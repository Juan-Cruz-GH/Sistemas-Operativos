<h1 align="center">Práctica 4B</h1>

## Docker

### Introducción

#### 1. Utilizando sus palabras, describa qué es Docker y enumere al menos dos beneficios que encuentre para el concepto de contenedores.

Docker es un proyecto de software open-source que automatiza el despliegue de aplicaciones dentro de **contenedores**, proporcionando una capa adicional de abstracción y automatización de virtualización de aplicaciones en múltiples sistemas operativos.

Docker usa características de aislamiento de recursos de Linux, tales como cgroups y espacios de nombres (namespaces) para permitir que contenedores independientes se ejecuten dentro de una única instancia de Linux, evitando el overhead de iniciar y mantener máquinas virtuales.

Beneficios de los contenedores:

- **Portabilidad**: Los contenedores se ejecutan de forma consistente entre entornos porque se empaquetan junto con todas sus dependencias.
- **Aislamiento**: Cada contenedor se ejecuta de forma aislada a todos los demás, reduciendo conflictos entre aplicaciones y mejorando la seguridad.
- **Escalabilidad**: Los contenedores trabajan bien con herramientas como Kubernetes que ayudan a escalar una aplicación.
- **Eficiencia**: Los contenedores comparten el kernel del SO host, lo cual los hace más rápidos y livianos que las máquinas virtuales tradicionales.

#### 2. ¿Qué es una imagen? ¿Y un contenedor? ¿Cuál es la principal diferencia entre ambos?

- **Imagen**:
  - Archivo read-only con el código, dependencias y configuraciones necesarias para ejecutar una aplicación.
  - Estático, nunca cambia.
  - No persiste datos ni estado al ejecutarse.
  - Plantilla para crear contenedores.
  - Se guarda en un repositorio (por ejemplo Docker Hub).
- **Contenedor**:
  - Instancia en ejecución de una imagen.
  - Dinámico → puede ejecutarse, apagarse, modificarse.
  - Puede mantener estado temporal mientras está activo.
  - Entorno real donde corre la aplicación.
  - Se ejecuta en un entorno de contenedores (por ejemplo Docker Engine).

La principal diferencia entre ambos es que **la imagen es la plantilla, mientras que el contenedor es una instancia de esa plantilla**. Pueden instanciarse N contenedores para una misma imagen.

#### 3. ¿Qué es Union Filesystem? ¿Cómo lo utiliza Docker?

Un Union Filesystem es un tipo de filesystem que permite superponer de forma transparente los archivos y directorios de varios filesystems (o "capas"), creando una vista única y coherente.

Sus características principales son:

- **Capas**: Los sistemas de archivos individuales se denominan capas.
- **Superposición transparente**: Los contenidos de las capas se fusionan de manera que el usuario ve un único sistema de archivos.
- **Prioridad**: Cuando varias capas contienen un archivo con el mismo nombre y ruta, una capa tiene prioridad sobre las demás (generalmente la capa superior).
- **Copy-on-write**: Si se realiza una escritura en un archivo que existe en una capa de solo lectura, el UnionFS no modifica la capa original. En su lugar, copia el archivo a una capa escribible (generalmente la capa superior) y aplica los cambios ahí. Esto permite que las capas inferiores permanezcan inalteradas.

Docker usa los Union Filesystems para construir y gestionar sus imágenes y contenedores de forma eficiente, de la siguiente manera:

- **Imágenes en capas**:
  - Cada instrucción en un Dockerfile crea una nueva capa read-only en la imagen.
  - Estas capas se apilan una encima de la otra. Por ejemplo, una imagen base de Ubuntu sería una capa, la instalación de un paquete sería otra capa, y el código de la aplicación sería una capa adicional.
  - Gracias a esto, las capas pueden ser compartidas y reutilizadas entre diferentes imágenes. Si tenemos varias imágenes que usan la misma base de Ubuntu, esa capa base se almacena una sola vez en disco.
- **Contenedores como capas escribibles**:
  - Cuando se ejecuta un contenedor a partir de una imagen, Docker toma todas las capas de la imagen y crea una nueva capa superior que es de lectura/escritura.
  - Esta capa superior es donde se almacenan todos los cambios que ocurren dentro del contenedor mientras está en ejecución (nuevos archivos, modificaciones, eliminaciones, etc).
  - El UnionFS presenta la vista combinada de todas las capas de la imagen más la capa de escritura del contenedor como un único filesystem para el proceso que se ejecuta dentro del contenedor.
- **Eficiencia con Copy-on-write**:
  - Si un archivo existente en una capa inferior (de solo lectura) es modificado o eliminado dentro del contenedor, el UnionFS utiliza el mecanismo CoW. El archivo original en la capa inferior permanece intacto. En cambio, una copia modificada del archivo se escribe en la capa superior de lectura/escritura del contenedor.
  - Esto ahorra espacio, ya que no se duplica todo el contenido de la imagen cada vez que se inicia un nuevo contenedor, si no que solo se almacenan los cambios.
  - Además, cuando se elimina un contenedor, su capa de escritura se descarta, dejando la imagen base completamente inalterada y lista para ser utilizada por otro contenedor. Esto garantiza que cada contenedor arranque con un estado "limpio" y predecible de la imagen.

#### 4. ¿Qué rango de direcciones IP utilizan los contenedores cuando se crean? ¿De dónde la obtiene?

