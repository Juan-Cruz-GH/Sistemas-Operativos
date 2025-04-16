<h1 align="center">Clase 1 - 12 de marzo, 2025</h1>

## Sistema Operativo

### Introducción

- Un sistema operativo es un **software** que hace de **intermediario entre el usuario y el hardware.**
- Gestiona/administra el hardware, pero también **necesita** al hardware para poder ejecutarse, ya que es software.
- Controla la ejecución de procesos: cuándo se ejecuta cada uno, cómo y por qué.
- Es una interfaz entre las aplicaciones y el hardware.

### Perspectivas:

#### Vista del usuario (top down)

- Esta perspectiva se centra en **cómo el usuario interactúa con el sistema operativo**.
- Se enfoca en el OS como una interfaz entre el usuario y el hardware.
- Los usuarios interactúan con servicios de alto nivel, como interfaces gráficas (GUI), gestión de archivos y ejecución de aplicaciones.
- El usuario ve el sistema operativo como un facilitador que administra los recursos y asegura una experiencia fluida sin necesidad de lidiar con las complejidades internas.
- El usuario se abstrae totalmente de la arquitectura de la computadora.
- Busca dar la mayor comodidad y friendliness.

#### Vista del sistema (bottom up)

- Esta perspectiva se centra en **cómo el sistema operativo gestiona el hardware y los procesos a bajo nivel**.
- Desde este ángulo, el sistema operativo se ve como un conjunto de componentes que controlan los recursos del hardware, como la CPU, la memoria y los dispositivos de entrada/salida (I/O).
- Involucra la gestión de procesos, la programación de tareas, el manejo de interrupciones y la asignación eficiente de recursos.

### Objetivos

- **Comodidad**: Hacer más simple el uso de la PC.
- **Eficiencia**: Hacer un uso más eficiente de los recursos del sistema.
- **Evolución**: Permitir la introducción de nuevas funciones al sistema sin interferir con funciones anteriores.

### Componentes

- Kernel.
- Shell (GUI o CLI).
- Herramientas (editores, compiladores, librerías, etc).

## Kernel

### Introducción

- Es una porción de código que se encuentra en gran parte en RAM.
- Se encarga de:
  - Manejar la memoria.
  - Manejar la CPU.
  - Administrar los procesos.
  - Comunicación y concurrencia.
  - Gestionar la E/S.
- Es de código abierto (en Linux).
- Funciona para todas las arquitecturas de hardware existentes.
- Usa licencia GPLv2.
- Estrictamente, "ES" el sistema operativo en sí.
- Utiliza las System Calls.
- Los procesos son **clientes** del SO.
- Escrito en C, Assembler y Rust (módulos).

### Modos de ejecución

- Los procesos se pueden ejecutar en modo privilegiado o modo usuario.
- En modo privilegiado se tiene acceso al conjunto entero de instrucciones, lo que permite acceder al hardware de forma irrestricta, direccionar la memoria, programar la CPU, etc.
- Los procesos de usuario comunes se ejecutan en modo usuario.
- El Kernel se ejecuta en modo privilegiado.
- Cuando un proceso necesita acceder al hardware, se lo pide al Kernel vía una System Call.
- Interrupción de Clock: Se debe evitar que un proceso se apropie de la CPU.
- Protección de la memoria: Se deben definir límites de memoria a los que puede acceder cada proceso (registros base y límite)
- El bit en la CPU indica el modo actual (privilegiado o usuario).
- Las instrucciones privilegiadas solo se pueden ejecutar si el bit está en modo privilegiado.
- En modo usuario, el proceso puede acceder **solo a su espacio de direcciones**, no el de otros procesos ni el del kernel.
- Cuando arranca la PC, el bit está en modo privilegiado. Cuando se comienza a ejecutar un proceso de usuario, se cambia el bit a modo usuario vía una interrupción.

### Tipos de Kernel

#### Monolítico

