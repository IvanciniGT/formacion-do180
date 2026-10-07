# Contenedor

Un entorno aislado en un SO (Linux) donde ejecutar procesos.
Aislado:
- Configuración propia de red -> Propia IP
- Sistema de archivos propio 
- Sus propias variables de entorno
- Podía tener limitaciones en recursos como CPU, memoria y almacenamiento.

Se crean desde imágenes de contenedor.

Las imágenes las sacamos de registros de repositorios de imágenes de contenedor.

Las imágenes se identifican por una url:   registry/repo:tag

    registry es opcional (por defecto se usa el que tenga configurado mi gestor de contenedores)
    tag es opcional (por defecto se usa "latest" que no es una buena práctica)

# Automatizar tareas vs procesos

Puedo tener una persiana, que subo y bajo con una cuerdita = PROCESO MANUAL

Puedo poner un motor con un botón y quitar la cuerdita -> AUTOMATIZACION DE UNA TAREA, la tarea de subir y bajar la persiana está automatizada.
Pero cuidado, sigo estando yo ahí para:
- Dar al botón para subir y bajar la persina. -> TAREA MANUAL
- Decidir cuándo dar al botón.                -> TAREA MANUAL

Ahora podría poner un ssensor de luz... que mire cuando hay mucha / poca luz fuera.
Y Podría montar un controlador, que reciba esa información y decida automáticamente cuándo "dar al botón y de hecho accione el botón/motor. 
Con eso he automatizado las 2 tareas que seguía haciendo yo. -> PROCESO DE SUBIR/BAJAR LA PERSIANA AUTOMATIZADO

Solo tiene sentido que me plantee automatizar el proceso si he automatizado las tareas individuales que lo componen.


---

# Donde entran los contenedores dentro del ciclo de vida de un sistema (DEVOPS)

Yo estoy desarrollando en mi máquina. Y para pruebas en local (pruebas para mi... no valen como criterio en el proyetco.. pero para mi si) necesito una BBDD... Nada mejor que un contenedor.
Ventajas frente a una instalación tradicional:
- Tiempo (30 segundos)
- Replicación (puedo pasar el fichero de creación del contenedor a todos mis compañeros y que tengan el mismo entorno de pruebas)
- Efímeros. Cuando voy a hacer pruebas, lo creo y arranco (10 segundos) y cuando termino lo destruyo, dejando el sistema limpio (no hay mierda en mi máquina, y la siguiente vez que hago pruebas empiezo con un entorno limpio no maleado!)

Se compila el proyecto de verdad de la buena (no en mi máquina), en un entorno reproducible. Un contenedor.
El mvn compile se ejecuta en un contenedor, creado con los requisitos de mi proyecto. Y eso es lo que usa siempre...
Y un entorno efimero... que creo desde cero cada vez que necesito compilar.
Las pruebas las ejecuto en un contenedor también, asegurando que el entorno de pruebas sea consistente y reproducible cada vez que se ejecutan Y AISLADO!

Mi app... quiero probar lo mismo que voy a desplegar!
Y no es solo la versión... es la instalación/configuración... Todo empaquetado en una imagen de contenedor.
Una imagen contiene SOFTWARE PREINSTALADO y CONFIGURACIÓN LISTA PARA USAR, lo que garantiza que el entorno de ejecución sea consistente en cualquier lugar donde se despliegue.
Desde esa imagen puedo generar un entorno de pruebas, con mi software instalado
Pero también generaré el entorno de producción, con la mismita instalación.

---

# Entornos de producción

