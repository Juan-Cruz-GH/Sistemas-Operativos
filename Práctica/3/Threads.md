<h1 align="center">Práctica 3</h1>

## Requisitos

Para realizar esta práctica se puede usar la misma máquina virtual de la práctica 1 o una de su elección si resulta más cómodo (por ejemplo una VM con interfaz gráfica y un IDE).

## Threading (ULT y KLT)

### Conceptos generales

#### 1. ¿Cuál es la diferencia fundamental entre un proceso y un thread?

La diferencia fundamental entre un proceso y un thread es que un proceso es **un programa en ejecución**, y un thread es **una unidad de ejecución de un proceso**.

Debido a esto, **diferentes procesos no pueden compartir el mismo espacio de direcciones, pero diferentes hilos de un mismo proceso sí**.

#### 2. ¿Qué son los User-Level Threads (ULT) y cómo se diferencian de los Kernel-Level Threads (KLT)?

Los User-Level Threads y los Kernel-Level Threads son dos enfoques para implementar hilos en un SO, y se diferencian principalmente en **quién los gestiona y cómo interactúan con el kernel**.

Los ULT son hilos gestionados completamente en espacio de usuario por una biblioteca (ej: pthread en Linux) sin soporte directo del kernel.

Los KLT son hilos gestionados directamente por el kernel.

#### 3. ¿Quién es responsable de la planificación de los ULT? ¿y los KLT? ¿Cómo afecta esto al rendimiento en sistemas con múltiples núcleos?

- El responsable de la planificación de los ULT es la biblioteca de manejo de hilos que se use.
- El responsable de la planificación de los KLT es el kernel.
- En sistemas con múltiples núcleos:
  - ULT no es ideal, ya que no provee paralelismo real, si no solo concurrencia: solo un hilo del proceso se ejecuta a la vez.
  - KLT es ideal, ya que provee paralelismo: pueden ejecutarse varios hilos del proceso en paralelo, uno por núcleo.

#### 4. ¿Cómo maneja el sistema operativo los KLT y en qué se diferencian de los procesos?

El SO maneja los KLT de la siguiente manera:

- **Planificación**: El kernel decide qué hilo se ejecuta, cuándo y en qué CPU, igual que con los procesos.
- **Context switch**: El kernel realiza el context switch entre hilos. Guarda y restaura el estado (registros, contador de programa, etc.).
- **Bloqueos y sincronización**: El SO puede bloquear un hilo sin detener el proceso completo, porque sabe que son hilos independientes.
- **Multiprocesamiento**: El kernel puede ejecutar hilos en múltiples núcleos simultáneamente.
- **Gestión de recursos compartidos**: Como los hilos de un mismo proceso comparten memoria y otros recursos, el kernel garantiza acceso concurrente seguro usando primitivas como semáforos, mutex, etc.

Los KLT se diferencian de los procesos de las siguientes formas:

| Característica     | Proceso                                   | KLT (Hilo a nivel de kernel)                                  |
| ------------------ | ----------------------------------------- | ------------------------------------------------------------- |
| Entidad del SO     | Independiente                             | Parte de un proceso                                           |
| Espacio de memoria | Tiene su **propio** espacio de memoria    | **Comparte** el espacio de memoria del proceso                |
| Contexto           | Contexto completo (memoria, registros...) | Contexto parcial (solo registros e hilos)                     |
| Coste de creación  | Alto                                      | Más bajo que un proceso                                       |
| Cambio de contexto | Más costoso                               | Más ligero                                                    |
| Comunicación       | Más difícil (IPC)                         | Más fácil (comparten memoria)                                 |
| Fallos             | Si un proceso falla, muere solo él        | Si un hilo comete un error grave, puede dañar todo el proceso |
| Paralelismo        | Puede ejecutarse en paralelo              | También puede (y con más flexibilidad)                        |

#### 5. ¿Qué ventajas tienen los KLT sobre los ULT? ¿Cuáles son sus desventajas?

Ventajas de los KLT:

- Provee paralelismo real, ya que el kernel puede asignar diferentes hilos del mismo proceso a distintos núcleos.
- Si un hilo se bloquea, los otros siguen funcionando, porque el kernel los ve como entidades independientes.
- Provee mejor rendimiento en cargas intensivas.
- El sistema operativo puede manejar la sincronización entre hilos.

Desventajas de los KLT:

