# Entorno

- Descargar kubectl y añadirlo al path
- Descargar un archivo llamado config y colocarlo en la carpeta:
  c:\Usuarios\<Mi-USUARIO-DE-WINDOWS>\.kube

  El archivo config contiene la configuración necesaria para que kubectl se conecte al cluster de Kubernetes:
  - Usuario
  - "Contraseña" del usuario (TOKEN)
  - Certificado CA que es la que firma el certificado https del cluster
  - URL del cluster

# Recursos

## NAMESPACE: Agrupación de recursos (habitualmente por entorno y aplicación):

    - Limitar los recursos físicos (CPU, RAM) que pueden ser utilizados por los pods dentro del namespace
    - Controlar el acceso/gestión de los recursos dentro del namespace (Usuarios)

    En kubernetes, la idea es acabar con un modelo de AUTOSERVICIO, donde los desarrolladores pueden solicitar NAMESPACES y desplegar aplicaciones sin necesidad de intervención manual del equipo de operaciones.

    ```yaml
    kind:           Namespace
    apiVersion:     v1
    metadata:
      name:         mi-namespace
    ```
    
    NOTA: `kubectl create namespace <mi-namespace>`

    Ese comando existe... como muchos otros comandos de tipo CREATE... y no los usamos nunca!
    Todo lo que ejecutemos en una terminal NO QUEDA REGISTRADO EN NINGUN SITIO.
    Cualquier cosa que vayamos a crear en un cluster de kubernetes la queremos en GIT, versionada!

    SOLO CUANDO ALGO NO QUEREMOS QUE QUEDE REGISTRADO (QUE DEJE HUELLA) QUE SEA REPRODUCIBLE usamos este tipo de comandos... y pocas cosas queremos que se creen sin dejar huella y sin ser reproducibles. Entre esas pocas cosas estarían los SECRETOS:

    `kubectl create secret generic <nombre-del-secreto> --from-literal=<clave>=<valor>`

## POD

Un pod es un conjunto de contenedores, que:
- Se despliegan en la misma máquina del cluster (nodo):
  - Comparten configuración de red: Interfaces de red:
    - Compartan IPs:
      - Comparten la IP del pod en la red de kubernetes
      - Comparten interfaz de loopback: 127.0.0.1 - localhost
  - Podían compartir volumenes de almacenamiento local (carpetas locales)
- Se despliegan juntos y escalan juntos... van en pack

    ```yaml
    kind:           Pod
    apiVersion:     v1
    metadata:
      name:         mi-pod
    spec:
      containers:
      - name:            mi-contenedor
        image:           nginx:latest
        imagePullPolicy: IfNotPresent
    ```

Ayer creamos un pod...

> Cuántos pods vamos a crear en un cluster "estandar" de kubernetes?

NINGUNO!
Evidentemente era pregunta trampa... Los podsNOSOTROS NO LOS CREAMOS. Los va a crear Kubernetes.
Pero voy a otro concepto... es que NI SIQUIERA VAMOS A DEFINIR NI UN SOLO FICHERO DE POD.

>> Si creo un pod... y quiero ahora tener 2? qué tengo que hacer?

Tengo que crear OTRO POD. Copio el archivo y le cambio el nombre = RUINA!

LO QUE DEFINIMOS EN UN CLUSTER DE KUBERNETES SON "PLANTILLAS DE PODS" OH!!!
Nosotros creamos una plantilla de pod. Y se la pasamos a Kubernetes.
Y le decimos CUANTOS PODs queremos que él cree desde esa plantilla.
Kubernetes es quien crea los fichero yaml del tipo:

```yaml
kind:           Pod
apiVersion:     v1
...
```

Para crear estas plantillas, kubernetes ofrece 3 tipos de recursos: Deployments, StatefulSets, DaemonSets:
- Deployment:   Plantilla de pod + Número inicial de réplicas (es decir, de pods que quiero que kubernetes cree desde esa plantilla)
- StatefulSet:  Plantilla de pod + Número inicial de réplicas + Plantilla de PVC
- Daemonset:    Plantilla de pod de la que Kubernetes crea tantas réplicas como nodos hay en el cluster (1 réplica por nodo)
                Los daemonsets son raros... y para despliegues de apps es muy muy raro que se usen.
                Son más para cosas de infraestructura:
                - Monitorización (necesito un programa (pod) en cada nodo del cluster)
                - ...