Qué diferencia un entorno de producción del resto de entornos? 
Qué características tiene ese entorno que no necesito o no se dan en otros entorno (desarrollo, pruebas)?
- Alta disponibilidad

    Tratar de garantizar que el producto estará en funcionamiento una determinada cantidad de tiempo , pactada previamente de forma contractual: SLA (Service Level Agreement).
    NECESITAMOS ESTE SISTEMA FUNCIONANDO DE L-V de 8:30-15:00.. Y necesito que me asegures que está funcionando al menos el 95% de ese tiempo.
    NECESITAMOS ESTE SISTEMA FUNCIONANDO DE L-D 24 horas.. Y necesito que me asegures que está funcionando al menos el 99% de ese tiempo.

    Puedo asegurar (me puedo comprometer) a que un sistema funcionará el 100% del tiempo? (Y ojo, suponiendo que el sistema no tiene bugs -que es mucho suponer-): NO... el sistema funciona sobre máquinas, que pueden romperse.

    Se suelen medien en 9s:

    90% de disponibilidad corresponde a aproximadamente 36,5 días al año con el sistema KO              | €
    95% de disponibilidad corresponde a aproximadamente 18,25 días al año con el sistema KO             | €€
    99% de disponibilidad corresponde a aproximadamente 3,65 días al año con el sistema KO              | €€€€€
        Soy una web de citas de una peluquería de barrio
        Tengo mi web instalada en el ordenador gaming de mi hijo
    99,9% de disponibilidad corresponde a aproximadamente 8,76 horas al año con el sistema KO           | €€€€€€€€€€€€
        Necesito tener la máquina disponible ya (la de respaldo, por si se jode la principal)
        e instalada de antemano
    99,99% de disponibilidad corresponde a aproximadamente 52,56 minutos al año con el sistema KO       | €€€€€€€€€€€€€€€€€€€€€€
        Primero: Necesito un sistema de detección temprano! MONITORIZACION AVANZADA
        No me da tiempo a enchufar y desenchufar máquinas...
        Deben estar ya a priori funcionando en apralelo:
            Cluster ACTIVO ACTIVO (y es complejo de configurar)
    99,999% de disponibilidad corresponde a aproximadamente 5,26 minutos al año con el sistema KO       | €€€€€€€€€€€€€€€€€€€€€€€€€€€€€€€€€€€€€€€€
                                                                                                        v
            Y si se va la luz? Y si falla el proveedor de internet? Y si hay un  incendio? y si hay un terremoto?
        Generadores electricos de emergenciua: DIESEL
        Varios proveedores de internet (incluso alguno por satelite)
        Sistemas de extinción de incendios (que no sean cona AGUITA CAYENDO A LAS MAQUINA) (muy caros)
        NEcesito tener la infra en al menos 2 ubicaciones geograficas muy alejadas (por si hay una catastrofe natural)

- Escalabilidad

    Capacidad de ajustar la infra a las necesidades de cada momento.

    > App1: App departamental, app interna de la empresa

       Día 1:     100 usuarios
       Día 100:   100 usuarios
       Día 1000:  100 usuarios

        Puede haberlas pero es muy raro.

    > App 2: Una app que va usandose más. y está bien

        Día 1:     100 usuarios
        Día 100:   1000 usuarios        El problema es que llegará un mmento que la maquina donde tengo esto instalado no da de si. RAM, CPU, DISCO, RED
        Día 1000:  10000 usuarios           Solución: MAS MAQUINA   -> Escalabilidad VERTICAL

    > App 3: 

        Día n:          100 usuarios
        Día n+1:    1000000 de usuarios
        Día n+2:          0 usuarios
        Día n+3: 10.000.000 de usuarios
        Día n+4:    500 usuarios

        Esto lo normal hoy en día : INTERNET


    Soy la web de peidos del telepi!
        00:00h       0 estoy cerrado
        08:00h       0 sigo cerrado
        12:00h       30 clientes
        14:00h       3000 clientes
        16:00h       6 clientes
        18:00h       100 estoy cerrado
        21:30h       Madrid - Barça de por medio: 10000000 agarra que nos vamos!
        23:59h       0 estoy cerrado

    A qué dimensióno la infra aquí?
    ESCALABILIDAD HORIZONTAL: MAS MAQUINAS o MENOS!
    Y además querré AUTOMATIZAR la tarea de poner/quitar máquinas!
    Pero esto no va solo de máquinas... hay programas en esas máquinas que hay que instalar, y configurar y meter en clusters. Y poner balanceadores de carga... Y necesito automatizarlo.

