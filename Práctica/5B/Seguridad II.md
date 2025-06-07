<h1 align="center">Práctica 5</h1>

## Seguridad II

### Notas:

### Utilizar un kernel completo (no el compilado en las prácticas 1 y 2).

### En Debian 12 (Woodworm) utilizar el kernel por defecto 6.1.0 para evitar incompatibilidades con apparmor-utils.

### Compilar el código C usando el Makefile provisto a fin de deshabilitar algunas medidas de seguridad del compilador y generar un código assembler más simple.

### Acceda al código necesario para la práctica en el repositorio de la materia.

### Se recomienda trabajar en una VM ya que como parte de la práctica se van a habilitar y deshabilitar medidas de seguridad, lo que puede generar vulnerabilidades o hacer que determinadas aplicaciones no funcionen.

### D - AppArmor

### Ayudas:

### Es útil habilitar el modo complain y volver a ejecutar aa-genprof para detectar más acciones y que se agreguen al profile.

### Seguro es necesario ajustar el archivo manualmente ya que aa-genprof no siempre muestra las opciones que necesitamos.

### Verificar que no se agreguen “include” adicionales ya que traen otras reglas que van a cambiar el comportamiento.

### Para permitir acceso a un directorio: `/path/terminado/en/barra/ r`

### Para permitir acceso a los subdirectorios: `/path/terminado/en/barra/** r`

### Para denegar es lo mismo agregando deny al principio.

### Para permitir listar / pero denegar el resto: `/ r` y `deny /* r`

### owner se usa para acceder solo a los recursos de los cuales el proceso es owner. No lo usaremos en esta práctica.

### Siempre verificar que el perfil esté en enforce en las pruebas, si está en complain el proceso podrá acceder a todos los recursos y no estaremos probando el perfil realmente.

#### 1. Instale las herramientas de espacio de usuario, perfiles por defecto de app-armor y auditd (necesario para generar perfiles de forma interactiva): `apt install apparmor apparmor-profiles apparmor-utils auditd`

```sh
root@so:/home/so# apt install apparmor apparmor-profiles apparmor-utils auditd
Leyendo lista de paquetes... Hecho
Creando árbol de dependencias... Hecho
Leyendo la información de estado... Hecho
apparmor ya está en su versión más reciente (3.0.8-3).
fijado apparmor como instalado manualmente.
Se instalarán los siguientes paquetes adicionales:
  libauparse0 python3-apparmor python3-libapparmor
Paquetes sugeridos:
  vim-addon-manager audispd-plugins
Se instalarán los siguientes paquetes NUEVOS:
  apparmor-profiles apparmor-utils auditd libauparse0
  python3-apparmor python3-libapparmor
0 actualizados, 6 nuevos se instalarán, 0 para eliminar y 0 no actualizados.
Se necesita descargar 539 kB de archivos.
Se utilizarán 2.269 kB de espacio de disco adicional después de esta operación.
¿Desea continuar? [S/n] s
Des:1 http://deb.debian.org/debian bookworm/main amd64 libauparse0 amd64 1:3.0.9-1 [61,9 kB]
Des:2 http://deb.debian.org/debian bookworm/main amd64 auditd amd64 1:3.0.9-1 [218 kB]
Des:3 http://deb.debian.org/debian bookworm/main amd64 apparmor-profiles all 3.0.8-3 [41,7 kB]
Des:4 http://deb.debian.org/debian bookworm/main amd64 python3-libapparmor amd64 3.0.8-3 [36,4 kB]
Des:5 http://deb.debian.org/debian bookworm/main amd64 python3-apparmor all 3.0.8-3 [87,8 kB]
Des:6 http://deb.debian.org/debian bookworm/main amd64 apparmor-utils all 3.0.8-3 [94,0 kB]
Descargados 539 kB en 0s (2.348 kB/s)
Seleccionando el paquete libauparse0:amd64 previamente no seleccionado.
(Leyendo la base de datos ... 45798 ficheros o directorios instalados actualmente.)
Preparando para desempaquetar .../0-libauparse0_1%3a3.0.9-1_amd64.deb ...
Desempaquetando libauparse0:amd64 (1:3.0.9-1) ...
Seleccionando el paquete auditd previamente no seleccionado.
Preparando para desempaquetar .../1-auditd_1%3a3.0.9-1_amd64.deb ...
Desempaquetando auditd (1:3.0.9-1) ...
Seleccionando el paquete apparmor-profiles previamente no seleccionado.
Preparando para desempaquetar .../2-apparmor-profiles_3.0.8-3_all.deb ...
Desempaquetando apparmor-profiles (3.0.8-3) ...
Seleccionando el paquete python3-libapparmor previamente no seleccionado.
Preparando para desempaquetar .../3-python3-libapparmor_3.0.8-3_amd64.deb ...
Desempaquetando python3-libapparmor (3.0.8-3) ...
Seleccionando el paquete python3-apparmor previamente no seleccionado.
Preparando para desempaquetar .../4-python3-apparmor_3.0.8-3_all.deb ...
Desempaquetando python3-apparmor (3.0.8-3) ...
Seleccionando el paquete apparmor-utils previamente no seleccionado.
Preparando para desempaquetar .../5-apparmor-utils_3.0.8-3_all.deb ...
Desempaquetando apparmor-utils (3.0.8-3) ...
Configurando python3-libapparmor (3.0.8-3) ...
Configurando apparmor-profiles (3.0.8-3) ...
Configurando libauparse0:amd64 (1:3.0.9-1) ...
Configurando python3-apparmor (3.0.8-3) ...
Configurando auditd (1:3.0.9-1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/auditd.service → /lib/systemd/system/auditd.service.
Configurando apparmor-utils (3.0.8-3) ...
Procesando disparadores para man-db (2.11.2-2) ...
Procesando disparadores para libc-bin (2.36-9+deb12u9) ...
```

