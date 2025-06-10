<h1 align="center">Práctica 6</h1>

## Multiprocesadores

### A - Conceptos teóricos

#### 1. Explique las características principales y formas de comunicación entre procesos en:

##### a. Multiprocesadores con memoria compartida

**Características principales**:

- La comunicación entre las CPUs es a través de la memoria compartida, que es un único espacio lógico de direcciones para todos los procesos.
- Cada CPU tiene el mismo acceso que otras a la memoria física a través de un único BUS físico.
- Para acceder a una palabra de memoria por lo general cada CPU requiere de 2 a 10 nanosegundos.

**Formas de comunicación entre procesos**:

- Variables compartidas: Acceso directo a datos comunes.
- Mutex/Semáforos: Para exclusión mutua.
- Monitores: Estructuras con sincronización integrada.
- Barriers: Para sincronizar múltiples hilos.
- Operaciones atómicas: Evitan interferencias sin locks.

##### b. Multicomputadora con memoria independiente / pasaje de mensajes

**Características principales**:

- Varios pares (CPU, memoria) se conectan a una interconexión de alta velocidad pasando mensajes.
- Cada CPU tiene su propia memoria local y privada, la cual solo puede ser utilizada directamente por esa CPU.
- El retardo del paso de mensajes entre CPUs es de entre 10 a 50 microsegundos.

**Formas de comunicación entre procesos**:

- Pasaje de mensajes: Envío/recepción de datos entre procesos.
- Protocolos estándar como MPI.
- Send/Receive: Comunicación básica punto a punto.
- Broadcast/Multicast: Envío a múltiples nodos.
- Sincronización explícita: Se necesita coordinar envíos y recepciones.

##### c. Sistemas Distribuidos

**Características principales**:

- Conecta sistemas de cómputo completos a través de una red.
- Cada sistema de cómputo es una computadora completa y se llama nodo.
- Cada nodo tiene su propia memoria, y se comunican mediante el pasaje de mensajes.
- El retardo del pasaje de mensajes es de entre 10 a 100 milisegundos.
- Provee heterogeneidad de sistemas y hardware.

**Formas de comunicación entre procesos**:

- Pasaje de mensajes: Principal mecanismo (sockets, RPC, MPI).
- Middleware: Software que abstrae la comunicación (CORBA, gRPC).
- Comunicación síncrona o asíncrona: Dependiendo del protocolo.
- Protocolos de red: TCP/IP, HTTP, UDP, etc.

#### 2. Indique en qué CPU/s se ejecuta el sistema operativo en sistemas de tipo:

##### a. Maestro-esclavo

El SO se ejecuta exclusivamente en la CPU maestra. Las CPUs esclavas no toman decisiones ni ejecutan código del SO → solo ejecutan tareas asignadas por la CPU maestra.

##### b. SMP

El SO se ejecuta en todas las CPUs, pero no como múltiples copias independientes.
Hay una **sola copia** del SO en memoria, y cada CPU puede ejecutar distintas partes del kernel al mismo tiempo, gracias a mecanismos de sincronización que evitan conflictos.
Todas las CPUs tienen los mismos privilegios y acceso simétrico a recursos del sistema.

#### 3. ¿Qué puede ocurrir si el kernel S.O. quiere acceder en paralelo a una estructura de datos en la memoria compartida en un sistema SMP? ¿Qué mecanismo permite manejar esta situación?

Si el kernel intenta acceder en paralelo a una estructura de datos compartida en un sistema SMP, pueden ocurrir **condiciones de carrera**. Esto significa que dos o más CPUs podrían:

- Leer datos inconsistentes.
- Modificar al mismo tiempo una estructura y así corromperla.
- Generar fallos impredecibles o errores de sincronización.

El mecanismo que permite manejar esta situación es la sincronización, que se puede llevar a cabo vía locks, semáforos y operaciones atómicas.

