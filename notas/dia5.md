
# Almacenamiento

## Cómo funciona el Sistema de Archios de un contenedor.

Qué lleva una imagen de contenedor?

    HOST
        
        /
            bin/
                ls
                cp
                apt
            etc/
            lib/
            usr/
            var/
                lib/
                    docker/
                        images/
                            nginx-latest/       < Aquí se descomprime la imagen de contenedor (No es exacta la representación pero conceptualmente si!)
                                bin/
                                    ls
                                    cp
                                    yum
                                etc/
                                    nginx/
                                        nginx.conf
                                lib/
                                usr/
                                var/
                                    www/
                                        index.html
                                opt/
                                    nginx/
                                        nginx    < Comando ejecutable
                                tmp/
                        containers/
                            mi-nginx/       < Aquí se crea una carpeta para el contenedor concreto (No es exacto la representación pero conceptualmente si!)
                                 /etc/nginx/nginx.conf
                                 /var/log/nginx/access.log
                                 /DATOS/FICHERO.txt
                            mi-nginx-2/       < Aquí se crea una carpeta para el contenedor concreto (No es exacto la representación pero conceptualmente si!)
                                /var/log/nginx/access.log
                                /etc/nginx/nginx.conf
            home/
            tmp/
            opt/
                bin/
                    docker/
                        docker    < Comando ejecutable
            ...

En una imágen de contenedor viene una estructura POSIX de carpetas con binarios ejecutables y configuraciones... Por ejemplo supongamos la de nginx:

        bin/
            ls
            cp
            yum
        etc/
            nginx/
                nginx.conf
        lib/
        usr/
        var/
            www/
                index.html
        opt/
            nginx/
                nginx    < Comando ejecutable
        tmp/

Eso viene comprimido. Cuando descargamos la imagen, eso se descomprime.

Ahora creamos un contenedor. Qué ocurre?
La carpeta de la imagen, se REUSA ENTRE CONTENEDORES.
2 contenedores que yo cree, no necesitan REPLICAR la carpeta de la imagen... LA REAPROVECHAN

Lo que hace el programa que gestiona el contenedor (acordaros que es un proceso) es engañar a los procesos que hay debajo, para hacerles creer que el ROOT
del sistema de archivos es:
            var/lib/docker/images/nginx-latest/   <- chroot

Los contenedores (los programas que corren en un contenedor) escribirá cosas en el FS/lo modificaran:
- Generan logs
- Crean archivos BBDD

Por ejemplo, el nginx genera los en: /var/log/nginx/access.log

La carpeta de la imagen es INALTERABLE, nadie puede hacer cambios en esas carpetas.
Y realmente el proceso es un poco más complejo. El FS se monta por capas (OVERLAY FILE SYSTEM).
La carpeta de la imagen es LA CAPA BASE.

Y sobre ella se superpone la CAP específica de cada contenedor.
Cualquier cambio que se hace en el filesystem se hace sobre la CAPA ESPECÍFICA DEL CONTENEDOR.

Lo que se presenta a los programas que corren en el contenedor es LA SUPERPOSICION DE ESAS CAPAS. Sobre la combinación de ellas es REALMENTE sobre lo que se hace el CHROOT.

LOS CONTENEDORS, a pesar de la falsa creencia de que NO TIENEN PERSISTENCIA EN LOS DATOS,si que la tienen.
Los datos se guardan en HDD del host, en la carpeta específica de cada contenedor dentro de /var/lib/docker/overlay2/.
Si reinicio un contendor, sus datos siguen ahí.

Cuándo si pierdo los datos? SI BORRO EL CONTENEDOR

PERO ESTO NO ES NADA ESPECIAL.

Si tengo una Máquina virtual y la reinicio, pierdo datos? NO
Y si la borro? SI.

Si tengo una Máquina FISICA y la reinicio, pierdo datos? NO 
Y si la borro? SI.

Los contenedores tienen la misma persistencia que las VMs oi las máquinas físicas.

