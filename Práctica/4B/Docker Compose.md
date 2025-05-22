<h1 align="center">Práctica 4B</h1>

## Docker Compose

### Introducción

#### 1. Utilizando sus palabras describa, ¿qué es docker compose?

Docker Compose es una herramienta que facilita el despliegue de aplicaciones que están compuestas por múltiples contenedores. Este conjunto de contenedores se comunican entre sí para brindar la funcionalidad objetivo.

Consideramos entonces a docker compose como el conjunto de la herramienta (el binario) y los archivos de configuración llamados “archivo compose” o “compose file” que definen los recursos deseados.

Debido a esta relación, Docker Compose necesita que Docker esté instalado y configurado para poder funcionar correctamente.

#### 2. ¿Qué es el archivo compose y cual es su función? ¿Cuál es el "lenguaje" del archivo?

El archivo Docker Compose (`docker-compose.yml`) es un archivo YAML que define y orquesta aplicaciones multi-contenedor de Docker.

A cada contenedor definido en este archivo se lo denomina **servicio**.

Las funciones principales de este archivo son:

- **Declarar servicios**: Especificar qué contenedores (web, base de datos, API, etc) forman la aplicación.
- **Configurar dependencias**: Indicar el orden de inicio y comunicación entre ellos.
- **Gestionar recursos**: Mapear puertos, volúmenes, redes y variables de entorno para cada servicio.

#### 3. ¿Cuáles son las versiones existentes del archivo `docker-compose.yaml` y qué características aporta cada una? ¿Son compatibles entre sí? ¿Por qué?

El archivo Docker Compose ha atravesado varias versiones:

- **Versión 1 (Legacy, sin etiqueta version)**:

  - No requería especificar una versión en el archivo Compose.
  - Características básicas para definir servicios, volúmenes y redes.
  - Todos los servicios se declaran en la raíz del documento.
  - No permite declarar volúmenes nombrados, redes o argumentos de construcción.
  - Todos los contenedores se conectan a la misma red predeterminada, de tipo Bridge.
  - Se encuentra obsoleta.

- **Versión 2.x (version: '2.x')**:

  - Introducción de la sección version.
  - Mejoras en la configuración de redes y volúmenes.
  - Todos los servicios se declaran dentro de la clave **services**.
  - Permite declarar volúmenes nombrados, dentro de la clave **volumes**.
  - Permite declarar redes, dentro de la clave **networks**.
  - Por defecto, los contenedores se conectan a una misma red, y se usa como nombre de host el nombre del servicio.
  - Las versiones 2.1 a 2.4 agregan otras claves y características.
  - Soporte para dependencias de servicios mediante **depends_on**.
  - Añadido soporte para configuraciones más avanzadas como **healthcheck**, **deploy**, y **secrets**.
  - Permite el uso de comandos de construcción (build) más avanzados.
  - Se encuentra superada, pero aún funcional con herramientas modernas.

- **Versión 3.x (version: '3.x')**

  - Ultima versión y la recomendada por Docker.
  - Enfocada en el uso con **Docker Swarm**, permitiendo la configuración de despliegues en un entorno de clúster.
  - Introducción de la sección **deploy** para especificar políticas de despliegue (número de réplicas, restricciones de recursos, etc.).
  - Soporte mejorado para secretos y configuraciones (configs).
  - Opciones avanzadas para redes y volúmenes, como configuraciones de driver y **driver_opts**.
  - **healthcheck** avanzado para verificar el estado de los servicios.
  - Soporte para configs y secrets para gestionar configuraciones sensibles y secretas.
  - Se remueven las claves:
    - volume_driver
    - volumes_from
    - cpu_shares
    - cpu_quota
    - cpuset
    - mem_limit
    - memswap_limit
    - extends
    - group_add
  - Las versiones 3.1 a 3.8 agregan otras claves y características.

- **Compose Specification (Usada por docker compose V2)**:
  - La etiqueta version es opcional/informativa.
  - Mejor retrocompatibilidad.
  - Introduce profiles y mejor soporte GPU.
  - Es el estándar actual y recomendado.

**Compatibilidad**:

- Hacia adelante (herramientas nuevas con archivos viejos) suele ser compatible. docker compose V2 maneja bien los formatos v2.x y v3.x.
- Hacia atrás (herramientas viejas con archivos nuevos) suele no ser compatible. Una herramienta antigua no entenderá la sintaxis o características de un formato de archivo más nuevo.
- Entre v2.x y v3.x hay diferencias importantes (ej. cómo se definen los recursos), por lo que no son directamente intercambiables sin ajustes.
- Las incompatibilidades se deben principalmente a que las nuevas versiones añaden palabras clave y estructuras que las versiones antiguas no entienden. Además, algunas opciones cambian de nombre, se mueven, o directamente se eliminan.