- Por defecto, Docker usa la subred 172.17.0.0/16 para los contenedores.
- El daemon de Docker se encarga de crear subredes dinámicas y asignar IPs a cada contenedor.
- Cada red tiene una máscara de subred y gateway por defecto.
- Las IPs se asignan vía un server DHCP interno de Docker.
- El usuario puede crear sus propias redes personalizadas con rangos IP específicos usando `docker network create`.

#### 5. ¿De qué manera puede lograrse que los datos sean persistentes en Docker? ¿Qué dos maneras hay de hacerlo? ¿Cuáles son las diferencias entre ellas?

Para la persistencia de datos en Docker, hay dos métodos principales:

- **Volúmenes**:
  - Docker se encarga de su creación y ubicación en el host.
  - Persisten incluso si el contenedor se elimina.
  - Son la mejor opción para datos de aplicaciones (bases de datos, etc.) por su portabilidad y rendimiento.
  - Se usan con `docker run -v nombre_volumen:/ruta/en/contenedor`.
  - Se almacenan en `/var/lib/docker/volumes`.
- **Bind Mounts**:
  - Vinculan directamente un directorio o archivo del host al contenedor.
  - Si se elimina la ruta en el host, los datos se pierden.
  - Ideal para desarrollo (compartir código), archivos de configuración o logs.
  - Se usan con `docker run -v /ruta/en/host:/ruta/en/contenedor`.
  - Se pueden almacenar en cualquier lugar de la computadora host.

Los volúmenes son la opción preferida para persistir datos de aplicaciones, gestionados por Docker y más portátiles. Los bind mounts ofrecen un control directo sobre los archivos del host, ideales para el desarrollo y escenarios específicos.

### Taller

### El siguiente taller le guiará paso a paso para la construcción de una imagen Docker utilizando dos mecanismos distintos para los cuales deberá investigar y documentar qué comandos y argumentos utiliza para cada caso.

#### 1. Instale Docker CE (Community Edition) en su sistema operativo. Ayuda: seguir las instrucciones de la página de Docker. La instalación más simple para distribuciones de GNU/Linux basadas en Debian es usando los repositorios.

#### 2. Usando las herramientas (comandos) provistas por Docker realice las siguientes tareas:

##### a. Obtener una imagen de la última versión de Ubuntu disponible. ¿Cuál es el tamaño en disco de la imagen obtenida? ¿Ya puede ser considerada un contenedor? ¿Qué significa lo siguiente: `Using default tag: latest`?

- Para obtener una imagen de la última versión de Ubuntu, uso el comando `docker pull ubuntu`.
- Para chequear que se descargó correctamente y a su vez saber cuál es su tamaño en disco, uso el comando `docker images`:

```
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
ubuntu        latest    a0e45e2ce6e6   3 weeks ago    78.1MB
hello-world   latest    74cc54e27dc4   3 months ago   10.1kB
```

- La imagen pesa 78.1 MB.
- Esta imagen NO es un contenedor, ya que un contenedor es una imagen en ejecución, y esta imagen de ubuntu no se encuentra actualmente en ejecución.
- Cuando ejecuto `docker pull ubuntu`, docker me dice `Using default tag: latest`. Esto significa que por defecto, docker traerá la versión más nueva, reciente y estable de la imagen, a no ser que se especifique un tag en particular con `nombre_imagen:tag`.

##### b. De la imagen obtenida en el punto anterior iniciar un contenedor que simplemente ejecute el comando `ls -l`.

Para iniciar un contenedor que ejecute el comando `ls -l` a partir de la imagen de ubuntu que obtuve, ejecuto `docker run --name mi-contenedor ubuntu ls -l` y obtengo el output:

```bash
juan@juan-Lenovo-IdeaPad-S145-15AST:~$ docker run --name mi-contenedor ubuntu ls -l
total 48
lrwxrwxrwx   1 root root    7 Apr 22  2024 bin -> usr/bin
drwxr-xr-x   2 root root 4096 Apr 22  2024 boot
drwxr-xr-x   5 root root  340 May 21 02:06 dev
drwxr-xr-x   1 root root 4096 May 21 02:06 etc
drwxr-xr-x   3 root root 4096 Apr 15 14:11 home
lrwxrwxrwx   1 root root    7 Apr 22  2024 lib -> usr/lib
lrwxrwxrwx   1 root root    9 Apr 22  2024 lib64 -> usr/lib64
drwxr-xr-x   2 root root 4096 Apr 15 14:04 media
drwxr-xr-x   2 root root 4096 Apr 15 14:04 mnt
drwxr-xr-x   2 root root 4096 Apr 15 14:04 opt
dr-xr-xr-x 290 root root    0 May 21 02:06 proc
drwx------   2 root root 4096 Apr 15 14:11 root
drwxr-xr-x   4 root root 4096 Apr 15 14:11 run
lrwxrwxrwx   1 root root    8 Apr 22  2024 sbin -> usr/sbin
drwxr-xr-x   2 root root 4096 Apr 15 14:04 srv
dr-xr-xr-x  13 root root    0 May 21 02:02 sys
drwxrwxrwt   2 root root 4096 Apr 15 14:11 tmp
drwxr-xr-x  12 root root 4096 Apr 15 14:04 usr
drwxr-xr-x  11 root root 4096 Apr 15 14:11 var
```

Puedo ver al contenedor, aunque se haya detenido (ya que no se lo borró) con `docker ps -a`:

```bash
juan@juan-Lenovo-IdeaPad-S145-15AST:~$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED              STATUS                          PORTS     NAMES
65d2a89763c5   ubuntu    "ls -l"   About a minute ago   Exited (0) About a minute ago             mi-contenedor
```

