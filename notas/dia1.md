
# DO-180 Redhat Openshift I: Contenedores y Kubernetes.

# Formas de instalar/desplegar software

## Método tradicional: Instalación a Hierro

        App1   +   App2   +   App3              Esta forma de instalación tiene problemas graves, especialmente en entornos de producción:
    -------------------------------------           - Atados a hardware
            Sistema Operativo                       - Aislamiento entre apps:
    -------------------------------------               - Puede haber incompatibilidades entre las herramientas o sus dependencias
            HIERRO (Hardware)                           - Puede haber incompatibilidades en configuraciones de sistema operativo
                                                        - Puede ser que app1 espie o acceda a datos de app2 (virus informático)
                                                        - Bug en App1 (CPU 100%)        App1 --> Offline
                                                                                        App2 --> Offline
                                                                                        App3 --> Offline
                                                    - ...

## Método basado en Máquinas virtuales

       App1    |      App2 + App3              Las máquinas virtuales las hemos usado INTENSIVAMENTE, especialmente en entornos de producción,
    ------------------------------------       ya que nos dan una solución a todos los problemas de ahí arriba.
       SO1     |       SO2                          Básicamente, nos dan un entorno aislado donde correr las distintas apps.
    ------------------------------------
       VM1     |       VM2                      El problema es que las VMs vienen con sus propios PROBLEMAS:
    ------------------------------------        - Configuraciones/Mantenimiento mucho más complejas
        Hipervisor:                             - Merma de recursos efectiva
        HyperV, KVM, esXI, VirtualBox           - Empeoramiento en el rendimiento de las apps
        ...                                     - Licencias
    ------------------------------------        - ...
            Sistema Operativo
    ------------------------------------
            HIERRO (Hardware)

## Método basado en Contenedores

Se formaliza en 2013... por una empresa (startup) llamada Docker Inc.
Pero las ideas detrás del mundo de los contenedores venían incluso de décadas atrás.

       App1    |      App2 + App3
    ------------------------------------
        C1     |        C2
    ------------------------------------
        Gestor de contenedores:
        Docker, Podman, LXC, 
        CRIO, ContainerD
    ------------------------------------
        Sistema Operativo LINUX!
    ------------------------------------
            HIERRO (Hardware)

    Los contenedores NO SON VIRTUALIZACIÓN!
    NO SE PARECEN EN NADA a las máquinas virtuales.
    Los contenedores NO SON MÁQUINAS VIRTUALES sin sistema operativo propio.
    
    Pero...hacen las veces de una máquina virtual... Es decir:
    Me permiten disponer de entornos AISLADOS donde ejecutar aplicaciones.

    Los contenedores me permiten resolver casi los mismos problemas que las máquinas virtuales, pero de manera más eficiente y ligera.
    Un contenedor ocupa muy poco espacio, por no tener un sistema operativo completo propio.
    Un contenedor no genera merma en los recursos del sistema, a diferencia de las máquinas virtuales. Casi es más óptimo en algunos escenarios.
    Un contenedor no baja rendimiento de apps.. ya que los procesos están en comunicación directa con el kernel del sistema operativo anfitrión.

    Son todas las ventajas de las instalaciones a hierro, pero sin los inconvenientes de las máquinas virtuales.

    Se han impuesto en el mercado.
    El 90% de lo que antes se hacía con VMs ahora se hace con contenedores.
    Las VMs siguen teniendo sus casos de uso... ya muy pocos.
    De hecho, grandes fabricantes de software de virtualización están adaptando sus productos para trabajar con contenedores en lugar de VMs.
        - Redhat tiene su propia solución de virtualización: Red Hat Virtualization (RHV)
          Pero su apuesta por los contenedores es enorme: PODMAN y Red Hat OpenShift.
        - VMWare, la empresa de referencia en virtualización, también está adaptando sus productos para trabajar con contenedores.
          Su plataforma de contenedores es VMware Tanzu.
        - Nutanix, otra empresa de virtualización, también está adaptando sus productos para trabajar con contenedores.
          Su plataforma de contenedores es Nutanix Karbon.