#### 2. Verifique si apparmor se encuentra habilitado con el comando `aa-enabled`. Si no se encuentra habilitado verifique el kernel que está ejecutando (el kernel de Debian de la VM lo trae habilitado por defecto).

```sh
root@so:/home/so# aa-enabled
S?
```

#### 3. Utilice la herramienta `aa-status` para determinar:

##### a. ¿Cuántos perfiles se encuentran cargados?

```sh
root@so:/home/so# /usr/sbin/aa-status
apparmor module is loaded.
31 profiles are loaded.
10 profiles are in enforce mode.
   /usr/bin/man
   /usr/lib/NetworkManager/nm-dhcp-client.action
   /usr/lib/NetworkManager/nm-dhcp-helper
   /usr/lib/connman/scripts/dhclient-script
   /{,usr/}sbin/dhclient
   lsb_release
   man_filter
   man_groff
   nvidia_modprobe
   nvidia_modprobe//kmod
21 profiles are in complain mode.
   avahi-daemon
   dnsmasq
   dnsmasq//libvirt_leaseshelper
   identd
   klogd
   mdnsd
   nmbd
   nscd
   php-fpm
   ping
   samba-bgqd
   samba-dcerpcd
   samba-rpcd
   samba-rpcd-classic
   samba-rpcd-spoolss
   smbd
   smbldap-useradd
   smbldap-useradd///etc/init.d/nscd
   syslog-ng
   syslogd
   traceroute
0 profiles are in kill mode.
0 profiles are in unconfined mode.
2 processes have profiles defined.
2 processes are in enforce mode.
   /usr/sbin/dhclient (449) /{,usr/}sbin/dhclient
   /usr/sbin/dhclient (450) /{,usr/}sbin/dhclient
0 processes are in complain mode.
0 processes are unconfined but have a profile defined.
0 processes are in mixed mode.
0 processes are in kill mode.
```

Hay **31** perfiles cargados.

##### b. ¿Cuántos procesos y cuáles procesos de tu sistema tienen perfiles definidos?

2 procesos tienen perfiles definidos:

- `/usr/sbin/dhclient (449) /{,usr/}sbin/dhclient`
- `/usr/sbin/dhclient (450) /{,usr/}sbin/dhclient`