Qué es lo que pasa? 
Que una VM o un MAQUINA FISICA NO SE BORRA NUNCA (a menos que ya no la necesite)
Pero los contenedores NOS PASAMOS EL DIA BORRANDOLOS... en el flujo NATURAL DE TRABAJO CON UN CONTENEDOR.

Si tengo un oracle montado en una VM y lo quiero actualizar, entro en la VM y actualizo.
Si tengo un oracle montado en un contenedor y lo quiero actualizar, lo borro y creo uno nuevo con la versión actualizada.

Si quiero mover un contenedor de una máquina a otra, realmente lo que hacemos es borrar el contenedor en la máquina original y crear uno nuevo en la máquina de destino con la misma imagen y configuración.

ESTE ES EL PUNTO... que los contenedores los borramos de continuo.
    

Al borrar los datos se pierden.. Se borra la carpeta del contenedor.
Cómo resolveríamos eso en una VM o física.
Tengo una carpeta /DATOS que cuando borre el HDD deuna máquina física NO QUIERO QUE SE PIERDAN SUS DATOS. Cómo lo resuelvo?
Esos datos no los guardo e el HDD del computador. Dónde los guardo?
En una cabina de almacenamiento, en una carpeta NFS de un servidor o un SAN externo.
En un cloud.

Y como hago que esa carpeta esté disponible en mi sistema de archivos?
Si quiero que los archivos de /DATOS seán los que hay en una carpeta de un NFS, que hago? MONTARLO en el sistema de archivos MOUNT.

    mount -t nfs <NFS_SERVER>:/ruta/en/nfs /DATOS         POSIX PELAO! Esto se hace en linux, unix, desde hace décadas!

    PUNTO DE MONTAJE EN EL filesystem DONDE SE ACCEDE A LOS DATOS EXTERNOS

Es como si en windows pongo la unidad X: y la apunto a una carpeta de red.

Esto, es lo que en contenedores llamamos un VOLUMEN.
Un volumen es un punto de montaje en el filesystem del contenedor que apunta a un almacenamiento externo al filesystem del contenedor.
Cuando trabajo con Docker, normalmente ese almacenamiento externo al contenedor es una carpeta el el FS del host. 



        /
            bin/
                ls
                cp
                apt
            etc/
            lib/
            usr/
            var/
                lib/
                    docker/
                        images/
                            filebeat/
                                bin/
                                    filebeat
                                etc/
                                lib/
                                usr/
                                var/
                            nginx-latest/       < Aquí se descomprime la imagen de contenedor (No es exacta la representación pero conceptualmente si!)
                                bin/
                                    ls
                                    cp
                                    yum
                                etc/
                                    nginx/
                                        nginx.conf
                                lib/
                                usr/
                                var/
                                    www/
                                        index.html
                                opt/
                                    nginx/
                                        nginx    < Comando ejecutable
                                tmp/
                        containers/
                            mi-nginx/       < Aquí se crea una carpeta para el contenedor concreto (No es exacto la representación pero conceptualmente si!)
                                 /etc/nginx/nginx.conf
                                 /var/log/nginx/ -> REALMENTE SU ALMACENAMIENTO REAL Es EN /DATOS del HOST
                                    access.log
                                 /DATOS/FICHERO.txt
                            mi-nginx-2/       < Aquí se crea una carpeta para el contenedor concreto (No es exacto la representación pero conceptualmente si!)
                                /var/log/nginx/access.log
                                /etc/nginx/nginx.conf
                            mi-filebeat/       < Aquí se crea una carpeta para el contenedor concreto (No es exacta la representación pero conceptualmente si!)
                                /etc/filebeat/filebeat.yml
            home/
            tmp/
            opt/
                bin/
                    docker/
                        docker    < Comando ejecutable
            DATOS/
                access.log

Y al crear el contenedor digo:
 En el contenedor mi-nginx la carpeta /var/log/nginx realmente apunta a la carpeta /DATOS del host
 HAGO UN MOUNT


# NOTA SOBRE LOS LOGS

Los logs de un conteendor son la salida estándar y la salida de error estándar del proceso principal del contenedo (comando) que se está ejecutando dentro del contenedor.
Normalmente los fabricantes de imagenes hacen que los logs en lugar de guardarse a ficheros se redirijan a la salida estándar y a la salida de error estándar.