ESTO ES LO QUE RESUELVE KUBERNETES!
TODOS estos escenarios.

Kubernetes es una herramienta para DEFINIR entornos de producción con un lenguaje DECLARATIVO.
Y Kubernetes (que es un programa... bueno muchos en realidad) se encarga de CREAR, INSTALAR, OPERAR Y MONITORIZAR AUTOMATIZAMENTE el entorno de producción, basado en las definiciones que hemos hecho de manera DECLARATIVA.
Y esto lo puede hacer GRACIAS a que el trabajo se hace (las cargas de trabajo, los procesos) en CONTENEDORES.
Pero para kubernetes, los contenedores son una ANECDOTA!
De hecho, kubernetes NO GESTIONA CONTENEDORES!

Los conceptos que manejamos en kubernetes son:
- Cluster activo-pasivo         Deployments, Statefulsets, Daemonsets
- Cluster Activo-Activo
- Proxy reverso                 Ingress Controllers / routes
- Balanceador de carga          Services
- Reglas de firewall            Network Policies

---

Qué comando arranca nginx?                              docker container start mi-nginx
Qué comando arranca oracle?                             docker container start mi-oracle
Qué comando arranca jenkins?                            docker container start mi-jenkins
Qué comando para jenkins?                               docker container stop mi-jenkins
Qué comando instala jenkins?                            docker container create --name mi-jenkins jenkins
Qué comando reinicia un apache kafka?                   docker container restart mi-apache-kafka
En qué carpeta guarda Apache httpd sus logs?            docker logs mi-apache-httpd

La gracia es que trabajo con contenedores, esos comandos / carpetas... quedan estandarizados/encapsulados

Da igual el programa que vaya a OPERAR, todo se opera de la misma manera: mediante comandos de contenedores.
Y si ya no hay que PREOCUPARSE de aprender 500 comandos.. y de que cada instalación sea un huerto...
sino que todo está totalmente ESTANDARIZADO, puedo hacer un PROGRAMA que:
- Instalar programas
- Operar programas
- Monitorizar programas

Y eso es Kubernetes.

Cuando trabajo con Kubernetes, los contenedores los siguen operando GESTORES DE CONTENEDORES: Container-d, CRIO...
Kubernetes solo les dice a esos GEstores de contenedores, que arrancar/parar/reiniciar y donde.
Nada más... Con respecto a los contenedores.

Por qué luego:
  Gestiona automaticamente servidores DNS
  Instala y configura Balanceadores de carga en automático
  Configura Proxies reversos en automatico
  Crea volumenes en cabinas de almacenamiento
  Gestiona la configuración de los programas de manera automática
  Escala horizontal y vertical de manera automática


---

A kubernetes le voy a decir:

Quiero un entorno de producción para la app 1.
En ese entorno quiero de 3 a 10 servidores tomcat en cluster (en base al uso de CPU > 50%).
Quiero que esos tomcat tengan un volumen compartido en una cabina nfs... donde se van a ir dejando ficheros.
Quiero ir recopilado de ellos todas esta s metricas (tamaño de cola, tiempo en cola, peticiones actuales, conexiones a bbdd ...)
Quiero tener un certificado https en los tomcat, que me lo generas tu PUBLICO.
Necesito un balanceador delante, que apunte a los tomcat (a los que haya en cada momento)
Quiero un proxy reverso dlante del balanceador... con una ruta configurada: https://miapp.miempresa.com/
Y necesito también un postgres con un volumen persistente para almacenar los datos.
Y lo quiero en cluster activo pasivo.. pero sin maquina "prealquilada"... Si se jode donde está corriendo el postgres.. me haces otra instalación sobre la marcha (10 segundos)
Eso si, al postgres, que solo se pueda llegar desde los tomcat... y a los tomcat desde el balanceador.
Quiero backups en el postgres todas las noches.
Ah.. y quiero que recopiles los logs de los tomcats y del postgres.. y me los dejes juntitos en algún sitio para echar un ojo si pasa algo.
Ah.. y quiero que cada minuto hagas una llamada por http a los tomcats para ver si contestan sano... y al postgres una query select 1.
Y si no los reinicias.
Los tomcats, INTENTA ponerlos en máquinas físicas diferenetes para evitar un único punto de fallo.
Los tomcat con 4 cores y 16 de ram
Y el postgres 6 cores y 32 de ram.
Los volumenes: el compartido de los tomcat: 1Tb (NFS)
Y el del postgres: 2Tb (iSCSI)