#### 4. ¿En el caso de una arquitectura SMP puede haber un impacto negativo por migrar un hilo de una CPU a otra?

Migrar un hilo de una CPU a otra en una arquitectura SMP puede tener impactos negativos, aunque en general los SO intentan minimizar estos efectos. Los impactos negativos son:

- Se pierde la caché (cache miss):
  - Cada CPU tiene su propia caché.
  - Si un hilo se migra, los datos que tenía en caché se tienen que volver a cargar en la nueva CPU.
  - Esto tiene un impacto en el rendimiento.
- Aumenta la latencia:
  - El cambio puede implicar más tiempo de scheduling y sincronización.
  - Especialmente crítico en hilos que hacen muchas syscalls o acceden a recursos compartidos.
- La sincronización se torna más compleja:
  - Si el hilo accede a estructuras compartidas, la migración puede generar más contención de locks o spinlocks.
- Se interrumpe la Afinidad de CPU:
  - Los hilos suelen tener afinidad con una CPU (por rendimiento o por razones térmicas).
  - Migrarlos rompe esta afinidad y puede afectar a la eficiencia del scheduler.

### B - Práctica guiada

### Nota: La práctica requiere ejecutar los programas `affinity` y `affinity_half_and_half` provistos en el repositorio de la cátedra. En cada caso si el programa tarda demasiado o muy poco ajuste la macro `ITERATIONS` y/o la macro `THREADS` para disminuir o aumentar la carga del sistema.

#### 1. Utilice `lscpu` para determinar cuántos cores lógicos tiene su computadora (si usa una máquina virtual asegúrese de configurar al menos 2 CPUs para llevar a cabo el resto de la práctica).

```sh
so@so:~/codigo-para-practicas/practica6$ lscpu
Arquitectura:                            x86_64
  modo(s) de operación de las CPUs:      32-bit, 64-bit
  Tamaños de las direcciones:            48 bits physical, 48 bits virtual
  Orden de los bytes:                    Little Endian
CPU(s):                                  6
  Lista de la(s) CPU(s) en línea:        0-5
ID de fabricante:                        AuthenticAMD
  Nombre del modelo:                     AMD Ryzen 5 7600 6-Core Processor
    Familia de CPU:                      25
    Modelo:                              97
    Hilo(s) de procesamiento por núcleo: 1
    Núcleo(s) por «socket»:              6
    «Socket(s)»:                         1
    Revisión:                            2
    BogoMIPS:                            7585,58
    Indicadores:                         fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36
                                          clflush mmx fxsr sse sse2 ht syscall nx mmxext fxsr_opt rdtscp lm cons
                                         tant_tsc rep_good nopl nonstop_tsc cpuid extd_apicid tsc_known_freq pni
                                          pclmulqdq ssse3 cx16 sse4_1 sse4_2 movbe popcnt aes rdrand hypervisor
                                         lahf_lm cmp_legacy cr8_legacy abm sse4a misalignsse 3dnowprefetch vmmca
                                         ll fsgsbase bmi1 bmi2 invpcid rdseed adx clflushopt sha_ni arat
Características de virtualización:
  Fabricante del hipervisor:             KVM
  Tipo de virtualización:                lleno
Cachés (suma de todas):
  L1d:                                   192 KiB (6 instancias)
  L1i:                                   192 KiB (6 instancias)
  L2:                                    6 MiB (6 instancias)
  L3:                                    192 MiB (6 instancias)
NUMA:
  Modo(s) NUMA:                          1
  CPU(s) del nodo NUMA 0:                0-5
Vulnerabilidades:
  Gather data sampling:                  Not affected
  Itlb multihit:                         Not affected
  L1tf:                                  Not affected
  Mds:                                   Not affected
  Meltdown:                              Not affected
  Mmio stale data:                       Not affected
  Reg file data sampling:                Not affected
  Retbleed:                              Not affected
  Spec rstack overflow:                  Mitigation; safe RET
  Spec store bypass:                     Not affected
  Spectre v1:                            Mitigation; usercopy/swapgs barriers and __user pointer sanitization
  Spectre v2:                            Mitigation; Retpolines; STIBP disabled; RSB filling; PBRSB-eIBRS Not af
                                         fected; BHI Not affected
  Srbds:                                 Not affected
  Tsx async abort:                       Not affected
```

