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

El archivo `arch/x86/entry/syscalls/syscall_64.tbl` forma parte del código fuente del Kernel y es esencial en el proceso de definición de las syscalls para arquitecturas de 64 bits en x86 (también conocidas como x86_64 o AMD64). En pocas palabras, este archivo le dice al kernel qué número de syscall corresponde a qué función del sistema.

Este archivo es básicamente una tabla, y cada línea representa una syscall. El formato de la tabla es:

`<número> <abi> <nombre_syscall> <nombre_función_kernel>`

- **número**: Número identificador de la syscall.
- **abi**: Application Binary Interface. Puede ser common, 64, x32, etc.
- **nombre_syscall**: Nombre simbólico de la syscall (write, read, etc).
- **nombre_función_kernel**: Nombre de la función interna que maneja esa syscall dentro del código del kernel.

#### 6. ¿Para qué sirve la herramienta strace? ¿Cómo se usa?

**strace** es una herramienta de línea de comandos en Linux que sirve para rastrear y mostrar las syscalls (llamadas al sistema) que hace un programa mientras se ejecuta.

Es útil para depurar errores, entender el comportamiento de programas, o ver por qué un proceso está fallando.

- Su uso básico es: `strace ./mi_programa`
- Seguir un proceso que está en ejecución: `strace -p <PID>`
- Filtrar por tipo de syscall: `strace -e trace=file ./mi_programa`

#### 7. ¿Para qué sirve la herramienta ausyscall? ¿Cómo se usa?

La herramienta **ausyscall** se usa para mapear números de llamadas al sistema (syscalls) con sus nombres correspondientes, y viceversa. Es útil para analizar registros de auditoría (como los generados por auditd) donde las llamadas al sistema aparecen como números y no como nombres legibles.

Sus funciones son:

- Traducir números de syscalls a nombres (ej: 59 → execve).
- Traducir nombres de syscalls a números (ej: openat → 257 en x86_64).
- Mostrar las syscalls disponibles según la arquitectura del sistema (x86, x86_64, arm, etc.).

Cómo se usa:

- Obtener el nombre de una syscall por su id: `ausyscall 59` → execve.
- Obtener el número de una syscall por su nombre: `ausyscall execve` → 59.
- Listar todas las syscalls de la arquitectura actual: `ausyscall --dump`.

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

Los macros **SYS_CALL_DEFINE** se usan para definir y mappear systemcalls a nombres simbólicos. Define un número de llamada al sistema (un entero) que se asigna a un nombre de macro específico. Este mapeo permite que las aplicaciones, en lugar de usar números, utilicen nombres de macros como sys_read, sys_write, etc. para solicitar servicios del sistema, lo cual es mucho más legible.

##### b. ¿Para que se utilizan las macros for_each_process y for_each_thread?

- `for_each_process` es un macro que permite loopear sobre todos los task_struct que existen en el sistema, es decir todos los procesos, incluyendo los procesos en estado running, sleeping, stopped y zombie.
- `for_each_thread` es un macro que permite loopear sobre todos los hilos de un proceso determinado.

##### c. ¿Para que se utiliza la función copy_to_user?

`copy_to_user` es una función del kernel que copia, de forma segura, datos de espacio kernel a espacio de usuario. Se usa debido a que el kernel se ejecuta en un espacio de memoria separado de otros programas. No se puede simplemente usar `memcpy()`, ya que hacer esto podría crashear el sistema o crear problemas de seguridad.

##### d. ¿Para qué se utiliza la función printk?, ¿porque no la típica printf?

La función `printk` es la versión del kernel de la función `printf` de la libc. Se usa para loggear mensajes al buffer del kernel, usualmente para debuggear.

No se usa `printf` en este contexto debido a que el kernel Linux no tiene acceso a la libc, ya que ésta corre en espacio de usuario.

##### e. Podría explicar que hacen las system call que hemos incluido?

1. La primer systemcall (**my_sys_call**) recibe un entero y lo escribe en el buffer del kernel usando `printk`. Este output va hacia **dmesg**, no a la terminal.
2. La segunda systemcall (**get_task_info**) loopea todos los procesos del sistema y para cada uno guarda en el buffer del kernel su PID, su nombre y el estado de ese proceso. Luego copia esta información hacia espacio de usuario para que el programa de usuario pueda leerlo.
3. La tercer systemcall (**get_threads_info**) es similar a la anterior. Loopea todos los procesos, y para cada uno, loopea todos los hilos del mismo. Recolecta, para cada proceso, su nombre + PID y nombre + TID de cada uno de sus hilos. Almacena todo esto en un buffer dinámico (**kbuffer con kmalloc**), el cual luego se copia a espacio de usuario.

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
444 common landlock_create_ruleset               sys_landlock_create_ruleset
445 common landlock_add_rule                     sys_landlock_add_rule
446 common landlock_restrict_self                sys_landlock_restrict_self
447 common memfd_secret                          sys_memfd_secret
448 common process_mrelease                      sys_process_mrelease
449 common futex_waitv                           sys_futex_waitv
450 common set_mempolicy_home_node               sys_set_mempolicy_home_node
451 common my_sys_call                           sys_my_sys_call
452 common get_task_info                         sys_get_task_info
453 common get_threads_info                      sys_get_threads_info
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