La elección de si crear un Deployment o un Statefulset NO LA TOMAMOS... VA TOTALMENTE CONDICIONADA POR EL TIPO DE PROGRAMA QUE QUEREMOS MONTAR
- Apache -> Deployment
- MariaDB -> StatefulSet
  
## Configmap y secrets

Conjuntos clave-valor para:
- Independizar el archivo del pod (desarrollador) de ciertos datos (operaciones)
- Permitir tener varios conjuntos de datos diferentes para distintos entornos o configuraciones.


---

## Sobre el cliente de kubernetes

La sintaxis es muy simple:

kubectl <VERBO> <TIPO_RECURSO> <ARGS>

> Tipos de recursos:

| TIPO_RECURSO                    | ALIAS |
|---------------------------------|-------|
| namespace (namespaces)          | ns    |
| pod (pods)                      | po    |
| configmap (configmaps)          | cm    |
| secret (secrets)                |       |

> VERBOS:

Hay algunos genéricos, que podemos aplicar sobre cualquier TIPO_RECURSO:

| VERBO                         | DESCRIPCIÓN                                       |
|-------------------------------|---------------------------------------------------|
| get                           | Listado de recursos                               |
| describe     <name>           | Describir un recurso en detalle                   |
| delete       <name>           | Eliminar un recurso                               |

> ARGS:

<name>                          Cuando quiero hacer operaciones que involucran un recurso específico, debo indicar su nombre aquí.
--namespace, -n <namespace>     Indica el namespace en el que se encuentra el recurso.
-o wide                         Muestra información adicional en el listado de recursos (GET).

### Además hay otros 3 comandos que ejecutamos con frecuencia

| VERBO                         | DESCRIPCIÓN                                        |
|-------------------------------|----------------------------------------------------|
| create       -f <archivo>     | Crear un recurso a partir de un archivo YAML       |
| apply        -f <archivo>     | Aplicar cambios a un recurso desde un archivo YAML |
| delete       -f <archivo>     | Eliminar un recurso a partir de un archivo YAML    |

---

