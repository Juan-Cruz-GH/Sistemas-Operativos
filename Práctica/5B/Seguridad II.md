<h1 align="center">Práctica 5</h1>

## Seguridad II

### Notas:

### Utilizar un kernel completo (no el compilado en las prácticas 1 y 2).

### En Debian 12 (Woodworm) utilizar el kernel por defecto 6.1.0 para evitar incompatibilidades con apparmor-utils.

### Compilar el código C usando el Makefile provisto a fin de deshabilitar algunas medidas de seguridad del compilador y generar un código assembler más simple.

### Acceda al código necesario para la práctica en el repositorio de la materia.

### Se recomienda trabajar en una VM ya que como parte de la práctica se van a habilitar y deshabilitar medidas de seguridad, lo que puede generar vulnerabilidades o hacer que determinadas aplicaciones no funcionen.

### D - AppArmor

#### 1. Instale las herramientas de espacio de usuario, perfiles por defecto de app-armor y auditd (necesario para generar perfiles de forma interactiva): `apt install apparmor apparmor-profiles apparmor-utils auditd`

#### 2. Verifique si apparmor se encuentra habilitado con el comando `aa-enabled`. Si no se encuentra habilitado verifique el kernel que está ejecutando (el kernel de Debian de la VM lo trae habilitado por defecto).

#### 3. Utilice la herramienta `aa-status` para determinar:

##### a. ¿Cuántos perfiles se encuentran cargados?

##### b. ¿Cuántos procesos y cuáles procesos de tu sistema tienen perfiles definidos?

#### 4. Detenga y deshabilite el servicio `insecure_service` creado en la parte 1 de la práctica de forma que no vuelva a iniciarse automáticamente:

```sh
systemctl stop insecure_service.service
systemctl disable insecure_service.service
```

#### 5. Ejecute `insecure_service` manualmente usando el usuario root y verifique que puede acceder libremente al filesystem en http://localhost:8080 (o la IP correspondiente donde se ejecuta el servicio): `/opt/sistemasoperativos/insecure_service`

#### 6. Generación de un nuevo profile:

##### a. Ejecutar `aa-genprof /...`

##### b. Abrir otra terminal, ejecutar `insecure_service` y navegue el sistema de archivos usando la interfaz web provista por el servicio.

##### c. Genere un perfil que permita:

###### i. Abrir conexiones tcp ipv4

###### ii. Abrir conexiones tcp ipv6

###### iii. Listar el contenido de / y /proc

###### iv. Ejecutar dash con los permisos del perfil actual (ix)

#### 7. Habilite el modo enforcing y verifique si funciona (`aa-enforcing`).

#### 8. Si necesita volver a generar un perfil puede usar `aa-complain` + `aa-logprofile` o editar el profile a mano y aplicar con `apparmor_parser -r`