Lo puedo borrar con: `docker rm mi-contenedor`.

##### c. ¿Qué sucede si ejecuta el comando `docker [container] run ubuntu /bin/bash`? ¿Puede utilizar la shell Bash del contenedor?

```bash
juan@juan-Lenovo-IdeaPad-S145-15AST:~$ docker run ubuntu /bin/bash
juan@juan-Lenovo-IdeaPad-S145-15AST:~$
```

Si ejecuto el comando descripto, no puedo usar la shell Bash del contenedor.

###### i. Modifique el comando utilizado para que el contenedor se inicie con una terminal interactiva y ejecutarlo. ¿Ahora puede utilizar la shell Bash del contenedor? ¿Por qué?

Para que el contenedor se inicie en una terminal interactiva, agrego la flag `-it` de esta forma `docker run -it ubuntu /bin/bash`.

Ahora puedo usar sin problemas la shell Bash del contenedor:

```bash
juan@juan-Lenovo-IdeaPad-S145-15AST:~$ docker run -it ubuntu /bin/bash
root@fcc6cd3872d7:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@fcc6cd3872d7:/#
```

Esto se debe a que la flag `-it` le dice a Docker que inicie el contenedor en modo interactivo, lo cual permite que la entrada y salida estándar del contenedor se redirijan a mi terminal, y por ende me permite interactuar.

###### ii. ¿Cuál es el PID del proceso bash en el contenedor? ¿Y fuera de éste?

- **En el contenedor posee PID = 1**:

```bash
root@fcc6cd3872d7:/# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 02:18 pts/0    00:00:00 /bin/bash
root          14       1  0 02:21 pts/0    00:00:00 ps -ef
```

- **Fuera del contenedor posee PID = 19978**:

```bash
juan@juan-Lenovo-IdeaPad-S145-15AST:~$ ps -ef | grep /bin/bash
juan       19938   17858  0 23:18 pts/0    00:00:00 docker run -it ubuntu /bin/bash
root       19978   19954  0 23:18 pts/0    00:00:00 /bin/bash
juan       20068   20044  0 23:22 pts/1    00:00:00 grep --color=auto /bin/bash
```

###### iii. Ejecutar el comando `lsns`. ¿Qué puede decir de los namespaces?

- **En el contenedor**:

```bash
root@fcc6cd3872d7:/# lsns
        NS TYPE   NPROCS PID USER COMMAND
4026531834 time        2   1 root /bin/bash
4026531837 user        2   1 root /bin/bash
4026533136 mnt         2   1 root /bin/bash
4026533137 uts         2   1 root /bin/bash
4026533138 ipc         2   1 root /bin/bash
4026533139 pid         2   1 root /bin/bash
4026533140 cgroup      2   1 root /bin/bash
4026533141 net         2   1 root /bin/bash
```

- **Fuera del contenedor**:

```bash
juan@juan-Lenovo-IdeaPad-S145-15AST:~$ lsns
        NS TYPE   NPROCS   PID USER COMMAND
4026531834 time       68  1183 juan /usr/bin/pipewire
4026531835 cgroup     68  1183 juan /usr/bin/pipewire
4026531836 pid        69  1183 juan /usr/bin/pipewire
4026531837 user       69  1183 juan /usr/bin/pipewire
4026531838 uts        68  1183 juan /usr/bin/pipewire
4026531839 ipc        68  1183 juan /usr/bin/pipewire
4026531840 net        68  1183 juan /usr/bin/pipewire
4026531841 mnt        68  1183 juan /usr/bin/pipewire
```

Solo comparten los namespaces `time` y `user`.

###### iv. Dentro del contenedor cree un archivo con nombre sistemas-operativos en el directorio raíz del filesystem y luego salga del contenedor (finalice la sesión de Bash utilizando las teclas Ctrl + D o el comando exit).

```bash
root@fcc6cd3872d7:/# touch sistemas-operativos
root@fcc6cd3872d7:/# exit
exit
```

###### v. Corrobore si el archivo creado existe en el directorio raíz del sistema operativo anfitrión (host). ¿Existe? ¿Por qué?

```bash
juan@juan-Lenovo-IdeaPad-S145-15AST:/$ ls
bin                boot   dev  home  lib32  lib.usr-is-merged  media  opt   root  sbin                srv       sys        tmp  var
bin.usr-is-merged  cdrom  etc  lib   lib64  lost+found         mnt    proc  run   sbin.usr-is-merged  swapfile  timeshift  usr
```

El archivo que creé desde el contenedor no existe en el SO host. Esto se debe a que el directorio raíz del contenedor es independiente al del SO host. Ese archivo solo existe dentro del filesystem del contenedor, no fuera.

##### d. Vuelva a iniciar el contenedor anterior utilizando el mismo comando (con una terminal interactiva). ¿Existe el archivo creado en el contenedor? ¿Por qué?

```bash
juan@juan-Lenovo-IdeaPad-S145-15AST:/$ docker run -it ubuntu /bin/bash
root@87fce3c4d700:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@87fce3c4d700:/#
```

Se puede ver que el archivo ya no está. Esto se debe a que esta nueva ejecución de la imagen es un contenedor nuevo, distinto al anterior, y por ende no tiene el mismo filesystem.

##### e. Obtenga el identificador del contenedor (container_id) donde se creó el archivo y utilícelo para iniciar con el comando `docker start -ia container_id` el contenedor en el cual se creó el archivo.

###### i. ¿Cómo obtuvo el container_id para para este comando?

