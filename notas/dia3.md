# POD

Un pod es un conjunto de contenedores, que:
- Los contenedores de un pod COMPARTEN CONFIGURACION DE RED: DIRECCION IP, INTERFACES
- Se despliegan juntos en el mismo host/nodo trabajador del cluster
  - Pueden hablar entre si por localhost (sin pisar red física)
  - Pueden compartir carpetas locales
- Se despliegan juntos y escalan juntos

Cuando creo un pod, yo decido que contenedores pongo en ese pod.
Un pod puede tener solo 1 contenedor....

> queremos montar un Wordpress en nuestro cluster

- Componentes:
  - Servidor web (apache httpd) con soporte de php
    Programas propios de wordpress
  - BBDD (MySQL, MariaDB...)

> La pregunta es: Cuantos contenedores CREAMOS? 1 o 2? 2
 
SIEMPRE UN PROGRAMA POR CONTENEDOR!

> La pregunta es: Cuantos pods CREAMOS? 1 o 2? DE NUEVO 2
                                                                                                LO NECESITO      LO QUIERO
- Los contenedores de un pod COMPARTEN CONFIGURACION DE RED: DIRECCION IP, INTERFACES               NO             puede ser/me interesa (menos latencia)
- Se despliegan juntos en el mismo host/nodo trabajador del cluster                                 NO             puede ser/me interesa (menos latencia)   
  - Pueden hablar entre si por localhost (sin pisar red física)                                     NO             puede ser/me interesa (menos latencia)   
  - Pueden compartir carpetas locales                                                               NO             NO
- Se despliegan juntos y escalan juntos                                                             
   Si actualizo uno se actualiza el otro?                                                           NO NECESARIAMENTE
                                                                                                    Hay muchos casos que no. Solo quiero subir versión de UNO DE ELLOS
                                                                                                    Si están en el mismo pod... Se borra el pod y se recrea completo
   Escalan juntos?                                                                                  NO             NI DE COÑA!
    Puedo tener 1 MYSQL y 3 tomcat... y quere pasar a 1 MYSQL y 5 tomcats
        ^^^
        ESTO PER SE ya hace imprescindible que vayan en pods separados.
        De hecho, si esalan por separado, NI DE COÑA LOS QUIERO EN EL MISMO HOST, ni hablando por localhost!
        Querré las instancias del tomcat en NODOS DISTINTOS (HA)... y entocnes localhost no vale para nada. ME IMPIDE ESO!

> En nuestro caso

    POD A -> APACHE/WORDPRESS
                contenedor1      apache-wordpress

    POD B -> MYSQL
                contenedor1      mysql

    Y el día de mañana podré hacer réplicas de los pods según sea necesario por separado

> En general

Programas distintos, contenedores distintos y pods distintos.
Solo casos muy muy muy especiales montaremos contenedores distintos , pero mismo pod.

>> Instalación más avanzada de wordpress

    PodB1 -> Mysql-Galera                        Pod A1 -> Apache/WP                            ELASTIC_SEARCH < KIBANA
                                                        apache.log
                                                            ^
                                                        Filebeat | Fluentd (tail -f apache.log)
                                                        Y cada linea la manda a ELASTICSEARCH / OPENSEARCH

    PodB2 -> Mysql-Galera                        Pod A2 -> Apache/WP
                                                        apache.log
                                                            ^
                                                        Filebeat | Fluentd (tail -f apache.log)
                                                        Y cada linea la manda a ELASTICSEARCH / OPENSEARCH

    PodB3 -> Mysql-Galera                        Pod A3 -> Apache/WP
                                                        apache.log
                                                            ^
                                                        Filebeat | Fluentd (tail -f apache.log)
                                                        Y cada linea la manda a ELASTICSEARCH / OPENSEARCH

                                                 Pod A4 -> Apache/WP
                                                            v
                                                        apache.log
                                                            ^
                                                        Filebeat | Fluentd (tail -f apache.log)
                                                        Y cada linea la manda a ELASTICSEARCH / OPENSEARCH

Lo normal es que configure 2 archivos rotados de 50Kb-100Kb para los logs. Con esto limito el uso de almacenamiento en los host de 100Kb-200Kb
En una instalación guay, lo que haría es que la carpeta /var/log/apache2/ la montaría como tmpfs con soporte en memoria RAM.