#### 4. Detenga y deshabilite el servicio `insecure_service` creado en la parte 1 de la práctica de forma que no vuelva a iniciarse automáticamente:

```sh
systemctl stop insecure_service.service
systemctl disable insecure_service.service
```

```sh
root@so:/home/so# systemctl stop insecure_service.service
root@so:/home/so# systemctl disable insecure_service.service
Removed "/etc/systemd/system/multi-user.target.wants/insecure_service.service".
```

#### 5. Ejecute `insecure_service` manualmente usando el usuario root y verifique que puede acceder libremente al filesystem en http://localhost:8080 (o la IP correspondiente donde se ejecuta el servicio).

```sh
root@so:/home/so/codigo-para-practicas/practica5/insecure_service# ./insecure_service
2025/06/06 22:39:12 Servidor iniciado en http://localhost:8080
2025/06/06 22:39:18 Browsing path: /, ulr path: /resources/

```

Puedo acceder al sitio sin problemas.

#### 6. Generación de un nuevo profile:

##### a. Ejecutar `aa-genprof /home/so/codigo-para-practicas/practica5/insecure_service/insecure_service`

```sh
root@so:/home/so# /usr/sbin/aa-genprof /home/so/codigo-para-practicas/practica5/insecure_service/insecure_service
Updating AppArmor profiles in /etc/apparmor.d.
        no es un ejecutable dinámico
Writing updated profile for /home/so/codigo-para-practicas/practica5/insecure_service/insecure_service.
Estableciendo /home/so/codigo-para-practicas/practica5/insecure_service/insecure_service al modo reclamar.

Before you begin, you may wish to check if a
profile already exists for the application you
wish to confine. See the following wiki page for
more information:
https://gitlab.com/apparmor/apparmor/wikis/Profiles

Profiling: /home/so/codigo-para-practicas/practica5/insecure_service/insecure_service

Please start the application to be profiled in
another window and exercise its functionality now.

Once completed, select the "Scan" option below in
order to scan the system logs for AppArmor events.

For each AppArmor event, you will be given the
opportunity to choose whether the access should be
allowed or denied.

[(S)can system log for AppArmor events] / (F)inalizar

```

##### b. Abrir otra terminal, ejecutar `insecure_service` y navegue el sistema de archivos usando la interfaz web provista por el servicio.

```sh
root@so:/home/so/codigo-para-practicas/practica5/insecure_service# ./insecure_service
2025/06/06 22:39:12 Servidor iniciado en http://localhost:8080
2025/06/06 22:39:18 Browsing path: /, ulr path: /resources/
2025/06/06 22:40:47 Browsing path: /mnt, ulr path: /resources/mnt
2025/06/06 22:40:51 Browsing path: /, ulr path: /resources/
2025/06/06 22:40:52 Browsing path: /boot, ulr path: /resources/boot
2025/06/06 22:40:53 Browsing path: /var, ulr path: /resources/var
2025/06/06 22:40:55 Browsing path: /media, ulr path: /resources/media
2025/06/06 22:40:57 Browsing path: /lost+found, ulr path: /resources/lost+found
2025/06/06 22:40:58 Browsing path: /home, ulr path: /resources/home
2025/06/06 22:41:01 Browsing path: /proc, ulr path: /resources/proc
2025/06/06 22:41:19 Browsing path: /sbin, ulr path: /resources/sbin
2025/06/06 22:41:22 Browsing path: /usr, ulr path: /resources/usr
```

##### c. Genere un perfil que permita:

###### i. Abrir conexiones tcp ipv4

###### ii. Abrir conexiones tcp ipv6

###### iii. El perfil debe incluir los siguientes perfiles (y ningún otro): `include <abstractions/base>` y `include <abstractions/nameservice>`

###### iv. Listar el contenido de `/` y `/proc` pero no de otros subdirectorios de `/`

###### v. Ejecutar con los permisos del perfil actual (mrix) los siguientes comandos: `/usr/bin/dash`, `/usr/bin/ip`,`/usr/bin/mawk` y `/usr/bin/ps`