NOTA: el postgres y los tomcat que se ejecuten contenedores.

LITERALMENTE ESTO ES LO QUE PEDIMOS EN KUBERNETES!

Yo le pido esto.
- Kubernetes crea ese entorno de producción por mi y lo mantiene!
    - A lo mejor el día de mañana cambia la spec: OYE , que el volumen de la BBDD ahora de 3Tbs. Y quero tener entre 3-20 tomcats.
      Y kubernetes dice: OK! Ahora empiezo a trabajar asi!
- Kubernetes opera ese entorno 24x7.
- Si el postgres muere, Kubernetes lo detecta y lo levanta en otro sitio
- Si un tomcat muere, Kubernetes lo detecta y lo levanta en otro sitio
- Si las cpus pasan del 50% Kubernetes escala horizontalmente los tomcats.

KUBERNETES REEMPLAZA ADMINISTRADORES DE SISTEMAS. (Personas físicas)

Aunque da un lenguaje unificado, está planteado para que ese lenguaje sea usado por 2 perfiles diferentes:
- Equipos de desarrollo
- Administradores de sistemas

Quién dice en qué puerto funciona la BBDD, o qué versión de BBDD se usa, o que juego de caracteres tiene la BBDD o cuánta RAM configuro a los tomcat?
DESARROLLO

Ahora... que tu desarrollador pidas 16 Gbs de RAM para los tomcats, significa que YO ADMINISTRADOR DE SISTEMAS te los vaya a dar? Eso ya lo veremos

Que tu pidas 1Tb de almacenamiento nfs significa que YO ADMINISTRADOR DE SISTEMAS te lo vaya a dar? Eso ya lo veremos.
Y en qué cabina física se va a crear ese volumen ? Eso lo decide el desarrollador? NO, lo decide el administrador de sistemas.
Y la contraseña de la BBDD en el entorno de producción, la decide desarrollo? EIN?=??= NO, lo decide el administrador de sistemas.

Está pensado para que cada uno haga su trabajo y no interfieran entre si.
Cada uno define su parte... y kubernetes trabaja en base a esas definiciones conjuntas.

---

He hablado varias veces ya de LENGUAJE DECLARATIVO!
Entendemos el concepto?
Es un paradigma de programación.. aunque los humanos, en lenguajes naturales también lo usamos...
Nos centramos en lo que queremos conseguir (ESTADO DESAEADO) no en como conseguirlo (LENGUAJE IMPERATIVO)

> Felipe, IF(Condicional) hay algo que no sea una silla debajo de la ventana:
    > Quítalo!   IMPERATIVO
> Felipe, IF (condicional) no hay silla debajo de la ventana:
    > Felipe, IF NOT SILLA. (silla == false) THEN
    > GOTO IKEA:
        > COMPRA SILLA
    > Felipe, pon una silla debajo de la ventana.   IMPERATIVO

Para qué meto todos esos condicionales / caminos alternativos?
Para conseguir IDEMPOTENCIA!
En el mundo IT usamos este concepto para simbolizar una operación que independientemente del estado incial, siempre deje el sistema en el mismo estado final.
La consigo a base de enumerar / tratar todos los posibles estados iniciales... 
Le digo a Felipe cómo debe actuar dependiendo del estado incial que se encuentre.

