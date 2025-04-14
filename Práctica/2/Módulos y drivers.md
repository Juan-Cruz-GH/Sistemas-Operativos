<h1 align="center">Práctica 2</h1>

## Requisitos

Para realizar esta práctica puede utilizar exactamente la misma versión del código fuente de
Linux utilizada en la práctica 1. Se puede usar la misma máquina virtual de la práctica 1 o una de su
elección si resulta más cómodo (por ejemplo una VM con interfaz gráfica y un IDE).

Si se usa la misma VM de la práctica 1 este directorio es `/home/so/kernel/linux-<version>/`.

https://gitlab.com/unlp-so/codigo-para-practicas/-/tree/main/practica

## Materiales de referencia

- [Agregar Syscalls](https://www.kernel.org/doc/html/latest/process/adding-syscalls.html)
- [Linux Kernel Hacking: A Crash Course - Speaker Deck](https://speakerdeck.com/linuxkernelhacking)

## [Módulos y Drivers](http://tldp.org/LDP/lkmpg/2.6/html/c38.html)

### Conceptos generales

#### 1. ¿Cómo se denomina en GNU/Linux a la porción de código que se agrega al kernel en tiempo de ejecución? ¿Es necesario reiniciar el sistema al cargarlo? Si no se pudiera utilizar esto. ¿Cómo deberíamos hacer para proveer la misma funcionalidad en GNU/Linux?

En GNU/Linux, la porción de código que se agrega al kernel en tiempo de ejecución se llama módulo, y es un archivo (usualmente con extensión .ko) que contiene código ejecutable que extiende las funcionalidades del kernel sin necesidad de reiniciar ni recompilar el sistema.

Si no se pudieran usar módulos, la única forma de extender la funcionalidad del kernel sería modificando el código fuente del mismo, lo cual nos fuerza a recompilar todo el kernel, y esto es costoso en tiempo.

#### 2. ¿Qué es un driver? ¿Para qué se utiliza?

Un driver en GNU/Linux es un componente de software que permite al kernel comunicarse con un dispositivo de hardware específico, como podría ser una tarjeta de red, un disco duro, una impresora, etc.

Se usa para que el sistema operativo pueda reconocer y controlar esa pieza de hardware de forma correcta, traduciendo las operaciones del sistema, como podría ser leer un archivo, en acciones que el dispositivo pueda entender.

#### 3. ¿Por qué es necesario escribir drivers?

Escribir drivers es necesario para que el SO pueda hacer uso efectivo de infinidad de dispositivos.

#### 4. ¿Cuál es la relación entre módulo y driver en GNU/Linux?

EN GNU/Linux, muchos drivers se implementan como módulos del kernel, es decir que **un módulo puede ser un driver**, o dicho de otra forma, un driver se puede escribir como un módulo del kernel. Esto permite que el driver se cargue dinámicamente, se descargue cuando ya no se necesita, y se desarrolle y pruebe con facilidad, sin necesidad de recompilar todo el kernel.

#### 5. ¿Qué implicancias puede tener un bug en un driver o módulo?

Como los drivers y módulos se ejecutan en modo kernel en GNU/Linux, un bug dentro de éstos puede fácilmente causar que el sistema colapse, ya sea afectando la estabilidad del SO, perdiendo datos, causando problemas de seguridad, o produciendo un kernel panic.

#### 6. ¿Qué tipos de drivers existen en GNU/Linux?

Existen 3 tipos principales de drivers en GNU/Linux:

- **Drivers de bloque**:
  - Son un grupo de bloques de datos persistentes.
  - Se lee y se escribe de a bloques, generalmente de tamaño 1024 bytes.
  - Ejemplos: Discos duros, SSDs, pendrives, CDs, DVDs, etc.
- **Drivers de carácter**:
  - Se accede byte a byte.
  - Ejemplos: Mouse, teclados, terminales, sensores, puertos serie, etc.
- **Drivers de red**:
  - Envío y recepción de paquetes de red.
  - Ejemplos: Tarjetas Ethernet, Wi-Fi, etc.

#### 7. ¿Qué hay en el directorio `/dev`? ¿Qué tipos de archivo encontramos en esa ubicación?

En el directorio `/dev` se encuentran todos los archivos especiales llamados archivos de dispositivo, que representan a los dispositivos del sistema. Esto se debe a que Linux trata a todos los dispositivos como si fueran archivos, en vez de interactuar de forma directa con el hardware.

Los tipos de archivo que encontramos en ese directorio son:

- **Dispositivos de bloque**:
  - Transfieren datos en bloques.
  - Ejemplos:
    - Discos duros: `/dev/sda`, `/dev/sdb`
    - Archivo montado como disco: `/dev/loop0`
    - Lector de CD/DVD: `/dev/sr0`
- **Dispositivos de carácter**:
  - Transfieren datos byte a byte, como un flujo continuo.
  - Ejemplos:
    - Terminal: `/dev/tty`
    - Consola del sistema: `/dev/console`
    - Dispositivos virtuales: `/dev/zero`, `/dev/null`
- **Enlaces simbólicos a dispositivos**:
  - Algunos archivos en /dev son accesos directos a dispositivos reales.
  - Ejemplos:
    - `/dev/cdrom` puede ser un symlink a `/dev/sr0`
- **Dispositivos virtuales o pseudo-dispositivos**:
  - No representan hardware real, pero cumplen funciones útiles:
  - Ejemplos:
    - `/dev/null`: borra todo lo que se escribe en él.
    - `/dev/random`: genera números aleatorios.
    - `/dev/full`: siempre "lleno".

#### 8. ¿Para qué sirve el archivo `/lib/modules/<version>/modules.dep` utilizado por el comando modprobe?

El archivo `/lib/modules/<version>/modules.dep` es un archivo de dependencias de módulos, contiene información sobre las dependencias entre los diferentes módulos del kernel cargados en el SO.

Cuando se carga un módulo del kernel, el SO necesita saber si ese módulo depende de otros módulos para funcionar correctamente. El archivo `modules.dep` especifica estas dependencias, lo que permite al SO cargar automáticamente todos los módulos necesarios cuando se carga un módulo específico.

#### 9. ¿En qué momento/s se genera o actualiza un initramfs?

El filesystem temporal `initramfs` se genera o actualiza durante la instalación del SO, después de modificar y recompilar el kernel, o cuando se realizan cambios significativos en la configuración del sistema.

#### 10. ¿Qué módulos y drivers deberá tener un initramfs mínimamente para cumplir su objetivo?

Debido a que este filesystem temporal tiene como objetivo preparar el entorno necesario para montar el sistema de archivos raíz (/), debe incluir como mínimo los siguientes componentes:

- Drivers de almacenamiento (dependen del tipo de disco que tengamos, SATA, NVMe, etc).
- Soporte para el sistema de archivos raíz (ext4, xfs, btrfs, etc).
- Soporte para la CPU y buses (firmware, UEFI, PCIe, etc).

### Práctica guiada - Desarrollando un módulo simple para Linux. El objetivo de este ejercicio es crear un módulo sencillo y poder cargarlo en nuestro kernel con el fin de consultar que el mismo se haya registrado correctamente.

#### 1. Crear el archivo memory.c con el siguiente código (puede estar en cualquier directorio, incluso fuera del directorio del kernel):

```c
#include <linux/module.h>
MODULE_LICENSE("Dual BSD/GPL");
```

#### 2. Crear el archivo Makefile con el siguiente contenido: `obj-m := memory.o`

##### a. Explique brevemente cual es la utilidad del archivo Makefile.

Este makefile se usa para compilar un módulo del kernel a partir de un archivo fuente.

Específicamente:

- La parte `obj-m` indica que memory.o debe compilarse como un módulo cargable del kernel y el resultado será **memory.ko**.
- La parte `memory.o` hace referencia al archivo objeto generado a partir de memory.c.

##### b. ¿Para qué sirve la macro MODULE_LICENSE? ¿Es obligatoria?

La macro MODULE_LICENSE especifica la licencia bajo la cual se distribuye el código.

No es estrictamente obligatoria (el módulo se compilará y cargará sin ella), pero:

- El kernel marcará el módulo como "no licenciado" ("unspecified").
- Se activará el taint (TAINT_NO_LICENSE), lo que puede afectar informes de errores (kernel oops).
- No se podran usar APIs reservadas para GPL.

#### 3. Ahora es necesario compilar nuestro módulo usando el mismo kernel en que correrá el mismo, utilizaremos el que instalamos en el primer paso del ejercicio guiado. `make -C <KERNEL_CODE> M=$(pwd) modules`

##### a. ¿Cuál es la salida del comando anterior?

El comando completo en mi caso es: `make -C $HOME/kernel/linux-6.13 M=$(pwd) modules`.

Al ejecutarlo, obtenemos la siguiente salida:

```
make: se entra en el directorio '/home/so/kernel/linux-6.13'
make[1]: se entra en el directorio '/home/so/practica2'
  CC [M]  memory.o
  MODPOST Module.symvers
  CC [M]  memory.mod.o
  CC [M]  .module-common.o
  LD [M]  memory.ko
make[1]: se sale del directorio '/home/so/practica2'
make: se sale del directorio '/home/so/kernel/linux-6.13'
```

##### b. ¿Qué tipos de archivo se generan? Explique para qué sirve cada uno.

A su vez, se crearon los siguientes archivos en el directorio donde se ejecutó el comando:

- **memory.ko**:
  - Módulo del kernel compilado: Kernel Object.
  - Es el archivo que se carga con `insmod` o `modprobe`.
  - Contiene el código binario del módulo listo para ejecutarse en el kernel.
- **memory.o**:
  - Objeto intermedio generado a partir de memory.c.
  - Contiene el código máquina del módulo antes de ser enlazado con dependencias del kernel. Se usa temporalmente durante la compilación.
- **memory.mod.c**:
  - Código C auxiliar generado automáticamente por el sistema de compilación del kernel.
  - Añade metadatos del módulo (como licencia, autor, dependencias) y las funciones init/exit si no fueron definidas en el código del módulo.
- **memory.mod.o**:
  - Objeto compilado de memory.mod.c.
  - Se enlaza con memory.o para generar memory.ko.
- **modules.order**:
  - Lista de módulos compilados en el orden correcto.
  - Ayuda al sistema de compilación a gestionar dependencias entre módulos.
- **Module.symvers**:
  - Archivo que almacena símbolos exportados por el módulo (funciones/variables visibles para otros módulos).
  - Permite que otros módulos usen funciones de este módulo (si se usa EXPORT_SYMBOL).
  - En nuestro caso, estará vacío.
- **memory.mod**:
  - Archivo temporal con información sobre el módulo (similar a memory.mod.c pero en otro formato).
  - Usado internamente por el sistema de compilación.

##### c. Con lo visto en la Práctica 1 sobre Makefiles, construya un Makefile de manera que si ejecuto:

- **make**, nuestro módulo se compila.
- **make clean**, limpia el módulo y el código objeto generado.
- **make run**, ejecuta el programa.

Makefile que hace lo pedido:

```make
# Makefile para módulo del kernel Linux
obj-m := memory.o
KDIR := /lib/modules/$(shell uname -r)/build
PWD := $(shell pwd)

# Compilar el módulo (genera memory.ko)
all:
	make -C $(KDIR) M=$(PWD) modules

# Limpiar todos los archivos generados (incluyendo .ko)
clean:
	make -C $(KDIR) M=$(PWD) clean

# Cargar el módulo en el kernel (requiere sudo)
run:
	sudo insmod memory.ko
	@echo "Módulo cargado (verificar con: dmesg | tail)"

# Descargar el módulo
stop:
	sudo rmmod memory
	@echo "Módulo descargado"

.PHONY: all clean run stop
```

#### 4. El paso que resta es agregar y eventualmente quitar nuestro módulo al kernel en tiempo de ejecución. Ejecutamos: `insmod memory.ko`. ¿Para qué sirven el comando insmod y el comando modprobe? ¿En qué se diferencian?

**NOTA**: El comando como está no me funcionó. Tuve que hacer su para loguearme como root, y luego hacer: `/sbin/insmod memory.ko`.

- El comando **insmod** sirve para cargar un módulo al kernel.
- El comando **modprobe** hace lo mismo.
- La diferencia es que **insmod** no resuelve dependencias automáticamente, mientras que **modprobe** sí.

#### 5. Ahora ejecutamos: `lsmod | grep memory`

##### a. ¿Cuál es la salida del comando? Explique cuál es la utilidad del comando lsmod.

La salida del comando es:

```
memory                  8192  0
```

La utilidad del comando **lsmod** es que nos muestra todos los módulos que están actualmente cargados en el kernel. En este caso, como estamos filtrando por módulos con nombre memory vía **grep**, solo nos muestra el que cargamos nosotros.

##### b. ¿Qué información encuentra en el archivo /proc/modules?

En el archivo `/proc/modules` encuentro la siguiente información:

```
memory 8192 0 - Live 0x0000000000000000 (OE)
intel_rapl_msr 16384 0 - Live 0xffffffffc0a75000
intel_rapl_common 40960 1 intel_rapl_msr, Live 0xffffffffc0a29000
intel_pmc_core 110592 0 - Live 0xffffffffc0a3c000
intel_vsec 16384 1 intel_pmc_core, Live 0xffffffffc0a44000
pmt_telemetry 12288 1 intel_pmc_core, Live 0xffffffffc0927000
pmt_class 12288 1 pmt_telemetry, Live 0xffffffffc08d5000
crct10dif_pclmul 12288 1 - Live 0xffffffffc0952000
crc32_pclmul 12288 0 - Live 0xffffffffc0877000
ghash_clmulni_intel 16384 0 - Live 0xffffffffc0954000
sha512_ssse3 45056 0 - Live 0xffffffffc0a2e000
snd_intel8x0 45056 0 - Live 0xffffffffc092a000
sha256_ssse3 28672 0 - Live 0xffffffffc085d000
snd_ac97_codec 184320 1 snd_intel8x0, Live 0xffffffffc09e9000
sha1_ssse3 32768 0 - Live 0xffffffffc0902000
vmwgfx 446464 0 - Live 0xffffffffc0960000
ac97_bus 12288 1 snd_ac97_codec, Live 0xffffffffc0956000
aesni_intel 118784 0 - Live 0xffffffffc0936000
snd_pcm 159744 2 snd_intel8x0,snd_ac97_codec, Live 0xffffffffc0908000
gf128mul 12288 1 aesni_intel, Live 0xffffffffc088b000
drm_client_lib 12288 1 vmwgfx, Live 0xffffffffc0885000
binfmt_misc 24576 1 - Live 0xffffffffc0865000
snd_timer 45056 1 snd_pcm, Live 0xffffffffc0843000
crypto_simd 12288 1 aesni_intel, Live 0xffffffffc083a000
drm_ttm_helper 12288 2 vmwgfx, Live 0xffffffffc07b4000
cryptd 24576 2 ghash_clmulni_intel,crypto_simd, Live 0xffffffffc08f9000
video 73728 0 - Live 0xffffffffc08dc000
snd 122880 4 snd_intel8x0,snd_ac97_codec,snd_pcm,snd_timer, Live 0xffffffffc08ae000
ttm 86016 2 vmwgfx,drm_ttm_helper, Live 0xffffffffc088d000
rapl 16384 0 - Live 0xffffffffc0798000
pcspkr 12288 0 - Live 0xffffffffc0879000
ac 16384 0 - Live 0xffffffffc086a000
soundcore 12288 1 snd, Live 0xffffffffc078e000
wmi 24576 1 video, Live 0xffffffffc0854000
drm_kms_helper 180224 3 vmwgfx,drm_client_lib,drm_ttm_helper, Live 0xffffffffc080b000
vboxguest 49152 0 - Live 0xffffffffc07b9000
button 20480 0 - Live 0xffffffffc07ab000
evdev 24576 3 - Live 0xffffffffc079d000
joydev 24576 0 - Live 0xffffffffc06e5000
serio_raw 16384 0 - Live 0xffffffffc06eb000
sg 45056 0 - Live 0xffffffffc06a6000
fuse 204800 1 - Live 0xffffffffc07cf000
efi_pstore 12288 0 - Live 0xffffffffc07c7000
drm 618496 6 vmwgfx,drm_client_lib,drm_ttm_helper,ttm,drm_kms_helper, Live 0xffffffffc06f1000
dm_mod 192512 0 - Live 0xffffffffc06ac000
configfs 53248 1 - Live 0xffffffffc068b000
ip_tables 36864 0 - Live 0xffffffffc0694000
x_tables 53248 1 ip_tables, Live 0xffffffffc067c000
autofs4 53248 2 - Live 0xffffffffc0500000
ext4 1040384 1 - Live 0xffffffffc0572000
crc16 12288 1 ext4, Live 0xffffffffc0454000
mbcache 12288 1 ext4, Live 0xffffffffc041f000
jbd2 167936 1 ext4, Live 0xffffffffc0553000
hid_generic 12288 0 - Live 0xffffffffc0446000
usbhid 69632 0 - Live 0xffffffffc04e2000
hid 225280 2 hid_generic,usbhid, Live 0xffffffffc04d3000
sr_mod 28672 0 - Live 0xffffffffc031b000
cdrom 69632 1 sr_mod, Live 0xffffffffc040d000
sd_mod 77824 2 - Live 0xffffffffc0320000
ata_generic 12288 0 - Live 0xffffffffc0315000
ahci 49152 2 - Live 0xffffffffc0520000
libahci 53248 1 ahci, Live 0xffffffffc050d000
ata_piix 40960 0 - Live 0xffffffffc02ad000
ohci_pci 16384 0 - Live 0xffffffffc04f8000
libata 380928 4 ata_generic,ahci,libahci,ata_piix, Live 0xffffffffc0472000
ohci_hcd 61440 1 ohci_pci, Live 0xffffffffc0458000
ehci_pci 16384 0 - Live 0xffffffffc0449000
ehci_hcd 102400 1 ehci_pci, Live 0xffffffffc0426000
usbcore 344064 5 usbhid,ohci_pci,ohci_hcd,ehci_pci,ehci_hcd, Live 0xffffffffc03af000
psmouse 200704 0 - Live 0xffffffffc0360000
crc32c_intel 12288 2 - Live 0xffffffffc03a5000
i2c_piix4 28672 0 - Live 0xffffffffc0350000
e1000 159744 0 - Live 0xffffffffc02b7000
scsi_mod 282624 4 sg,sr_mod,sd_mod,libata, Live 0xffffffffc02cc000
i2c_smbus 16384 1 i2c_piix4, Live 0xffffffffc029f000
usb_common 12288 3 ohci_hcd,ehci_hcd,usbcore, Live 0xffffffffc02a1000
scsi_common 12288 5 sg,sr_mod,sd_mod,libata,scsi_mod, Live 0xffffffffc0296000
```

Es decir, nos muestra la lista de todos los módulos actualmente cargados en el kernel, similar al comando **lsmod**.

##### c. Si ejecutamos more /proc/modules encontramos los siguientes fragmentos ¿Qué información obtenemos de aquí?:

```
memory 8192 0 - Live 0x0000000000000000 (OE)
binfmt_misc 24576 1 - Live 0x0000000000000000
intel_rapl_msr 16384 0 - Live 0x0000000000000000
intel_rapl_common 32768 1 intel_rapl_msr, Live 0x0000000000000000
```

- La primera columna contiene el nombre del módulo.
- La segunda columna se refiere al tamaño (en memoria) del módulo en bytes.
- La tercera columna enumera cuántas instancias del módulo están cargadas actualmente. Un valor de cero representa un módulo descargado.
- La cuarta columna indica si el módulo depende de la presencia de otro módulo para funcionar y enumera esos otros módulos.
- La quinta columna enumera en qué estado de carga se encuentra el módulo: Live, Loading o Unloading son los únicos valores posibles.
- La sexta columna enumera el desplazamiento de memoria del kernel actual para el módulo cargado. Esta información puede resultar útil para fines de depuración o para herramientas de creación de perfiles como oprofile.

##### d. ¿Con qué comando descargamos el módulo de la memoria?

Se puede descargar al módulo de la memoria usando el comando `rmmod memory`.

#### 6. Descargue el módulo memory. Para corroborar que efectivamente el mismo ha sido eliminado del kernel ejecute el siguiente comando: `lsmod | grep memory`

**NOTA**: El comando como está no me funcionó. Tuve que hacer su para loguearme como root, y luego hacer: `/sbin/rmmod memory`.

Efectivamente el mismo fue eliminado, debido a que el output de `lsmod | grep memory` devuelve vacío.

#### 7. Modifique el archivo memory.c de la siguiente manera:

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>

MODULE_LICENSE("Dual BSD/GPL");

static int hello_init(void) {
    printk("Hello world!\n");
    return 0;
}

static void hello_exit(void) {
    printk("Bye, cruel world\n");
}
module_init(hello_init);
module_exit(hello_exit);
```

##### a. Compile y cargue en memoria el módulo.

Simplemente ejecuto `make clean`, luego `make`, luego `make run`.

##### b. Invoque al comando dmesg

Al ejecutar **dmesg**:

```
...
[ 9609.424581] memory: loading out-of-tree module taints kernel.
[ 9609.424586] memory: module verification failed: signature and/or required key missing - tainting kernel
[10296.460887] Hello world!
```

Es decir, se ejecutó la parte `module_init()` del módulo.

##### c. Descargue el módulo de memoria y vuelva a invocar a dmesg

Ejecuto `make stop` y luego `dmesg`, y veo lo siguiente:

```
[ 9609.424581] memory: loading out-of-tree module taints kernel.
[ 9609.424586] memory: module verification failed: signature and/or required key missing - tainting kernel
[10296.460887] Hello world!
[10403.798208] Bye, cruel world
```

Es decir, se ejecutó la parte `module_exit()` del módulo.

#### 8. Responda lo siguiente:

##### a. ¿Para qué sirven las funciones module_init y module_exit?. ¿Cómo haría para ver la información del log que arrojan las mismas?

- La macro `module_init` especifica la función que se va a ejecutar cuando se **cargue** el módulo.
- La macro `module_exit` especifica la función que se va a ejecutar cuando se **descargue** el módulo.
- Para ver la info del log que arrojan las mismas se usa el comando dmesg, que permite ver el log interno del kernel.

##### b. Hasta aquí hemos desarrollado, compilado, cargado y descargado un módulo en nuestro kernel. En este punto y sin mirar lo que sigue. ¿Qué nos falta para tener un driver completo?

Para tener un driver completo nos faltaría especificar un dispositivo físico asociado a nuestro módulo, junto con su archivo descriptor.

##### c. Clasifique los tipos de dispositivos en Linux. Explique las características de cada uno.

- **Dispositivos de carácter**:

  - Estos dispositivos transfieren datos, pero un caracter a la vez.
  - Se ven muchos pseudodispositivos (/dev/null) como dispositivos de carácter y estos dispositivos no están realmente conectados físicamente a la máquina, pero permiten al sistema operativo una mayor funcionalidad.

- **Dispositivos de bloque**:

  - Estos dispositivos transfieren datos, pero en grandes bloques de tamaño fijo.
  - Lo más común es que se vean dispositivos que utilizan bloques de datos como dispositivos de bloque. Estos pueden ser discos duros, sistemas de archivos, etc.

- **Dispositivo de pipe**:

  - Los pipes permiten que dos o más procesos se comuniquen entre sí.
  - Son similares a los dispositivos de caracteres, pero en lugar de enviar la salida a un dispositivo, se envía a otro proceso.

- **Dispositivo de Socket**:

  - Los dispositivos socket facilitan la comunicación entre procesos, son similares a los dispositivos pipe pero pueden comunicarse con muchos procesos a la vez.

- **Caracterización de dispositivos**:
  - Los dispositivos se caracterizan usando dos números:
    - Número de dispositivo mayor.
    - Número de dispositivo menor.

### Práctica guiada - Desarrollando un Driver. Ahora completamos nuestro módulo para agregarle la capacidad de escribir y leer un dispositivo. En nuestro caso el dispositivo a leer será la memoria de nuestra CPU, pero podría ser cualquier otro dispositivo.

#### 1. Modifique el archivo memory.c para que tenga el siguiente [código](https://gitlab.com/unlp-so/codigo-para-practicas/-/blob/main/practica2/crear_driver/1_memory.c).

#### 2. Responda lo siguiente:

##### a. ¿Para qué sirve la estructura ssize_t y memory_fops? ¿Y las funciones register_chrdev y unregister_chrdev?

- **ssize_t**:
  - Es un tipo de dato definido en el kernel (signed size type).
  - Representa el tamaño de datos transferidos en operaciones de lectura/escritura (read, write).
  - Se usa en funciones como read y write para indicar bytes leídos/escritos o errores (valores negativos).
- **memory_fops**:
  - Es una estructura que define las operaciones soportadas por un driver, que son llamadas desde el espacio de usuario.
  - Asocia funciones del driver (ej: open, read, write) con las llamadas del sistema.
- **register_chrdev**:
  - Registra un driver de caracteres en el kernel (asocia un major number y file_operations).
- **unregister_chrdev**:
  - Libera el major number y desregistra el driver.

##### b. ¿Cómo sabe el kernel que funciones del driver invocar para leer y escribir al dispositivo?

- Cuando un programa en espacio de usuario llama a `open("/dev/mi_dispositivo", ...)`, el kernel busca el major number asociado al dispositivo en `/proc/devices`.
- Usa el major number para encontrar el **file_operations** registrado por el driver.
- Ejecuta la función correspondiente (ej: .open = mi_open).

##### c. ¿Cómo se accede desde el espacio de usuario a los dispositivos en Linux?

Los dispositivos en Linux se acceden como archivos especiales en `/dev`. Por ejemplo:

```
echo -n A > /dev/memory     # Escribe una letra
cat /dev/memory             # Lee lo que se escribió
```

```c
int fd = open("/dev/memory", O_RDWR);
write(fd, "B", 1);
char buf[1];
read(fd, buf, 1);
```

##### d. ¿Cómo se asocia el módulo que implementa nuestro driver con el dispositivo?

1. Cuando ejecutamos **insmod** o **modprobe**, el kernel llama a `memory_init()`.
2. Dentro de esa función, en nuestro ejemplo, hacemos `register_chrdev(memory_major, "memory", &memory_fops);`.
3. Esto asocia el major number (60, definido más arriba en el código) con la estructura de funciones memory_fops.
4. Luego creamos el archivo de dispositivo: `mknod /dev/memory c 60 0` para que se condiga con el 60 definido en memory.c
5. Cuando se abre /dev/memory, el kernel usa el número mayor (60) para buscar a este driver y llama a las funciones definidas.

##### e. [¿Qué hacen las funciones copy_to_user y copy_from_user?](https://developer.ibm.com/technologies/linux/articles/l-kernel-memory-access/)

- **copy_to_user**: Copia datos del kernel al espacio de usuario.
- **copy_from_user**: Copia datos de espacio de usuario hacia el espacio de kernel.

#### 3. Ahora ejecutamos lo siguiente: `mknod /dev/memory c 60 0`

#### 4. Y luego: `insmod memory.ko`

##### a. ¿Para qué sirve el comando mknod? ¿qué especifican cada uno de sus parámetros?.

El comando mknod crea una entrada de directorio y el correspondiente i-nodo para un archivo especial. El primer parámetro es el nombre del dispositivo de entrada.

Mknod tiene dos formas con diferentes flags:

- En la primer forma el segundo parámetro puede ser b o c según se trate de dispositivos de carácter o de bloque.
  - Los últimos dos parámetros son números que especifican el major y minor device number
  - El major device number ayuda al sistema a encontrar el código del driver del dispositivo.
  - El minor device number es el número de unidad o línea que puede ser decimal u octal.
- En la segunda forma se usa la flag p para crear pipelines FIFO.

##### b. ¿Qué son el “major” y el “minor” number? ¿Qué referencian cada uno?

- El major number identifica el **tipo** de dispositivo, como disco duro, impresora, etc.
- El minor number identifica un dispositivo especifico dentro de un tipo, como una partición de un disco duro o un puerto determinado de una tarjeta de red.

#### 5. Ahora escribimos a nuestro dispositivo: `echo -n abcdef > /dev/memory`

#### 6. Ahora leemos desde nuestro dispositivo: `more /dev/memory`

#### 7. Responda lo siguiente:

##### a. ¿Qué salida tiene el anterior comando?, ¿Porque? (ayuda: siga la ejecución de las funciones memory_read y memory_write y verifique con dmesg)

La salida de `more /dev/memory` es:

```
f
```

Esto se debe a que el dispositivo solo está almacenando el **último** carácter que se le escribe, y no todo.

##### b. ¿Cuántas invocaciones a memory_write se realizaron?

```
...
[12083.110774] <1>Inserting memory module
[12099.945030] memory_write()
[12099.945034] memory_write()
[12099.945034] memory_write()
[12099.945035] memory_write()
[12099.945035] memory_write()
[12099.945036] memory_write()
[12106.263011] memory_read()
[12106.263032] memory_read()
```

Se realizaron 6 invocaciones a `memory_write()`, una por cada carácter del string que le pasamos (**abcdef**, 6 caracteres).

##### c. ¿Cuál es el efecto del comando anterior? ¿Por qué?

Al ser un dispositivo de carácter, va escribiendo uno por uno.

##### d. Hasta aquí hemos desarrollado un ejemplo de un driver muy simple pero de manera completa, en nuestro caso hemos escrito y leído desde un dispositivo que en este caso es la propia memoria de nuestro equipo.

##### e. En el caso de un driver que lee un dispositivo como puede ser un file system, un dispositivo usb, etc. ¿Qué otros aspectos deberíamos considerar que aquí hemos omitido? ayuda: semáforos, ioctl, inb, outb.

Cuando un driver lee un dispositivo como un file system, un usb, etc, se deberían considerar varios aspectos adicionales:

- **Sincronización de acceso**:
  - Al leer un dispositivo, especialmente en un entorno multi-hilo o multi-proceso, es importante garantizar la sincronización adecuada del acceso al dispositivo para evitar interferencias.
  - Esto puede lograrse mediante el uso de semáforos, mutexes u otros mecanismos de sincronización proporcionados por el kernel de Linux.
- **Operaciones de control**:
  - Es posible que el driver necesite admitir operaciones de control adicionales más allá de simplemente leer y escribir.
  - Las llamadas ioctl (Input/Output Control) pueden ser utilizadas para implementar estas operaciones de control. Las ioctl permiten que los programas de usuario envíen comandos al driver para realizar diversas funciones, como configurar parámetros del dispositivo, obtener información adicional o realizar acciones específicas propias del dispositivo.
- **Acceso a registros de hardware**:
  - En el caso de dispositivos de hardware específicos puede ser necesario acceder directamente a los registros de hardware para configurar el dispositivo o leer datos. - Esto puede implicar el uso de funciones como inb() y outb() para leer y escribir bytes de los puertos de E/S del hardware.