- Todos los servicios del SO están en un **único bloque de código enorme** que se ejecuta en modo privilegiado.
- Posee distintos subsistemas y la funcionalidad de cada uno es accedida directamente desde otro a través de funciones públicas.
- Provee gestión completa de recursos (memoria, CPU, E/S).
- Posee **acceso total al hardware** (más eficiente al no requerir cambios de modo mientras se ejecuta).
- Maneja las excepciones e interrupciones de forma directa.
- Es **complejo de administrar y mantener**.
- Ejemplo: Linux, Unix.

#### Microkernel

- Minimiza la cantidad de código que se ejecuta en modo privilegiado con el fin de hacerlo más liviano.
- Incluye funciones más esenciales, como la gestión de procesos, comunicación entre procesos y gestión de memoria. Otros servicios (como controladores de hardware, sistemas de archivos y red) se ejecutan en modo usuario.
- Es altamente modular permitiendo su personalización a través de la adición o eliminación de módulos de funcionalidad.
- Dado que la mayoría de los servicios se ejecutan en modo usuario, suele ser más **estable** y **seguro** que los monolíticos.
- Provee un **rendimiento inferior** por los cambios de modo constantes que requiere para su ejecución.
- Su desarrollo y mantenimiento es más eficiente.
- Ejemplo: Minix.

#### Híbrido

- Combina características del monolíticos y el microkernel.
- Tiene un núcleo más pequeño que un monolítico. Incluye algunos servicios en modo núcleo (eficiencia) y otros se ejecutan en modo usuario (seguridad).
- Son modulares. Ofrecen un **equilibrio** entre rendimiento y modularidad lo que permite actualizaciones más sencillas y una mayor flexibilidad y facilidad a su desarrollo.
- Ofrecen un **mejor rendimiento** que los microkernels y **mayor modularidad/seguridad** que los monolíticos.
- Suelen ser una alternativa equilibrada y atractiva.
- Ejemplo: Windows.

#### ExoKernel

- Provee servicios muy **básicos** al sistema operativo, evitando alto nivel de abstracción (como hilos o archivos).
- Ofrece acceso directo a recursos del hardware, permitiendo a los
  desarrolladores construir soluciones **personalizadas**.
- Provee primitivas básicas para la gestión de memoria y deja las políticas de gestión de memoria a bibliotecas en espacio de usuario.
- Ofrece primitivas para que las aplicaciones gestionen sus propios procesos.
- Los controladores se ejecutan en espacio de usuario.
- La mínima cantidad de código reduce los vectores de ataque y los errores, lo que hace a este tipo de kernel ideal para entornos especializados como sistemas en tiempo real o embebidos.

#### Kernel de Tiempo Real

- Debe garantizar que las operaciones se completen dentro de **marcos de tiempo conocidos y predecibles**.
- Utiliza algoritmos de scheduling de CPU avanzados para asegurar que las tareas se ejecuten en el **orden correcto** y cumplir con los plazos establecidos.
- Implementa mecanismos de asignación de memoria estática o dinámica optimizados para prevenir retrasos indeseados, evitando deseablemente los fallos de páginas.
- Soporta multitasking apropiativo donde las tareas de alta prioridad suspenden a las de menor prioridad.
- Provee mecanismos de seguridad robustos.

---

<h1 align="center">Clase 2 - 19 de marzo, 2025</h1>

## Kernel

### Desarrollo

- El desarrollo del Kernel de GNU/Linux es colaborativo: participan empresas, universidades y devs independientes.
- Su ciclo de desarrollo es: sprint de 3 a 4 meses → merge window de 1 a 2 semanas → ventana de corrección de bugs → próximo sprint.
- En la merge window se discute el agregado de nuevos features.
- En la merge window van apareciendo "rc"s o release candidates.
- Linus Torvalds mantiene el Kernel de GNU/Linux y aprueba o no los merge requests.

### Subsistemas

- El Kernel de GNU/Linux se divide en muchos subsistemas debido a lo enorme que es.
- Cada subsistema es mantenido por uno o más responsables.
- Estos responsables aceptan o no patches o pull requests de los devs. Luego interactúan con Linus para incluir los cambios en una versión rc o no.
- Cada responsable tiene su propio tree de git para el desarrollo.