# Para qué sirven los volúmenes?

- Tener los datos guardados en un sitio que sobreviva a la muerte (BORRADO) del contenedor. PERSISTENCIA
- Compartir ficheros/carpetas entre contenedores.
        apache
            v
            access.log --> Si esto lo guardo en un sitio externo a los contenedores, ambos 2 contenedores pùeden MONTAR en su fs ese volumen externo, y uno escribir y el otro ver.
            ^
        filebeat 

            A priori, los procesos dentro del contenedor de filebeat (que tiene su propio FS) no pueden acceder al FS de otros contenedores.
- Inyectar archivos/carpetas dentro de un contenedor

    Tengo mi propio archivo de configuración de nginx... El que viene en la imagen NO ME VALE! es uno genérico.
    Lo tengo guardado en un volumen externo al contenedor.
    Cuando creo el contenedor, hago un mount de ese volumen en la ruta donde nginx espera su archivo de configuración.
        Por ejemplo: Filebeat define el proceso que debe hacer para leer los logs de otros contenedores en un fichero llamado filebeat.yml.
        Y lo que hago es inyectar ese archivo al contenedor.

---

ESTO ESTA GUAY... pero... En KUBERNETES (Entorno de producción) esto se queda pobre.

 NODO TRABAJADOR 1
    mariadb-1
        ~~DATOS: /var/lib/mysql (en el propio filesystem del contenedor = RUINA!)~~
        ~~DATOS: /var/lib/mysql (en una carpeta del host - volumen tipo docker-) ME VALE? TAMPOCO~~
        ~~Si el contenedor es necesario crearlo en otro nodo, los datos no estarán disponibles.~~
        DATOS: /var/lib/mysql -> Un almacenamiento externo al host, accesible desde cualquier nodo, por ejemplo un NFS.
 NODO TRABAJADOR 2
    mariadb-1
        DATOS: /var/lib/mysql -> Un almacenamiento externo al host, accesible desde cualquier nodo, por ejemplo un NFS. 
 NODO TRABAJADOR 3
 

Kubernetes, de serie define más de 20 tipos de volumenes... Pero instalando plugins acabamos con cientos!

En Kubernetes hay 3 grupos de tipos de volúmenes:
- Volumenes para compartir datos entre contenedores
- Volumenes para persistencia de datos
- Volumenes para inyectar archivos o configuraciones en contenedores

En kubernetes, los volúmenes se definen a nivel de POD.
Y se usan en contenedores.

## Volumenes para compartir datos entre contenedores: EMPTYDIR

```yaml
# Ejemplo simplificado del filebeat y el neginx

kind: Pod
apiVersion: v1

metadata:
  name:             filebeat-nginx-pod
spec:
  containers:
  - name:           nginx
    image:          nginx
    volumeMounts:
    - mountPath:    /var/log/nginx
      name:         carpeta-de-los-logs
  - name:           filebeat
    image:          docker.elastic.co/beats/filebeat:7.10.0
    volumeMounts:
    - mountPath:    /ficheros-a-leer
      name:         carpeta-de-los-logs
  volumes:
  - name:           carpeta-de-los-logs
    emptyDir:       {}    # Ojo a los {} Eso en yaml era un MAPA VACIO... este tipo de volumen no requiere de configuración adicional
                          # Lo que hace es crear una "carpeta vacia" en el HOST. EFIMERA! Si se borra el pod, se borra también la carpeta.
```

# Inyectar archivos o configuraciones en contenedores: CONFIGMAP, SECRET

Los configmap y los secrets SOLO SO CONJUNTOS CLAVE-VALOR.
Para qué usamos esos datos?
- Para rellenar variables de entorno de los contenedores
- Para inyectar archivos de configuración en los contenedores