```bash
juan@juan-Lenovo-IdeaPad-S145-15AST:~$ docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED              STATUS                      PORTS     NAMES
87fce3c4d700   ubuntu    "/bin/bash"   About a minute ago   Up About a minute                     mystifying_ritchie
fcc6cd3872d7   ubuntu    "/bin/bash"   17 minutes ago       Exited (0) 4 minutes ago              boring_hermann
65d2a89763c5   ubuntu    "ls -l"       26 minutes ago       Exited (0) 26 minutes ago             mi-contenedor
```

El ID del contenedor donde se creó el archivo es **fcc6cd3872d7**.

###### ii. Chequee nuevamente si el archivo creado anteriormente existe. ¿Cuál es el resultado en este caso? ¿Puede encontrar el archivo creado?

```bash
juan@juan-Lenovo-IdeaPad-S145-15AST:~$ docker start -ia fcc6cd3872d7
root@fcc6cd3872d7:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  sistemas-operativos  srv  sys  tmp  usr  var
root@fcc6cd3872d7:/#
```

Ahora, como iniciamos el contenedor desde el cual se creó el archivo, si lo podemos ver.

##### f. ¿Cuántos contenedores están actualmente en ejecución? ¿En qué estado se encuentra cada uno de los que se han ejecutado hasta el momento?

```bash
juan@juan-Lenovo-IdeaPad-S145-15AST:/$ docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS                      PORTS     NAMES
87fce3c4d700   ubuntu    "/bin/bash"   5 minutes ago    Exited (0) 16 seconds ago             mystifying_ritchie
fcc6cd3872d7   ubuntu    "/bin/bash"   21 minutes ago   Up About a minute                     boring_hermann
65d2a89763c5   ubuntu    "ls -l"       30 minutes ago   Exited (0) 30 minutes ago             mi-contenedor
```

Uno solo está en ejecución (el boring_hermann, estado Up), los otros dos están parados (exited) y no tuvieron errores (0).

##### g. Elimine todos los contenedores creados hasta el momento. Indique el o los comandos utilizados.

```bash
juan@juan-Lenovo-IdeaPad-S145-15AST:~$ docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS                          PORTS     NAMES
87fce3c4d700   ubuntu    "/bin/bash"   8 minutes ago    Exited (0) 3 minutes ago                  mystifying_ritchie
fcc6cd3872d7   ubuntu    "/bin/bash"   24 minutes ago   Exited (0) About a minute ago             boring_hermann
65d2a89763c5   ubuntu    "ls -l"       33 minutes ago   Exited (0) 33 minutes ago                 mi-contenedor
juan@juan-Lenovo-IdeaPad-S145-15AST:~$ docker container prune
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
Deleted Containers:
87fce3c4d700106b1277f9e7e1aab1b94cfc1c4fbcfb3140ad87bbca41975676
fcc6cd3872d784a8e49e0f9bcb5d03443d54149d1ac095f427024103654e07df
65d2a89763c5b46d84e1f1b0fa4ad46d8039ac901f09d39dc73a1b713cf570cc

Total reclaimed space: 105B
juan@juan-Lenovo-IdeaPad-S145-15AST:~$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

Se puede usar `docker container prune` para eliminar todos los containers con estado Exited.

#### 3. Creación de una imagen a partir de un contenedor. Siguiendo los pasos indicados a continuación genere una imagen de Docker a partir de un contenedor:

##### a. Inicie un contenedor a partir de la imagen de Ubuntu descargada anteriormente ejecutando una consola interactiva de Bash.

```bash
juan@juan-Lenovo-IdeaPad-S145-15AST:~$ docker run --name ejercicio-3 -it ubuntu /bin/bash
root@81365036ed93:/#
```

##### b. Instale el servidor web [Nginx](https://nginx.org/en/) en el contenedor utilizando los siguientes comandos:

```bash
export DEBIAN_FRONTEND=noninteractive
export TZ=America/Buenos_Aires
apt update -qq
apt install -y --no-install-recommends nginx

: '
Los dos primeros comandos exportan dos variables de ambiente para que
la instalación de una de las dependencias de nginx (el paquete tzdata)
no requiera que interactivamente se respondan preguntas sobre la
ubicación geográfica a utilizar
'
```

```bash
root@81365036ed93:/# export DEBIAN_FRONTEND=noninteractive
root@81365036ed93:/# export TZ=America/Buenos_Aires
root@81365036ed93:/# apt update -qq
All packages are up to date.
root@81365036ed93:/# apt install -y --no-install-recommends nginx
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  iproute2 libbpf1 libcap2-bin libelf1t64 libgssapi-krb5-2 libk5crypto3
  libkeyutils1 libkrb5-3 libkrb5support0 libmnl0 libtirpc-common libtirpc3t64
  libxtables12 nginx-common
Suggested packages:
  iproute2-doc python3:any krb5-doc krb5-user fcgiwrap nginx-doc ssl-cert
Recommended packages:
  libatm1 libpam-cap krb5-locales
The following NEW packages will be installed:
  iproute2 libbpf1 libcap2-bin libelf1t64 libgssapi-krb5-2 libk5crypto3
  libkeyutils1 libkrb5-3 libkrb5support0 libmnl0 libtirpc-common libtirpc3t64
  libxtables12 nginx nginx-common