- Los context switch se vuelven más costosos y frecuentes ya que requieren cambiar a modo kernel y guardar/recuperar más información.
- El kernel tiene que gestionar más estructuras por cada hilo, lo cual añade overhead.
- El planificador del SO decide la ejecución, así que no se puede personalizar tan fácilmente como con ULT.

#### 6. Qué retornan las siguientes funciones:

##### a. `getpid()`

- Es una systemcall que provee la libc.
- Retorna el process ID (PID) del proceso que llama a la función.

##### b. `getppid()`

- Es una systemcall que provee la libc.
- Retorna el process ID (PID) del proceso padre del que llama a la función.

##### c. `gettid()`

- Es una systemcall del kernel de linux.
- Retorna el thread ID (TID) del hilo que llama a la función.
  - En un proceso monohilo, el TID es igual al PID.
  - En un proceso multihilo, cada hilo tiene el mismo PID pero distinto TID.

##### d. `pthread_self()`

- Es una función de PThreads (librería que provee hilos KLT).
- Retorna el TID del thread actual a nivel PThreads en forma de un tipo **pthread_t**, el cual no necesariamente es el mismo que retorna `gettid()`.
  - Se usa para comparar hilos en PThreads.

##### e. `pth_self()`

- Es una función de PTh (librería que provee hilos ULT).
- Retorna el TID del thread actual a nivel PTh en forma de un tipo **pth_t**, el cual no necesariamente es el mismo que retorna `gettid()`.
  - Se usa para comparar hilos en PTh.

#### 7. ¿Qué mecanismos de sincronización se pueden usar? ¿Es necesario usar mecanismos de sincronización si se usan ULT?

Para sincronización se pueden usar:

- Mutex.
- Semáforos.
- Variables condición.
- Barreras.

Tanto en ULT como en KLT es necesario usar mecanismos de sincronización si nuestro programa tiene una o más variables que son manipuladas por varios hilos.

#### 8. Procesos

- `fork()` crea un nuevo proceso que es una copia exacta del proceso actual.
- `exec()` reemplaza el contenido del proceso actual (sus páginas) con un nuevo programa.
- `wait()` permite a un proceso esperar a que uno o más de sus procesos hijos terminen.

##### a. ¿Qué utilidad tiene ejecutar `fork()` sin ejecutar `exec()`?

Ejecutar `fork()` sin ejecutar `exec()` sería util cuando queremos que el proceso hijo ejecute una parte del mismo programa que el padre, pero realizando una tarea distinta en paralelo. Por ejemplo, si queremos procesar datos en paralelo o manejar múltiples conexiones en un servidor, podemos crear varios procesos hijos que ejecuten código distinto al del padre, pero dentro del mismo binario.

##### b. ¿Qué utilidad tiene ejecutar `fork()` + `exec()`?

Ejecutar `fork()` + `exec()` sería útil cuando queremos que el proceso hijo ejecute un programa completamente distinto al del proceso padre.

##### c. ¿Cuál de las 2 asigna un nuevo PID, `fork()` o `exec()`?

La función que asigna un nuevo PID es `fork()`, ya que ésta crea efectivamente un nuevo proceso. `exec()` no crea un nuevo proceso, por ende no asigna un nuevo PID.

##### d. ¿Qué implica el uso de Copy-On-Write (COW) cuando se hace `fork()`?

Cuando se llama a `fork()`, el SO no copia inmediatamente toda la memoria del proceso padre al hijo. Eso sería lento y muchas veces innecesario. En su lugar, usa una técnica llamada Copy-On-Write (COW).

Esta técnica consiste en:

- Padre e hijo comparten las mismas páginas de memoria al principio.
- Esas páginas se marcan como solo lectura.
- Si alguno de los dos intenta escribir en una página, recién ahí el SO:
  - Crea una copia privada de esa página solo para ese proceso.
  - Actualiza su tabla de páginas para que tenga su propia versión.

COW se usa porque en la mayoría de los casos se suele usar `exec()` luego de `fork()`, lo que reemplaza todo el espacio de memoria del proceso, por ende desperdiciando esa copia grande que se hubiera hecho al inicio si no se usa COW.

##### e. ¿Qué consecuencias tiene no hacer `wait()` sobre un proceso hijo?