#### 5. Ahora vamos a verificar que nuestras system calls nuevas ya son parte del kernel, para esto ejecutamos: `grep get_task_info "/boot/System.map-$(uname -r)"`

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

#### Nota: Cuando utilizamos llamadas al sistema, por ejemplo open() que permite abrir un archivo, no es necesario invocarlas de manera explícita, ya que por defecto la librería libc tiene funciones que encapsulan las llamadas al sistema. Luego lo compilamos para obtener nuestro programa. Para ello ejecutamos: `gcc -o get_task_info get_task_info.c`

#### Por último nos queda ejecutar nuestro programa y ver el resultado. `./get_task_info`

#### Con lo visto en la Práctica 1 sobre Makefiles, construya un Makefile de manera que si ejecuto:

- **make**, nuestro programa se compila get_task_info.c.
- **make clean**, limpia el ejecutable y el código objeto generado.
- **make run**, ejecuta el programa.

El makefile que usaré es:

```make
TARGET = get_task_info
SRC = get_task_info.c

all:
	gcc -o $(TARGET) $(SRC)

clean:
	rm -f $(TARGET)

run: all
	./$(TARGET)
```

### Monitoreando System Calls

#### 1. Ejecute el programa anteriormente compilado: `./get_task_info`. Cual es el output del programa?

El output del programa es:

```
Información de los procesos en ejecución:
----------------------------------------
PID: 1 | Nombre: systemd | Estado: 1
PID: 2 | Nombre: kthreadd | Estado: 1
PID: 3 | Nombre: pool_workqueue_ | Estado: 1
PID: 4 | Nombre: kworker/R-rcu_g | Estado: 8
PID: 5 | Nombre: kworker/R-sync_ | Estado: 8
PID: 6 | Nombre: kworker/R-slub_ | Estado: 8
PID: 7 | Nombre: kworker/R-netns | Estado: 8
PID: 9 | Nombre: kworker/0:1 | Estado: 8
PID: 12 | Nombre: kworker/R-mm_pe | Estado: 8
PID: 13 | Nombre: rcu_tasks_kthre | Estado: 8
PID: 14 | Nombre: rcu_tasks_rude_ | Estado: 8
PID: 15 | Nombre: rcu_tasks_trace | Estado: 8
PID: 16 | Nombre: ksoftirqd/0 | Estado: 1
PID: 17 | Nombre: rcu_preempt | Estado: 8
PID: 18 | Nombre: rcu_exp_par_gp_ | Estado: 1
PID: 19 | Nombre: rcu_exp_gp_kthr | Estado: 1
PID: 20 | Nombre: migration/0 | Estado: 1
PID: 21 | Nombre: idle_inject/0 | Estado: 1
PID: 22 | Nombre: cpuhp/0 | Estado: 1
PID: 23 | Nombre: cpuhp/1 | Estado: 1
PID: 24 | Nombre: idle_inject/1 | Estado: 1
PID: 25 | Nombre: migration/1 | Estado: 1
PID: 26 | Nombre: ksoftirqd/1 | Estado: 1
PID: 28 |
----------------------------------------
```

#### 2. Luego de ejecutar el programa ahora ejecute: `sudo dmesg`. ¿Cuál es el output? Por qué? (recuerde printk y lea el man de dmesg)

Lo que veo al ejecutar `dmesg` son los mensajes generados por las llamadas a `printk` que realiza la systemcall **get_task_info**.

Este `printk()` se ejecuta dentro del bucle `for_each_process(task)`, lo que significa que cada vez que la syscall recorre un proceso activo, imprime su PID y su nombre al log del kernel.

#### 3. Ejecute el programa anteriormente compilado con la herramienta strace: `strace ./get_task_info`. Aclaración: Si el programa strace no está instalado, puede instalarlo en distribuciones basadas en Debian con: `sudo apt-get install strace`.

#### En alguna parte del log de strace debería ver algo similar a lo siguiente: `syscall_0x1c4(0xffffdf859ba0, 0x400, 0xaaaabe110740, 0xffff9cc790c0, 0xbd2cc5d5aef6ff14, 0xffff9cc22078) = 0x400`. Si luego ejecuto: `# echo $((0x1C4))`. ¿Qué valor obtengo? Por qué?

El output al correr el comando `strace ./get_task_info` es:

```
execve("./get_task_info", ["./get_task_info"], 0x7ffef0f082b0 /* 32 vars */) = 0
brk(NULL)                               = 0x557ae4200000
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f55696bc000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No existe el fichero o el directorio)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=22426, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 22426, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f55696b6000
close(3)                                = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\20t\2\0\0\0\0\0"..., 832) = 832
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
newfstatat(3, "", {st_mode=S_IFREG|0755, st_size=1922136, ...}, AT_EMPTY_PATH) = 0
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
mmap(NULL, 1970000, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f55694d5000
mmap(0x7f55694fb000, 1396736, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x26000) = 0x7f55694fb000
mmap(0x7f5569650000, 339968, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x17b000) = 0x7f5569650000
mmap(0x7f55696a3000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1ce000) = 0x7f55696a3000
mmap(0x7f55696a9000, 53072, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f55696a9000
close(3)                                = 0
mmap(NULL, 12288, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f55694d2000
arch_prctl(ARCH_SET_FS, 0x7f55694d2740) = 0
set_tid_address(0x7f55694d2a10)         = 2977
set_robust_list(0x7f55694d2a20, 24)     = 0
rseq(0x7f55694d3060, 0x20, 0, 0x53053053) = 0
mprotect(0x7f55696a3000, 16384, PROT_READ) = 0
mprotect(0x557ac622c000, 4096, PROT_READ) = 0
mprotect(0x7f55696f4000, 8192, PROT_READ) = 0
prlimit64(0, RLIMIT_STACK, NULL, {rlim_cur=8192*1024, rlim_max=RLIM64_INFINITY}) = 0
munmap(0x7f55696b6000, 22426)           = 0
syscall_0x1d4(0x7ffe80ee6130, 0x400, 0x557ac622cdd8, 0, 0x7f55696c8680, 0x7f55696f6ad0) = 0x400
newfstatat(1, "", {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0), ...}, AT_EMPTY_PATH) = 0
getrandom("\x60\xe4\xd2\x6d\x6b\xad\xec\xf0", 8, GRND_NONBLOCK) = 8
brk(NULL)                               = 0x557ae4200000
brk(0x557ae4221000)                     = 0x557ae4221000
write(1, "\n", 1
)                       = 1
write(1, "Informaci\303\263n de los procesos en "..., 44Información de los procesos en ejecución:
) = 44
write(1, "--------------------------------"..., 41----------------------------------------
) = 41
write(1, "PID: 1 | Nombre: systemd | Estad"..., 1014PID: 1 | Nombre: systemd | Estado: 1
PID: 2 | Nombre: kthreadd | Estado: 1
PID: 3 | Nombre: pool_workqueue_ | Estado: 1
PID: 4 | Nombre: kworker/R-rcu_g | Estado: 8
PID: 5 | Nombre: kworker/R-sync_ | Estado: 8
PID: 6 | Nombre: kworker/R-slub_ | Estado: 8
PID: 7 | Nombre: kworker/R-netns | Estado: 8
PID: 9 | Nombre: kworker/0:1 | Estado: 8
PID: 12 | Nombre: kworker/R-mm_pe | Estado: 8
PID: 13 | Nombre: rcu_tasks_kthre | Estado: 8
PID: 14 | Nombre: rcu_tasks_rude_ | Estado: 8
PID: 15 | Nombre: rcu_tasks_trace | Estado: 8
PID: 16 | Nombre: ksoftirqd/0 | Estado: 1
PID: 17 | Nombre: rcu_preempt | Estado: 8
PID: 18 | Nombre: rcu_exp_par_gp_ | Estado: 1
PID: 19 | Nombre: rcu_exp_gp_kthr | Estado: 1
PID: 20 | Nombre: migration/0 | Estado: 1
PID: 21 | Nombre: idle_inject/0 | Estado: 1
PID: 22 | Nombre: cpuhp/0 | Estado: 1
PID: 23 | Nombre: cpuhp/1 | Estado: 1
PID: 24 | Nombre: idle_inject/1 | Estado: 1
PID: 25 | Nombre: migration/1 | Estado: 1
PID: 26 | Nombre: ksoftirqd/1 | Estado: 1
) = 1014
write(1, "PID: 28 |\n", 10PID: 28 |
)             = 10
write(1, "--------------------------------"..., 41----------------------------------------
) = 41
exit_group(0)                           = ?
+++ exited with 0 +++
```

Aproximadamente a la mitad del output, veo la línea similar a la pedida:

`syscall_0x1d4(0x7ffe80ee6130, 0x400, 0x557ac622cdd8, 0, 0x7f55696c8680, 0x7f55696f6ad0) = 0x400`

Cuando ejecuto `echo $((0x1d4))` recibo como resultado el número entero 468, que no solo es el número de la syscall que llamamos.

Lo que ocurre acá es que **strace** solo sabe los nombres de las syscalls estándar (como read, write, fork, etc.). Como nuestra syscall es nueva, no conoce su nombre semántico y solo conoce su número, el cual es 0x1d4 en hexadecimal, o 468 en decimal.