# COMUNICACIONES EN UN CLUSTER DE KUBERNETES

    192.168.0.100:30080
    192.168.0.101:30080
    192.168.0.102:30080
    192.168.0.201:30080
    192.168.0.202:30080
    192.168.0.203:30080                 *.miempresa -> 192.168.0.10    http://miapp.empresa
      |                                   |                                     |
   192.168.0.10:80  Balanceador          DNS                                 MenchuPC
      |                                   |                                     |  
 +----+-----------------------------------+-------------------------------------+-- red de la empresa   (192.168.0.0 /24)
 |
 ++-- 192.168.0.100 -- ControlPlane1
 ||                      Linux + Kubelet
 ||                         netfilter:
 ||                             10.10.0.200:3307   -> 10.10.0.100:3306 
 ||                             10.10.0.201:80     -> 10.10.0.101:80 | 10.10.0.102:80
 ||                             192.168.0.100:30080 -> 10.10.0.202:80
 ||                             10.10.0.202:80      -> 10.10.0.103:80
 ||                      gestor de contenedores (CRIO-ContainerD)
 |+------------------------- core-dns-1
 ||                                 mibbdd -> 10.10.0.200
 ||                                 wp     -> 10.10.0.201
 ||                                 ingress-controller -> 10.10.0.202
 |+------------------------- kube-proxy
 ||
 ++-- 192.168.0.101 -- ControlPlane2
 ||                      Linux + Kubelet
 ||                         netfilter:
 ||                             10.10.0.200:3307 -> 10.10.0.100:3306 
 ||                             10.10.0.201:80   -> 10.10.0.101:80 | 10.10.0.102:80
 ||                             192.168.0.101:30080 -> 10.10.0.202:80
 ||                             10.10.0.202:80      -> 10.10.0.103:80
 ||                      gestor de contenedores (CRIO-ContainerD)
 |+------------------------- core-dns-2
 ||                                 mibbdd -> 10.10.0.200
 ||                                 wp     -> 10.10.0.201
 ||                                 ingress-controller -> 10.10.0.202
 |+------------------------- kube-proxy
 ||
 ++-- 192.168.0.102 -- ControlPlane3
 ||                      Linux + Kubelet
 ||                         netfilter:
 ||                             10.10.0.200:3307 -> 10.10.0.100:3306 
 ||                             10.10.0.201:80   -> 10.10.0.101:80 | 10.10.0.102:80
 ||                             192.168.0.102:30080 -> 10.10.0.202:80
 ||                             10.10.0.202:80      -> 10.10.0.103:80
 ||                      gestor de contenedores (CRIO-ContainerD)
 |+------------------------- kube-proxy
 ||
 ++-- 192.168.0.201 -- Worker1
 ||                      Linux + Kubelet
 ||                         netfilter:
 ||                             10.10.0.200:3307 -> 10.10.0.100:3306 
 ||                             10.10.0.201:80   -> 10.10.0.101:80 | 10.10.0.102:80
 ||                             192.168.0.201:30080 -> 10.10.0.202:80
 ||                             10.10.0.202:80      -> 10.10.0.103:80
 ||                      gestor de contenedores (CRIO-ContainerD)
 |+------------------------- kube-proxy
 |+-- 10.10.0.101 ---------- pod-apache
 ||                                 contenedor apache+wp ":80"     ->      10.10.0.101:80
 ||                                         Aquí hay un fichero llamado wp-config.php. Y dentro pone:    db_url=mibbdd:3307
 |+-- 10.10.0.103 ---------- pod-nginx(ingress-Controller)
 ||                                 contenedor nginx ":80"         ->      10.10.0.103:80
 ||                                         Daré una regla (virtual host)
 ||                                          http://miapp.empresa/:80 -> wp:80                  INGRESS
 ||
 ++-- 192.168.0.202 -- Worker2
 ||                      Linux + Kubelet
 ||                         netfilter:
 ||                             10.10.0.200:3307 -> 10.10.0.100:3306 
 ||                             10.10.0.201:80   -> 10.10.0.101:80 | 10.10.0.102:80
 ||                             192.168.0.202:30080 -> 10.10.0.202:80
 ||                             10.10.0.202:80      -> 10.10.0.103:80
 ||                      gestor de contenedores (CRIO-ContainerD)
 |+------------------------- kube-proxy
 |+-- 10.10.0.102 ---------- pod-apache
 ||                                 contenedor apache+wp ":80"     ->      10.10.0.102:80
 ||                                         Aquí hay un fichero llamado wp-config.php. Y dentro pone:    db_url=mibbdd:3307
 ||
 ++-- 192.168.0.203 -- Worker3
  |                      Linux + Kubelet
  |                         netfilter:
  |                             10.10.0.200:3307 -> 10.10.0.100:3306 
  |                             10.10.0.201:80   -> 10.10.0.101:80 | 10.10.0.102:80
  |                             192.168.0.203:30080 -> 10.10.0.202:80
  |                             10.10.0.202:80      -> 10.10.0.103:80
  |                      gestor de contenedores (CRIO-ContainerD)
  +------------------------- kube-proxy
  +-- 10.10.0.100 ---------- pod-mariadb
  |                                 contenedor mariadb ":3306"     ->      10.10.0.100:3306
  |
  Red overlay (VLAN, VXLAN...) propia del cluster de kubernetes. Las máquinas del cluster se conectan a esa red. 10.10.0.0/24
  Y los pods se conectan a esta red también


mibbdd -> fqdn resoluble vía un servidor DNS

Eso lo podemos crear en kubernetes... De hecho si recodaís kubernetes tiene su propio DNS: coreDNS

Ese es el concepto de un nuevo tipo de recurso del que aún no hemos hablado: SERVICE

En Kubernetes hay 3 tipos de SERVICIOS:
- ClusterIP

    Un SERVICE CLUSTERIP es una entrada en el DNS de Kubernetes que apunta a una IP FIJA DE BALANCEO QUE GENERA KUBERNETES... a la que se asocia un puerto, que apunta al puerto de TODOS LOS CONTENEDORES con la etiqueta... en balanceo!
        Podría crear un service llamado "mibbdd" -> 10.10.0.200
            El tema es que necesito que cuando alguien llame a esa IP 10.10.0.200:3307 -> 10.10.0.102:3306
        Podría crear un service llamado "wp"     -> 10.10.0.201

    ```yaml
    apiVersion:         v1
    kind:               Service
    metadata:
    name:             mibbdd  # Este es el fqdn que se da alta en el DNS de Kubernetes
    spec:
    ports:
        - port:         3307    # Este es el puerto de la IP FIJA que se va a asociar al fqdn en DNS
          targetPort:   3306    # Este es el puerto del contenedor al que se redirige el tráfico
    selector:
        app:            bbdd    # Label que ponemos en el template del que se generan los pods
    ```
    Los servicios clusterip ofrecen 2 cosas:
    - Acceso vía un nombre de dominio resoluble a una IP DENTROL DEL CLUSTER (DNS)
    - HA (Balanceo de carga)