> Felipe, debajo de la ventana tiene que haber una silla. Es tu responsabilidad.   DECLARATIVO

El lenguaje declarativo me ofrece de manera natural LA IDEMPOTENCIA.
Ya que lo que hago es decir SOLO EL ESTADO FINAL DESEADO, y no cómo conseguirlo.
El cómo conseguir ese estado final lo dejo en manos de Felipe. LO DELEGO.

Esto es cada vez más deseado en el mundo del desarrollo de software.... el usar lenguajes declarativos.
Todas las herramientas que lo están petando, lo están petando precisamente por usar lenguajes declarativos:
- Docker compose                YAML
- Kubernetes                    YAML
- Ansible      (sysadmins)      YAML
- Terraform
- SpringBoot (desarrollo)       YAML
- Angular

---

En el caso de kubernetes, la descripción de los entornos la haremos en archivos YAML.
- Gitlab CI/CD - YAML
- La configuración de red en una máquina ubuntu (netplan) - YAML

---

# Como es un documento YAML de kubernetes

A kubernetes lo que vamos a hacer es explicarle COMO QUEREMOS NUESTRO ENTORNO DE PRODUCCION.
Para ello, cargaremos en Kubernetes DOCUMENTOS con DEFINICIONES de COSAS que quiero en MI ENTORNO DE PRODUCCION.
Cada COSA que quiera en mi entorno de producción tendrá su propio DOCUMENTO YAML en Kubernetes.
Que quiero un cluster activo activo de MariaDB  -> 1 DOCUMENTO YAML
Que quiero un balanceador de carga              -> 1 DOCUMENTO YAML
Que quiero un volumen de almacenamiento         -> 1 DOCUMENTO YAML
Que quiero una regla de firewall                -> 1 DOCUMENTO YAML

COSA? Cosa puede ser muy variado:
- Un cluster activo activo de MariaDB
- Un balanceador de carga
- Un volumen de almacenamiento
- Una regla de firewall
- Un fichero de configuración que quiero aplicar a mi aplicación (application.properties)
- Un certificado SSL-
- ...

En kubernetes a esas COSAS se les llama RECURSOS.
Y kubernetes define unos 50 TIPOS de RECURSO diferentes.
En este curso aprenderemos unos 20-25 TIPOS DE RECURSO.

En el curso DO280, que es la continuación de este curso DO180, se ven otros tantos y más!

Y aquí pasa algo especial.
Kubernetes es una plataforma extensible. Puedo instalar PLUGINS (En el mundo kubernetes se les conoce como OPERADORES) que amplian la funcionaldiad de Kubernetes y permiten definir NUEVOS TIPOS DE RECURSOS.

Openshift es una distribución de Kubernetes (la de redhat).
Es un Kubernetes con un monton de OPERADORES instalados sobre él, preseleccionados por Red Hat.
Kubernetes estandar permite definir unos 50 TIPOS DE RECURSO.
En Openshift, salido de fabrica (que luego también se puede tunear) podemos definir más de 500 TIPOS DE RECURSO.

Los de kubernetes son la base! SON LOS QUE NECESITAMOS APRENDER BIEN!
Pero el objetivo de este curso es DOBLE:
- Aprender los tipos de recursos principales de Kubernetes.
- Aprender a manejarnos CON CUALQUIER TIPO DE RECURSO POSIBLE... Todos se operan de la misma forma... luego cada uno tendrá sus peculiaridades... y para eso: DOCUMENTACION!

# Tipos de RECURSOS PRINCIPALES DE KUBERNETES

- Node
- Namespace
- Pod
- Deployment
- Statefulset
- Replicaset
- Job
- CronJob
- ConfigMap
- Secret
- PersistentVolume
- PersistentVolumeClaim
- HorizontalPodAutoscaler
- Service
- Ingress
- NetworkPolicy
- PodDisruptionBudget
- ResourceQuota
- LimitRange
- ...