0 upgraded, 15 newly installed, 0 to remove and 0 not upgraded.
Need to get 2684 kB of archives.
After this operation, 7795 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libelf1t64 amd64 0.190-1.1ubuntu0.1 [57.8 kB]
Get:2 http://archive.ubuntu.com/ubuntu noble/main amd64 libbpf1 amd64 1:1.3.0-2build2 [166 kB]
Get:3 http://archive.ubuntu.com/ubuntu noble/main amd64 libmnl0 amd64 1.0.5-2build1 [12.3 kB]
Get:4 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libkrb5support0 amd64 1.20.1-6ubuntu2.5 [34.1 kB]
Get:5 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libk5crypto3 amd64 1.20.1-6ubuntu2.5 [82.0 kB]
Get:6 http://archive.ubuntu.com/ubuntu noble/main amd64 libkeyutils1 amd64 1.6.3-3build1 [9490 B]
Get:7 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libkrb5-3 amd64 1.20.1-6ubuntu2.5 [347 kB]
Get:8 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libgssapi-krb5-2 amd64 1.20.1-6ubuntu2.5 [143 kB]
Get:9 http://archive.ubuntu.com/ubuntu noble/main amd64 libtirpc-common all 1.3.4+ds-1.1build1 [8094 B]
Get:10 http://archive.ubuntu.com/ubuntu noble/main amd64 libtirpc3t64 amd64 1.3.4+ds-1.1build1 [82.6 kB]
Get:11 http://archive.ubuntu.com/ubuntu noble/main amd64 libxtables12 amd64 1.8.10-3ubuntu2 [35.7 kB]
Get:12 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libcap2-bin amd64 1:2.66-5ubuntu2.2 [34.2 kB]
Get:13 http://archive.ubuntu.com/ubuntu noble/main amd64 iproute2 amd64 6.1.0-1ubuntu6 [1120 kB]
Get:14 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 nginx-common all 1.24.0-2ubuntu7.3 [31.2 kB]
Get:15 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 nginx amd64 1.24.0-2ubuntu7.3 [520 kB]
Fetched 2684 kB in 3s (1057 kB/s)
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package libelf1t64:amd64.
(Reading database ... 4381 files and directories currently installed.)
Preparing to unpack .../00-libelf1t64_0.190-1.1ubuntu0.1_amd64.deb ...
Unpacking libelf1t64:amd64 (0.190-1.1ubuntu0.1) ...
Selecting previously unselected package libbpf1:amd64.
Preparing to unpack .../01-libbpf1_1%3a1.3.0-2build2_amd64.deb ...
Unpacking libbpf1:amd64 (1:1.3.0-2build2) ...
Selecting previously unselected package libmnl0:amd64.
Preparing to unpack .../02-libmnl0_1.0.5-2build1_amd64.deb ...
Unpacking libmnl0:amd64 (1.0.5-2build1) ...
Selecting previously unselected package libkrb5support0:amd64.
Preparing to unpack .../03-libkrb5support0_1.20.1-6ubuntu2.5_amd64.deb ...
Unpacking libkrb5support0:amd64 (1.20.1-6ubuntu2.5) ...
Selecting previously unselected package libk5crypto3:amd64.
Preparing to unpack .../04-libk5crypto3_1.20.1-6ubuntu2.5_amd64.deb ...
Unpacking libk5crypto3:amd64 (1.20.1-6ubuntu2.5) ...
Selecting previously unselected package libkeyutils1:amd64.
Preparing to unpack .../05-libkeyutils1_1.6.3-3build1_amd64.deb ...
Unpacking libkeyutils1:amd64 (1.6.3-3build1) ...
Selecting previously unselected package libkrb5-3:amd64.
Preparing to unpack .../06-libkrb5-3_1.20.1-6ubuntu2.5_amd64.deb ...
Unpacking libkrb5-3:amd64 (1.20.1-6ubuntu2.5) ...
Selecting previously unselected package libgssapi-krb5-2:amd64.
Preparing to unpack .../07-libgssapi-krb5-2_1.20.1-6ubuntu2.5_amd64.deb ...
Unpacking libgssapi-krb5-2:amd64 (1.20.1-6ubuntu2.5) ...
Selecting previously unselected package libtirpc-common.
Preparing to unpack .../08-libtirpc-common_1.3.4+ds-1.1build1_all.deb ...
Unpacking libtirpc-common (1.3.4+ds-1.1build1) ...
Selecting previously unselected package libtirpc3t64:amd64.
Preparing to unpack .../09-libtirpc3t64_1.3.4+ds-1.1build1_amd64.deb ...
Adding 'diversion of /lib/x86_64-linux-gnu/libtirpc.so.3 to /lib/x86_64-linux-gn
u/libtirpc.so.3.usr-is-merged by libtirpc3t64'
Adding 'diversion of /lib/x86_64-linux-gnu/libtirpc.so.3.0.0 to /lib/x86_64-linu
x-gnu/libtirpc.so.3.0.0.usr-is-merged by libtirpc3t64'
Unpacking libtirpc3t64:amd64 (1.3.4+ds-1.1build1) ...
Selecting previously unselected package libxtables12:amd64.
Preparing to unpack .../10-libxtables12_1.8.10-3ubuntu2_amd64.deb ...
Unpacking libxtables12:amd64 (1.8.10-3ubuntu2) ...
Selecting previously unselected package libcap2-bin.
Preparing to unpack .../11-libcap2-bin_1%3a2.66-5ubuntu2.2_amd64.deb ...
Unpacking libcap2-bin (1:2.66-5ubuntu2.2) ...
Selecting previously unselected package iproute2.
Preparing to unpack .../12-iproute2_6.1.0-1ubuntu6_amd64.deb ...
Unpacking iproute2 (6.1.0-1ubuntu6) ...
Selecting previously unselected package nginx-common.
Preparing to unpack .../13-nginx-common_1.24.0-2ubuntu7.3_all.deb ...
Unpacking nginx-common (1.24.0-2ubuntu7.3) ...
Selecting previously unselected package nginx.
Preparing to unpack .../14-nginx_1.24.0-2ubuntu7.3_amd64.deb ...
Unpacking nginx (1.24.0-2ubuntu7.3) ...
Setting up libkeyutils1:amd64 (1.6.3-3build1) ...
Setting up libtirpc-common (1.3.4+ds-1.1build1) ...
Setting up libelf1t64:amd64 (0.190-1.1ubuntu0.1) ...
Setting up libkrb5support0:amd64 (1.20.1-6ubuntu2.5) ...
Setting up libcap2-bin (1:2.66-5ubuntu2.2) ...
Setting up libmnl0:amd64 (1.0.5-2build1) ...
Setting up libk5crypto3:amd64 (1.20.1-6ubuntu2.5) ...
Setting up libxtables12:amd64 (1.8.10-3ubuntu2) ...
Setting up libkrb5-3:amd64 (1.20.1-6ubuntu2.5) ...
Setting up libbpf1:amd64 (1:1.3.0-2build2) ...
Setting up libgssapi-krb5-2:amd64 (1.20.1-6ubuntu2.5) ...
Setting up libtirpc3t64:amd64 (1.3.4+ds-1.1build1) ...
Setting up iproute2 (6.1.0-1ubuntu6) ...
Setting up nginx (1.24.0-2ubuntu7.3) ...
invoke-rc.d: unknown initscript, /etc/init.d/nginx not found.
invoke-rc.d: could not determine current runlevel
Setting up nginx-common (1.24.0-2ubuntu7.3) ...
Processing triggers for libc-bin (2.39-0ubuntu8.4) ...
root@81365036ed93:/#
```

##### c. Salga del contenedor y genere una imagen Docker a partir de éste. ¿Con qué nombre se genera si no se especifica uno?

```bash
root@81365036ed93:/# exit
juan@juan-Lenovo-IdeaPad-S145-15AST:~$
juan@juan-Lenovo-IdeaPad-S145-15AST:~$ docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS                     PORTS     NAMES
81365036ed93   ubuntu    "/bin/bash"   4 minutes ago   Exited (0) 6 seconds ago             ejercicio-3
juan@juan-Lenovo-IdeaPad-S145-15AST:~$ docker commit 81365036ed93
sha256:69d691cba053e0a3395c16c0a0393d88cb3bb3bd89bd922e2ac617068378510a
juan@juan-Lenovo-IdeaPad-S145-15AST:~$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
<none>        <none>    69d691cba053   4 seconds ago   135MB
ubuntu        latest    a0e45e2ce6e6   3 weeks ago     78.1MB
hello-world   latest    74cc54e27dc4   3 months ago    10.1kB
```

Con `docker commit <container-id>` puedo generar una nueva imagen. Por defecto, se genera con nombre \<none>.

##### d. Cambie el nombre de la imagen creada de manera que en la columna Repository aparezca nginx-so y en la columna Tag aparezca v1.

```bash
juan@juan-Lenovo-IdeaPad-S145-15AST:~$ docker tag 69d691cba053 nginx-so:v1
juan@juan-Lenovo-IdeaPad-S145-15AST:~$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
nginx-so      v1        69d691cba053   2 minutes ago   135MB
ubuntu        latest    a0e45e2ce6e6   3 weeks ago     78.1MB
hello-world   latest    74cc54e27dc4   3 months ago    10.1kB
```

Con `docker tag <image-id> <image-name:tag>` podemos cambiar el nombre y tag de una imagen.

##### e. Ejecute un contenedor a partir de la imagen nginx-so:v1 que corra el servidor web nginx atendiendo conexiones en el puerto 8080 del host, y sirviendo una página web para corroborar su correcto funcionamiento. Para esto:

###### I. En el Sistema Operativo anfitrión (host) sobre el cual se ejecuta Docker crear un directorio que se utilizará para este taller. Éste puede ser el directorio nginx-so dentro de su directorio personal o cualquier otro directorio - para los fines de este enunciado haremos referencia a éste como `/home/so/nginx-so`, por lo que en los lugares donde se mencione esta ruta usted deberá reemplazarla por la ruta absoluta al directorio que haya decidido crear en este paso.

Voy a crear el directorio en la ruta `/home/juan/nginx-so`.

###### II. Dentro de ese directorio, cree un archivo llamado `index.html` que contenga el código HTML de [este gist de GitHub](https://gist.github.com/ncuesta/5b959fce1c7d2ed4e5a06e84e5a7efc8).

1. `cd nginx-so/`.
2. `touch index.html`.
3. Copiar el código HTML del link.
4. `xed index.html`.
5. Pegar el código HTML.

###### III. Cree un contenedor a partir de la imagen nginx-so:v1 montando el directorio del host (`/home/so/nginx-so`) sobre el directorio `/var/www/html` del contenedor, mapeando el puerto 80 del contenedor al puerto 8080 del host, y ejecutando el servidor nginx en primer plano (para iniciar el servidor nginx en primer plano usar el comando `nginx -g 'daemon off;'`). Indique el comando utilizado.

- Para montar un directorio del host a un directorio del contenedor, se usa el argumento `docker run -v dir-host:dir-contenedor`.
- Para mapear un puerto del contenedor a un puerto del host, se usa el argumento `docker run -p puerto-host:puerto-contenedor`
- Por lo tanto el comando que se necesita ejecutar es:

```bash
juan@juan-Lenovo-IdeaPad-S145-15AST:~/nginx-so$ docker run -p 8080:80 -v /home/juan/nginx-so:/var/www/html nginx-so:v1 nginx -g 'daemon off;'