**Tengo 6 cores lógicos en la máquina virtual**.

#### 2. Analice el código y comentarios del programa `affinity.c`.

```c
#define _GNU_SOURCE // For sched_getcpu()
#include <stdio.h>
#include <stdarg.h>
#include <pthread.h>
#include <sched.h>

#define THREADS 100
#define ITERATIONS 1000000

// Mutex for synchronizing printf output
pthread_mutex_t print_lock = PTHREAD_MUTEX_INITIALIZER;

/** printf_sync - A thread-safe printf function
 *  @fmt: The format string for printf
 *  @...: The values to format and print
 *
 *  This function ensures that printf is thread-safe by using a mutex to lock the output
 *  buffer while printing. This is important in multi-threaded or multi-processed applications
 *  where multiple threads or processes may attempt to write to stdout simultaneously.
 *  This function also flushes the output buffer immediately to ensure that the output is consistent
 *  across threads or processes.
 */
void printf_sync(const char *fmt, ...)
{
    // Lock the mutex to ensure that only one thread can print at a time
    pthread_mutex_lock(&print_lock);
    va_list args;
    va_start(args, fmt);
    vprintf(fmt, args);
    va_end(args);
    // Flush the output buffer to ensure immediate display
    fflush(stdout);
    pthread_mutex_unlock(&print_lock);
}

void *worker(void *rank_p)
{
    // Cast the void pointer to an int pointer and dereference it to get the thread rank (not id)
    int rank = *(int *)rank_p;
    // Get the CPU on which this thread is running
    int cpu = sched_getcpu();
    // Print a message indicating the worker thread's rank
    printf_sync("Worker thread %d, running on CPU %d\n", rank, cpu);
    // Simulate some work by the worker thread
    for (int i = 0; i < ITERATIONS; i++)
    {
        // Do some computation
        for (int j = 0; j < 1000; j++)
        {
            // do nothing, just simulate work
        }
        int new_cpu = sched_getcpu(); // Get the CPU again
        if (new_cpu != cpu)           // Check if the CPU has changed
        {
            printf_sync("Worker thread %d moved from CPU %d to CPU %d\n", rank, cpu, new_cpu);
            cpu = new_cpu; // Update the CPU variable
        }
    }
    return NULL;
}

int main()
{
    // Create an array of worker threads
    pthread_t threads[THREADS];
    // Create an array to hold thread IDs
    int thread_ranks[THREADS];
    // Create and start each worker thread
    for (int i = 0; i < THREADS; i++)
    {
        thread_ranks[i] = i;
        if (pthread_create(&threads[i], NULL, worker, &thread_ranks[i]) != 0)
        {
            perror("Failed to create thread");
            return 1;
        }
    }
    // Wait for all worker threads to finish
    for (int i = 0; i < THREADS; i++)
    {
        pthread_join(threads[i], NULL);
    }
    return 0;
}
```

NOTA: el código de la cátedra tenía un pequeño error en la línea: `for (int j = 0; j < 1000; i++)`, se debe incrementar j y no i.

**Funcionamiento del código**:

1. Se usa la librería Pthreads.
2. Se crean 100 hilos, asignando a cada uno su ID.
3. Cada hilo ejecuta la función `worker()`.
4. Dentro de esa función, cada hilo obtiene el número de CPU donde se está ejecutando actualmente y lo imprime con exclusión mutua.
5. Luego los hilos simulan trabajar y chequean constantemente si la CPU donde se están ejecutando ha cambiado o no (es decir, si el scheduler los migró a otra CPU o no).