## Qué es un Contenedor?

Un contenedor es un entorno AISLADO dentro de un Sistema Operativo rodando LINUX, en el que ejecuto procesos.
Aislado:
- Cada contenedor tiene su propia configuración de red... y por ende, su propia IP.
- Cada contenedor tiene su propio Filesystem (sistema de archivos) aislado del resto de contenedores y del sistema anfitrión.
- Cada contenedor tiene sus propias variables de entorno... Como entorno que es.
- Los contenedores PUEDEN tener limitado el acceso a recursos del sistema, como CPU, memoria y almacenamiento.

Todos los procesos que corren en contenedores dentro de un equipo comparten KERNEL del sistema operativo anfitrión.

Máquina virtual sin sistema operativo.
Burbuja con código/librerías para ponerlo a funcionar en entornos diferentes/sin compatibilidad.
Parte de software y librerías para ejecutar un software sin la parte de sistema operativo(kernel) que se usa en comón con otros contenedores.

Los contenedores nos dan una alternativa a las VMs y a las instalaciones a hierro a la hora de poner software a funcionar...
Con diferencias importantes a favor:
- Cuando trabajo con contenedores: NO INSTALO SOFTWARE, lo despliego!

Los contenedores se crean desde IMAGENES DE CONTENEDOR!
Una máquina virtual se crea desde:
- Imagen de disco (ISO)
- Appliance (OVA)

# Imagen de contenedor

Un triste archivo comprimido (tar) que tiene dentro:
- Un esquema/estructura de carpetas (habituialmente) siguiendo la estructura POSIX.
     etc/       Configuraciones
     var/       Datos, logs
     bin/       Binarios de usuario
     opt/       Aplicaciones 
     home/  
     root/
     lib/
     usr/
- Preinstalados en esas carpetas un **MONTONAZO** de programas
  Habitualmente, entre ellos, el programa PRINCIPAL objeto de la imagen de contenedor.
  Por ejemplo, me descargo una imagen de NGINX (para usarlo como servidor web).
  - Vendrá instalado dentro nginx
  - Pero vendrán instalados otros 200 programas!
- Preconfiguraciones / configuraciones por defecto para esos programas, listas para su uso

Además de esa estructura de carpetas y programas preinstalados, las imágenes de contenedor incluyen METADATOS:
- En qué carpetas guarda el programa principal que hay dentro de la imagen sus datos.
- Qué puertos usa ese programa con la preconfiguración por defecto.
- Qué variables de entorno puede leer ese programa para funcionar y qué valores por defecto tienen.
- Cual es el comando que se ejecutará por defecto en el contenedor (cuando se cree desde esta imagen)
- ...

Las imágenes las descargamos de un REGISTRO de REPOSITORIOS DE IMAGENES DE CONTENEDOR.
Hay muchos:
- Docker hub
- Quay.io      (el de redhat)
- Microsoft Artifact Registry
- Google Container Registry
- Oracle Container Registry

TODO EL SOFTWARE EMPRESARIAL HOY EN DIA SE DISTRIBUYE MEDIANTE IMAGENES DE CONTENEDOR! TODO!
Es la forma predilecta de poner a funcionar software hoy en día.

Las imágenes (QUE ES LO QUE DESCARGAMOS) se encuentran en REPOS de REGISTROS.

Una imagen no la descagamos apretando un botón.
Las imágenes las descagan los GESTORES DE CONTENEDORES:

docker image pull <url-de-la-imagen>
podman pull <url-de-la-imagen>

Para ello, debo pasarles una URL:
      registry/repo:tag

      De esas 3 partes, solo el repo es obligatorio.
      Si no pongo registry, el gestor de contenedores usa el registry que tenga configurado por defecto.
      Si no pongo tag, se usa por defecto el tag: latest

`docker image pull nginx`
Realmente la URL de la imagen completa sería:  `docker.io/library/nginx:latest`