#### 4. Investigue y describa la estructura de un archivo compose. Desarrolle al menos sobre los siguientes bloques indicando para qué se usan:

##### a. services

Define los servicios que componen a la aplicación. Cada servicio corresponde a un contenedor y posee su propia configuración (imagen, puertos, opciones de build, variables de entorno, etc).

##### b. build

Especifica cómo construir la imagen Docker para el servicio:

- La ruta al Dockerfile.
- El nombre alternativo del Dockerfile, si es necesario.
- Argumentos de construcción.

##### c. image

Nombre de la imagen Docker que se usará para el servicio. Puede ser una imagen local o de un registro.

##### d. volumes

Configura los montajes de volumen:

- **Bind mounts**: Mapean un directorio del host a uno del contenedor (./local:/ruta/contenedor).
- **Named volumes**: Volúmenes administrados por Docker (nombre-volumen:/ruta/contenedor).

##### e. restart

Política de reinicio del contenedor:

- "no".
- "always".
- "on-failure".
- "unless-stopped".

##### f. depends_on

Establece dependencias entre servicios. Esto garantiza que un servicio se inicie sólo después de que los servicios de los cuales depende ya se hayan iniciado.

##### g. environment

Variables de entorno que se le pasan al contenedor. Puede ser:

- Una lista de VAR=valor.
- Un diccionario (en formato YAML).

##### h. ports

Mapea puertos del host a puertos del contenedor en formato HOST:CONTAINER. También se puede definir el protocolo a usar (TCP o UDP) para cada puerto.

##### i. expose

Expone puertos sin publicarlos en el host. Solo son accesibles para otros servicios en la misma red.

##### j. networks

Permite definir redes personalizadas para los servicios. Estas permiten que los contenedores se comuniquen entre sí a través de redes aisladas.

Se pueden especificar configuraciones de red como alias, direcciones IP y conectividad externa.

Hay un network raíz y un network en cada servicio particular. En el network raíz se definen todas las redes que se van a crear.

##### k. healthcheck

Se utiliza para definir pruebas de salud personalizadas que verifican si un contenedor está funcionando de forma correcta.

#### 5. Conceptualmente: ¿Cómo se podrían usar los bloques "healthcheck" y "depends_on" para ejecutar una aplicación Web dónde el backend debería ejecutarse si y sólo si la base de datos ya está ejecutándose y lista?

En este caso, se tendrían dos servicios:

- Backend.
- Base de datos.

El backend tendría un `depends_on` con la base de datos y usaría la condición `service_healthy` que asegura que el backend solo se inicia si el `healthcheck` de la DB es exitoso.

El healthcheck en la DB se usaría para verificar que la misma esté lista para recibir conexiones antes de que el backend se empiece a ejecutar.

#### 6. Indique qué hacen y cuáles son las diferencias entre los siguientes comandos:

##### a. `docker compose create` y `docker compose up`

- El primero prepara a todos los contenedores del `docker-compose.yaml` sin iniciarlos. Crea las networks, volumes, y configuraciones pero deja a los contenedores en estado stopped.
- El segundo buildea, recrea e inicia a todos los contenedores del `docker-compose.yaml`. Además muestra en la terminal los logs de cada uno de ellos.

##### b. `docker compose stop` y `docker compose down`

- El primero detiene a todos los contenedores en ejecución del `docker-compose.yaml`, sin eliminarlos. Los contenedores y sus redes/volúmenes asociados permanecen intactos.
- El segundo detiene **y elimina** a todos los contenedores, redes y (opcionalmente) volúmenes e imágenes que fueron creados.

##### c. `docker compose run` y `docker compose exec`

- El primero crea e inicia un nuevo contenedor para ejecutar un comando específico y termina.
- El segundo ejecuta un comando dentro de un contenedor en ejecución. El contenedor sigue en ejecución después de que se completa el comando.

##### d. `docker compose ps`

Lista todos los contenedores gestionados por Docker Compose, mostrando su estado, nombre y puertos asociados.

##### e. `docker compose logs`

Muestra los logs de salida de los contenedores.

#### 7. ¿Qué tipo de volúmenes puede utilizar con docker compose? ¿Cómo se declara cada tipo en el archivo compose?