```yaml
kind:               ConfigMap
apiVersion:         v1
metadata:
  name:             un-configmap
data:
    clave:          valor
    clave2:         valor2
    nginx.conf: |
                    server {
                        listen 80;
                        server_name localhost;
                        location / {
                            root /usr/share/nginx/html;
                            index index.html index.htm;
                        }
                    }
---
kind:               Secret
apiVersion:         v1
metadata:
  name:             un-secret
data:
    certificado.crt: <contenido_base64_del_certificado>
    clave.key:       <contenido_base64_de_la_clave>
---
kind:               Pod
apiVersion:         v1
metadata:
  name:             mi-nginx-pod
spec:
  containers:
  - name:           nginx
    image:          nginx
    volumeMounts:
    - mountPath:    /configuraciones
      name:         configuracion-nginx
    - mountPath:    /secret
      name:         secret-nginx
  volumes:
  - name:           configuracion-nginx
    configMap:
      name:         un-configmap
  - name:           secret-nginx
    secret:
      secretName:   un-secret
```
Cuando aplique esto, y se cree el contenedor,
En el contenedor habrá una carpeta llamada `/configuraciones`.
En ella habrá 2 archivos:
- Un archivo llamado `clave` cuyo contenido será: `valor`
- Un archivo llamado `clave2` cuyo contenido será: `valor2`
- Un archivo llamado `nginx.conf` cuyo contenido será :
    ```
    server {
        listen 80;
        server_name localhost;
        location / {
            root /usr/share/nginx/html;
            index index.html index.htm;
        }
    }
    ```
El Secret, funciona igual

# Volumenes para persistencia:

En kubernetes hay muchos tipos de volumenes para persistencia, como por ejemplo:
- nfs
- iscsi
- fiber channel
- ceph
- glusterfs
- aws EBS
- azure Disk
- gce Persistent Disk

Kubernetes trae unos cuantos....
Pero los fabricantes de cabinas de almacenamiento a menudo proporcionan sus propios controladores para integrarse con Kubernetes.
Y definen sus propios tipos de volumenes.

Cada tipo de volumen requiere una configuración diferente:

```yaml
volumes:
    - name:           mi-volumen-nfs
      nfs:
        server:       <direccion_del_servidor_nfs>
        path:         <ruta_del_recurso_compartido>
    - name:           mi-volumen-iscsi
      iscsi:
        targetPortal:  <direccion_del_servidor_iscsi>
        iqn:           <iqn_del_recurso_iscsi>
        lun:           <numero_del_lun>
        fsType:        <tipo_de_sistema_de_archivos>
    - name:           mi-volumen-aws
      awsElasticBlockStore:
        volumeID:     <id_del_volumen_aws>
        fsType:       <tipo_de_sistema_de_archivos>
```

PROBLEMON!

```yaml

kind:            Pod
apiVersion:      v1

metadata:   
  name:             mi-mariadb
spec:
  containers:
  - name:           mariadb
    image:          mariadb
    volumeMounts:
    - mountPath:    /var/lib/mysql
      name:         mariadb-persistent-storage
  volumes:
  - name:           mariadb-persistent-storage
    #nfs:
    #  server:       <direccion_del_servidor_nfs>
    #  path:         <ruta_del_recurso_compartido>
      # o enun iscsi:
    iscsi:
      targetPortal:  <direccion_del_servidor_iscsi>
      iqn:           <iqn_del_recurso_iscsi>
      lun:           <numero_del_lun>
      fsType:        <tipo_de_sistema_de_archivos>
```

Dónde está el problemón?
Quién dijimos que rellena este archivo?  DESARROLLO/NEGOCIO
Y ese sabe dónde se guardarán los datos? NO

Otro problemón:
- Si despliego el pod en un entorno (namespace) de desarrollo, los datos son los mismo que uso para el entorno de producción?
  Se guardan en el mismo sitio? NOOO!!!

PUES PROBLEMON IDENTIFICADO!

---

# Qué me pediría negocio/desarrollo para una app? En qué términos me habla?

