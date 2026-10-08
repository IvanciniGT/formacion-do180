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
- StatefulSet:  Plantilla de pod + Número inicial de réplicas + ???
- Daemonset:    Plantilla de pod de la que Kubernetes crea tantas réplicas como nodos hay en el cluster (1 réplica por nodo)
                Los daemonsets son raros... y para despliegues de apps es muy muy raro que se usen.
                Son más para cosas de infraestructura:
                - Monitorización (necesito un programa (pod) en cada nodo del cluster)
                - ...
La elección de si crear un Deployment o un Statefulset NO LA TOMAMOS... VA TOTALMENTE CONDICIONADA POR EL TIPO DE PROGRAMA QUE QUEREMOS MONTAR

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