En Docker Compose se pueden usar varios tipos de volúmenes para persistir datos o compartirlos entre contenedores.

- **Named Volumes**:

  - Son volúmenes administrados por Docker, almacenados en `/var/lib/docker/volumes/`.
  - Ideales para persistir datos de forma eficiente y segura.
  - Docker gestiona su ciclo de vida.
  - Se le pueden especificar drivers (como local, nfs, etc.) o opciones como driver_opts.
  - Declaración:

  ```yaml
  services:
  app:
    image: nginx
    volumes:
      - mi_volumen:/ruta/en/el/contenedor

  volumes:
    mi_volumen: # Docker lo crea automáticamente si no existe
  ```

- **Anonymous Volumes**:

  - Volúmenes temporales creados automáticamente por Docker.
  - No tienen nombre explícito.
  - Se eliminan al hacer `docker compose down` (a menos que se use -v).
  - Se suelen usar para datos temporales o cuando no se necesita persistencia.
  - Declaración:

  ```yaml
  services:
    app:
      image: nginx
      volumes:
        - /ruta/en/el/contenedor # Sin especificar host
  ```

- **Bind Mounts**:

  - Vincula un directorio o archivo específico del host con el contenedor.
  - Ideal para desarrollo (código fuente, configs, hot-reload).
  - Permite sincronización en tiempo real.
  - El host debe tener la ruta accesible.
  - Declaración:

  ```yaml
  services:
    app:
      image: nginx
      volumes:
        - ./ruta/local:/ruta/en/el/contenedor # Ruta absoluta o relativa
        - /home/usuario/config.conf:/etc/nginx/config.conf
  ```

- **Volúmenes tmpfs**:

  - Almacena datos en la RAM del host.
  - No persisten después de reiniciar el contenedor.
  - Útil para datos sensibles que no deben escribirse en disco (ej: tokens temporales).
  - Declaración:

  ```yaml
  services:
    app:
      image: nginx
      tmpfs:
        - /ruta/en/el/contenedor
  ```

- **Volúmenes externos**:

  - Volúmenes preexistentes creados manualmente (por ejemplo con `docker volume create`). Docker Compose los reutiliza.
  - Declaración:

  ```yaml
  services:
    app:
      image: nginx
      volumes:
        - volumen_externo:/ruta/en/el/contenedor

  volumes:
    volumen_externo:
      external: true # Indica que ya existe
      name: nombre_del_volumen_externo # Opcional (si difiere del nombre en Compose)
  ```

#### 8. ¿Qué sucede si en lugar de usar el comando `docker compose down` utilizo `docker compose down -v/--volumes`?

- `docker compose down` detiene y elimina los contenedores, pero no elimina sus volúmenes (ni siquiera los anónimos).
- `docker compose down -v/--volumes` detiene y elimina los contenedores, y además elimina todos sus volúmenes named pero no los externos.

### Instalación

#### En la práctica anterior se instaló el entorno de ejecución Docker-CE. Es requisito para esta práctica tener dicho entorno instalado y funcionando en el dispositivo donde se pretenda realizar la misma.

#### En [este sitio](https://docs.docker.com/compose/install/) se puede encontrar la guía para instalar docker-compose en distintos SO.

#### Docker-compose es simplemente un binario, por lo que lo único que se necesita es descargar el binario, ubicarlo en algún lugar que el PATH de nuestro dispositivo pueda encontrarlo y que tenga los permisos necesarios para ser ejecutado.

#### En la actualidad existen 2 versiones del binario docker-compose. Vamos a utilizar la versión 2. Para instalar la versión 2.18.1, vamos a descargarla y ubicarla en el directorio `/usr/local/bin/docker-compose`, para que de esta manera quede accesible mediante el PATH de nuestra CLI:

```bash
~$ sudo curl -SL
https://github.com/docker/compose/releases/download/v2.18.1/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
```

#### Una vez descargado, le damos permiso de ejecución:

```bash
~$ sudo chmod +x /usr/local/bin/docker-compose
```

#### De esta manera ya tendremos docker-compose disponible. Para asegurarnos que esté instalado correctamente, verificamos la versión instalada corriendo desde la consola:

```bash
~$ docker compose --version
Docker Compose version v2.18.1
```

### Ejercicio guiado

#### Dado el siguiente código de archivo compose:

```yaml
version: "3.9"

services:
  db:
    image: mysql:5.7
    networks:
      - wordpress
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    networks:
      - wordpress
    volumes:
      - ${PWD}:/data
      - wordpress_data:/var/www/html
    ports:
      - "127.0.0.1:8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
volumes:
  db_data: {}
  wordpress_data: {}
networks:
  wordpress:
```