El perfil se encuentra en `/etc/apparmor.d/home.so.codigo-para-practicas.practica5.insecure_service.insecure_service`

```sh
root@so:/etc/apparmor.d# cat home.so.codigo-para-practicas.practica5.insecure_service.insecure_service
# Last Modified: Fri Jun  6 22:46:36 2025
abi <abi/3.0>,

include <tunables/global>

/home/so/codigo-para-practicas/practica5/insecure_service/insecure_service {
  include <abstractions/apache2-common>
  include <abstractions/base>
  include <abstractions/opencl-pocl>

  capability net_admin,

  deny owner /etc/ r,
  deny owner /etc/ld.so.cache r,
  deny owner /usr/ r,

  /home/so/codigo-para-practicas/practica5/insecure_service/insecure_service mr,
  /usr/bin/dash mrix,
  owner /proc/ r,
  owner /proc/sys/net/core/somaxconn r,
  owner /sys/kernel/mm/transparent_hugepage/hpage_pmd_size r,

}
```

Lo modifico para que cumpla todo lo solicitado:

```sh
root@so:/etc/apparmor.d# cat home.so.codigo-para-practicas.practica5.insecure_service.insecure_service
# Last Modified: Fri Jun  6 22:46:36 2025
abi <abi/3.0>,

include <tunables/global>

/home/so/codigo-para-practicas/practica5/insecure_service/insecure_service {
  include <abstractions/base>
  include <abstractions/nameservice>

  // Conexiones TCP IPv4 e IPv6
  network inet stream,
  network inet6 stream,

  // Listar solo contenido de / y /proc, pero no sus subdirectorios
  / r,
  /proc/ r,

  // Permitir ejecución de comandos con permisos completos del perfil
  /usr/bin/dash mrix,
  /usr/bin/ip mrix,
  /usr/bin/mawk mrix,
  /usr/bin/ps mrix,

  capability net_admin,

  deny owner /etc/ r,
  deny owner /etc/ld.so.cache r,
  deny owner /usr/ r,

  /home/so/codigo-para-practicas/practica5/insecure_service/insecure_service mr,
  /usr/bin/dash mrix,
  owner /proc/ r,
  owner /proc/sys/net/core/somaxconn r,
  owner /sys/kernel/mm/transparent_hugepage/hpage_pmd_size r,

}
root@so:/etc/apparmor.d# cat home.so.codigo-para-practicas.practica5.insecure_service.insecure_service
# Last Modified: Fri Jun  6 22:46:36 2025
abi <abi/3.0>,

include <tunables/global>

/home/so/codigo-para-practicas/practica5/insecure_service/insecure_service {
  include <abstractions/base>
  include <abstractions/nameservice>

  # Ejecutable
  /home/so/codigo-para-practicas/practica5/insecure_service/insecure_service mr,

  # Conexiones TCP IPv4 e IPv6
  network inet stream,
  network inet6 stream,

  # Listar solo contenido de / y /proc, pero no sus subdirectorios
  / r,
  /proc/ r,

  # Permitir ejecución de comandos con permisos completos del perfil
  /usr/bin/dash mrix,
  /usr/bin/ip mrix,
  /usr/bin/mawk mrix,
  /usr/bin/ps mrix,
}

root@so:/usr/sbin# /usr/sbin/apparmor_parser -r /etc/apparmor.d/home.so.codigo-para-practicas.practica5.insecure_service.insecure_service
```

#### 7. Habilite el modo enforcing y verifique si funciona (`aa-enforcing`).

```sh
root@so:/usr/sbin# /usr/sbin/aa-enforce /etc/apparmor.d/home.so.codigo-para-practicas.practica5.insecure_service.insecure_service
Setting /etc/apparmor.d/home.so.codigo-para-practicas.practica5.insecure_service.insecure_service to enforce mode.
```

#### 8. Si necesita volver a generar un perfil puede usar `aa-complain` + `aa-logprofile` o editar el profile a mano y aplicar con `apparmor_parser -r`