---

# Intro a la arquitectura de Kubernetes

Cuando trabajamos con Kubernetes lo que manejamos es un cluster de máquinas! Es lo que hace falta en un entorno de producción.

    Fuera de esas máquinas, se instala kubernetes. Pero kubernetes no es un programa.... son muchos programas.
    Esos programas de hecho se montan mediante CONTENEDORES! que corren dentro del propio cluster que gestiona kubernetes ¿?
    Kubernetes se instala en Kubernetes.
    Lo que pasa es que esos contenedores no se montan en los nodos de trabajo

    Máquina CP 1 - Control Plane (Antiguamente se les llamaba maestros)
        Linux
        Kubelet (agente de Kubernetes - Servicio a nivel de os)
        Gestor de contenedores: Docker (o CRI-O, containerd, etc.)
            kube-proxy
            apiserver-1
            controller-manager-1
            core-dns-1
            etcd-1
    Máquina CP 2 - Control Plane
        Linux
        Kubelet (agente de Kubernetes - Servicio a nivel de os)
        Gestor de contenedores: Docker (o CRI-O, containerd, etc.)
            kube-proxy
            apiserver-2
            core-dns-2
            scheduler-1
            etcd-2
    Máquina CP 3 - Control Plane
        Linux
        Kubelet (agente de Kubernetes - Servicio a nivel de os)
        Gestor de contenedores: Docker (o CRI-O, containerd, etc.)
            kube-proxy
            controller-manager-2
            scheduler-2
            etcd-3

    Máquina 1 - Nodo de trabajo
        Linux
        Kubelet (agente de Kubernetes - Servicio a nivel de os)
        Gestor de contenedores: Docker (o CRI-O, containerd, etc.)
            kube-proxy
    Máquina 2 - Nodo de trabajo
        Linux
        Kubelet (agente de Kubernetes - Servicio a nivel de os)
        Gestor de contenedores: Docker (o CRI-O, containerd, etc.)
            kube-proxy
    Máquina 3 - Nodo de trabajo
        Linux
        Kubelet (agente de Kubernetes - Servicio a nivel de os)
            v
        Gestor de contenedores: Docker (o CRI-O, containerd, etc.)
            kube-proxy
            nginx-app

Lo normal es que sean nodos físicos... aunque en ocasiones trabajamos con nodos virtuales.
Puede haber clusters pequeños. 5 nodos, 10 nodos
Puede haber clusters grandes: 500 nodos

En ocasiones, reservamos al menos 2 máquinas de trabajo para programas especiales.. que ya hablaré de ellos: nodos de infraestructura.

Necesitamos al menos 3 máquinas para el Control Plane. Y al menos 2 máquinas de trabajo, para tener HA (Alta Disponibilidad).
Qué programas forman parte de kubernetes:
- Kubelet (es un programa que se instala a HIERRO, como un servicio... en cada nodo del cluster, con independencia del tipo de nodo)
- Kubeadm (es una herramienta cli, que instalamos a hierro, en cada nodo del cluster, con independencia del tipo de nodo)
  Nos ayuda con tareas básicas de gestión del cluster: Instalación inicial, añadir nuevos nodos al cluster y otras operaciones de mantenimiento.