## Historia de Linux

### Versiones

- En **1991** Linus empieza a programar el Kernel Linux basado en Minix (Clon de Unix desarrollado por Tanenbaum en 1987).
- En **octubre de ese año**, se anuncia la primer versión oficial de Linux: 0.02.
- En **1992**, con la release de la 0.12, se decide cambiar a una licencia GNU.
- En **marzo de 1994** Linus considera que todos los componentes del Kernel estaban lo suficientemente maduros y lanza la versión 1.0.
- En **1995** Linux se porta a arquitecturas DEC Alpha y Sun SPARC. Con el correr de los años se portó a otra decena de arquitecturas.
- En **mayo de 1996** se decide adoptar a Tux como mascota oficial de Linux.
- En **julio de 1996** se lanza la versión 2.0 y se define un sistema de nomenclatura.
- Se desarrolló hasta **febrero de 2004** y terminó con la versión 2.0.40. Esta versión comenzó a brindar soporte a sistemas multiprocesadores.
- En **2001** se lanza la versión 2.4 y se deja de desarrollar a fines del 2010 con la 2.4.37.11. **La versión 2.4 fue la que catapultó a GNU/Linux como un sistema operativo estable y robusto**.
- A **fines de 2003** se lanza la versión 2.6. Esta versión mejoró mucho el Kernel: soporte de threads, mejoras de scheduling, soporte de nuevo hardware.
- El **3 de agosto de 2011** se lanza la versión 2.6.39.4, la última de Linux 2.X.
- El **17 julio de 2011** se lanza la versión 3.0.
  - No agrega mayores cambios.
  - Se lanzó como celebración por 20 años de Linux.
  - Es totalmente compatible con Kernel 2.6.
  - Provee mejoras en Virtualización y FileSystems.
- El **12 de abril de 2015** se lanza la versión 4.0.
  - Una de sus principales mejoras es la posibilidad de aplicar parches y actualizaciones sin necesidad de reiniciar el SO.
  - Soporte para nuevas CPU.
- El **3 de marzo de 2019** se lanza la versión 5.0.
  - Soporte de energy-aware scheduling:
    - Pensado idealmente para minimizar el consumo energético en smartphones.
    - Utiliza decisiones de consumo de energía para hacer la planificación.
  - Soporte de namespaces para binderfs que permite ejecutar múltiples instancias de Android.
  - Soporte de archivos swap en BTRFS.
  - Kernel Lockdown Mode previene el acceso de procesos de usuario (incluso root) a memoria del kernel. (Habilitado por defecto en modo Secure Boot de EFI) - 5.4 (24 Noviembre 2019).
  - Soporte inicial de USB 4 - 5.6 (29 de Marzo 2020).
  - Nuevo mecanismo para manejar systemcalls de Windows para programas como Wine - 5.11 (14 de Febrero 2021).
  - Landlock security module: Permite restringir las acciones que un conjunto de programas puede ejecutar en un filesystem de forma simple - 5.13 (27 de Junio de 2021).
  - Nueva system call para crear áreas de memoria secretas que no pueden ser accedidas ni por el usuario root (por ejemplo para guardar claves) - 5.14 (29 de Agosto de 2021).
  - Soporte opcional para paquetes IPv6 de más de 64KB - 5.19 (31 de Julio de 2021).
- El **2 de octubre de 2022** se lanza la versión 6.0.
  - Primer release que soporta módulos escritos en Rust - 6.1 (11 de Diciembre de 2022).
  - Mejoras en la confiabilidad de la implementación de RAID5/6 en Btrfs - 6.2 (19 de Febrero 2023).
  - Mejoras de rendimiento y fragmentación para Btrfs 6.3 (23 de Abril 2023).
  - Soporte inicial para USB4.0 v2 - 6.5 (27 de Agosto 2023).
  - Nuevo scheduler de tareas (EEVDF) que mejora el fairness respecto al algoritmo anterior (CFS) Linux 6.6 (29 de Octubre de 2023).
  - Soporte para un nuevo Filesystem BCachefs con características similares a ZFS y Btrfs - 6.7 (7 de Enero de 2024).
  - Optimización de las estructuras de datos usadas para networking que mejora la performance de TCP cuando hay muchas conexiones concurrentes en hasta un 40% - 6.8 (10 de Marzo de 2023).