Tengo un equipo que quiero desplegar un sistema en un entorno de producción.
Y piden a su dpto de sistemas que les prepare la infra.
Qué le dices?

    Voy a montar un Wordpress,
    Necesito un servidor para el apache. Dame 4 cores y 16 Gbs de RAM... y dame 500 Gbs de almacenamiento.
    Necesito también una base de datos. Dame 2 cores y 8 Gbs de RAM... y dame 2Tbs de almacenamiento.
    Necesito otro volumen de almacenamiento para backups, de al menos 1Tb.
    Con respecto al almacenamiento, como mucho dire cosas del tipo:
    - Necesito que sea rapidito!
    - Necesito que sea redundante!
    - Necesito que vaya encriptado!
    - El de backups lentito... que me cobra caro si es rapidito.

---

Cómo resuelve kubernetes el problema del lenguaje y el problema dedatos distintos apra distintos entornos?

Mediante 2 nuevos tipos de recursos: 
- PersistentVolume (PV)
- PersistentVolumeClaim (PVC)

## PersistentVolumeClaim (PVC)

Es una petición de almacenamiento, realizada por negocio/desarrollo, que indica cuánto almacenamiento necesita y con qué características, sin preocuparse de los detalles de la infraestructura subyacente.

```yaml
apiVersion:     v1
kind:           PersistentVolumeClaim
metadata:
  name:         mi-peticion-de-almacenamiento-para-apache
spec:
  resources:
    requests:
      storage:      100Gi
  storageClassName: rapidito-redundante
  accessModes:
    - ReadWriteOnce # Ese volumen lo usaré en una máquina!
    # - ReadOnlyMany # Ese volumen lo podrán usar varias máquinas, pero solo en modo lectura.
    # - ReadWriteMany # Ese volumen lo podrán usar varias máquinas, y en modo lectura y escritura.
```

El accessMode condiciona totalmente el tipo de volumen que se puede usar para satisfacer esta petición.
Por ejemplo, NFS permite varios usuarios con escrituras concurrentes, por lo que es compatible con los accessModes ReadOnlyMany y ReadWriteMany.
En cambio iSCSI puede llegar a permitir varios escribiendo... pero solo con sistemas de archivos que soporten bloqueos a nivel de cliente (GFS, ext4 por ejemplo no lo soporta correctamente), por lo que generalmente solo es compatible con ReadWriteOnce.

NFS es un almacenamiento orientado a ficheros. Va muy bien para tener a varias personas dejando y leyendo archivos de manera concurrente.
En cambio, iSCSI es un almacenamiento orientado a bloques. Es mucho más eficiente que NFS, sobre todo cuando no voy a copia/mover archivos enteros... sino que actualizo solo bloques dentro de esos archivos.


---
# Wordpress
 
    BBDD                              Apaches
    mariadb-galera-1                  apache1
    mariadb-galera-2                  apache2
    mariadb-galera-3
    

Pregunta. Los datos en la BBDD. Cada instancia guarda sus datos indepndientes o usan un almacenamiento comun?
En el caso de mariadb y mysql es independiente el almacenamiento.
El montar un cluster activo-activo es para mejorar rendimiento , ademas de ofrecer HA.
Si solo quiero HA normalmente monto un activo-pasivo, mucho más sencillo.

    mariadb-galera1     dato1    dato2
    mariadb-galera2     dato1    dato3
    mariadb-galera3     dato2    dato3

    En 2 unidades de tiempo puedo hacer 3 operaciones. Mejora potencial de rendimiento de un 50%, lo que es frustrante porque hemos hecho un 300% en la infraestructura.

    MariaDB cada dato lo guarda en un fichero independiente o tiene un fichero gordo, que cuando llega un dato lo modifica ese trozo del fichero.
    TODO EN UN FICHERO.

    Dado que cada mariadb va a tener su propio almacenamiento independiente, y dado que los datos se guardan en un único fichero (o en pocos)... que se van editando a trozos, me interesa más NFS o iSCSI?
    Me interesa más iSCSI, porque es un almacenamiento orientado a bloques y permite modificar solo los bloques que cambian dentro de un fichero grande, lo que es más eficiente que NFS en este caso.

    Necesito 3 volumenes independientes iscsi para las bbdd.

Miremos ahora el apache con el wordpress.
Al Wordpress un usuario le sube un pdf.
Ese pdf tiene que estar disponible en todos los apaches... el pdf no se guarda en bbdd. En bbdd se guardan metadatos del pdf: título, tamaño...