> Apache y Filebeat... 2 programas

1 contenedor o 2 contenedores? 2 contenedores SIEMPRE... programas distintos, con contenedores distintos.

>> 1 pod o 2?

                                                                                                LO NECESITO      LO QUIERO
- Los contenedores de un pod COMPARTEN CONFIGURACION DE RED: DIRECCION IP, INTERFACES               NO             NO (no me molesta que ocurra... pero no es algo que quira ni necesite)
- Se despliegan juntos en el mismo host/nodo trabajador del cluster                                 SI
  - Pueden hablar entre si por localhost (sin pisar red física)                                     NO             NO 
  - Pueden compartir carpetas locales (en HDD o en RAM)                                             SI             SI
- Se despliegan juntos y escalan juntos                                                             
   Si actualizo uno se actualiza el otro?                                                           NO             NO
   Escalan juntos?                                                                                  POR SUPUESTO. Cada Apache, debe tener un filebeat
   Van en una relación 1 a 1

    1 pod!
    En ese pod, el apache sería el contenedor PRINCIPAL.
    El Filebeat es algo accesorio, que necesito... pero que justifico solo porque hay un apache del que leer. Si no hubiera apache, no necesito filebeat
    El contenedor de filebeat es lo que llama un contenedor sidecar.

>>> Pregunta! Si hay un problema, donde miro? 

>>> PROBLEMA 1: 
En los logs... cuántos logs tengo? Al menos 7.
Claro .. que si yo gestiono 10 apps, tendré 50 logs.
Los voy mirando de uno en uno?

>>> PROBLEMA 2: 
Después de 1 año de funcionamiento... peto los servidores con archivos de log
Quiero conservar logs antiguos? Posiblemente SI (Muchos beneficios)
Los quiero en los HDD de los apaches y los mysql= EVIDENTEMENTE NO
Necesito un sistema central de gestión de logs.

La herramienta estrella hoy en día para eso es: ELASTICSEARCH / OPENSEARCH





Cada fabricante me da su IMAGEN de contenedor, y los contenedores los creamos desde las imagenes.
No hay una imagen con mysql + apache + php + wordpress. 
Puedo crearla yo... ABSURDO . No lo haría nunca.

Además, imaginad que el día de mañana quiero actualizar la BBDD.. no necesito todar el wordpress.
Si estuviera junto tendria que tocar las 2 cosas.
Por otro lado, dijimos que la gracia de los contenedores es que unos procesos no interfieran con otros. NO HAY NECEIDAD NI APORTA NADA juntar programas en un contenedor. TODO LO CONTRARIO. ES MUCHO MEJOR tener un contenedor por cada program:
- Más sencillo de mantener
- Más controlado el consumo/limitación de recursos

Pero luego hay otra... Imaginad que quiero ESCALAR el sistema.. para que acepten más usuarios. La BBDD y el APACHE escalan en paralelo? Por cada Apche necesito una instancia nueva de BBDD?
Puedo tener 4 apaches contra la misma BBDD... o en una instalación más gorda: 10 apaches contra 3 BBDD.

---

NOTA:

Imaginad que tengo una VM que he creado con Ubuntu.
Y dentro he instalado MySQL versión 5.7.2.

Quiero pasar a MYSQL versión 5.7.3. Cuál sería el procedimiento de trabajo?
Entrar en la VM y actualizar de MySQL 5.7.2 a 5.7.3. Según instrucciones del fabricante.

Cómo trabajamos con contenedores:

Si esa instalación en lugar de en un VM la tuviera en un contenedor, el procedimiento cambia.
Nunca entraríamos a un contenedor a actualizar el programa... da igual el programa.

Lo que hacemos es: BORRAR EL CONTENEDOR y CREAR UNO NUEVO desde la imagen actualizada del programa.
LISTO!

El flujo de trabajo con contenedores NO SE PARECE EN NADA al flujo de trabajo con máquinas virtuales.

Una VM en un entorno de producción NO SE BORRA NUNCA (hasta que no quiero desmantelar el proyecto).
En cambio, los contenedores los borramos de continuo.