### Versionado previo a la versión 2.6 - X.Y.Z

- Antes de esta versión, el versionado seguía la siguiente anatomía (X.Y.Z):
- X: Serie principal. Cambiaba al agregar o quitar una funcionalidad muy importante.
- Y: Versión de producción (número par) o versión de desarrollo (número impar).
- Z: Bugfixes.

### Versionado entre la versión 2.6 a 3.0 - A.B.C.[D]

- Entre estas versiones, el versionado seguía la siguiente anatomía (A.B.C.D):
- A: Denota Versión. Cambia con menor frecuencia (cada varios años).
- B: Denota revisión mayor.
- C: Denota revisión menor. Solo cambia cuando hay nuevos drivers o caracterı́sticas.
- D: Se utiliza cuando se corrige un grave error sin agregar nueva funcionalidad.

### Versionado posterior a la versión 3.0 - A.B.C[-rcX]

- En el año 2011, cuando se pasó de la versión 2.6.39 a la 3.0 se volvió a un esquema de 3 números:
- A: Denota revisión mayor. Cambia con menor frecuencia (cada varios años).
- B: Denota revisión menor. Solo cambia cuando hay nuevos drivers o caracterı́sticas.
- C: Número de revisión.
- rcX: Versiones de prueba.

## Compilación del Kernel

### Motivación

- El Kernel se puede recompilar por varias razones:
  - Soportar nuevos dispositivos como por ejemplo una placa de video.
  - Agregar mayor funcionalidad (soporte de un nuevo filesystem).
  - Optimizar funcionamiento de acuerdo al sistema en que corre.
  - Adaptarlo al sistema donde corre (quitar soporte de hardware no usado).
  - Corregir bugs.

### Qué se necesita?

- **gcc**: Compilador de C.
- **make**: Ejecuta las directivas definidas en el makefile.
- **binutils**: Assembler, linker.
- **libc6**: Archivos de encabezados y bibliotecas de desarrollo.
- **ncurses**: Bibliotecas de menú de ventanas (solo si se usa menuconfig).
- **initrd-tools**: Herramientas para crear discos RAM.

### Proceso de compilación del Kernel

1. Obtener el source code → http://www.kernel.org.
2. Preparar el árbol de archivos del source code.
3. Configurar el kernel.
4. Construir el kernel a partir del source code e instalar los módulos.
5. Reubicar el kernel.
6. Creación del initramfs.
7. Configurar y ejecutar el gestor de arranque (GRUB en general).
8. Reiniciar el sistema y probar el nuevo kernel.

---

<h1 align="center">Clase 3 - 26 de marzo, 2025</h1>

## Llamadas al Sistema

### Definición

- Una llamada al sistema o system call/syscall es el mecanismo usado por un proceso de usuario para solicitarle al SO un **servicio**, ya que éstos son protegidos por el mismo.
- En este sentido, se puede ver al SO como un **servidor** y a los procesos de usuario como los **clientes** que realizan peticiones al SO vía system calls.
- Concretamente, las syscalls son métodos que el SO provee vía una API.
  - Por esto, cada método recibe una cierta cantidad de parámetros de determinado tipo y tiene un valor de retorno.
- En Unix, GNU/Linux la API que define estos mecanismos se llama libc.

### Flujo de ejecución

Por ejemplo, si usamos la función `read()` en C:

1. Se agregan a la pila los parámetros enviados en la llamada al `read()` de la librería desde nuestro código.
2. Se invoca a la función `read()` implementada en la librería y se comienza su ejecución.
3. Dentro de la librería, `read()` es una función simple que solo indicará el número de syscall que se quiere ejecutar y permitirá realizar la llamada al sistema correspondiente.
4. Esta función ejecuta el TRAP (Interrupción por Software) para cambiar a modo Kernel y pasarle el control al SO.
5. Para todas las syscalls se usa la misma interrupción.
6. La forma de identificar a la syscall invocada es a través del valor del registro.
7. Una vez que el SO tiene el control, verificará cuál es la llamada al sistema que debe atender y ejecutará el código correspondiente.
8. En este punto se accede al dispositivo de almacenamiento para obtener el archivo solicitado y leerlo en memoria