- NodePort = ClusterIP + NAT PortForwarding en cada nodo del cluster en el puerto que elija (PERO... debe estar por encima del 30000)

    ```yaml
    apiVersion:         v1
    kind:               Service
    metadata:
    name:               wp  # Este es el fqdn que se da alta en el DNS de Kubernetes
    spec:
    ports:
        - port:         80    # Este es el puerto de la IP FIJA que se va a asociar al fqdn en DNS
          targetPort:   80    # Este es el puerto del contenedor al que se redirige el tráfico
          nodePort:     30080
    selector:
        app:            bbdd    # Label que ponemos en el template del que se generan los pods
    type:               NodePort   # ClusterIP (default), NodePort, ...
    ```

- LoadBalancer = NodePort + Kuberneter configura automaticamente un balanceador de carga COMPATIBLE (y que se debe de instalar por separado)
    Cuando trabajo con un cluster de kubernetes que alquilo en un cloud (AWS, GCP, Azure...) esos clouds me "regalan" (€€€€) un balanceador externo compatible con kubernetes.
    Si monto yo un cluster de kubernetes on prem, necesito montar YO un balanceador de carga compatible con kubernetes: METALLB. MetalLB realmente no hace balanceo, hace failover... Manda todo a un nodo...si ese se cae, le pega el cambiazo por otro.



> Netfilter?

Os suena iptables? SI
Netfilter es un componente del Kernel de Linux.
Cualquier paquete de red que circula por el kernel de Linux pasa primero por Netfilter.
IPTable genera reglas para netfliter

> En un cluster de kubernetes "estandar", cuántos servicios de cada tipo voy a tener?

                         Cuántos?
    -------------------------------
    ClusterIP             Todos-1    Servicios internos al cluster: MariaDB
    -------------------------------
    NodePort                 0       Servicios accesibles desde fuera del cluster: Apache + WP          
    LoadBalancer             1       PROXY REVERSO!  Que en kubernetes recibe el nombre de INGRESS CONTROLLER

Los servicios son por app. Cada app tiene su servicio:
    - MariaDB   -> Servicio 1
    - Apache    -> Servicio 2

---

    MariaDB <- Tomcat  <--
               Tomcat  <--  Balanceador de carga <- Proxy REVERSO      <--- Proxy? <--- Clientes?
    -------------------------------------------------------------      ---------------------------
                                                          INVERSO

# Qué hace un proxy?

- Un proxy es una herramienta que PROTEGE A LOS CLIENTES
  Cuando el cliente hace una petición, el proxy la intercepta
  Ralmente, la petición el cliente se la hace al PROXY.
  El proxy NO REDIRIGE... El proxy deja al clienmte a la espera... Y EL (PROXY) hace la petición en nombre del cliente.
  El cliente DELEGA en el proxy la responsabilidad de comunicarse con el servidor final.
  El servidor ve la IP del PROXY... es quien llama.
  El proxy coje la respuesta y la valida (MIRA QUE NO TENGA VIRUS, QUE NO haya un intento de phising, valida certificados...DE PASO NOS JODEN Y NOS FILTRAN) y la manda al cliente

# Qué hace un proxy reverso

- Un proxy reverso es una herramienta que PROTEGE A LOS SERVIDORES
  Cuando un cliente hace una petición (vía proxy o no), el proxy reverso la intercepta
  Realmente, la petición del cliente se la hace al PROXY REVERSO.
  El proxy reverso NO REDIRIGE... El proxy deja al servidor a la espera... Y EL (PROXY REVERSO) hace la petición en nombre del cliente.
  El cliente ve la IP del proxy reverso... es quien llama.
  Si un cliente hace 500 peticiones/segundo, el proxy reverso las rechaza (DROP) EVITA UN ATAQUE DDOS.
  Si un cliente abre 20 conexiones.. y empieza a mandar 1 byte por segundo, el proxy reverso puede detectar este comportamiento como un intento de ataque de tipo Slowloris y tomar medidas para mitigarlo.
  Puede actuar de cache, para quitar peticiones a los servidores.

  NGINX