Si un proceso padre no hace `wait()`, su proceso hijo puede convertirse en un proceso zombie. Esto ocurre porque, al finalizar, el hijo libera su memoria, pero su información de salida (PID, código de retorno, etc.) permanece en la tabla de procesos del sistema operativo. Esta información se conserva hasta que el padre llame a `wait()`, permitiendo al sistema limpiar completamente al hijo. Si no se hace, los zombies pueden acumularse y agotar la tabla de procesos del sistema.

##### f. ¿Quién tendrá la responsabilidad de hacer el `wait()` si el proceso padre termina sin hacer `wait()`?

Si un proceso padre termina sin hacer `wait()`, su proceso hijo queda huérfano y **es adoptado automáticamente por el proceso init**. Cuando ese hijo termina su ejecución, init realiza un `wait()` implícito para liberar sus recursos y evitar que se convierta en zombie.

Nota: Esto solo ocurre si el padre muere antes de que el hijo haya terminado. Si el padre sigue vivo pero no hace `wait()`, el hijo sí se convierte en zombie al finalizar.

Situaciones que pueden ocurrir:

- **El padre muere**: init lo adopta → el hijo no queda zombie.
- **El padre sigue vivo y no hace `wait()`**: el hijo queda zombie.
- **El padre hace `wait()`**: el hijo es limpiado correctamente.

#### 9. Kernel Level Threads

##### a. ¿Qué elementos del espacio de direcciones comparten los threads creados con `pthread_create()`?

Los hilos creados con `pthread_create()` comparten código, datos y heap, pero cada hilo tiene su propio stack.

##### b. ¿Qué relaciones hay entre `getpid()` y `gettid()` en los KLT?

Las funciones `getpid()` y `gettid()` **no se comportan exactamente igual** cuando las usamos en ULT vs KLT:

- En programas con ULT:

  - `getpid()` devuelve el mismo ID para todos los hilos del proceso.
  - `gettid()` también devuelve el mismo ID para todos los hilos ULT, ya que el kernel no tiene idea de estos hilos, son gestionados por la aplicación.

- En programas con KLT:
  - `getpid()` devuelve el mismo ID para todos los hilos del proceso.
  - `gettid()` devuelve un ID único para cada hilo kernel, permitiendo identificar individualmente cada KLT.

##### c. ¿Por qué `pthread_join()` es importante en programas que usan múltiples hilos? ¿Cuándo se liberan los recursos de un hilo zombie?

La función `pthread_join()` es importante en programas que usan múltiples hilos ya que realiza tres tareas fundamentales:

- Espera a que un hilo termine y recoge su valor de retorno.
- Libera los recursos del hilo (stack, estructuras del kernel).
- Evita que el hilo quede en estado zombie (finalizado pero no recolectado).

Los recursos de un hilo zombie se liberan cuando otro hilo llama a `pthread_join()` sobre él. Si no se hace, hay **fuga de recursos**.

##### d. ¿Qué pasaría si un hilo del proceso bloquea en `read()`? ¿Afecta a los demás hilos?

Si un hilo del proceso se bloquea en un `read()`, los demás hilos no se verán afectados. Esto se debe a que cada hilo es gestionado por el kernel con su propio contexto de ejecución, de forma concurrente, o paralela si tenemos un multicore.

##### e. Describí qué ocurre a nivel de sistema operativo cuando se invoca `pthread_create()` (¿es syscall? ¿usa clone?).

A nivel de SO, cuando se invoca `pthread_create()`, la cual es una función de espacio de usuario y **no una syscall**, ésta internamente invoca a la syscall `clone()` la cual puede recibir varios flags como:

- **CLONE_VM**: Comparte el espacio de memoria (mismo proceso)
- **CLONE_FS**: Comparte el sistema de archivos
- **CLONE_FILES**: Comparte descriptores de archivo
- **CLONE_SIGHAND**: Comparte manejadores de señales
- **CLONE_THREAD**: Indica que es un hilo (no un proceso independiente)

Esto permite que el nuevo hilo comparta recursos con el proceso padre (como memoria y archivos abiertos), pero tenga su propio stack y Thread ID (TID).

Luego de esto, el kernel asigna un nuevo Thread Control Block (TCB) y lo gestiona como una entidad planificable independiente, aunque pertenezca al mismo proceso.

Diferencia con `fork()`:

- `fork()` usa `clone()` sin los flags anteriores, resultando en un **proceso hijo con memoria independiente** (COW).
- `pthread_create()` (`vía clone()`) comparte memoria y recursos, creando un **hilo dentro del mismo proceso**.

#### 10. User Level Threads