### Registros

- **EAX**: número de syscall.
- EBX: primer parámetro.
- ECX: segúndo parámetro.
- EDX: ...
- La instrucción que inicia la system call: int 80h (32 bits, syscall en 64 bits).

### Categorías

Existen varias categorías de syscalls según su propósito:

- Control de procesos (`fork()`, `waitpid()`, `execve()`, `exit()`).
- Manejo de archivos (`òpen()`, `close()`, `read()`, `write()`, `lseek()`, `stat()`).
- Manejo de dispositivos.
- Mantenimiento de info del sistema.
- Comunicaciones.

NOTA: Los nombres semánticos de las systemcalls son distintos entre UNIX y Win32.

### Características de las syscalls en GNU/Linux

- Son identificadas unívocamente por un número.
- Pueden tener 6 parámetros como máximo.
- Para x86 en 32 bits están definidas en `arch/x86/entry/syscalls/syscall_32.tbl`.
- Para x86 en 64 bits están definidas en `arch/x86/entry/syscalls/syscall_64.tbl`.
- La primer tarea que realiza el dispatcher cuando se produce una interrupción es verificar el número en la tabla correspondiente y ejecutar las funciones asociadas.

![Tabla](https://i.imgur.com/DeAbe1q.png)

### Cuidados con los parámetros

- Los parámetros de la syscall deben manejarse con mucho cuidado, dado que **se configuran en el espacio de usuario**:
  - No se puede asumir que sean correctos.
  - En el caso de pasarse punteros, no pueden apuntar al espacio del Kernel por cuestiones de seguridad (de no verificarse, en un read por ejemplo el buffer podría tener una dirección del Kernel y sobrescribir datos sensibles).
  - Los punteros deben ser siempre válidos (que apunten a direcciones que existen), si no, podría producirse un Kernel Panic.
  - El Kernel deberá tener acceso al espacio de usuario con APIs especiales que garanticen que se accede al espacio de direcciones de quien invocó la syscall (`get_user()`, `put_user()`, `copy_from_user()`, `copy_to_user()`).

---

<h1 align="center">Clase 4 - 9 de abril, 2025</h1>

## Hilos

### Manejo de procesos en sistemas operativos antiguos

- El proceso es la unidad básica de uso de CPU y representa a un programa en ejecución.
- Es también la unidad de asignación de los recursos: a los procesos se les asigna CPU, memoria, dispositivos.
- Cada proceso tiene:
  - Su espacio de direcciones propio y privado.
  - Punteros a los recursos asignados (stacks, archivos, etc).
  - Estructuras como la PCB.
  - **Un solo hilo de control**:
    - Un único flujo secuencial de ejecución.
    - Se ejecuta una instrucción y cuando finaliza se ejecuta la siguiente.
- Para ejecutar otro proceso, se debe llevar adelante un context switch.

### Evolución del hardware

- **Sistemas Dual-processor (DP)**:
  - Tiene 2 procesadores físicos en el mismo chasis.
  - Pueden estar en la misma motherboard o no.
  - Cache y controlador independientes.
- **Sistemas Dual-core**:
  - Una CPU con dos cores por procesador físico.
  - Un circuito integrado tiene 2 procesadores completos.
  - Los 2 procesadores comparten cache y controlador.
- En ambos casos, las APIC (Advanced Programmable Interrupt Controllers) están separadas por procesador. De esta manera proveen administración de interrupciones por procesador.

#### Multithreading Simultáneo

- También conocido como Hyper Threading, la implementación de Intel de este concepto.
- Permite que el software programado para ejecutar múltiples hilos (multi-threaded) procese los hilos en paralelo dentro de un único procesador.
- Simula dos procesadores lógicos dentro de un único procesador físico.
  - Duplica solo algunas “secciones” de un procesador:
    - Registros de Control (MMU, Interrupciones, Estado, etc).
    - Registros de Propósito General (AX, BX, PC, Stack, etc).
- Resultado: mejora en el uso del procesador de entre 20 y 30%.

### Evolución del software

- Es común dividir un proceso en diferentes “tareas” que, independientemente o colaborativamente, solucionan el problema.
- Es común contar con un pool de procesadores para ejecutar nuestros procesos de forma simultánea.
- Debido a que el hardware evolucionó, esto forzó al software a evolucionar también para poder aprovechar al máximo a los multicore.
- Como resultado, se incrementa el rendimiento de aplicaciones y se evitan bloqueos.
- Un ejemplo típico de esta evolución es la librería pthreads en C que nos permite realizar programas multihilados, pero además de esta librería:
  - Java: heredar de “Thread”, implementar la interface “Runnable”.
  - Delphi: Heredar de “TThread”.
  - C#, C, etc.
  - Ruby: Thread.new{CODIGO}.
  - PHP: Heredar de Thread.
  - Javascript: HTML5 Web Workers.
  - Etc.

### Manejo de procesos e hilos en sistemas operativos actuales

- Actualmente, tenemos tanto procesos como hilos.
- **Proceso**:
  - Espacio de direcciones.
  - Unidad de propiedad de recursos.
  - Conjunto de threads, uno o más.
- **Thread**:
  - Unidad de trabajo → hilo de ejecución.
  - Contexto del procesador.
  - Stacks de usuario y de kernel.
  - Variables propias.
  - Acceso a memoria y a recursos del proceso.
- Motivación: Por qué dividir una aplicación en threads?
  - Para obtener paralelismo/ejecución en background y así mejorar los tiempos de respuesta y la usabilidad.
  - Para aprovechar las ventajas de los multicore: con N CPUs pueden ejecutarse N hilos al mismo tiempo.
  - A su vez, hacer esto trae sus complejidades:
    - Sincronización.
    - Escalabilidad: cantidad de threads, excesivos context switch de los hilos del mismo proceso, etc.
- 3 procesos monohilo vs un proceso con 3 hilos:

![3 procesos monohilo vs un proceso con 3 hilos](https://i.imgur.com/UzGfqWk.png)

- Uso de hilos para aprovechar tiempos muertos:

![Uso de hilos para aprovechar tiempos muertos](https://i.imgur.com/kRm9rBb.png)

### Estructura de un hilo

Cada hilo tiene:

- Un **estado** de ejecución.
- Un **contexto** de procesador.
- **Stacks** (uno en modo usuario y otro en modo kernel).
- Variables propias.
- Acceso a memoria y recursos del proceso:
  - Archivos abiertos.
  - Señales.
  - Código.
  - Todos estos datos se comparten entre todos los hilos del proceso.
- TCB (Thread Control Block).
- Es la unidad básica de utilización de CPU.

### Ventajas del uso de hilos

| Característica | Procesos                                                                                                                                           | Hilos                                                                                                                                                            |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Context switch | El SO debe intervenir con el fin de salvar el ambiente del proceso saliente y recuperar el ambiente del nuevo.                                     | El cambio de contexto solo se realiza a nivel de registros y no espacio de direcciones. Lo lleva a cabo el proceso sin necesidad de intervención del SO.         |
| Creación       | Implica la creación de un nuevo espacio de direcciones, PCB, PC, etc. Lo lleva a cabo el SO.                                                       | Implica la creación de una TCB, registros, PC y un espacio para el stack. Lo hace el mismo proceso sin intervención del SO.                                      |
| Destrucción    | El SO debe intervenir con el fin de salvar el ambiente del proceso saliente y eliminar su PCB.                                                     | La tarea se realiza dentro del proceso sin necesidad de intervención del SO.                                                                                     |
| Planificación  | Es llevada a cabo por el sistema operativo. El cambio implica cambios de contexto continuos.                                                       | Es responsabilidad del desarrollador quien debe planificar sus hilos. Es menos costoso, pero puede traer desventajas aparejadas.                                 |
| Protección     | El SO garantiza la protección a través de distintos mecanismos de seguridad. La comunicación entre ellos implica el uso de técnicas mas avanzadas. | La protección debe darse desde el lado del desarrollo. Todos los hilos comparten el mismo espacio de direcciones. Un hilo podría bloquear la ejecución de otros. |

### User Level Thread (ULT)

- El programa, en modo usuario, se encarga de la gestión de los hilos por medio de una librería de threading que provee primitivas para crear, destruir, planificar, etc, a los hilos.
- El kernel no se entera de estos threads, son invisibles para él.
- Ejemplos:
  - Java VM.
  - POSIX Threads.
  - Solaris Threads.
- **Ventajas ✅**:
  - Todos los hilos del proceso comparten el espacio de direcciones.
  - Cada proceso planifica sus hilos como más le convenga.
  - Podrían reemplazarse llamadas al sistema bloqueantes por otras que no bloqueen.
  - Portabilidad: pueden correr en distintas plataformas.
  - No requiere cambios para su “existencia”.
  - No es necesario que el SO soporte hilos.
- **Desventajas ❌**:
  - No se puede ejecutar hilos del mismo proceso en distintos procesadores.
  - Si un hilo produce un Page Fault, todo el proceso se bloquea.
  - Un hilo podría monopolizar el uso de la CPU por parte del proceso.
  - Bloqueo del proceso durante una System Call bloqueante.

### Kernel Level Thread (KLT)

- La gestión completa de los hilos se realiza en modo Kernel.
- Ejemplos:
  - Windows NT/2000.
  - Linux.
- **Ventajas ✅**:
  - Se puede multiplexar hilos del mismo proceso en diferentes procesadores.
  - Independencia de bloqueos entre Threads de un mismo proceso.
- **Desventajas ❌**:
  - Cambios de modo de ejecución para la gestión (planificación, creación, destrucción, etc).

### Combinaciones

- Se puede combinar a los ULT con los KLT de muchas formas.
- En este tipo de sistemas, la creación de hilos se realiza a nivel de usuario y éstos son mapeados a una cantidad igual o menor de KLT.
- La sincronización de hilos en este modelo permite que un hilo se bloquee y otros hilos del mismo proceso sigan ejecutándose.
- Permite que hilos de usuario mapeados a distintos KLT puedan ejecutarse en distintos procesadores.
- Este enfoque aprovecha las ventajas de ambos tipos de hilos.
- Hay 3 tipos:
  - Uno a uno.
  - Muchos a uno.
  - Muchos a muchos.

#### Uno a uno (1:1)

- Cada ULT se mapea con un KLT.
- Cada vez que se necesita un ULT se debe crear un KLT → Introduce un costo alto.
- Si se bloquea un ULT, otro hilo del proceso puede seguir ejecutándose.
- La concurrencia y/o paralelismo es máximo, ya que cada hilo puede correr en un procesador distinto.
- Ejemplos:
  - Implementaciones UNIX tradicionales.

#### Muchos a uno (M:1)

- Muchos ULT mapean a un único KLT.
- Usado en sistemas que no soportan KLT.
- Si se bloquea un ULT, se bloquea todo el proceso.
- Java sobre un sistema que no soporta KLT.
- Ejemplos:
  - Windows NT.
  - Solaris.
  - Linux.
  - OS/2.
  - OS/390.
  - MACH.

#### Muchos a muchos (M:N)

- Muchos ULT mapean a muchos KLT.
- Este modelo multiplexa los ULT en KLT, logrando un balanceo razonable:
  - No tiene el costo del modelo 1:1.
  - Minimiza los problemas de bloqueo del modelo M:1.
- Ejemplos:
  - TRIX.

---

<h1 align="center">Clase 5 - 16 de abril, 2025</h1>

## Virtualización

---

<h1 align="center">Clase 6 - 23 de abril, 2025</h1>

##

---

<h1 align="center">Clase 7 - 30 de abril, 2025</h1>

##