Los tags son los que identifican las IMAGES de un repo. Un tag es un texto... que suele contener:
- La versión del software contenido en la imagen
  - Puede ser en forma de version semántica: A.B.C... completa o parcial 
  - Puede ser en base al canal de distribución (stable, beta, nightly, etc.)
  - Los fabricantes además suelen generar un tag llamado latest, para indicar la versión más reciente o estable de la imagen.
    PERO OJO... no es obligatorio el generar tag latest. De hecho esta considerado una muy mal práctica.
    Hay muchos fabricantes que NI SIQUIERA generan un tag latest.

    En muchos casos encontraré cosas como:
        - httpd:latest
        - httpd:2
        - httpd:2.4        *** ESTA ES LA MEJOR OPCION!
        - httpd:2.4.42
    De esas, cuál quiero usar normalmente? Cuál es la mejor elección?

    Si mi aplicación necesita un MySQL, necesitará una determinada funcionalidad de MYSQL, que proporciona un determinado Minor:
        5.7
    Qué versión me interesa montar en ese caso? La que tenga mayor PATCH... más cantidad de bugs arreglados
    Me interesa montar la 5.8.0? Ahi vienen funcionalidades que no uso... y que pueden haber metido bugs nuevos.
    Me interesa usar la 6.0.0? Ni de broma.. quizás mi app deja de funcionar. Vienen breaking changes.
    Me interesa latests? En absoluto... Lo primero que no sé ni que versión es. Un día latest puede apuntar a la 5.7.14 y otro día a la 6.0.0 y que mi sistema deje de funcionar.

    Hay 2 tipos de tags:
    - Fijos: 2.4.42   Siempre apuntan a la misma versión
    - Flotantes: latest   Pueden apuntar a diferentes versiones a lo largo del tiempo
                 2.4
                 2

    Si quiero ser ultraconservador, uso siempre tags fijos: 2.4.42
    En general, en entornos de producción nos interesa fijar minor: 2.4

- Otra cosa que puede venir es información de programas adicionales que vengan instalados.
    tomcat:9.0.17-jdk17 
    tomcat:9.0.17-jdk21 
    tomcat:9.0-jdk21 

- A veces, viene también el nombre la imagen BASE de contenedor desde la que se ha generado la imagen.





Ejemplos de registries.
    docker hub tiene su url de registry: docker.io
    quay.io tiene su url de registry: quay.io
    Microsoft Artifact Registry tiene su url de registry: mcr.microsoft.com
    Google Container Registry tiene su url de registry: gcr.io
    Oracle Container Registry tiene su url de registry: container-registry.oracle.com

## Redes en contenedores

    MAQUINA/HOST
     - loopback  <- Red virtual, dentro de mi máquina - PRIVADA     (127.0.0.0/8)
                    No está asociada a una NIC ni a una red física.
                    Mi máquina pilla la ip 127.0.0.1
                    Esta red se usa para qué?
                      Comunicar procesos por TCP/IP dentro de la misma máquina, sin pisar red física.
                      Oracle: localhost:1521 <- Tomcat
     - eth       <-> NIC (Network Interface Card) - Pinchada a una red física (192.168.X.X/16)
       IP: 192.168.2.127 

Cuando trabajamos con contenedores, los gestores de contenedores crean redes virtuales similares a las de loopback.
En el caso de docker, por defecto crea una red 172.17.0.0/16
El host pilla la ip: 172.17.0.1 (y actúa de gateway)
Los contenedores que se creen en esa red tendrán ips del rango 172.17.0.0/16 secuenciales , a pertir de la 172.17.0.2

     +---------------------------- red de mi empresa (192.168.0.0/24) -------------------+-------
     |                                                                                   |
    192.168.0.200                                                                    Menchu PC
     |
    HOST === 172.17.0.1 ==========+================ red privada virtual de docker (172.17.0.0/16)
     |                            ||
    127.0.0.1                     += 172.17.0.2 - mi-nginx    
     |                                              \ nginx -g 'daemon off;'... Y Este proceso abre puerto 80... en la ip 172.17.0.2
     |
     +---------------------------- red de loopback (127.0.0.0/8)