##### a. ¿Por qué los ULTs no se pueden ejecutar en paralelo sobre múltiples núcleos?

Los ULTs no se pueden ejecutar en paralelo sobre múltiples núcleos porque el kernel no tiene conocimiento de su existencia: solo los gestiona la biblioteca de hilos en espacio de usuario. Por lo tanto, el scheduler del SO solo asigna CPU al proceso completo (un único KLT), sin distribuir los ULTs entre núcleos.

##### b. ¿Qué ventajas tiene el uso de ULTs respecto de los KLTs?

Usar ULTs en vez de KLTs tiene varias ventajas:

1. **Mayor eficiencia en la creación y gestión de hilos**: La creación, eliminación y cambio de contexto entre ULTs es mucho más rápida ya que no requiere intervención del kernel, evitando syscalls costosas.
2. **Menor sobrecarga**: Los ULTs tienen menos sobrecarga ya que se gestionan completamente en espacio de usuario, sin necesidad de cambiar entre modo usuario y modo kernel.
3. **Planificación personalizada**: Las aplicaciones pueden implementar sus propias políticas de planificación de hilos adaptadas a sus necesidades específicas, en lugar de depender de la política general del SO.
4. **Mejor portabilidad**: Los ULTs pueden implementarse en cualquier SO, incluso en aquellos que no soportan nativamente hilos a nivel de kernel.
5. **Control más preciso**: La aplicación tiene control absoluto sobre el comportamiento de los hilos, permitiendo implementaciones más específicas y optimizadas.
6. **Escalabilidad mejorada**: Se pueden crear miles de ULTs sin impactar significativamente los recursos del sistema, mientras que los KLTs están más limitados por los recursos del kernel.
7. **Menor impacto en bloqueos**: Si un ULT se bloquea, solo afecta a los hilos dentro del mismo proceso, mientras que con KLTs, podría afectar a la planificación global del sistema.

##### c. ¿Qué relaciones hay entre `getpid()`, `gettid()` y `pth_self()` (en GNU Pth)?

- `getpid()` retorna el PID del proceso actual. Este valor es compartido por todos los hilos del proceso, ya sean ULT o KLT.
- `gettid()` retorna el TID del hilo actual (hilo KLT).
- `pth_self()` retorna el TID del hilo actual en la librería Pth (hilo ULT).

##### d. ¿Qué pasaría si un ULT realiza una syscall bloqueante como `read()`?

Si un ULT realiza una syscall bloqueante como `read()`, todos los hilos ULT de ese proceso se bloquean hasta que termine la syscall.

##### e. ¿Qué tipos de scheduling pueden tener los ULTs? ¿Cuál es el más común?

En los ULT, es la biblioteca de hilos la que maneja el scheduling de éstos. Existen varios tipos de scheduling que este tipo de bibliotecas usan:

- **Cooperativo (el más común)**:
  - Non-preemptive (no apropiativo).
  - Cada hilo cede voluntariamente el control.
  - El scheduler no puede interrumpir a un hilo de forma arbitraria.
  - Puede provocar bloqueos.
- **Round-robin**:
  - Preemptive (apropiativo).
  - Los hilos se programan en órdenes circulares, donde cada uno obtiene la CPU por un determinado tiempo (quantum).
  - No se suele usar.

#### 11. Global Interpreter Lock

##### a. ¿Qué es el GIL (Global Interpreter Lock)? ¿Qué impacto tiene sobre programas multi-thread en Python y Ruby?