```

##### f. Verifique que el contenedor esté ejecutándose correctamente abriendo un navegador web y visitando la URL [localhost](http://localhost:8080).

![Contenedor ejecutándose correctamente](https://i.imgur.com/aRyhcDS.png)

##### g. Modifique el archivo `index.html` agregándole un párrafo con su nombre y número de alumno. ¿Es necesario reiniciar el contenedor para ver los cambios?

Luego de editar el archivo HTML, al hacer F5 en http://localhost:8080, puedo ver los cambios. No hizo falta reiniciar el contenedor.

Esto se debe a que usamos un bind mount:

- Cuando uso el argumento `-v /home/juan/nginx-so:/var/www/html`, Docker comparte directamente el directorio de mi host con el contenedor.
- No es una copia, sino un acceso directo: cualquier cambio en los archivos del host se refleja inmediatamente en el contenedor y viceversa.
- Nginx sirve los archivos "en vivo" desde `/var/www/html`, que en realidad es el mismo directorio que `/home/juan/nginx-so` en mi PC.

##### h. Analice: ¿por qué es necesario que el proceso nginx se ejecute en primer plano? ¿Qué ocurre si lo ejecuta sin `-g 'daemon off;'`?

```bash
juan@juan-Lenovo-IdeaPad-S145-15AST:~/nginx-so$ docker run -p 8080:80 -v /home/juan/nginx-so:/var/www/html nginx-so:v1 nginx
juan@juan-Lenovo-IdeaPad-S145-15AST:~/nginx-so$ docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS                          PORTS     NAMES
4744a891b21a   nginx-so:v1   "nginx"                  5 seconds ago    Exited (0) 4 seconds ago                  naughty_gould
e9dc9aad4780   nginx-so:v1   "nginx -g 'daemon of…"   11 minutes ago   Exited (0) About a minute ago             mystifying_merkle
81365036ed93   ubuntu        "/bin/bash"              32 minutes ago   Exited (0) 27 minutes ago                 ejercicio-3
```

Al correr el contenedor sin el argumento `-g 'daemon off;'`, el contenedor se inicia pero se cierra inmediatamente después.

El contenedor se cierra porque, sin el argumento mencionado, Nginx se ejecuta en segundo plano (como demonio) y el proceso principal termina inmediatamente. Docker solo mantiene el contenedor activo si su proceso principal sigue corriendo.

#### 4. Creación de una imagen Docker a partir de un archivo Dockerfile. Siguiendo los pasos indicados a continuación, genere una nueva imagen a partir de los pasos descritos en un Dockerfile.

##### a. En el directorio del host creado en el punto anterior (`/home/so/nginx-so`), cree un archivo Dockerfile que realice los siguientes pasos:

###### i. Comenzar en base a la imagen oficial de Ubuntu.

###### ii. Exponer el puerto 80 del contenedor.

###### iii. Instalar el servidor web nginx.

###### iv. Copiar el archivo `index.html` del mismo directorio del host al directorio `/var/www/html` de la imagen.

###### v. Indicar el comando que se utilizará cuando se inicie un contenedor a partir de esta imagen para ejecutar el servidor nginx en primer plano: `nginx -g 'daemon off;'`. Use la forma exec ([La documentación oficial de Docker describe las tres formas posibles para indicar el comando principal de una imagen](https://docs.docker.com/engine/reference/builder/#cmd)) para definir el comando, de manera que todas las señales que reciba el contenedor sean enviadas directamente al proceso de nginx. Ayuda: las instrucciones necesarias para definir los pasos en el Dockerfile son FROM, EXPOSE, RUN, COPY y CMD.

```dockerfile
# Comenzar en base a la imagen oficial de Ubuntu.
FROM ubuntu