- Componentes del plano de control (Control Plane) de Kubernetes: Esos se ejecutan como contenedores dentro del cluster, gestionados por el propio Kubernetes:
  - kube-apiserver: Es quien recibe TODAS LAS PETICIONES y actúa como puerta de entrada al cluster.
    Cualquier cosa que se quiera hacer, pasa a través del kube-apiserver.
    Tendremos varios clientes para hablar con el cluster:
        - Cli (linea de comandos): kubectl, oc (OpenShift CLI)...
        - GUI: Dashboard de Kubernetes (deprecated), headlamp (nuevo GUI Web de Kubernetes), Openshift  web console.
        Cualquiera de estos clientes, habla con el kube-apiserver.
  - core-dns: Es un servidor dns interno al cluster
  - controller-manager: Es el componente que hace los trabajos. Cualquier trabajo que haya que hacer, es realizado por el controller-manager.
  - scheduler: Es un componente con una única responsabildiad. Decicir en que máquina del cluster se despliega una aplicación.
  - etcd: BBDD interna de kubernetes. Todos los datos que guarda kubernetes: Configuraciones nuestras que vamos haciendo, estado de los sistemas... 
  - kube-proxy.... YA OS CONTARÉ!!!! Cuando hablemos de comunicaciones.


`kubectl apply -f nginx.yaml`
 -> nfinx.yaml -> apiserver
 -> api-server : valida el archivo
                -> controller-manager: Oye, que me piden tener esto!
                    -> etcd: Guardando ese fichero en la base de datos interna.
                    -> scheduler: Necesito saber donde guardar esto -> Nodo trabajor 3
                    -> kubelet nodo trabajor 3 -> Crea un contenedor para el nginx. 
                        -> kubelet -> gestor de contenedores: Docker (o CRI-O, containerd, etc.) -> Crea el contenedor para el nginx. 


# Presentación de los RECURSOS más básicos de Kubernetes / Un paseo por el esquema YAML básico de RECURSOS de Kubernetes

## Namespace

Un namespace es una agrupación LOGICA de RECURSOS dentro de un cluster.
Para qué la usamos? 2 cosas:
- Tener todas las configuraciones/recursos de un tema/despliegue/sistema/entorno juntos
    NAMESPACE: app1-produccion           app1-clienteA          
        cluster-mariadb
        cluster tomcats
        balanceador de carga para los tomcat
        ficheros de configuración de la bbdd
        contraseñas de la bbdd
        volumenes de almacenamiento de la bbdd
        ...
    ---> Esto lo hacen desarrolladores de la app1

- Limitar lo que puede ocurrir con esos recursos. (seguridad,uso de recursos)
     Solo tales usuarios pueden manejar los recursos de tal namespace.
     En ese namespace solo pueden usarse como mucho 10 cores y 20 Gbs de RAM.. y 1Tb de almacenamiento
    ---> Esto lo hacen sysadmins del cluster

Entramos en un modelo nuevo: MODELO DE AUTOSERVICIO: ESTA IDEA ES CLAVE EN KUBERNETES

La idea es que tenemos a 2 equipos trabajando en paralelo.
Necesito un entorno para desplegar un sistema.
- Quien gestiona el cluster GENERA ESE ENTORNO (namespace) y define las políticas de uso y acceso.
  Crean un usuario con permisos para ese namespace
  Y lo entregan a negocio/desarrollo, quién lleve ese sistema
- Quien lleva el sistemna: Negocio, desarrollo:
  Entra con su usaurio y solo ve las cositas de su namespace.
  Y ahí pueden crear lo que necesiten... es su problema.
  Eso si.. si no funciona, luego que no lloren a nadie!!


Más antiguamente reinaba la ANARQUÍA: cada equipo trabajaba como le venía en gana... -> PROBLEMON!
    Finales de los años 60... cada equipo trabajaba a su bola. Sin control, sin estandares, sin patrones, sin arquitecturas, sin metodologias -> RUINA GIGANTE: CRISIS Del software
    Esto dió lugar a la INGENIERIA DE SOFTWARE COMO DISCIPLINA.
    -> Metodologías waterfall
    -> Patrones
    -> Arquitecturas monolíticas
    -> Departamento de IT Centrales.


Antiguamente había un departamento de sistemas CENTRALIZADO: HIPERBUROCRACIA! INEFICIENCIA! LENTITUD!