Los wordpres editan trocitos de archivos o suben y bajan archivos completos? ARCHIVOS COMPLETOS
Me interesa más un almacenamiento orientado a archivos, que además permita que varios apaches accedan concurrentemente a los mismos archivos, como NFS.
    
Para los apaches me interesa un almacenamiento orientado a archivos, como NFS, que permita el acceso concurrente a los mismos archivos.
1 solo volumen, compartido entre todos los apaches.

```yaml
kind:      PersistentVolumeClaim
apiVersion: v1
metadata:
  name: peticion-volumen-apaches
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  storageClassName: rapidito
---
kind:       PersistentVolumeClaim
apiVersion: v1
metadata:
  name: peticion-volumen-mariadb-1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: rapidito
---
kind:       PersistentVolumeClaim
apiVersion: v1
metadata:
  name: peticion-volumen-mariadb-2
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: rapidito
---
kind:       PersistentVolumeClaim
apiVersion: v1
metadata:
  name: peticion-volumen-mariadb-3
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: rapidito
```

# PersistentVolume

Un persistent volume es un REGISTRO / ALTA de un volumen que debe existir en un backend de almacenamiento, como NFS, iSCSI, o un disco local
que hacemos en kubernetes.

Es decir, INFORMAR a KUBERNETES que existe un volumen físico en el backend de almacenamiento con unas determinadas características.

```yaml
kind:               PersistentVolume
apiVersion:         v1
metadata:
  name:             volumen-1
spec:
  capacity:
    storage: 20Gi
  storageClassName: rapidito
  accessModes:
    - ReadWriteMany
    - ReadWriteOnce
    - ReadOnlyMany
  nfs:
    path: /ruta/en/nfs/volumen-1
    server: servidor.nfs.local
---
kind:               PersistentVolume
apiVersion:         v1
metadata:
  name:             volumen-2
spec:
  capacity:
    storage: 20Gi
  storageClassName: rapidito
  accessModes:
    - ReadWriteOnce
  iscsi:
    targetPortal: iscsi.servidor.local:3260
    iqn: iqn.2024-06.com.example:storage.target01
    lun: 0
    fsType: ext4
```
Esto es lo que rellena un administrador del cluster de kubernetes.

Y OJO, el volumen debe existir en la cabina... Tendré que haber entrado antes en la cabina (YO ADMINISTRADOR DEL CLUSTER) a crear la lun correspondiente.... la cabina me da el iqn, el lun, y demás parámetros necesarios para rellenar el PersistentVolume en Kubernetes.

Esos trabajos, el definir la pv y el definir la pvc se hacen en paralelo. Desarrollo hace su parte y Operaciones hace la suya.

PVs y PVCs no llevan identificadores entre si.

Cuando los cargo en un cluster (kubectl apply -f <archivo>.yaml) entra kubernetes.
Kubernetes es el TINDER de los volumenes... El que hace MATCH entre pvc y la pv.
Kubernetes busca un PV que sea capaz de satisfacer la petición de un determinado pvc.

Por ejemplo:
```yaml
kind:               PersistentVolume
apiVersion:         v1
metadata:
  name:             volumen-1
spec:
  capacity:
    storage: 20Gi
  storageClassName: rapidito
  accessModes:
    - ReadWriteMany
    - ReadWriteOnce
    - ReadOnlyMany
  nfs:
    path: /ruta/en/nfs/volumen-1
    server: servidor.nfs.local
---
kind:      PersistentVolumeClaim
apiVersion: v1
metadata:
  name: peticion-volumen-apaches
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  storageClassName: rapidito
```

Esa pv es capaz de satisfacer esa pvc? CLARO.
Desarrollo pide:
- 10Gi
- ReadWriteMany
- storageClassName: rapidito
El PV tiene? 
- Al menos 10Gi
- ReadWriteMany
- storageClassName: rapidito
Respuesta SI... pues los vincula!

> Pregunta... le da los 20Gi o 10Gi? 