Desde el host, si quiero llegar al NGINX, que escribo? http://172.17.0.2:80
                       Funcionaría http://mi-nginx:80? NO.. ¿por qué? Es un fqdn que tendré que tener registrado en el Servidor DNS de mi host.
                       Y lo normal es que mi host tenga configurado un DNS de mi empresa.
Desde el MenchuPC, qué pongo? 
    Funciona: http://172.17.0.2:80? No... menchu no tiene acceso (no está enchufada) a mi red virtual privada...

Habría alguna forma de que Menchu llegase a mi contenedor?
Lo que necesito simplemente es buscar el punto (NEXO) entre las 2 redes.. La que está conectada menchu y la que tiene el contenedor.
    - Mi Host y menchu están en la misma red? SI.. por ende, pueden verse/comunicarse entre ellas? SI
    - Mi host y el contenedor están en la misma red virtual privada de docker? SI.. por ende, pueden verse/comunicarse entre ellas? SI

El HOST Es el punto en común.. el nexo entre las 2 redes.
Lo que hago es configurar un NAT( Network Address Translation: Port Forwarding ) en el HOST:
- Creo una regla en el host (sistetma oeprativo) que diga:
  - Si llega una pecición TCP por la ip 192.168.0.200 en el puerto 9999 (o el que me de la real gana), redirige esa petición a la IP: 172.17.0.2 en el puerto 80.

Docker me regala el hacer esa configuración.
Cuando creo un contendor, puedo poner:
`docker container create --name mi-nginx -p 192.168.0.200:9999:80 nginx`

    -p: Quiero que configures un NAT (port forwarding) en el host
        192.168.0.200:9999:80
        192.168.0.200 - La IP que recibe las peticiones que queremos redirigir.
        9999          - El puerto en el que recibimos las peticiones en esa IP
        80            - El puerto al que redirigimos las peticiones en el contenedor.

    Puedo poner en lugar de la IP 192.168.0.200, la máscara 0.0.0.0, que significa "todas las IPs del host".

     +---------------------------- red de mi empresa (192.168.0.0/24) -------------------+-------
     |                                                                                   |
    192.168.0.200                                                                    Menchu PC
     |
    HOST === 172.17.0.1 ==========+================ red privada virtual de docker (172.17.0.0/16)
     |                            ||
    127.0.0.1                     += 172.17.0.2 - mi-nginx    
     |                                              \ nginx -g 'daemon off;'... Y Este proceso abre puerto 80... en la ip 172.17.0.2
     |
     +---------------------------- red de loopback (127.0.0.0/8)

    Si pongo 0.0.0.0:9999:80, desde el host, cómo puedo llegar al nginx?
        - 192.168.0.200:9999
        - 172.17.0.1:9999
        - 127.0.0.1:9999      = localhost:9999
        - 172.17.0.2:80
    Menchu, solo podría acceder usando:
        - 192.168.0.200:9999

0.0.0.0 es el valor por defecto. Puedo escribir directamente:
`docker container create --name mi-nginx -p 9999:80 nginx`

A veces me interesa usar:
`docker container create --name mi-nginx -p 127.0.0.1:9999:80 nginx`

Eso no permite a nadie externo acceder a mi nginx, pero me da una IP Estable interna para acceder al nginx.
A priori, si solo yo quiero acceder a mi nginx, puedo usar su IP... pero tengo que buscarla... y de un arranque a otro puede variar.
Si pongo eso, siempre puedo usar la dirección: localhost:9999 para acceder a mi nginx desde el host.
Siempre funciona y no tengo que buscar IPs.

DOCKER (ni con swarm) no son herramientas ideales para entornos de producción.
Si quiero que otros accedan a mis servicios... en un entorno de producción, no vamos a usar docker. 
Ahí viene Kubernetes.
Eso si... el tema de usar un fqdn (fully qualified domain name) para acceder a los servicios y no una IP depende de una entrada en DNS... que es otro tema!

# Volúmenes.

TODO: Pendiente para mañana

# Devops

Es una cultura, un movimiento, una filosofía en pro de la automatización.
Cuando en la empresa decimos: OYE chic@s vamos a automatizar TODO el curro, entre el DEV y la OPS... decimos que estamos adoptando la cultura DevOps.