El GIL (Global Interpreter Lock) es un mecanismo de exclusión mutua (mutex) presente en implementaciones como CPython (Python) y MRI (Matz's Ruby Interpreter), que permite que **solo un hilo ejecute código del intérprete a la vez, incluso en programas multithread**.

Esto significa que, aunque un programa esté compuesto por múltiples hilos, solo uno de ellos puede ejecutar bytecode del lenguaje a la vez.

El impacto del GIL es especialmente notable en programas CPU-bound, ya que impide el aprovechamiento real de múltiples núcleos del procesador. En estos casos, el GIL crea un cuello de botella y limita la escalabilidad.

Para programas I/O-bound en cambio, el GIL tiene menos impacto, porque los hilos pueden liberar el GIL mientras esperan a que terminen las operaciones de entrada/salida que realizaron.

NOTA: El GIL es una limitación de los intérpretes más comunes de estos lenguajes, no de los lenguajes en sí. Tanto Ruby como Python poseen intérpretes alternativos que no tienen GIL.

##### b. ¿Por qué en CPython o MRI se recomienda usar procesos en vez de hilos para tareas intensivas en CPU?

En CPython (intérprete principal de Python) y en MRI (intérprete principal de Ruby), se recomienda usar procesos en vez de hilos para tareas intensivas en CPU debido al bottleneck que presenta el GIL, impidiendo la ejecución paralela real de múltiples hilos. Los procesos, en cambio, no comparten GIL (cada uno tiene el suyo, debido a que cada proceso tiene su propio intérprete), lo que permite que el SO los distribuya entre varios núcleos y se obtenga paralelismo verdadero y por ende mejor rendimiento.

### Práctica guiada

#### 1. Instale las dependencias necesarias para la práctica (strace, git, gcc, make, libc6-dev, libpth-dev, python3, htop y podman):

```
apt update
apt install build-essential strace git libpth-dev python3 python3-venv htop podman
```

#### 2. Clone el repositorio con el código a usar en la práctica:

```
git clone https://gitlab.com/unlp-so/codigo-para-practicas.git
```

#### 3. Resuelva y responda utilizando el contenido del directorio practica3/01-strace:

##### a. Compile los 3 programas C usando el comando make.

##### b. Ejecute cada programa individualmente, observe las diferencias y similitudes del PID y THREAD_ID en cada caso. Conteste en qué mecanismo de concurrencia las distintas tareas:

- Ejecutando `./01-subprocess`:

```
Parent process: PID = 1102, THREAD_ID = 1102
Child process: PID = 1103, THREAD_ID = 1103
```

- Ejecutando `./02-kl-thread`:

```
Parent process: PID = 1104, THREAD_ID = 1104
Child process: PID = 1104, THREAD_ID = 1105
```

- Ejecutando `./03-ul-thread`:

```
Parent process: PID = 1114, THREAD_ID = 1114, PTH_ID = 94419127625968
Child process: PID = 1114, THREAD_ID = 1114, PTH_ID = 94419127631584
```

###### i. Comparten el mismo PID y THREAD_ID

Las tareas tienen el mismo PID y mismo THREAD_ID en el programa **03-ul-thread**, ya que al usar User Level Threads el SO no tiene conocimiento de los ULTs.

###### ii. Comparten el mismo PID pero con diferente THREAD_ID

Las tareas tienen el mismo PID pero con diferente THREAD_ID en el programa **02-kl_thread**, ya que al usar Kernel Level Threads el SO asigna un TID distinto a cada hilo del proceso, pero todos poseen el mismo PID ya que son todos hilos del proceso en cuestión.

###### iii. Tienen distinto PID

Las tareas tienen distinto PID únicamente en el programa **01-subprocess**, ya que al hacer `fork()` se crea un nuevo proceso individual con su propio PID distinto al de su padre.

##### c. Ejecute cada programa usando strace (strace ./nombre_programa > /dev/null) y responda:

###### i. ¿En qué casos se invoca a la systemcall clone o clone3 y en cuál no? ¿Por qué?

- Ejecutando `strace ./01-subprocess > /dev/null` se invoca a la systemcall **clone**:
  - Esto se debe a que la instrucción `fork()` usada en este programa usa internamente a la systemcall clone para crear un nuevo proceso.

```
clone(child_stack=NULL, flags=CLONE_CHILD_CLEARTID | CLONE_CHILD_SETTID | SIGCHLD, child_tidptr=0x7f09895cea10) = 1177
```

- Ejecutando `strace ./02-kl-thread > /dev/null` se invoca a la systemcall **clone3**:
  - Esto se debe a que los KLT, como los que se crean con pthreads, son gestionados por el kernel. Para crearlos, la biblioteca de hilos (como glibc) utiliza internamente la systemcall **clone** o **clone3**, que permite compartir el espacio de direcciones y otros recursos entre hilos.

```
clone3({flags=CLONE_VM | CLONE_FS | CLONE_FILES | CLONE_SIGHAND | CLONE_THREAD | CLONE_SYSVSEM | CLONE_SETTLS | CLONE_PARENT_SETTID | CLONE_CHILD_CLEARTID, child_tid=0x7f80e31a990, parent_tid=0x7f80e312a990, exit_signal=0, stack=0x7f80e292a000, stack_size = 0x7fff80, tls=0x7f80e312a6c0} => {parent_tid=[1191]}, 88) = 1191
```

- Ejecutando `strace ./03-ul-thread > /dev/null` **NO se invoca a la systemcall clone ni clone3**:
  - Esto se debe a que los ULT son manejados exclusivamente en espacio de usuario por la librería de hilos (como Pth). No se usan systemcalls.

###### ii. Observe los flags que se pasan al invocar a clone o clone3 y verifique en qué caso se usan los flags CLONE_THREAD y CLONE_VM.

- Tanto la flag CLONE_THREAD como la flag CLONE_VM se usan al invocar **clone3** en el contexto de los KLT.

###### iii. Investigue qué significan los flags CLONE_THREAD y CLONE_VM usando la manpage de clone y explique cómo se relacionan con las diferencias entre procesos e hilos.

- **CLONE_THREAD** indica que el nuevo hilo pertenece al mismo grupo de hilos que el hilo principal (es decir, todos los hilos comparten PID de grupo).
- **CLONE_VM** indica que el nuevo hilo comparte el mismo espacio de direcciones que el hilo principal.

###### iv. `printf()` eventualmente invoca la syscall write (con primer argumento 1, indicando que el file descriptor donde se escribirá el texto es STDOUT). Vea la salida de strace y verifique qué invocaciones a write(1, ...) ocurren en cada caso.

- Ejecutando `strace ./01-subprocess > /dev/null`:

```
write(1, "Parent process: PID = 1277, THRE"..., 45) = 45
```

- Ejecutando `strace ./02-kl-thread > /dev/null`:

```
write(1, "Parent process: PID = 1290, THRE"..., 88) = 88
```

- Ejecutando `strace ./03-ul-thread > /dev/null`:

```
write(1, "Parent process: PID = 1298, THRE"..., 138) = 138
```

###### v. Pruebe invocar de nuevo strace con la opción -f y vea qué sucede respecto a las invocaciones a write(1, …). Investigue qué es esa opción en la manpage de strace. ¿Por qué en el caso del ULT se puede ver la invocación a write(1, …) por parte del thread hijo aún sin usar -f?

Por defecto, **strace** solo sigue las systemcalls del proceso principal del programa, y no de sus procesos hijos o de sus hilos. Si usamos la opción -f, strace trackea a todos los procesos hijos del programa, y a todos sus hilos.

- Ejecutando `strace -f ./01-subprocess > /dev/null`:

```
[pid  1459] write(1, "Parent process: PID = 1458, THRE"..., 89) = 89

write(1, "Parent process: PID = 1458, THRE"..., 45) = 45
```

- Ejecutando `strace -f ./02-kl-thread > /dev/null`:

```
write(1, "Parent process: PID = 1506, THRE"..., 88) = 88
```

- Ejecutando `strace -f ./03-ul-thread > /dev/null`:

```
write(1, "Parent process: PID = 1548, THRE"..., 138) = 138
```

En el caso del ULT se puede ver la invocación a `write(1, …)` por parte del thread hijo aún sin usar -f porque el "hilo" del ULT no es un hilo real del sistema. No se usa `clone()` ni `fork()`, por lo tanto el kernel no ve más de un proceso. Desde el punto de vista de strace, todo pasa en un solo PID, y por eso se ven todas las llamadas.

#### 4. Resuelva y responda utilizando el contenido del directorio practica3/02-memory:

##### a. Compile los 3 programas C usando el comando make.

##### b. Ejecute los 3 programas.

- Ejecutando `./01-subprocess`:

```
Parent process: PID = 1891, THREAD_ID = 1891
Parent process: number = 42
Child process: PID = 1892, THREAD_ID = 1892
Child process: number = 84
Parent process: number = 42
```

- Ejecutando `./02-kl-thread`:

```
Parent process: PID = 1911, THREAD_ID = 1911
Parent process: number = 42
Child thread: PID = 1911, THREAD_ID = 1912
Child process: number = 84
Parent process: number = 84
```

- Ejecutando `./03-ul-thread`:

```
Parent process: PID = 1949, THREAD_ID = 1949, PTH_ID = 94409932740848
Parent process: number = 42
Child thread: PID = 1949, THREAD_ID = 1949, PTH_ID = 94409932743392
Child thread: number = 84
Parent process: number = 84
```

##### c. Observe qué pasa con la modificación a la variable number en cada caso. ¿Por qué suceden cosas distintas en cada caso?

- **En 01-subprocess**:
  - El proceso principal crea a un proceso hijo y espera a que éste termine.
  - El proceso hijo multiplica por 2 una variable global del programa y la actualiza.
  - El proceso hijo imprime el nuevo valor.
  - Ya que el proceso hijo ahora terminó, el proceso original deja de esperar e imprime el valor del número también.
  - Explicación de por qué el proceso padre no ve la modificación que hizo el hijo:
    - `fork()` crea un nuevo proceso hijo que es una copia del proceso padre, incluyendo su espacio de direcciones (código, datos, variables globales, etc.).
    - Debido a la técnica de Copy-On-Write, el SO no copia inmediatamente toda la memoria, sino que ambos procesos comparten la misma memoria físicamente **hasta que uno de ellos la modifica**.
    - Cuando el proceso hijo modifica la variable global number, se genera una copia privada de esa página de memoria, y el cambio solo es visible para el hijo.
    - El padre sigue con su versión original de la variable number, que permanece en 42.
- **En 02-kl-thread**:
  - En este caso, cuando el hilo hijo modifica la variable global, el hilo padre ve esta modificación también.
  - Esto se debe a que cuando usamos Kernel Level Threads, los diferentes hilos del proceso todos comparten el mismo espacio de direcciones.
- **En 03-ul-thread**:
  - Ocurre lo mismo que en el programa anterior, todos los hilos ULT comparten el espacio de direcciones, y por ende la modificación que realiza el hilo hijo es visible para el hilo padre.

#### 5. El directorio practica3/03-cpu-bound contiene programas en C y en Python que ejecutan una tarea CPU-Bound (calcular el enésimo número primo).

##### a. Ejecute htop en una terminal separada para monitorear el uso de CPU en los siguientes incisos.

##### b. Ejecute los distintos ejemplos con make (usar make help para ver cómo) y observe cómo aparecen los resultados, cuánto tarda cada thread y cuanto tarda el programa completo en finalizar.

- Ejecutando `make run_klt`:

```

```

- Ejecutando `make run_ult`:

```

```

- Ejecutando `make run_klt_py`:

```

```

- Ejecutando `make run_ult_py`:

```

```

- Ejecutando `make run_klt_py_nogil`:

```

```

##### c. ¿Cuántos threads se crean en cada caso?

##### d. ¿Cómo se comparan los tiempos de ejecución de los programas escritos en C (ult y klt)?

##### e. ¿Cómo se comparan los tiempos de ejecución de los programas escritos en Python (ult.py y klt.py)?

##### f. Modifique la cantidad de threads en los scripts Python con la variable NUM_THREADS para que en ambos casos se creen solamente 2 threads, vuelva a ejecutar y comparar los tiempos. ¿Nota algún cambio? ¿A qué se debe?

##### g. ¿Qué conclusión puede sacar respecto a los ULT en tareas CPU-Bound?

#### 6. El directorio practica3/04-io-bound contiene programas en C y en Python que ejecutan una tarea que simula ser IO-Bound (tiene una llamada a sleep lo que permite interleaving de forma similar al uso de IO).

##### a. Ejecute htop en una terminal separada para monitorear el uso de CPU en los siguientes incisos.

##### b. Ejecute los distintos ejemplos con make (usar make help para ver cómo) y observe cómo aparecen los resultados, cuánto tarda cada thread y cuanto tarda el programa completo en finalizar.

##### c. ¿Cómo se comparan los tiempos de ejecución de los programas escritos en C (ult y klt)?

##### d. ¿Cómo se comparan los tiempos de ejecución de los programas escritos en Python (ult.py y klt.py)?

##### e. ¿Qué conclusión puede sacar respecto a los ULT en tareas IO-Bound?

#### 7. Diríjase nuevamente en la terminal a practica3/03-cpu-bound y modifique klt.py de forma que vuelva a crear 5 threads.

##### a. Ejecute htop en una terminal separada para monitorear el uso de CPU en los siguientes incisos.

##### b. Ejecute una versión de Python que tenga el GIL deshabilitado usando: `make run_klt_py_nogil` (esta operación tarda la primera vez ya que necesita descargar un container con una versión de Python compilada explícitamente con el GIL deshabilitado).

##### c. ¿Cómo se comparan los tiempos de ejecución de klt.py usando la versión normal de Python en contraste con la versión sin GIL?

##### d. ¿Qué conclusión puede sacar respecto a los KLT con el GIL de Python en tareas CPU-Bound?