# INGRESS

Un ingress es una REGLA para un PROXY REVERSO.

            http://miapp.empresa:80/ -> wp:80                  INGRESS

```yaml
kind:               Ingress
apiVersion:         networking.k8s.io/v1
metadata:
  name:             miapp-regla
spec:
  rules:
  - host:               miapp.empresa
    http:
      paths:
      - path:           /
        pathType:       Prefix
        backend:
          service:
            name:       wp
            port:
              number:   80
```

Ésto se conveertiría en automático en una regla dentro del fichero de configuración de NGINX, del tipo:

```nginx.conf
server {
    listen       80;
    server_name  miapp.empresa;

    location / {
        proxy_pass http://wp:80;
    }
}
```

Claro.. que si en lugar de nginx, usase apache:
```apache.conf
<VirtualHost *:80>  
    ServerName miapp.empresa

    ProxyPass / http://wp:80/
    ProxyPassReverse / http://wp:80/

</VirtualHost>
```

En HAProxy o envoy... otras sintaxis diferentes.

INGRESS me da una sintaxis agnóstica.... independiente del Proxy Reverso concreto que se use... que a mi como desarrollador me la trae al peiro!
Es más... el día de mañana puede cambiar.
A mi me aisla de ese concepto (INFRA)

De echo lo que se instala en un cluster (LOS ADMINISTRADORES DEL CLUSTER) es un INGRESS CONTROLLER:
Qué es un INGRESS CONTROLLER:
- Un PROXY REVERSO
- Un programa que es capaz de generar la sintaxis requerida por ese PROXY REVERSO a partir de las reglas definidas en los INGRESS.


---

# Operación del cluster

Kubernetes es quien opera el cluster.
Y continuamente está monitorizando los pods, y sus contenedores.

Pero cómo sabe kubernetes si un pod está operativo o no?
- Lo primero que hace (no kubernetes sino) el gestor de contenedores que corre el contenedor es mirar si el proceso principal que corre dentro (nginx master...) sigue vivo (NO EXISTED). Si kubernetes ve que el proceso cayó, inmediatemente inicia un nuevo pod.

PERO... esto no es suficiente.
Qué un proceso esté vivo significa ya que el programa que corre está bien? NO

Por eso kubernetes permite definir PROBES en los contenedores.
Los PROBES son pruebas que kubernetes ejecuta con la periodicidad que le indiquemos para verificar si el contenedor está funcionando correctamente.
- Pueden ser scripts que se ejecuten (y se mira si el codigo de salida es 0)
- Puede ser una llamada por http (y se mira el codigo de respuesta)
- ...

Pero... la gracia es que define 3 TIPOS DE PROBES.

- Startup probes
- Liveness probes
- Readyness probes

Cada tipo de probe tiene un propósito específico:
- Startup probes: Verifican si la aplicación dentro del contenedor ha arrancado correctamente. está arrancando aún.
    Se le da un timeout.
    Si transcurrido el tiempo indicado no responde bien la prueba       -> EL POD ES REINICIADO O RECREADO.
        Tengo un MARIADB.. y puede tardar 5 minutos en arrancar (está en una migración)

- Liveness probes: Verifican si la aplicación sigue viva y funcionando.
    Una vez la prueba de arranque ha dado ok, comienzan las pruebas de liveness.
    Si van mal (se configura timeout, retries)                          -> EL POD ES REINICIADO O RECREADO.
        Ejecuta un mysql -u admin A... A ver si contesta.  No tiene porque ser a usuarios
        Quizás está haciendo un backup ... y tarda 15 minutos... dejalo!

- Readyness probes: Verifican si la aplicación está lista para recibir tráfico.
    Si la prueba de vida da bien, se ejecutan pruebas de readyness, apra ver si el contenedor está listo para prestar servicio
    Si van mal                                                          -> SE SACA EL POD DE BALANCEO (SERVICIO)
        Ejecuta SELECT 1