##### ¿Cuántos contenedores se instancian?

##### ¿Por qué no se necesitan Dockerfiles?

##### ¿Por qué el servicio identificado como "wordpress" tiene la siguiente línea? `depends_on: - db`

##### ¿Qué volúmenes y de qué tipo tendrá asociado cada contenedor?

##### ¿Por que uso el volumen nombrado `db_data:/var/lib/mysql` para el servicio db en lugar de dejar que se instancie un volumen anónimo con el contenedor?

##### ¿Qué genera la línea `${PWD}:/data` en la definición de wordpress?

##### ¿Qué representa la información que estoy definiendo en el bloque environment de cada servicio? ¿Cómo se "mapean" al instanciar los contenedores?

##### ¿Qué sucede si cambio los valores de alguna de las variables definidas en bloque "environment" en solo uno de los contenedores y hago que sean diferentes? (Por ej: cambio SOLO en la definición de wordpress la variable WORDPRESS_DB_NAME)

##### ¿Cómo sabe comunicarse el contenedor "wordpress" con el contenedor "db" si nunca doy información de direccionamiento?

##### ¿Qué puertos expone cada contenedor según su Dockerfile? (pista: navegue [este sitio](https://hub.docker.com/_/wordpress) y [este otro](https://hub.docker.com/_/mysql)) para acceder a los Dockerfiles que generaron esas imágenes y responder esta pregunta.

##### ¿Qué servicio se "publica" para ser accedido desde el exterior y en qué puerto? ¿Es necesario publicar el otro servicio? ¿Por qué?

#### Instanciando

##### Cree un directorio llamada `docker-compose-ej-1` donde prefiera, ubíquese dentro de éste y cree un archivo denominado `docker-compose.yml` pegando dentro el código anterior. La herramienta docker-compose, por defecto, espera encontrar en el directorio desde donde se la invoca un archivo `docker-compose.yml` (por eso lo creamos con ese nombre). Si existe, lee este archivo compose y realiza el despliegue de los recursos allí definidos.

##### Ahora, desde ese directorio ejecute el comando `docker compose up`, lo que resulta en el comienzo del despliegue de nuestros servicios. Como es la primera vez que lo corremos y si no tenemos las imágenes en la caché local de nuestro dispositivo, se descargan las imágenes de los dos servicios que estamos iniciando (recordar lo visto en la práctica anterior).

```bash
~$ docker compose up
[+] Running 34/34
✔ wordpress 21 layers [⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿] 0B/0B Pulled
    121.3s
    ✔ f03b40093957 Pull complete
    8.2s
    ✔ 662d8f2fcdb9 Pull complete
    9.4s
    ✔ 78fe0ef5ed77 Pull complete
    27.5s
    ………..
    108.8s
✔ db 11 layers [⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿] 0B/0B Pulled
    104.9s
    ✔ e83e8f2e82cc Pull complete
    31.6s
    ✔ 0f23deb01b84 Pull complete
    38.2s
…………
[+] Building 0.0s (0/0)
[+] Running 5/5
✔ Network so_wordpress Created
    2.4s
✔ Volume "so_wordpress_data" Created
    1.1s
✔ Volume "so_db_data" Created
    1.1s
✔ Container so-db-1 Created
    8.2s
✔ Container so-wordpress-1 Created
    3.0s

Attaching to so-db-1, so-wordpress-1
so-db-1 | 2023-06-05 20:10:12+00:00 [Note] [Entrypoint]: Entrypoint script for
MySQL Server 5.7.42-1.el7 started.
so-db-1 | 2023-06-05 20:10:12+00:00 [Note] [Entrypoint]: Switching to dedicated
user 'mysql'
so-db-1 | 2023-06-05 20:10:12+00:00 [Note] [Entrypoint]: Entrypoint script for
MySQL Server 5.7.42-1.el7 started
……..
```

##### En este punto, quedará la consola conectada a los servicios y estaremos viendo los logs exportados de los servicios. Si cerramos la consola o detenemos el proceso con ctrl + c, los servicios se darán de baja porque iniciamos los servicios en modo foreground. Para no quedar "pegados" a la consola podemos iniciar los servicios en modo "detached" de modo que queden corriendo en segundo plano (background), igual que como se hace con el comando `docker run -d IMAGE` de esta forma:

```bash
~$ docker compose up -d
```

##### De esta manera veremos sólo información de que los servicios se inician y su nombre, pero la consola quedará "libre".