Los 20!
Un volumen es un volumen... es como si voy a comprar un HDD al mediamarkt.
Oiga quiero un HDD para guardar mis fotos, que ocupan 3.6Gb.
Y el tio tiene 2.. uno de 2Gb que mo me vale.. y otro de 5Gbs que si me vale.
Pero me da el disco entero de Gb... no puede partir el disco.

Definidas la pv  la pvc. Qué podríamos en el pod?

```yaml
kind: Pod
apiVersion: v1
metadata:
  name:     mi-pod-apache
spec:
  containers:
    - name: mi-contenedor-apache
      image: wordpress
      volumeMounts:
        - mountPath: /carpeta-para-ficheros
          name: mi-volumen
        - mountPath: /carpeta-para-backups
          name: mi-volumen-2
  volumes:
    - name: mi-volumen
      persistentVolumeClaim:      # Aqui no pongo el tipo de volumen... pongo la referencia a la PVC que quiero usar
        claimName: peticion-volumen-apaches 

    - name: mi-volumen-2
      persistentVolumeClaim:      # Aqui no pongo el tipo de volumen... pongo la referencia a la PVC que quiero usar
        claimName: peticion-volumen-apaches-2
```

Pero... aún hay 2 problemas!

- Una de las gracias de kubernetes es el escalado dinámico.
  lo puedo hacer vía: kubectl scale deployment <nombre-del-deployment> --replicas=<numero-de-replicas>

  También se puede hacer sin intervención humana:
  ```yaml
        apiVersion: autoscaling/v1
        kind: HorizontalPodAutoscaler
        metadata:
            name: mi-escalador-apaches
        spec:
            scaleTargetRef:
            apiVersion: apps/v1
            kind: Deployment
            name: despliegue-apaches
            minReplicas: 3
            maxReplicas: 10
            targetCPUUtilizationPercentage: 50
  ```

  Todos mis apaches trabajan contra el mismo volumen.
  Hay problema si mañana meto 3 apaches nuevos? NINGUNO

  Pero... los mariadbs nofuncionan así... Cada mariadb necesita su propio volumen persistente.
  Y si mañana escalo el despliegue de mariadbs? Necesitaré un nuevo volumen persistente para cada nueva instancia.



    BBDD                              Apaches
    volumen1 < mariadb-galera-1                  apache1
    volumen2 < mariadb-galera-2                  apache2
    volumen3 < mariadb-galera-3                  apache3    
    volumen4 < mariadb-galera-4
                                                    VVV
                                            Volumen NFS compartido  

Cómo se resuelve eso? Con los statefulsets.

Un statefulset es como un deployment, pero donde a cada réplica se le asigna un volumen persistente único, garantizando que cada instancia tenga su propio almacenamiento dedicado. Y para ello define una plantilla de petición de volumen persistente dentro de su especificación.
Y kubernetes, cuando escala el pod, no solo crea replicas del pod, sino que también crea y asigna automáticamente nuevas peticiones de volúmenes persistentes únicos para cada nueva réplica.

```yaml
kind:               StatefulSet
apiVersion:         apps/v1
metadata:
  name:            mi-statefulset-mariadb
spec:
  serviceName: "mi-servicio-mariadb"
  replicas:         3
  selector:
    matchLabels:
      app:          mariadb
  template:
    metadata:
      labels:
        app:        mariadb
    spec:
      containers:
      - name: mariadb
        image: mariadb:latest
        volumeMounts:
        - name: mariadb-storage
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mariadb-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
      storageClassName: rapidito
```

Para cada mariadb en el statefulset, Kubernetes creará automáticamente una petición de volumen persistente única basada en la plantilla definida en `volumeClaimTemplates`.


El otro problema.
Llega un desarrollador.. y lanza su pvc a kubernetes. 
Habrá una pv que cumpla con lo solicitado en la pvc del desarrollador? Normalmente NO.
Entonces? 
El desarrollador que hace, espera? Manda un ticket al CAU para que operaciones le cree la PV correspondiente. Esto del modelo de autoservicio se va a la porra!

Los administradores de cluster normalmente hacen otra cosa.
Montan un PROVISIONADOR DINAMICO DE VOLUMENES.