# Exponer el puerto 80 del contenedor.
EXPOSE 80

# Instalar el servidor web nginx.
RUN apt-get update && apt install -y --no-install-recommends nginx

# Copiar el archivo `index.html` del mismo directorio del host al directorio `/var/www/html` de la imagen.
COPY index.html /var/www/html/

# Comando que se utilizará cuando se inicie un contenedor a partir de esta imagen para ejecutar el servidor nginx en primer plano.
CMD ["nginx", "-g", "daemon off;"]
```

##### b. Utilizando el Dockerfile que generó en el punto anterior construya una nueva imagen Docker guardándola localmente con el nombre nginx-so:v2.

```bash
juan@juan-Lenovo-IdeaPad-S145-15AST:~/nginx-so$ ls
Dockerfile  index.html
juan@juan-Lenovo-IdeaPad-S145-15AST:~/nginx-so$ docker build -t nginx-so:v2 /home/juan/nginx-so/
[+] Building 19.1s (8/8) FINISHED                                                                                       docker:default
 => [internal] load build definition from Dockerfile                                                                              0.0s
 => => transferring dockerfile: 560B                                                                                              0.0s
 => [internal] load metadata for docker.io/library/ubuntu:latest                                                                  0.0s
 => [internal] load .dockerignore                                                                                                 0.0s
 => => transferring context: 2B                                                                                                   0.0s
 => [1/3] FROM docker.io/library/ubuntu:latest                                                                                    0.0s
 => [internal] load build context                                                                                                 0.0s
 => => transferring context: 959B                                                                                                 0.0s
 => [2/3] RUN apt-get update && apt install -y --no-install-recommends nginx                                                     18.6s
 => [3/3] COPY index.html /var/www/html/                                                                                          0.0s
 => exporting to image                                                                                                            0.4s
 => => exporting layers                                                                                                           0.3s
 => => writing image sha256:66d51f4c95f5267041abc2392aa8169d22696372ef0ccb12d5cf811d39cf673e                                      0.0s
 => => naming to docker.io/library/nginx-so:v2                                                                                    0.0s
