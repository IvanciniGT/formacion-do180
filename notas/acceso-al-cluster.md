# Acceso al clúster de Kubernetes

Cada uno tenéis vuestro propio espacio dentro del clúster del curso: un
**namespace**. Dentro de él podéis crear lo que queráis —pods, deployments,
services, volúmenes— sin molestar a los demás ni que os molesten.

Este documento es para dejarlo todo funcionando en vuestro portátil. Son cuatro
pasos y se hace una sola vez.


## Vuestros datos de acceso

| Dato                     | Valor                                                          |
|--------------------------|----------------------------------------------------------------|
| **Headlamp (navegador)** | https://cluster.ivanosuna.com                                  |
| **Usuario**              | `alumnoN`, donde N es vuestro número de la tabla de abajo      |
| **Contraseña**           | la misma para todos — **la digo en clase, no va escrita aquí** |
| **Namespace**            | `alumnoN`, el mismo número                                     |
| **API del clúster**      | `https://kube-api.ivanosuna.com:6443`                          |

La contraseña no os la piden cambiar al entrar.


## Quién es quién

|  Nº | Alumno          | Usuario    | Namespace  |
|----:|-----------------|------------|------------|
|   1 | Juan Manuel     | `alumno1`  | `alumno1`  |
|   2 | José Ángel      | `alumno2`  | `alumno2`  |
|   3 | Mónica          | `alumno3`  | `alumno3`  |
|   4 | M. Purificación | `alumno4`  | `alumno4`  |
|   5 | Julio Antonio   | `alumno5`  | `alumno5`  |
|   6 | Ricardo         | `alumno6`  | `alumno6`  |
|   7 | David P.        | `alumno7`  | `alumno7`  |
|   8 | Isidro          | `alumno8`  | `alumno8`  |
|   9 | David M.        | `alumno9`  | `alumno9`  |
|  10 | Miguel Ángel    | `alumno10` | `alumno10` |
|  11 | Jesús Ángel     | `alumno11` | `alumno11` |
|  12 | Diego           | `alumno12` | `alumno12` |
|  13 | M. Elena        | `alumno13` | `alumno13` |
|  14 | Isaac           | `alumno14` | `alumno14` |
|  15 | Gustavo         | `alumno15` | `alumno15` |

Hay dos **David** en clase, así que ahí va la inicial del primer apellido:
**David P.** es Pascual y **David M.** es Martín.


## Paso 1 — Conseguir vuestro fichero de credenciales

Para que `kubectl` sepa a qué clúster conectarse y con qué identidad, necesita un
fichero llamado **kubeconfig**. El vuestro está esperándoos dentro de vuestro
propio namespace, y lo recogéis desde el navegador.

**Escribid esta dirección completa, cambiando el número por el vuestro:**

```
https://cluster.ivanosuna.com/c/main/secrets/alumno1/kubeconfig
```

Os pedirá autenticaros: usuario `alumnoN` y la contraseña que doy en clase.

No busquéis vuestro namespace por los menús de Headlamp. El desplegable de
namespaces no os lo va a mostrar, porque para rellenarlo hace falta un permiso
sobre todo el clúster que no tenéis. Con la dirección escrita a mano entráis
igual, porque sobre vuestro propio namespace sí tenéis permiso.

Ya dentro, lo que necesitáis es el valor de la clave **`config`**. Copiadlo
entero.

> **Comprobad que habéis copiado lo correcto.** Los secretos de Kubernetes se
> guardan codificados en base64, y Headlamp tiene un botón para mostrar el valor
> ya descodificado. Lo que tenéis que llevaros empieza así:
>
> ```yaml
> apiVersion: v1
> clusters:
> - cluster:
>     server: https://kube-api.ivanosuna.com:6443
> ```
>
> Si lo que habéis copiado es un churro largo de letras y números sin saltos de
> línea, ese es el base64: buscad el botón de mostrar el valor y copiad lo otro.

Ese contenido es vuestra llave de acceso al clúster. No la compartáis.

## Paso 2 — Instalar kubectl en Windows

`kubectl` es **un único ejecutable**. No hay instalador, no hace falta
administrador y no toca nada del sistema: es un fichero que se descarga y ya
está.

Haceos una carpeta para el curso donde os venga bien —en el escritorio, en
Documentos, donde queráis— y guardadlo ahí. En lo que sigue, cada vez que veáis
`<TU_CARPETA_PARA_EL_CURSO>` ponéis la ruta de la vuestra.

Nuestro clúster es la versión **1.36.4**, así que es esta:

```
https://dl.k8s.io/release/v1.36.4/bin/windows/amd64/kubectl.exe
```

Podéis pegar la dirección en el navegador y guardar el fichero en vuestra
carpeta, o hacerlo desde PowerShell:

```powershell
cd <TU_CARPETA_PARA_EL_CURSO>
curl.exe -LO "https://dl.k8s.io/release/v1.36.4/bin/windows/amd64/kubectl.exe"
```