##### Si quisiéramos conectarnos a alguno de los contenedores que docker-compose inició, por ejemplo el contenedor de wordpress, podemos hacerlo de la manera tradicional que se vio en la práctica de Docker (`docker exec [OPTIONS] CONTAINER COMMAND [ARG...]`) utilizando el identificador apropiado para el contenedor, o mediante el comando que docker-compose también brinda para hacerlo y usar su nombre de servicio ("wordpress" en este caso):

```bash
~$ docker compose exec wordpress /bin/bash
root@4dd0bcce2cb1:/var/www/html#
```

##### Aquí puedo enviar el comando `/bin/bash` porque el contenedor lo soporta; si eso no funcionase, la mayoría soportan al menos `/bin/sh`.

##### Y una vez dentro del contenedor, puedo navegar sus directorios normalmente. Si nos dirigimos al directorio `/data`, veremos dentro el contenido de nuestro directorio `docker-compose-ej-1` (solo tenemos el archivo `docker-compose.yml`) ya que montamos ese directorio como un volumen:

```bash
root@4dd0bcce2cb1:/var/www/html# cd /data/
root@4dd0bcce2cb1:/data# ls
docker-compose.yml
```

##### Y si creamos algún archivo dentro de este directorio, lo vemos también reflejado afuera del contenedor (el volumen montado como rw funciona en ambas direcciones)

##### Dentro del contenedor

```bash
root@4dd0bcce2cb1:/data# touch test
root@4dd0bcce2cb1:/data# ls
docker-compose.yml test
```

##### En el host:

```bash
host:/docker compose-ej-1$ ls
docker-compose.yml test
```

##### Ahora que tenemos todo instanciado y funcionando, vamos a listar los servicios que iniciamos. Para esto vamos a correr:

```bash
~$ docker compose ps
Name                    Command                 State           Ports
---------------------------------------------------------------------------------
docker-compose-         docker-entrypoint.sh    Up      3306/tcp, 33060/tcp
ej-1_db_1               mysqld
docker-compose-         docker-entrypoint.sh    Up      127.0.0.1:8000->80/tcp
ej-1_wordpress_1        apach ...
```

##### Como se puede observar, el servicio denominado `docker-compose-ej-1_wordpress_1` está exponiendo el puerto 80 del contenedor en la dirección 127.0.0.1 puerto 8000 de nuestro dispositivo "host". Esto quiere decir que tenemos en nuestro dispositivo un puerto 8000 "abierto" aceptando conexiones y si ingresamos desde un navegador a la direccion "127.0.0.1:8000" veremos la página de inicio de la aplicación Wordpress:

![Página de inicio de la aplicación Wordpress](https://i.imgur.com/ge2Dyhl.png)

##### De este modo, hemos realizado el despliegue de una aplicación wordpress y de su base de datos mediante el uso de contenedores y la herramienta docker-compose. Desde este punto, solo queda continuar con la instalación de wordpress desde el browser.

##### Si queremos detener los servicios podemos ejecutar el comando:

```bash
~$ docker-compose stop
Stopping docker-compose-ej-1_wordpress_1 ... done
Stopping docker-compose-ej-1_db_1 ... done
```

##### y para eliminarlos:

```bash
~$ docker compose down
Removing docker-compose-ej-1_wordpress_1 ... done
Removing docker-compose-ej-1_db_1 ... done
Removing network docker-compose-ej-1_wordpress
```

##### Pero atención, esto elimina los contenedores pero no SUS VOLÚMENES DE DATOS, por lo que si volvemos a levantar los servicios por más que hayamos eliminado los contenedores, veremos que todas las modificaciones que hayamos realizado en la instalación de wordpress y datos agregados a la base de datos aún están presentes. Si queremos eliminar todo rastro de un despliegue previo, tendremos que eliminar los contenedores y también los volúmenes asociados utilizando el flag `-v` en `docker-compose down`:

```bash
~$ docker-compose down -v
Stopping docker-compose-ej-1_wordpress_1 ... done
Stopping docker-compose-ej-1_db_1 ... done
Removing docker-compose-ej-1_wordpress_1 ... done
Removing docker-compose-ej-1_db_1 ... done
Removing network docker-compose-ej-1_wordpress
Removing volume docker-compose-ej-1_db_data
Removing volume docker-compose-ej-1_wordpress_data
```

##### De esta manera, hemos eliminado todo lo instanciado por el docker compose. Solo quedan las imágenes Docker descargadas en la caché local del dispositivo, las cuales deben eliminar por su cuenta.