El propio kubernetes BORRARÁ NUESTROS CONTENEDORES VARIAS VECES AL DIA!
No existe el concepto de MOVER UN CONTENEDOR DE UNA MAQUINA A OTRA.
Lo que se hace es borrar el contenedor de una máquina y crear uno nuevo en otra máquina.

Nos pasamos el día borrando contenedores y creando otros nuevos.


---

 tomcat                 apache
    app1.war                php con los programas de wordpress


 tomcat                 apache
    app1.war                php con los programas de wordpress  


 tomcat                 apache
    app1.war                php con los programas de wordpress  


 tomcat                 apache
    app1.war                php con los programas de wordpress  


 tomcat                 apache
    app1.war                php con los programas de wordpress  


mysql galera
mysql galera
mysql galera


redis cache

DESPLIEGUE = INFRA + PROGRAMAS                      v1.0.0                  ->  v1.1.0                            -> v.1.1.1
    version apache                                  v.2.24.7   3 instancias       v.2.25.0        3-5 instancias.    v.2.25.0        3-5 instancias
    version mysql                                   v.7.5.1    1 instancia        v.8.0.0         3 instancias       v.8.0.0         3 instancias  
    version redis                                   v.3.1.7    1 instancia        v3.1.7          1 instancia        v3.1.7          1 instancia
    
    balanceador de carga para el mysql                  X                          Si lo necesito 
    volumen de datos para el apache                    10 Gb                     10 Gb                                  20Gb 
    Cambio en la configuracion de logs

# IaC (Infrastructure as Code)

Pero infraestructura como código no es solo tener la infra definida en ficheros de texto.
ES TRATAR LA INFRA COMO SI FUERA CODIGO.... y lo  primero que hago es SOMETER LA INFRA A CONTROL DE VERSION!

La infra evoluciona en versiones. Y hay dependencias entre versiones de la infra y de las apps que enchufo en la infra.



# Asignación de recursos a un POD

```yaml
      resources:
        limits:
          memory:               128Mi
          cpu:                  "1"
        requests:
          memory:               64Mi
          cpu:                  250m
```

Definimos los recursosque asignamos al pod, pero... dando solicitudes(peticiones / requests) y límites (limits) de CPU y memoria.
Antes de nada entendemos los datos:

    `128Mi`        Eso no son 128 Megas.... Son 128 Mebibytes

    1KB = 1000 bytes        NO SON 1024.... SON 1000... Cambió hace 25 años. Antiguamente eran 1024 bytes.
    1MB = 1000 KB
    1GB = 1000 MB

Se cambió para que los prefijos encajasen con los de sistema internacional. En una norma ISO.
Y crearon una nueva unnidad de medida: Los bibytes
    1 Kibibyte = 1024 bytes
    1 Mebibyte = 1024 Kibibytes
    1 Gibibyte = 1024 Mebibytes
    1 Tebibyte = 1024 Gibibytes

    Kubernetes, los clouds hablan en bibytes. VAMOS ... LOS MEGAS DE TODA LA VIDA!

    `cpu: "1"`       1 core
    `cpu: 250m`    250 milicores

    Y eso no significa que al contenedor le deje 1 core para el...
    Sino que le dejo usar el equivalente a 1 core por unidad de tiempo.
        Puede ser un core de la máquina al 100%
        Pueden ser 2 cores al 50%
        O 4 cores al 25%

        1000 milicores = 1 core
        500 milicores = 0.5 core
        250 milicores = 0.25 core


Ahora bien... dicho esto.. que son los request y los limits. Y es complejo.

El request es lo que se garantiza al contenedor, caso que lo necesite. Si no lo necesita... no se usa.
El limit es lo que le permito usar si hay hueco disponible en el nodo.

El dato del request el primero que lo usa es el SCHEDULER (junto con muchos otros datos) para decidir en qué nodo colocar el pod.

                HARDWARE        REQUEST        USO REAL      COMPROMETIDO.     SIN USAR
                CPU   RAM       CPU   RAM      CPU   RAM      CPU   RAM        CPU   RAM
    NODO 1      10    20        0     0         5    5        10    20         5     15
        POD1 Tomcat             5     10        1    2
        POD2 Tomcat             5     10        5    10
    NODO 2      10    20
        POD3 Tomcat             5     10        

    POD Tomcat 
        request: CPU 5  RAM 10
        limits:  CPU 7  RAM 15

