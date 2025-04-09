<h1 align="center">Práctica 2</h1>

## Requisitos

Para realizar esta práctica puede utilizar exactamente la misma versión del código fuente de Linux utilizada en la práctica 1. Se puede usar la misma máquina virtual de la práctica 1 o una de su elección si resulta más cómodo (por ejemplo una VM con interfaz gráfica y un IDE).

Si se usa la misma VM de la práctica 1 este directorio es `/home/so/kernel/linux-<version>/`.

https://gitlab.com/unlp-so/codigo-para-practicas/-/tree/main/practica

## Materiales de referencia

- [Agregar Syscalls](https://www.kernel.org/doc/html/latest/process/adding-syscalls.html)
- [Linux Kernel Hacking: A Crash Course - Speaker Deck](https://speakerdeck.com/linuxkernelhacking)

## System Calls

### Conceptos generales

#### 1. ¿Qué es una System Call? ¿Para qué se utiliza?

Una System Call es un pedido que realiza un programa al kernel del sistema operativo para realizar una tarea específica que el programa no puede hacer por sí mismo debido a que se trata de una operación protegida.

Las System Calls se pueden usar para trabajar con archivos, para manejar procesos, para comunicación en redes, para acceder a dispositivos de E/S, etc.

#### 2. ¿Para qué sirve la macro syscall? Describa el propósito de cada uno de sus parámetros. [Ayuda](http://www.gnu.org/software/libc/manual/html_mono/libc.html#System-Calls)

La macro **syscall** se usa en Linux para hacer llamadas directas al sistema operativo desde un programa en espacio de usuario. Es parte del encabezado `<unistd.h>` o `<sys/syscall.h>` en C y permite invocar funciones del kernel sin pasar por los envoltorios (wrappers) estándar de la biblioteca C (glibc), como open(), read(), write(), etc.

La ventaja de usar syscall() directamente es que nos permite hacer llamadas que tal vez no estén expuestas por la biblioteca estándar, además de proveer una forma más directa o controlada de invocar una syscall específica.

La firma de la función es:

```c
long syscall(long number, ...);
```

Posee un número variable de argumentos:

- **number**: es el número de la syscall que se quiere ejecutar. Cada llamada al sistema tiene un número único que la identifica. Estos números están definidos en archivos como `/usr/include/asm/unistd.h` o `/usr/include/x86_64-linux-gnu/asm/unistd_64.h` (write → 1; read → 0; exit → 60, etc). Las constantes de estos valores están en `<sys/syscall.h>`.
- **...**: argumentos variables, dependen de cada syscall. Por ejemplo, para write se necesita el descriptor del archivo, un buffer, y la cantidad de bytes a escribir.

#### 3. Ejecute el siguiente comando e identifique el propósito de cada uno de los archivos que encuentra `ls -lh /boot | grep vmlinuz`

- En la carpeta `/boot` se guardan los archivos del sistema necesarios para arrancar (bootear) Linux. Acá están los kernels, los inits, y algunos archivos de configuración de arranque.
- vmlinuz es el kernel **comprimido** de Linux que se carga en el arranque del sistema. Cuando el sistema arranca, el bootloader (como GRUB) carga uno de estos archivos vmlinuz-\* y empieza el proceso de iniciar el sistema operativo.
- Cuando ejecuto el comando en la máquina virtual de la cátedra, obtengo tres líneas:
  - **vmlinuz-6.1.0-29-amd64**: Patch específico de la versión 6.1.0 del kernel.
  - **vmlinuz-6.1.0-31-amd64**: Idem.
  - **vmlinuz-6.13.7**: La versión a la que actualizamos en la práctica 1. No posee amd64 debido a que fue compilada manualmente.

#### 4. Acceda al codigo fuente de GNU Linux, sea visitando https://kernel.org/ o bien trayendo el código del kernel (cuidado, como todo software monolítico son unos cuantos gigas) `git clone https://github.com/torvalds/linux.git`

#### 5. ¿Para qué sirve el siguiente archivo? `arch/x86/entry/syscalls/syscall_64.tbl`

#### 6. ¿Para qué sirve la herramienta strace? ¿Cómo se usa?

#### 7. ¿Para qué sirve la herramienta ausyscall? ¿Cómo se usa?

### Práctica guiada - La System Calls que vamos a implementar accederán a la estructura [task_struct](https://alex-xjk.github.io/post/taskstruct/) que representa cada proceso en el sistema. Ha evolucionado con el tiempo, pero en las versiones más recientes del kernel (6.x), sigue teniendo los mismos principios básicos con nuevas adiciones y modificaciones. Es la estructura utilizada por el [scheduler](https://docs.kernel.org/scheduler/sched-eevdf.html) para planificar las tareas del Sistema Operativo.

#### Estas estructuras junto a otras conforman lo que en los libros de Sistemas Operativos se denomina la PCB (Process Control Block).

#### Accederemos con nuestra llamada al sistema a algunos datos almacenados en los de la estructura task_struct.

#### Para ello modificaremos los siguientes archivos del código fuente del Kernel para declarar nuestras system calls

```
arch/arm64/include/asm/unistd.h
arch/x86/entry/syscalls/syscall_64.tbl
include/uapi/asm-generic/unistd.h
```

#### Y además agregaremos estos dos nuevos archivos dónde colocaremos la implementación de nuestras system call

```
kernel/Makefile
kernel/my_sys_call.c
```

### Agregamos una nueva System Call

#### 1. Añadiremos el siguiente archivo con el código de nuestra system call:

```c
// kernel/my_sys_call.c

#include <linux/kernel.h>
#include <linux/syscalls.h>
#include <linux/sched.h>
#include <linux/uaccess.h>
#include <linux/sched/signal.h>
#include <linux/slab.h> // Para kmalloc y kfree

SYSCALL_DEFINE1(my_sys_call, int, arg) {
    printk(KERN_INFO "My syscall called with arg: %d\n", arg);
    return 0;
}

SYSCALL_DEFINE2(get_task_info, char __user *, buffer, size_t, length) {
    struct task_struct *task;
    char kbuffer[1024]; // Buffer en el espacio del kernel
    int offset = 0 ;

    for_each_process(task) {
        offset += snprintf(kbuffer + offset, sizeof(kbuffer) - offset, "PID: %d | Nombre: %s | Estado: %d \n", task->pid, task->comm, task_state_index(task));
        if (offset >= sizeof(kbuffer)) // Evita sobrepasar el tamaño del buffer
            break;

        printk(KERN_INFO "PID: %d | Nombre: %s\n", task->pid, task->comm);
    }

    // Copia la información al espacio de usuario
    if (copy_to_user(buffer, kbuffer, min(length, (size_t)offset)))
        return - EFAULT;
    return min(length, (size_t)offset);
}

SYSCALL_DEFINE2(get_threads_info, char __user *, buffer, size_t, length) {
    struct task_struct *task, *thread;
    char *kbuffer;
    int offset = 0;

    // Asignar memoria dinámica para el buffer
    kbuffer = kmalloc(2048, GFP_KERNEL);
    if (!kbuffer)
        return - ENOMEM;

    for_each_process(task) {
        offset += snprintf(kbuffer + offset, 2048 - offset, "Proceso: %s (PID: %d)\n", task->comm, task->pid);

        for_each_thread(task, thread) {
            offset += snprintf(kbuffer + offset, 2048 - offset, " ├── Hilo: %s (TID: %d)\n", thread->comm, thread->pid);
            if (offset >= 2048)
                break;
        }

        if (offset >= 2048)
            break;
    }

    if (copy_to_user(buffer, kbuffer, min(length, (size_t)offset))) {
        kfree(kbuffer);
        return -EFAULT;
    }

    kfree(kbuffer);
    return min(length, (size_t)offset);
}
```

#### Mirando el código anterior, investigue y responda lo siguiente:

##### a. ¿Para qué sirven los macros SYS_CALL_DEFINE?

##### b. ¿Para que se utilizan la macros for_each_process y for_each_thread?

##### c. ¿Para que se utiliza la función copy_to_user?

##### d. ¿Para qué se utiliza la función printk?, ¿porque no la típica printf?

##### e. Podría explicar que hacen las sytem call que hemos incluido?

#### 2. Modificaremos uno de los archivos Makefile del código del Kernel para indicar la compilación de nuestro código agregado en el paso anterior:

```
// kernel/Makefile
obj-y = fork.o exec_domain.o panic.o \
    cpu.o exit.o softirq.o resource.o \
    sysctl.o capability.o ptrace.o user.o \
    signal.o sys.o umh.o workqueue.o pid.o task_work.o \
    extable.o params.o \
    kthread.o sys_ni.o nsproxy.o \
    notifier.o ksysfs.o cred.o reboot.o \
    async.o range.o smpboot.o ucount.o regset.o \
    my_sys_call.o
```

#### 3. Añadir una entrada al final de la tabla que contiene todas las System Calls, la syscall table. En nuestro caso, vamos a dar soporte para nuestra syscall a la arquitectura x86_64.

#### Atención:

- El archivo donde añadiremos la entrada para la system call está estructurado en columnas de la siguiente forma: `<number> <abi> <name> <entry point>`.
- Buscaremos la última entrada cuya ABI sea “common” y luego agregaremos una línea para nuestra system call.
- Debemos asignar un número único a nuestra system call, de modo que aumentaremos en 1 el número de la última.

```
444 commonlandlock_create_ruleset               sys_landlock_create_ruleset
445 commonlandlock_add_rule                     sys_landlock_add_rule
446 commonlandlock_restrict_self                sys_landlock_restrict_self
447 commonmemfd_secret                          sys_memfd_secret
448 commonprocess_mrelease                      sys_process_mrelease
449 commonfutex_waitv                           sys_futex_waitv
450 commonset_mempolicy_home_node               sys_set_mempolicy_home_node
451 common my_sys_call                          sys_my_sys_call
452 common get_task_info                        sys_get_task_info
453 common get_threads_info                     sys_get_threads_info
```

#### Ahora incluimos la declaración de nuestras system calls en los headers del kernel junto a las otras system calls. Es importante recordar que debemos aumentar el valor de \_\_NR_syscalls de acuerdo a la cantidad de system calls que hemos agregado, ya que este es el tamaño de un array interno dónde están los punteros a los manejadores de las system calls.

```c
// include/uapi/asm-generic/unistd.h

#define __NR_set_mempolicy_home_node 450
__SYSCALL(__NR_set_mempolicy_home_node, sys_set_mempolicy_home_node)

#define __NR_my_sys_call 451
__SYSCALL(__NR_my_sys_call, sys_my_sys_call)

#define __NR_get_task_info 452
__SYSCALL(__NR_get_task_info, sys_get_task_info)

#define __NR_get_threads_info 453
__SYSCALL(__NR_get_threads_info, sys_get_threads_info)

#undef __NR_syscalls
#define __NR_syscalls 454
```

#### 4. Lo próximo que debemos realizar es compilar el Kernel con nuestros cambios. Una vez seguidos todos los pasos de la compilación como lo vimos en el trabajo práctico 1, acomodamos la imagen generada y arrancamos el sistema con el nuevo kernel.

#### 5. Ahora vamos a verificar que nuestras system calls nuevas ya son parte del kernel, para esto ejecutamos: `$ grep get_task_info "/boot/System.map-$(uname -r)"`

#### Aquí deberíamos ver el mapa de símbolos correspondiente a nuestra system call en el System.map del Kernel recientemente compilado

#### 6. Nuestro último paso es realizar un programa que llame a la System Call.

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/syscall.h>
#include <string.h>

#define SYS_get_task_info 452

void print_task_info(const char *info) {
    printf("\nInformación de los procesos en ejecución:\n");
    printf("----------------------------------------\n");
    printf("%s", info);
    printf("\n----------------------------------------\n");
}

int main() {
    char buffer[1024]; // Buffer donde se almacenará la información de las tareas
    long bytes_read;

    // Llamada al sistema para obtener la información de los procesos
    bytes_read = syscall(SYS_get_task_info, buffer, sizeof(buffer));

    // Comprobamos si la llamada al sistema fue exitosa
    if (bytes_read < 0 ) {
        perror("Error al invocar la llamada al sistema");
        return 1;
    }

    // Mostrar la información obtenida de los procesos
    print_task_info(buffer);

    return 0;
}
```

#### Nota: Cuando utilizamos llamadas al sistema, por ejemplo open() que permite abrir un archivo, no es necesario invocarlas de manera explícita, ya que por defecto la librería libc tiene funciones que encapsulan las llamadas al sistema. Luego lo compilamos para obtener nuestro programa. Para ello ejecutamos: `$ gcc -o get_task_info get_task_info.c`

#### Por último nos queda ejecutar nuestro programa y ver el resultado. `$ ./get_task_info`

#### Con lo visto en la Práctica 1 sobre Makefiles, construya un Makefile de manera que si ejecuto:

- **make**, nuestro programa se compila get_task_info.c.
- **make clean**, limpia el ejecutable y el código objeto generado.
- **make run**, ejecuta el programa.

### Monitoreando System Calls

#### 1. Ejecute el programa anteriormente compilado`: `$ ./get_task_info`. Cual es el output del programa?

#### 2. Luego de ejecutar el programa ahora ejecute: `$ sudo dmesg`. ¿Cuál es el output? Por qué? (recuerde printk y lea el man de dmesg)

#### 3. Ejecute el programa anteriormente compilado con la herramienta strace: `$ strace get_task_info`. Aclaración: Si el programa strace no está instalado, puede instalarlo en distribuciones basadas en Debian con: `$ sudo apt-get install strace`.

#### En alguna parte del log de strace debería ver algo similar a lo siguiente: `syscall_0x1c4(0xffffdf859ba0, 0x400, 0xaaaabe110740, 0xffff9cc790c0, 0xbd2cc5d5aef6ff14, 0xffff9cc22078) = 0x400`. Si luego ejecuto: `# echo $((0x1C4))`. ¿Qué valor obtengo? Por qué?