## Qué significa Automatizar?

Automatizar es crear una máquina (o cambiar el comportamiento de ella mediante un programa) para que realice la labor que antes un humano hacía con sus manos.

Puedo automatizar el lavado de la ropa: LAVADORA
A la lavadora por cierto, le puedo cambiar su comportamiento mediante PROGRAMAS de lavado: frio, prendas delicadas, ropa de color, etc.

La lavadora automatiza la TAREA de lavar la ropa.
YA no hace falta un humano para esa tarea.

Ahora bien... automatizar la tarea de lavar la ropa es lo mismo que automatizar el PROCESO de lavar la ropa?
El lavado de la ropa, como proceso, implica muchas tareas: 
- Separar la ropa por colores y tipos de tejido.
- Llenar la lavadora con detergente y suavizante.   
- Meter la ropa en la lavadora.
- Seleccionar el programa de lavado adecuado.
- Iniciar el ciclo de lavado.
- Luego se lava la ropa         <<<<< ESTA TAREA ES LA QUE AUTOMATIZAMOS CON LA LAVADORA.
- Se saca de la lavadora.
- Y ya vemos.. que viene el proceso de Secado y Guardado de la ropa.

Una cosa es el proceso, otra cosa son las tareas.

## Cuándo puedo plantearme automatizar un proceso?

Cuando tengo automatizadas sus tareas.

# Tareas en un proyecto de software

Devops (heredado de ALM = Application Lifecycle Management) define una serie de tareas de alto nivel: GRUPOS DE TAREAS:

                Automatizables          Herramientas?
- Plan               poco
- Code               cada día más...
- Build              totalmente             JAVA: maven , gradle
                                            JS/TS: npm, yarn, webpack
                                            C#: dotnet, msbuild, nuget
                                            ...
- Test              
  - desarrollo       cada día más...
  - ejecución        totalmente
                                            Herramientas propias del lenguaje (unitarias, integración.. incluso a veces de sistema) 
                                            JAVA: JUnit, TestNG, Mockito
                                            JS/TS: Jest, Mocha, Sinon, Karma, Cypress
                                            C#: xUnit, NUnit
                                            Herramientas para tareas concertas:
                                             - Calidad de código: Pruebas estáticas: SonarQube, ESLint, StyleCop
                                             - Servicios Web: SoapUI, ReadyAPI, Postman, Karate...
                                             - Rendimiento: JMeter, Gatling, Locust
                                             - Frontales Web: Selenium, Cypress
                                             - Frontales mobile: Appium, Detox
                                             - ...
                    Dónde ejecutamos las pruebas?
                    - Me fío de las pruebas que se ejecutan en la máquina del desarrollador? No... su entorno está maleao!
                    - Me fío de las pruebas que se ejecutan en la máquina del tester?        No... su entorno está maleao!
                    - Me fío de las pruebas que se ejecutan en un entorno controlado de pruebas, creado el día 1 de proyecto:
                      Desarrollo / Pruebas(Preproducción) / Producción
                      Antiguamente SI me fiaba de estas pruebas. Era el entorno donde las hacía.
                      Hoy en día NO. POR QUE? Por el cambio en las formas de trabajo. 
                      - Con una met tradicional, cuántas veces hacía pruebas? 1 al acabar.. por ende cuatas veces instalaba en pruebas? 1, 2 Cuando acababa y se iban a hacer las pruebas
                      - Con una met. ágil, cuándo hago pruebas? Todo el santo día!
                      Y después de 20 instalaciones en el entorno de pruebas, cómo va a estar ese entorno? MALEAO!
                    - Hoy en día creamos entornos de pruebas de usar y tirar:
                     - Necesito hacer puebas? creo el entorno, instalo, hago pruebas y liquido el entorno.
                        Y estos entornos como los creo? Me paso el día creando entornos...
                        Tengo que automatizar la creación y destrucción de estos entornos:
                        - Docker, Kubernetes
                        - Terraform, Vagrant, Ansible, puppet, chef, salt.

- Release
- Deploy
- Operate
- Monitor






































# Kubernetes