Linux es un kernel de sistema operativo de tiempo compartido.
Las tareas se van mandando a las cpus... pero el kernel de linux es quien hace ese trabajo... y tiene capacidad para en un momento dado ir encolando peticiones de un proceso... y reteniéndolas.

A un proceso le puedo retener peticiones a la cpu ... que vaya más lento.
Pero a un proceso no le puedo quitar RAM si ya la tiene asignada. Qué trozo le quita el SO?
Decisión de kubernetes:
    kill SIG_QUIT al pod 1 de tomcat (apagate!) y tienes un timeout para hacerlo.
    No lo haces?
    kill SIG_KILL al pod 1 de tomcat (muerte inmediata).
    kill -9 al pod 1 de tomcat (muerte inmediata).

    No se pone ni colorao el kubernetes para hacer esto.
    REINICIA EL POD AL MOMENTO

Por eso "SIEMPRE" ponemos LIMIT RAM = REQUEST RAM, a no ser que me de absolutamente igual que el pod pueda ser reiniciado por falta de memoria.
Y hay casos (muchos) donde me da igual.

En JAVA, que los procesos corren en una VM, configuramos:
 -Xms128m -Xmx128m   # request RAM = limit RAM
 Cual es la buena práctica / recomendación en JAVA: poner esos valores iguales!
 La idea es que si vas a llegar a necesitar una determinada cantidad de memoria RAM, pídela desde el principio.

En el caso de kubernetes es: Si vas a llegar a necesitar una determinada cantidad de memoria RAM, pídela desde el principio mediante los requests = limit, o te arriesgas a que si hace falta RAM te crujan.

> En qué casos me puede interesa limit != request?

Hay bastantes. Para SERVICIOS a los que acceden los clientes: NUNCA !
Pero hay otro tipo de servicios INTERNOS DEL CLUSTER... que se ejecutan de higos a brevas:
- Renovar/generar un certificado SSL                      CREACION: ANTES DE DESPLEGAR EL SISTEMA          RENOVACION: 3 meses? 1 año?
- Crear un volumen de almacenamiento en una cabina        CREACION: ANTES DE DESPLEGAR UNA APP
- ...

Pero esos certificados o esos volumenes son creados por programas.
- Gestor de certificados
- Gestor de volumenes

Esos programas se ejecutan de higos a brevas (hacen carga de trabajo)... pero estan arrancados de continuo.
De estos programas en cluster de kubernetes acabamos con decenas o cientos.
Si a cada uno le configuro un limir RAM = request RAM, estoy reservando memoria para algo que solo se ejecuta de higos a brevas, y dejo máquinas enteras bloqueadas (sin recursos disponibles) para estos programas.

En este tipo de programas se suele poner limit RAM >> request RAM, y un limit CPU >> request CPU.
Lo que quier es en 2 o 3 máquinas (NODOS DE INFRAESTRUCTURA) tener la hueva de programas de este tipo.
Que un programa necesita recursos y no hay... que se mate otro de los programas. NO AFECTA A SERVICIO (SLA)
Puede afectar a que un certificado o un volumen tarde más en generarse.

```yaml
    resources:
        limits:
            memory: "1024Mi"
            cpu: "2"
        requests:
                memory: "64Mi"
                cpu: "10m"
```

En servicios:
```yaml
    resources:
        limits:
            memory: "512Mi"     # La que necesite realmente el servicio en trabajo diario
            cpu:    "4"         # Aqui no hay problema... si hay hueco... que tire!
        requests:
            memory: "512Mi"
            cpu:    "2"
```

En un entorno de producción, los datos no se guardan en los HDD de los servidores.
Los servidores tienen sus HDD, pero para su sistema operativo y sus programas.
Los datos se guardan en cabinas de almacenamiento.



---

Para configurar entorno:
- Descarga de kubectl.exe y meterlo en el path
- Crear una carpeta en c:>\usuarios\Felipe\.kube
- Y dentro ponemos un fichero llamado config
   Ese fichero lleva:
   - URL del cluster
   - Usuario
   - "Contraseña" del usuario
   - Certificado CA que es la que firma el certificado https del cluster