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

##### b. Instale el servidor web [Nginx](https://nginx.org/en/) en el contenedor utilizando los siguientes comandos:

```bash
export DEBIAN_FRONTEND=noninteractive
export TZ=America/Buenos_Aires
apt update -qq
apt install -y --no-install-recommends nginx

: '
Los dos primeros comandos exportan dos variables de ambiente para que
la instalación de una de las dependencias de nginx (el paquete tzdata
no requiera que interactivamente se respondan preguntas sobre la
ubicación geográfica a utilizar
'
```

##### c. Salga del contenedor y genere una imagen Docker a partir de éste. ¿Con qué nombre se genera si no se especifica uno?

##### d. Cambie el nombre de la imagen creada de manera que en la columna Repository aparezca nginx-so y en la columna Tag aparezca v1.

##### e. Ejecute un contenedor a partir de la imagen nginx-so:v1 que corra el servidor web nginx atendiendo conexiones en el puerto 8080 del host, y sirviendo una página web para corroborar su correcto funcionamiento. Para esto:

###### I. En el Sistema Operativo anfitrión (host) sobre el cual se ejecuta Docker crear un directorio que se utilizará para este taller. Éste puede ser el directorio nginx-so dentro de su directorio personal o cualquier otro directorio - para los fines de este enunciado haremos referencia a éste como `/home/so/nginx-so`, por lo que en los lugares donde se mencione esta ruta usted deberá reemplazarla por la ruta absoluta al directorio que haya decidido crear en este paso.

###### II. Dentro de ese directorio, cree un archivo llamado `index.html` que contenga el código HTML de [este gist de GitHub](https://gist.github.com/ncuesta/5b959fce1c7d2ed4e5a06e84e5a7efc8).

###### III. Cree un contenedor a partir de la imagen nginx-so:v1 montando el directorio del host (`/home/so/nginx-so`) sobre el directorio `/var/www/html` del contenedor, mapeando el puerto 80 del contenedor al puerto 8080 del host, y ejecutando el servidor nginx en primer plano (para iniciar el servidor nginx en primer plano usar el comando `nginx -g 'daemon off;'`). Indique el comando utilizado.

##### f. Verifique que el contenedor esté ejecutándose correctamente abriendo un navegador web y visitando la URL [localhost](http://localhost:8080).

##### g. Modifique el archivo `index.html` agregándole un párrafo con su nombre y número de alumno. ¿Es necesario reiniciar el contenedor para ver los cambios?

##### h. Analice: ¿por qué es necesario que el proceso nginx se ejecute en primer plano? ¿Qué ocurre si lo ejecuta sin `-g 'daemon off;'`?

#### 4. Creación de una imagen Docker a partir de un archivo Dockerfile. Siguiendo los pasos indicados a continuación, genere una nueva imagen a partir de los pasos descritos en un Dockerfile.

##### a. En el directorio del host creado en el punto anterior (`/home/so/nginx-so`), cree un archivo Dockerfile que realice los siguientes pasos:

###### i. Comenzar en base a la imagen oficial de Ubuntu.

###### ii. Exponer el puerto 80 del contenedor.

###### iii. Instalar el servidor web nginx.

###### iv. Copiar el archivo `index.html` del mismo directorio del host al directorio `/var/www/html` de la imagen.

###### v. Indicar el comando que se utilizará cuando se inicie un contenedor a partir de esta imagen para ejecutar el servidor nginx en primer plano: `nginx -g 'daemon off;'`. Use la forma exec ([La documentación oficial de Docker describe las tres formas posibles para indicar el comando principal de una imagen](https://docs.docker.com/engine/reference/builder/#cmd)) para definir el comando, de manera que todas las señales que reciba el contenedor sean enviadas directamente al proceso de nginx. Ayuda: las instrucciones necesarias para definir los pasos en el Dockerfile son FROM, EXPOSE, RUN, COPY y CMD.

##### b. Utilizando el Dockerfile que generó en el punto anterior construya una nueva imagen Docker guardándola localmente con el nombre nginx-so:v2.

##### c. Ejecute un contenedor a partir de la nueva imagen creada con las opciones adecuadas para que pueda acceder desde su navegador web ala página a través del puerto 8090 del host. Verifique que puede visualizar correctamente la página accediendo a [localhost](http://localhost:8090).

##### d. Modifique el archivo `index.html` del host agregando un párrafo con la fecha actual y recargue la página en su navegador web. ¿Se ven reflejados los cambios que hizo en el archivo? ¿Por qué?

##### e. Termine el contenedor iniciado antes y cree uno nuevo utilizando el mismo comando. Recargue la página en su navegador web. ¿Se ven ahora reflejados los cambios realizados en el archivo HTML? ¿Por qué?

##### f. Vuelva a construir una imagen Docker a partir del Dockerfile creado anteriormente, pero esta vez dándole el nombre nginx-so:v3. Cree un contenedor a partir de ésta y acceda a la página en su navegador web. ¿Se ven reflejados los cambios realizados en el archivo HTML? ¿Por qué?