Kubernetes es una herramienta para definir / operar mediante lenguaje declarativo entornos de producción basados en contenedores.




---

# Qué era UNIX?

Unix era un Sistema Operativo... Lo hacía los Laboratorios Bell, de la Americana de telecomunicaciones AT&T.
AT&T licenciaba Unix a otras empresas y universidades, no es como hoy en día, que llevan un EULA (End User License Agreement)
Esas empresas (principalmente fabricantes de computadoras) y las universidades adaptaban Unix a sus necesidades/hardware concreto. Muchas luego lo licenciaban a usuarios finales.Cada empresa sacaba su DISTRIBUCION de UNIX.

Problema.. llegó a haber más de 300-400 distribuciones de UNIX diferente, y algunas mostraban incompatibilidades entre sí.
Generaron 2 estándares en paralelo, para poner orden:
- SUS (Single UNIX Specification)
- POSIX (Portable Operating System Interface)

UNIX dejó de fabricarse a principios de los 2000.
Pero los estándares siguen evolucionando.

# Qué es UNIX?

UNIX hoy en día, es un adjetivo que ponemos a aquellos sistemas operativos que cumplen con los estándares SUS y POSIX.
- IBM -> AIX (UNIX®)
- HP -> HP-UX (UNIX®)
- Oracle -> Solaris (UNIX®)
- Apple -> macOS (UNIX®)

# LINUX

Linux NO ES UN SISTEMA OPERATIVO.
Linux es un Kernel de SO.

Un sistema operativo NO ES UN PROGRAMA QUE INSTALO.
Un sistema operativo son MILES DE PROGRAMAS QUE INSTALO.
Esos programas los clasificamos en CAPAS o NIVELES:
- Kernel: Son los programas que:
  - Gestionan el hardware y los recursos del sistema.
  - Controlan la planificación del almacenamiento y la memoria.
  - Getionan las cargas de trabajo : Procesos, hilos y tareas del sistema.. y las llevan a CPU.
  - Seguridad (Usuarios, permisos, autenticación, etc.)
  - Redes (Configuración y gestión de interfaces de red, protocolos, etc.)
  - Y muchos más.
- Herramientas para interactuar con el SO:
  - CLI (Command Line Interface)
  - GUI (Graphical User Interface)
- Herramientas de utilidad:
  - Formatear discos y particiones
  - Ver los procesos que tenemos corriendo
  - Ver los archivos y carpetas que tengo en el sistema.
  - ...

Es un kernel que INICIAMENTE se baso en los estándares de UNIX... hoy en día pasa más bien al contrario... que los estándares de UNIX van tomando ideas de LINUX.
LINUX lleva su linea de desarrollo independiente, sin depender directamente de los estándares de UNIX.

¿Cuál es el kernel de SO más usado del mundo? LINUX

Hay muchos SO que corren el kernel de LINUX:
- En las empresas y algunas personas en sus computadoras se suele usar mucho un SO llamado GNU/Linux.
  Ese sistema operativo se ofrece en forma de distribuciones (cada una lleva unos componentes diferentes):
    - Red Hat Enterprise Linux (RHEL): Fedora, Rocky, Alma, Oracle Linux
    - Debian: Ubuntu, Linux Mint, Pop!_OS
    - Arch Linux: Manjaro, EndeavourOS
    - SuSE: openSUSE, SUSE Linux Enterprise
- Android (corre el kernel de Linux)
- Windows? corre el kernel de Linux?
  Si... nativamente a través del Subsistema de Windows para Linux (WSL). Esto es una característica que puedo habilitar en cualquier Windows...
  Lo ofrece Microsoft. No hay que instalar nada. Entro en características de Windows y habilito WSL.
  Microsoft hizo esto hace muchos años... precisamente para habilitar la ejecución de contenedores Linux en Windows.
  Microsoft, junto con Redhat y AWS fueron los primeros en firmar un acuerdo, con esa startup que nadie conocia llamada Docker.

# Windows