#### 3. Compile los programas provistos en practica6 con el comando `make`.

```sh
root@so:/home/so/codigo-para-practicas/practica6# make
cc -Wall -Werror    affinity.c   -o affinity
cc -Wall -Werror    affinity_half_and_half.c   -o affinity_half_and_half
```

#### 4. Ejecute `./affinity` y conteste:

##### a. ¿Qué información muestra el programa?

NOTA: cambié la cantidad de hilos de 100 a 12 y las iteraciones de 1_000_000 a 20_000_000 para que el output sea más ameno.

```sh
root@so:/home/so/codigo-para-practicas/practica6# ./affinity
Worker thread 0, running on CPU 5
Worker thread 1, running on CPU 0
Worker thread 2, running on CPU 3
Worker thread 2 moved from CPU 3 to CPU 0
Worker thread 9, running on CPU 1
Worker thread 11, running on CPU 4
Worker thread 3, running on CPU 2
Worker thread 5, running on CPU 3
Worker thread 4, running on CPU 2
Worker thread 6, running on CPU 1
Worker thread 6 moved from CPU 1 to CPU 2
Worker thread 7, running on CPU 1
Worker thread 7 moved from CPU 1 to CPU 2
Worker thread 8, running on CPU 3
Worker thread 3 moved from CPU 2 to CPU 3
Worker thread 4 moved from CPU 2 to CPU 0
Worker thread 10, running on CPU 4
Worker thread 6 moved from CPU 2 to CPU 1
Worker thread 7 moved from CPU 2 to CPU 1
Worker thread 4 moved from CPU 0 to CPU 5
Worker thread 6 moved from CPU 1 to CPU 0
Worker thread 2 moved from CPU 0 to CPU 2
Worker thread 5 moved from CPU 3 to CPU 1
Worker thread 10 moved from CPU 4 to CPU 2
Worker thread 6 moved from CPU 0 to CPU 2
Worker thread 8 moved from CPU 3 to CPU 4
Worker thread 6 moved from CPU 2 to CPU 5
```

##### b. ¿Los hilos se ejecutan siempre en el mismo core desde su creación?

Algunos hilos se ejecutan siempre en el mismo core de principio a fin, mientras que otros son migrados a otros cores a lo largo de la ejecución.

##### c. Para más claridad puede elegir un hilo y seguir su ejecución con grep: `./affinity | grep "thread 4[, ]"`

```sh
root@so:/home/so/codigo-para-practicas/practica6# ./affinity | grep "thread 4[, ]"
Worker thread 4, running on CPU 3
root@so:/home/so/codigo-para-practicas/practica6# ./affinity | grep "thread 4[, ]"
Worker thread 4, running on CPU 5
root@so:/home/so/codigo-para-practicas/practica6# ./affinity | grep "thread 4[, ]"
Worker thread 4, running on CPU 5
root@so:/home/so/codigo-para-practicas/practica6# ./affinity | grep "thread 4[, ]"
Worker thread 4, running on CPU 3
Worker thread 4 moved from CPU 3 to CPU 1
```

Nuevamente, a veces el scheduler migra los hilos y a veces no.

#### 5. Utilice `taskset` para ejecutar todos los hilos del programa `affinity` en el core 0.

##### a. ¿Cuánto tiempo tardó la ejecución comparativamente con invocar `./affinity` sin taskset? Puede usar el comando `time` y sumar los 3 valores que devuelve para obtener un valor preciso.

#### 6. Analice el código y comentarios de `affinity_half_and_half.c`.

##### a. ¿En qué core se ejecutarán los procesos con `rank < THREADS / 2`?

##### b. Ejecute `./affinity_half_and_half` y observe la asignación de cores de forma similar al punto 4. De nuevo puede filtrar un hilo con `grep` para más claridad.

##### c. ¿Los hilos que arrancan un core dado siguen toda su ejecución en el mismo core? ¿Por qué?