> Si vuestro portátil es ARM (Surface con Snapdragon y similares), cambiad
> `amd64` por `arm64` en la dirección.

Desde esa carpeta ya funciona, escribiéndolo con `.\` delante:

```powershell
.\kubectl.exe version --client
```

### Para no tener que estar siempre en esa carpeta

Añadid vuestra carpeta del curso al PATH de vuestro usuario. Así `kubectl`
funciona desde cualquier sitio:

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";<TU_CARPETA_PARA_EL_CURSO>", "User")
```

Tres advertencias sobre esto, que es donde se pierde el tiempo:

- **Cerrad la terminal y abrid otra.** El PATH solo se lee al abrir una nueva. Si
  no lo hacéis, parecerá que no ha funcionado.
- **Si la ruta tiene espacios**, ponedla entre comillas al escribir el `cd`:
  `cd "C:\Users\vuestro-usuario\Mis documentos\curso"`.
- **Ejecutadlo una sola vez.** Cada vez que lo lanzáis añade la carpeta otra vez
  al PATH, y acaba lleno de repetidos.

Comprobad que ya os responde desde cualquier carpeta:

```powershell
kubectl version --client
```

## Paso 3 — Colocar el kubeconfig

`kubectl` busca el fichero en una ruta fija de vuestro perfil:

```
C:\Users\<vuestro-usuario>\.kube\config
```

Creadla y poned ahí el contenido del fichero que os he dado:

```powershell
mkdir $HOME\.kube
notepad $HOME\.kube\config
```

> **Ojo con esto, que es el fallo más habitual:** el fichero se llama `config`,
> **sin extensión**. El Bloc de notas le añade `.txt` por su cuenta y sin
> avisar, y entonces `kubectl` no lo encuentra. Al guardar, elegid
> «Todos los archivos» en el tipo, y comprobad con:
>
> ```powershell
> dir $HOME\.kube
> ```
>
> Tiene que salir `config`, no `config.txt`.


## Paso 4 — Comprobar que funciona

```powershell
kubectl version
kubectl auth whoami
kubectl get pods
```

Qué deberíais ver:

- `kubectl version` saca dos versiones, la vuestra y la del servidor (1.36.4).
- `kubectl auth whoami` dice con qué identidad entráis.
- `kubectl get pods` responde **`No resources found`** la primera vez. Eso está
  bien: significa que conecta, que os reconoce y que vuestro namespace está
  vacío todavía.

Fijaos en que **no hace falta poner `-n alumnoN`**. Vuestro kubeconfig ya trae el
namespace puesto por defecto, así que todos los comandos van a vuestro espacio
sin tener que decirlo.

Una primera prueba, para ver algo vivo:

```powershell
kubectl run prueba --image=nginx
kubectl get pods
kubectl describe pod prueba
kubectl delete pod prueba
```


## Lo que podéis y lo que no

**Dentro de vuestro namespace tenéis permisos de administrador.** Crear, borrar y
modificar lo que necesitéis.

**Fuera, no.** No podéis ver ni tocar los namespaces de los demás ni los del
clúster. Si lanzáis un `kubectl get pods --all-namespaces` os dará un error de
permisos: es lo esperado, no es que esté roto.

**Tenéis un límite de recursos** por namespace, que es la suma de todos vuestros
pods juntos:

| Recurso | Petición | Límite |
|---------|----------|--------|
| CPU     | 1        | 2      |
| Memoria | 1 Gi     | 2 Gi   |

Si un pod se queda en `Pending` y `kubectl describe` menciona la cuota, es que
habéis llegado al techo: borrad algo que no estéis usando. Conviene acostumbrarse
a poner `requests` y `limits` modestos en los manifiestos, que además es buena
práctica y entra en el examen.

**La red entre namespaces está cerrada.** Vuestros pods se hablan entre ellos sin
problema dentro de vuestro espacio, pero no pueden alcanzar los pods de un
compañero. Si en algún ejercicio probáis comunicación entre namespaces y no
funciona, no os volváis locos: es así a propósito.


## Si algo no va

| Lo que veis                                  | Qué pasa                                                                        |
|----------------------------------------------|---------------------------------------------------------------------------------|
| `kubectl` no se reconoce como comando        | No habéis cerrado y abierto la terminal después de tocar el PATH                |
| `error: no configuration has been provided`  | No encuentra el kubeconfig: casi siempre es el `config.txt` del Bloc de notas   |
| `Unable to connect to the server`            | Sin red, o la dirección del servidor mal pegada                                 |
| `error: You must be logged in`               | El contenido del kubeconfig está incompleto o mal copiado                       |
| `forbidden` al mirar otros namespaces        | Correcto: solo tenéis el vuestro                                                |
| Versiones muy distintas en `kubectl version` | Tenéis otro `kubectl` por delante en el PATH; comprobad con `where.exe kubectl` |

Si tenéis **Docker Desktop** instalado, ya trae su propio `kubectl` apuntando a
su clúster local. Es la causa más habitual de resultados raros: `where.exe
kubectl` os dice cuál se está usando de verdad.