Windows NO ES UN SISTEMA OPERATIVO.
Windows es UNA FAMILIA DE SISTEMAS OPERATIVOS:
- Windows 10
- Windows 11
- Windows Server 2019
- Windows Server 2022
- Windows Server 2022 R2
- Windows 95

Windows tiene su kernel:
- DOS -> MS-DOS, Windows 2, 3, 9x (95, 98, ME)
- NT (New Technology): NT 4.0, Windows 2000, Windows XP, Windows Vista, Windows 7, Windows 8, Windows 10, Windows 11, Windows Server 2003, Windows Server 2008, Windows Server 2012, Windows Server 2016, Windows Server 2019, Windows Server 2022, Windows Server 2022 R2

Windows tiene sus CLIs:
- Command Prompt (cmd.exe)
- PowerShell (powershell.exe)

GUI:
- Metro (Windows 8)
- Fluent Design (Windows 10)

# Nginx

Es un PROXY REVERSO... que con el tiempo ganó funcionalidades de SERVIDOR WEB.
Le pasó lo contario que al Apache httpd... que nació como SERVIDOR WEB y con el tiempo ganó funcionalidades de PROXY REVERSO.

# Esquema semántico de versiones

Casi todos los programas de software siguen un esquema semántico de versiones para su versionado:

    A.B.C

              ¿Cuándo suben?
A = MAJOR     Cuando se produce un BREAKING CHANGE: Cambio que rompe la compatibilidad con versiones anteriores
                Y básicamente es cuando se QUITA ALGO (con o sin reemplazo nuevo)
B = MINOR     Cuando se añaden nuevas funcionalidades
              Cuando se marca algo como obsoleto (deprecated)
C = PATCH     Cuando se corrigen errores: Bugfix

El minor es lo que marca la funcionalidad del software

Al subir un major puede ser que mi sistema deje de funcionar. Si estoy usando una funcionalidad que se ha eliminado o cambiado, tendré que adaptarme a la nueva versión.


# Imágenes BASE DE CONTENEDOR

Cuando un fabricante de un software (Oracle, Microsoft, Redhat, YO) quiere montar una imagen de contenedor, lo que hace es partir de una imagen base:
- Ubuntu
- Debian
- CentOS
- Fedora
- Alpine

Esas imágenes base contienen:
- Las 4 carpetas de posix: /bin, /sbin, /usr/bin, /usr/sbin /etc, /var, /opt...
- Los 4 comandos de posix: ls, cp, mv, rm, cat, vi, echo, tail..., sh
- Programas adicionales que suelen venir con esas distros concretas.

Ubuntu: apt, apt-get
Fedora: dnf, yum
Alpine: apk

Que shell (cli) se utiliza en:
- Ubuntu: bash
- Alpine: sh

Eso es lo que viene en las imágenes base de contenedor.
Es un triste archivo comprimido que sirve como punto de partida para construir imágenes de contenedor más complejas.
Lo que hacemos para generar nuestra imagen de contenedor es descomprimir esa imagen base y añadirle los programas, configuraciones y archivos necesarios para nuestra aplicación... y reempaquetarla.


# MacOS

Ejecuta kernel linux? NO
Ejecuta kernel XNU? SÍ

Entonces, mi máquina puede correr contenedores? No directamente.
He instalado un programa llamado docker desktop.
Ese programa cuando lo arranco, crea un VM Linux en mi máquina MacOS donde los contenedores corren.

# GNU/Linux: Ubuntu

En una máquina Linux (HOST LINUX), puedo correr directamente contenedores.
Si hiciera un ps -eaf de mi máquina Linux, vería todos los procesos en ejecución, incluyendo los procesos de los contenedores que estoy corriendo.
Al fin y al cabo, cuantos kernels hay en esa máquina? UNO
Pues ese UNO, controla TODOS los procesos, incluyendo los de los contenedores.

Los procesosque se ejecutan en un contenedor corren dentro de un proceso QUE ES EL CONTENEDOR.
El contenedor en Linux no es sino un proceso que genera un entorno aislado para ejecutar otros procesos.

Lo que veríamos es el proceso del contenedor en ejecución, y dentro de ese proceso, los procesos que se están ejecutando en el contenedor.