juan@juan-Lenovo-IdeaPad-S145-15AST:~/nginx-so$
juan@juan-Lenovo-IdeaPad-S145-15AST:~/nginx-so$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
nginx-so      v2        66d51f4c95f5   3 minutes ago    135MB
nginx-so      v1        69d691cba053   44 minutes ago   135MB
ubuntu        latest    a0e45e2ce6e6   3 weeks ago      78.1MB
hello-world   latest    74cc54e27dc4   3 months ago     10.1kB
```

- Para crear una imagen a partir de un Dockerfile (que debe estar presente en el mismo directorio desde el cual se corre el comando), se usa `docker build`.
- Para especificar su nombre y tag, se usa el argumento `-t`.

##### c. Ejecute un contenedor a partir de la nueva imagen creada con las opciones adecuadas para que pueda acceder desde su navegador web a la página a través del puerto 8090 del host. Verifique que puede visualizar correctamente la página accediendo a [localhost](http://localhost:8090).

```bash
juan@juan-Lenovo-IdeaPad-S145-15AST:~/nginx-so$ docker run -p 8090:80 --name ejercicio4 nginx-so:v2

```

![Contenedor v2 corriendo exitosamente en el puerto 8090](https://i.imgur.com/0t3ZbYr.png)

##### d. Modifique el archivo `index.html` del host agregando un párrafo con la fecha actual y recargue la página en su navegador web. ¿Se ven reflejados los cambios que hizo en el archivo? ¿Por qué?

Al modificar el archivo HTML y hacer F5 en http://localhost:8090, no se ven reflejados los cambios. Esto se debe a que el contenedor está usando la versión de `index.html` de la imagen que se creó, y no del host.

##### e. Termine el contenedor iniciado antes y cree uno nuevo utilizando el mismo comando. Recargue la página en su navegador web. ¿Se ven ahora reflejados los cambios realizados en el archivo HTML? ¿Por qué?

```bash
juan@juan-Lenovo-IdeaPad-S145-15AST:~/nginx-so$ docker run -p 8090:80 --name ejercicio4-e nginx-so:v2

```

Nuevamente, no se ven reflejados los cambios. La razón es la misma, al iniciar un nuevo contenedor, se está usando otra vez el archivo HTML de la imagen original que se creó con la versión anterior de ese archivo, y no del host.

##### f. Vuelva a construir una imagen Docker a partir del Dockerfile creado anteriormente, pero esta vez dándole el nombre nginx-so:v3. Cree un contenedor a partir de ésta y acceda a la página en su navegador web. ¿Se ven reflejados los cambios realizados en el archivo HTML? ¿Por qué?

```bash
juan@juan-Lenovo-IdeaPad-S145-15AST:~/nginx-so$ ls
Dockerfile  index.html
juan@juan-Lenovo-IdeaPad-S145-15AST:~/nginx-so$ docker build -t nginx-so:v3 /home/juan/nginx-so/
[+] Building 0.1s (8/8) FINISHED                                                                                        docker:default
 => [internal] load build definition from Dockerfile                                                                              0.0s
 => => transferring dockerfile: 560B                                                                                              0.0s
 => [internal] load metadata for docker.io/library/ubuntu:latest                                                                  0.0s
 => [internal] load .dockerignore                                                                                                 0.0s
 => => transferring context: 2B                                                                                                   0.0s
 => [internal] load build context                                                                                                 0.0s
 => => transferring context: 995B                                                                                                 0.0s
 => [1/3] FROM docker.io/library/ubuntu:latest                                                                                    0.0s
 => CACHED [2/3] RUN apt-get update && apt install -y --no-install-recommends nginx                                               0.0s
 => [3/3] COPY index.html /var/www/html/                                                                                          0.0s
 => exporting to image                                                                                                            0.0s
 => => exporting layers                                                                                                           0.0s
 => => writing image sha256:4446b5893a77e49fbb415205e83d7c0c296e93bc3d4a084bfdccea0229c9fe1b                                      0.0s
 => => naming to docker.io/library/nginx-so:v3                                                                                    0.0s
juan@juan-Lenovo-IdeaPad-S145-15AST:~/nginx-so$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
nginx-so      v3        4446b5893a77   25 seconds ago   135MB
nginx-so      v2        66d51f4c95f5   14 minutes ago   135MB
nginx-so      v1        69d691cba053   55 minutes ago   135MB
ubuntu        latest    a0e45e2ce6e6   3 weeks ago      78.1MB
hello-world   latest    74cc54e27dc4   3 months ago     10.1kB
juan@juan-Lenovo-IdeaPad-S145-15AST:~/nginx-so$ docker run -p 8090:80 --name ejercicio4-f nginx-so:v3

```

![Contenedor v3 corriendo exitosamente en el puerto 8090](https://i.imgur.com/3sPGgjj.png)

Esta vez si se ven reflejados los cambios, ya que creamos una imagen nueva la cual fue a buscar el archivo `index.html` en nuestra PC, y como es la versión más actual, sí posee el párrafo con la fecha.
