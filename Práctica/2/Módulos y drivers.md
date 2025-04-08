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

#### 1. ¿Cómo se denomina en GNU/Linux a la porción de código que se agrega al kernel en tiempo de ejecución? ¿Es necesario reiniciar el sistema al cargarlo? Si no se pudiera utilizar esto. ¿Cómo deberíamos hacer para proveer la misma funcionalidad en Gnu/Linux?

#### 2. ¿Qué es un driver? ¿Para qué se utiliza?

#### 3. ¿Por qué es necesario escribir drivers?

#### 4. ¿Cuál es la relación entre módulo y driver en GNU/Linux?

#### 5. ¿Qué implicancias puede tener un bug en un driver o módulo?

#### 6. ¿Qué tipos de drivers existen en GNU/Linux?

#### 7. ¿Qué hay en el directorio /dev? ¿Qué tipos de archivo encontramos en esa ubicación?

#### 8. ¿Para qué sirven el archivo /lib/modules/<version>/modules.dep utilizado por el comando modprobe?

#### 9. ¿En qué momento/s se genera o actualiza un initramfs?

#### 10. ¿Qué módulos y drivers deberá tener un initramfs mínimamente para cumplir su objetivo?

### Práctica guiada - Desarrollando un módulo simple para Linux. El objetivo de este ejercicio es crear un módulo sencillo y poder cargarlo en nuestro kernel con el fin de consultar que el mismo se haya registrado correctamente.

#### 1. Crear el archivo memory.c con el siguiente código (puede estar en cualquier directorio, incluso fuera del directorio del kernel):

```c
#include <linux/module.h>
MODULE_LICENSE("Dual BSD/GPL");
```

#### 2. Crear el archivo Makefile con el siguiente contenido: `obj-m := memory.o`

##### a. Explique brevemente cual es la utilidad del archivo Makefile.

##### b. ¿Para qué sirve la macro MODULE_LICENSE? ¿Es obligatoria?

#### 3. Ahora es necesario compilar nuestro módulo usando el mismo kernel en que correrá el mismo, utilizaremos el que instalamos en el primer paso del ejercicio guiado. `$ make -C <KERNEL_CODE> M=$(pwd) modules`

##### a. ¿Cuál es la salida del comando anterior?

##### b. ¿Qué tipos de archivo se generan? Explique para qué sirve cada uno.

##### c. Con lo visto en la Práctica 1 sobre Makefiles, construya un Makefile de manera que si ejecuto:

- **make**, nuestro módulo se compila.
- **make clean**, limpia el módulo y el código objeto generado.
- **make run**, ejecuta el programa.

#### 4. El paso que resta es agregar y eventualmente quitar nuestro módulo al kernel en tiempo de ejecución. Ejecutamos: `# insmod memory.ko`. ¿Para qué sirven el comando insmod y el comando modprobe? ¿En qué se diferencian?

#### 5. Ahora ejecutamos: `$ lsmod | grep memory`

##### a. ¿Cuál es la salida del comando? Explique cuál es la utilidad del comando lsmod.

##### b. ¿Qué información encuentra en el archivo /proc/modules?

##### c. Si ejecutamos more /proc/modules encontramos los siguientes fragmentos ¿Qué información obtenemos de aquí?:

```
memory 8192 0 - Live 0x0000000000000000 (OE)
binfmt_misc 24576 1 - Live 0x0000000000000000
intel_rapl_msr 16384 0 - Live 0x0000000000000000
intel_rapl_common 32768 1 intel_rapl_msr, Live 0x0000000000000000
```

##### d. ¿Con qué comando descargamos el módulo de la memoria?

#### 6. Descargue el módulo memory. Para corroborar que efectivamente el mismo ha sido eliminado del kernel ejecute el siguiente comando: `lsmod | grep memory`

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

##### b. Invoque al comando dmesg

##### c. Descargue el módulo de memoria y vuelva a invocar a dmesg

#### 8. Responda lo siguiente:

##### a. ¿Para qué sirven las funciones module_init y module_exit?. ¿Cómo haría para ver la información del log que arrojan las mismas?

##### b. Hasta aquí hemos desarrollado, compilado, cargado y descargado un módulo en nuestro kernel. En este punto y sin mirar lo que sigue. ¿Qué nos falta para tener un driver completo?

##### c. Clasifique los tipos de dispositivos en Linux. Explique las características de cada uno.

### Práctica guiada - Desarrollando un Driver. Ahora completamos nuestro módulo para agregarle la capacidad de escribir y leer un dispositivo. En nuestro caso el dispositivo a leer será la memoria de nuestra CPU, pero podría ser cualquier otro dispositivo.

#### 1. Modifique el archivo memory.c para que tenga el siguiente [código](https://gitlab.com/unlp-so/codigo-para-practicas/-/blob/main/practica2/crear_driver/1_memory.c)

#### 2. Responda lo siguiente:

##### a. ¿Para qué sirve la estructura ssize_t y memory_fops? ¿Y las funciones register_chrdev y unregister_chrdev?

##### b. ¿Cómo sabe el kernel que funciones del driver invocar para leer y escribir al dispositivo?

##### c. ¿Cómo se accede desde el espacio de usuario a los dispositivos en Linux?

##### d. ¿Cómo se asocia el módulo que implementa nuestro driver con el dispositivo?

##### e. [¿Qué hacen las funciones copy_to_user y copy_from_user?](https://developer.ibm.com/technologies/linux/articles/l-kernel-memory-access/)

#### 3. Ahora ejecutamos lo siguiente: `# mknod /dev/memory c 60 0`

#### 4. Y luego: `# insmod memory.ko`

##### a. ¿Para qué sirve el comando mknod? ¿qué especifican cada uno de sus parámetros?.

##### b. ¿Qué son el “major” y el “minor” number? ¿Qué referencian cada uno?

#### 5. Ahora escribimos a nuestro dispositivo: `echo -n abcdef > /dev/memory`

#### 6. Ahora leemos desde nuestro dispositivo: `more /dev/memory`

#### 7. Responda lo siguiente:

##### a. ¿Qué salida tiene el anterior comando?, ¿Porque? (ayuda: siga la ejecución de las funciones memory_read y memory_write y verifique con dmesg)

##### b. ¿Cuántas invocaciones a memory_write se realizaron?

##### c. ¿Cuál es el efecto del comando anterior? ¿Por qué?

##### d. Hasta aquí hemos desarrollado un ejemplo de un driver muy simple pero de manera completa, en nuestro caso hemos escrito y leído desde un dispositivo que en este caso es la propia memoria de nuestro equipo.

##### e. En el caso de un driver que lee un dispositivo como puede ser un file system, un dispositivo usb,etc. ¿Qué otros aspectos deberíamos considerar que aquí hemos omitido? ayuda: semáforos, ioctl, inb,outb.