Otro modelo que se ha probado es el modelo externalizado:
Que cada equipo se monte sus mierdas! Esto se ha hecho mucho durante los últimos años: CLOUDS.
Resultado: RUINA TAMBIEN! Falta de estandarización y control.
Que no tenga que mover a una persona de un equipo a otro para gestionar los recursos, cada equipo se vuelve autónomo pero caótico.

Hoy en día optamos más por un modelo FEDERADO!
Hay un minidepartamento de sistemas que gestiona las políticas generales y una infraestructura común, mientras que los equipos de desarrollo/negocio les doy libertad, para que dentro de lo definido , actúen como quieran... ESTE PARECE SER EL PUNTO DULCE... En esto estamos... Esto ofrece KUBERNETES.


¿Cómo creamos un namespace en kubernetes?
NOSOTROS NO CREAMOS NAMESPACES EN KUBERNETES. Quién crea los namespaces? KUBERNETES!
Yo lo que diré a Kubernetes es: QUIERO TENER UN NAMESPACE LLAMADO X en el cluster. <- Qué tipo de lenguajes estamos usando? DECLARATIVO
Y cónde vamos a decirle eso? En un YAML

```yaml
kind:                    Namespace  # Tipo de Recurso que estamos definiendo
apiVersion:              v1         # Esto es raro al principio
                                    # Hemos dicho que kubernetes permite gestionar tipos de recursos diferentes.
                                    # Y hemos dicho que kubernetes puede ampliar los tipos de recursos que puede gestionar al instalarle "plugins"
                                    # Cada tipo de recurso va gestionado por una LIBRERIA específica.
                                    # Cuando monto un plugion, el plugin define:
                                    # - Nuevos tipos de recursos: CRD (Custom Resource Definitions)
                                    # - Nuevas librerías para gestionar esos recursos.
                                    # Apiversion indica qué libreria gestiona este tipo de recurso que estoy creando.
                                    # Qué libreria es la que gestiona los Namespace en este caso
                                    # Y es que puedo tener 2 librerias diferentes que gestionen tipos de recursos con el mismo nombre
                                    # Libreria para gestionar Certificados: Kind: Certificate
                                    # Pero quizás tengo 2 librerías que gestionan certificados. Una libreria que gestiona certificados PUBLICOS y otra AUTOFIRMADOS.
                                    # El apiVersion tiene esta pinta:     libreria/version(mayor)
                                    # Por ejemplo                           certmanager.io/v1            
                                    # Kubernetes viene de serie con algunas librerias.
                                    # Para la librería básica de kubernetes , la que define la mayor parte de sus objetos, NO SE PONE EL NOMBRE DE LA LIBRERIA... solo la version.
                                    # Otras libresrias que incluye Kubernetes:
                                         # - networking.k8s.io/v1 (para gestionar recursos de red como Ingress)
                                         # - rbac.authorization.k8s.io/v1 (para gestionar roles y permisos)
                                         # - apps/v1 (para gestionar despliegues y otros recursos de aplicaciones)
                                         # - batch/v1 (para gestionar trabajos por lotes como CronJobs y Jobs)

metadata:
  name:                     app1-produccion    # Sirve de identificador del recurso
                                               # Este nombre es UNICO..Para ese tipo de recurso, para el NAMESPACE en el que el recurso sea creado
```

Y YA, acabado!

```yaml
kind:                    Namespace
apiVersion:              v1
metadata:
  name:                  app2-produccion
```

Hay verbos en ese documento? NO LOS HAY.... Es puro declarativo
El verbo, que SI HACE FALTA! se da después.

    `kubectl create -f <archivo.yaml>`      Da de alta en el cluster los recursos definidos en el archivo YAML
    `kubectl apply -f <archivo.yaml>`       Da de alta o modifica los recursos definidos en el archivo YAML
    `kubectl delete -f <archivo.yaml>`      Borra los recursos que se definieron en el archivo YAML del cluster

    AQUI SI HAY LENGUAJE IMPERATIVO... En la acción que quiero hacer sobre esos recuros que tengo DEFINIDOS (en lenguaje DECLARATIVO)


    