Eso es un programa que :
- Cuando se lanza una pvc va al backend correspondiente, y crea allí el volumén físico
- Saca los datos del backend para ese volumen
- Y registra en kubernetes la nueva PV correspondiente a la PVC del desarrollador.

Lo hace solo.

De hecho lo normal es que en un cluster existan VARIOS PROVISIONADORES DINAMICOS DE VOLUMENES, cada uno encargado de un backend diferente o de un tipo de almacenamiento específico.
- volumenes de ficheros
- bloques de almacenamiento
- almacenamiento en la nube (EBS, GCE PD, etc.)

Y para evitar que la gente se flipe! en kubernetes ya diimos que en un NAMESPACE puedo meter limitaciones de acceso a recursos del cluster.
Igual que limito cuanta RAM y CPU puede usar un namespace, también puedo limitar el acceso a ciertos tipos de almacenamiento o a ciertos provisionadores dinámicos de volúmenes.

Eso son los ResourceQuota:
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: storage-quota
spec:
  hard:
    requests.cpu: "4"
    requests.memory: "8Gi"
    requests.storage: 100Gi
    persistentvolumeclaims: "5"
    rapidito.storage: 50Gi
```

Los provisionadores dinámicos se asocian al StorageClass.

rapidito-ficheros -> Cabina huawei con ssds
lentito-ficheros -> Cabina huawei con discos mecánicos
rapidito-bloques -> Cabina huawei con ssds
lentito-bloques -> Cabina huawei con discos mecánicos


Los desarrolladores pueden ver que TIPOS DE ALMACENAMIENTO (STORAGE CLASS) están disponibles en el cluster usando el comando:
```bash
kubectl get storageclass
oc get storageclass
```

Evidentemente para esto hace falta que ese programa sepa hablar con el backend correspondiente, ya sea una cabina de almacenamiento, un servicio de bloques o un proveedor de almacenamiento en la nube.

Eso es lo que os enseñé antes que había montado para un cliente y sus cabinas HUAWEI.

Y cada vez queda más separado el uso/operaciones que hacen los desarrolladores del mantenimiento/operaciones que hacen los administradores de cluster.

Y cada uno tiene sus objetos de kubernetes.. sus tipos de recursos.

Lo que pasa es que lo que antes necesitaba un equipo de 10 sysadmins, ahora lo hago con 2.
En cuanto dejan configurado el cluster...Kubernetes(y sus provisionadores...) se encargan de todo.


minikube
openshift local

5x16 = 80 GB RAM
8x5 = 40 cores x 2 hilos = 80 core-virtuales

redhat ofrece un cluster de 30 dias renovable gratis de 1 nodo

Developer Sandbox

---

Lo normal noo es escribir estos archivos de manifiesto YAML...
Eso no lo hacemos A NO SER QUEQUERAMOS DESPLEGAR UNA APP PROPIA EN EL CLUSTER. ENTONCES SI!
Y siempre querre desplegar apps propias... Aunque más de la mitad de lo que despliegue será estandar:
- BBDD
- Sistema de mensajería
- Sistema de cache
- ElasticSearch
- Gitlab
- Jenkins
- SonarQube

Y lo que hay son PLANTILLAS.
El el mundo kubernetes existe una herramienta llamada HELM (el apt, yum de kubernetes).
Y las plantillas de helm se llaman CHARTS.
Y hay decenas de miles de charts.
Ahora... esas plantillas... agarrate!
Que no es apt install apache!!!!

Que llevan ficheros parametrizables YAML de miles de lineas cada una.

---

Con este nuevo modelo entonces asumimos que los servicios los dejan de administrar los sysadmin y pasan a ser cosa de los desarrolladores, sin duda es un atribución gorda de funciones..

TOTALMENTE! Pero cambiemos el nombre: DESARROLLO -> NEGOCIO!

Los equip de negocio tienen:
- sus desarrolaldores
- sus testers
- sus propios sysadmins (mal llamados devops)

Lo que vamos a es a un modelo FEDERADO!

Este modelo aplica igual en clouds, que también trabajan con un modelo de AUTOSERVICIO para los equipos de negocio.