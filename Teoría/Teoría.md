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

### Concepto

- Técnica que permite realizar una abstracción de los recursos de una computadora: Es una capa de abstracción sobre el hardware para obtener una mejor utilización de los recursos y flexibilidad.
- Es una capa abstracta que desacopla el hardware físico del sistema.
- Permite ocultar detalles técnicos a través de encapsulación.
- Permite, entre otras cosas, que una computadora pueda realizar el trabajo de varias, a través de la compartición de recursos de un único dispositivo de hardware.
- Permite que haya múltiples máquinas virtuales (VM), o entornos virtuales (EV), con distintos (o iguales) sistemas operativos corriendo de manera aislada.
- Cada VM tiene su propio conjunto de hardware virtual (RAM, CPU, NIC, etc.) sobre el cual se ejecuta el SO “guest”.
- El SO “guest” ve un conjunto consistente de hw, no el hardware real (aunque a través de ciertas configuraciones podría ver parte del hardware real).
- Las VMs se representan y son encapsuladas en archivos dentro del filesystem.
- Fácil de almacenar, copiar.
- Fácil de hacer backup y restaurar.
- Simple de expandir y agregar recursos.
- Sistemas completos (aplicaciones ya configuradas, SO, hardware virtual) pueden moverse de un servidor a otro rápidamente.

### Características esenciales de una máquina virtual

- **Equivalencia / Fidelidad**:
  - Un programa ejecutándose sobre un VMM debería comportarse de forma idéntica a si se estuviera ejecutando directamente sobre el hardware subyacente.
- **Control de recursos / Seguridad**:
  - El VMM tiene que controlar completamente y en todo momento el conjunto de recursos virtualizados que proporciona a cada guest.
- **Eficiencia / Performance**:
  - Una fracción estadísticamente dominante de instrucciones tienen que ser ejecutadas sin la intervención del VMM, o en otras palabras, directamente por el hardware.

### Tipos de virtualización

- **Process Level**:
  - Permite lograr portabilidad entre diferentes sistemas.
  - Java Virtual Machine.
- **Storage Level**:
  - Presenta una vista lógica del almacenamiento al usuario.
  - RAID, LVM, etc.
- **Network Level**:
  - Integra recursos de hardware de red con recursos de software.
- **OS Level**:
  - Permite la existencia de varias instancias de espacio de usuario aisladas (containers).
- **System Level**:
  - Permite la creación de máquinas virtuales.

### Motivación

Podemos virtualizar por muchas razones:

- Tengo muchas máquinas servidores, poco usados.
- Tengo que correr aplicaciones heredadas (legacy) que no pueden ejecutarse en hardware o SOs actuales.
- Tengo que probar aplicaciones no seguras.
- Tengo que crear un SO o entorno de ejecución con recursos limitados.
- Tengo que simular la computadora real, pero con un subconjunto de recursos
- Necesito usar un hardware que no tengo (necesito “crear la ilusión” de hardware).
- Necesito simular redes de computadoras independientes.
- Tengo que correr varios y distintos SO simultáneamente.
- Necesito hacer testeo y monitoreo de performance.
- Necesito que SOs existentes se ejecuten en ambientes multiprocesador que comparten memoria.
- Necesito facilidad de migración.
- Necesito ahorrar energía (tendencias de green IT o tecnología verde).

### Software Host y Guest

- El software host es el que simula.
- El software guest es lo que se quiere simular, que puede ser un sistema operativo.

### Monitor de máquinas virtuales (VMM)

- También conocido como Hipervisor.
- Es una porción de software que separa a las aplicaciones y al SO del hardware subyacente.
- Provee una plataforma de virtualización que permite múltiples SO corriendo en un host al mismo tiempo.
- Programa que se corre sobre el hardware para implementar las máquinas virtuales.
- Se encarga de controlar los recursos y de la planificación de los guests.
- Necesita ejecutarse en modo supervisor.
- El software guest se ejecuta en modo usuario.
- Las instrucciones privilegiadas en los guests implican traps al VMM.
- El VMM interpreta/emula las instrucciones privilegiadas.

### Tipos de instrucciones

- **Inocuas o no privilegiadas**: Se ejecutan nativamente.
- **Privilegiadas**: Provocan una interrupción al ser ejecutadas en modo usuario.
- **Sensibles**: Se deben ejecutar en modo kernel.
- Para construir un VMM alcanza con que todas las instrucciones que podrían afecta al correcto funcionamiento del VMM (instrucciones sensibles) siempre generen una excepción y pasen el control al VMM.
- Las instrucciones no privilegiadas deben ejecutarse nativamente en el hardware (es decir, eficientemente).
- Para que un sistema soporte virtualización, las instrucciones sensibles deben ser un subconjunto de las privilegiadas.
- Cuando estoy virtualizando, el guest (corriendo en modo usuario) emitirá una instrucción privilegiada que NO debe ser ignorada, sino que debe generar un trap al SO.
  - Mecanismo conocido como trap-and-emulate.
  - No aplicable en arquitectura x86.

### Mecanismo Trap And Emulate

- Funciona similar a la emulación pero realiza una interpretación selectiva.
- Las aplicaciones y el SO se ejecutan en modo usuario.
- Aplicaciones ejecutan nativamente en el hardware.
- VMM ejecuta en modo privilegiado.
- Cuando se ejecuta una instrucción privilegiada en el guest SO (en modo usuario) se produce un “trap” al VMM.
- VMM ejecuta las instrucciones necesarias y retorna el control el guest SO.
- No puede ser utilizado en todas las ISAs. Debe cumplir con el teorema de Popek and Goldberg.
- x86 no cumple con el teorema: **no todas las instrucciones sensitivas son privilegiadas** (por ej. popf).

### Tipos de hipervisores

#### Tipo 1

- Se ejecuta en modo kernel.
- Se ejecuta sobre el hardware.
- Cada VM se ejecuta como un proceso de usuario en modo usuario.
- El SO guest no requiere ser modificado.
- Existen un modo kernel virtual y modo usuario virtual.
- El SO guest se ejecuta en modo kernel virtual.
- Siempre que la VM ejecuta una instrucción sensible, se produce una trap que procesa el hipervisor.
  - Algunos hipervisores introducen extensiones que le evitan tener que traducir todas las instrucciones.
- Debe tener asistencia del hardware siempre.

#### Tipo 2

- Se ejecuta como un programa de usuario sobre un SO host.
- Arriba de él están los SO guests.
- Interpreta un conjunto de instrucciones de máquina.
- El SO host es quien se ejecuta sobre el hardware.
- Su función principal de interpretar un subconjunto de las instrucciones de hardware de la máquina sobre la que corre.
- Debe emularse el hardware que se mapea a los SO guest.

### Técnicas de virtualización

#### Emulación

- Provee toda la funcionalidad del procesador deseado a través de software (ej: QUEMU, MAME).
- Se puede emular un procesador sobre otro tipo de procesador.
- Aplicación/SO emulado ejecuta en modo usuario.
- Se reescribe el conjunto completo de instrucciones.
- Todas las instrucciones son capturadas por el emulador.
- Cada instrucción es interpretada y traducida a una (o varias) equivalente adecuada al hardware subyacente.
- Tiende a ser lenta.

#### Full Virtualization

- Se trata de particionar un procesador físico en distintos contextos, donde cada uno de ellos corre sobre el mismo procesador.
- Los SO guest deben ejecutar la misma arquitectura de hardware sobre la que corren.
- No requiere que los guest se modifiquen.
- Es en general la técnica mas utilizada.
- El VMM analiza el flujo de ejecución.
  - Los bloques que contienen instrucciones sensibles son modificados.
  - Los bloques con instrucciones inocuas se ejecutan directamente en el hardware.
- Se combina traducción binaria con ejecución directa.
- Instrucciones no sensibles ejecutan directamente sobre el hardware.
- El hipervisor continuamente analiza en runtime el flujo de ejecución (bloques de código) de los SO guest y “traduce” las instrucciones sensibles por llamadas al hipervisor.
- Los bloques traducidos son ejecutados por la CPU directamente.
- Permite mejorar el rendimiento al poner en cache los bloques traducidos.

#### Asistida por hardware

- Se necesitan CPU “virtualizables”.
- Intel la llama VT (Tecnología de virtualización) y usa Root Mode y Non-Root Mode.
- AMD la llama SVM (máquina virtual segura) y usa Host Mode y Guest Mode.
- VMM ejecuta en Root Mode, VMs en Non-Root Mode.
- Ms generan traps al hipervisor cuando se ejecutan instrucciones sensibles (Non-Root Mode).

#### Paravirtualización

- Los hipervisors tipo 1 y 2 ejecutan SO guests no modificados.
- Se trata de tener SO guests modificados para mejorar el rendimiento.
- Cuando se quiere ejecutar una instrucción sensible, el SO guest la transforma en una llamada al VMM que expone una API específica.
- El VMM:
  - No realiza traducción binaria completa.
  - No debe emular instrucciones de hardware, lo cual hace que las llamadas se resuelvan de modo mas sencillo.
- El SO guest es como un proceso de usuario que hace llamadas al SO (el hipervisor).
- El hipervisor cuenta con una API, que es un conjunto de llamadas a procedimientos.
- Los guests, en vez de invocar instrucciones sensibles, invocan a estas llamadas (hipercalls).
- Se eliminan las instrucciones sensibles de los guest y se reemplazan por llamadas a la API especializada.
- El hipervisor se transforma en un microkernel.
- Decimos que un SO está paravirtualizado cuando se han eliminado, intencionalmente, algunas instrucciones sensibles (si se eliminan todas es paravirtualización completa si solo se eliminan algunas, se la llama paravirtualización parcial).
- Si no se eliminan TODAS las instrucciones sensibles, el VMM deberá realizar traducción binaria.
- Se puede implementar de dos formas:
  - Recompilando el kernel del sistema guest:
    - Los drivers y la forma de invocar a la API residen en el kernel.
    - Es necesario instalar un sistema operativo modificado/específico.
  - Instalando drivers paravirtualizados:
    - La paravirtualización es parcial (para algunas funciones y dispositivos).
    - Generalmente utilizada para placas de red, o gráficas.
    - Esta es la técnica que se utiliza hoy en día.
- Virtualización vs paravirtualización:
  - Virtualización completa o nativa puede generar problemas de performance ya que tiene que emular la totalidad del hardware.
  - Paravirtualización completa en kernel tiene mejor performance, pero soporta pocos SO, pues necesita modificar el SO original.
  - Paravirtualización parcial en drivers es una solución intermedia.

### Softwares virtualizadores

#### VMWare Workstation

- Es un hipervisor tipo 2.
- Explora código buscando bloques básicos (instrucciones seguidas que no cambien el
  program counter).
- Si hay instrucciones sensibles, sustituye cada una por una llamada a VMware.
- El bloque se pone en la cache de VMware, luego se ejecuta.
- Si no hay instrucciones sensibles, el bloque se ejecuta tan rápido como si fuera nativa.
- La acción de atrapar instrucciones sensibles y emularlas se conoce como traducción automática.
- Generalmente tiene un costo en la performance:
  - ~2% sobre CPU y RAM.
  - Entre ~8% y ~20% para dispositivos de I/O.

#### KVM

- Kernel Based Virtual Machine
- Infraestructura de virtualización para el kernel de Linux.
- Soportado nativamente en el kernel desde 2.6.20.
- Se lo considera tipo 1 ya que está embebida en el kernel del SO.
- Virtualiza plataformas x86 de 32 y 64 bits.
- Requiere un procesador con extensión para virtualización.
- Sistemas Operativos guest que admite: versions de Linux, BSD, Solaris, Windows, Haiku, ReactOS, Plan 9, AROS Research Operating System, Android 2.2, GNU/Hurd (Debian K16), Minix 3.1.2a, Solaris 10 U3, Darwin 8.0.1, entre otros.
- Soporta paravirtualización de algunos dispositivos para Linux, OpenBSD, FreeBSD, NetBSD, Plan 9 y Windows mediante API.

#### ProxmoxVE

- No es en si un hipervisor, sino que es una distribución GNU/Linux basada en Debian que agrupa funcionalidades de virtualización
- Utiliza KVM como hypervisor y LXC para la gestión de contenedores. También utiliza QEMU para emular ciertas VMs.
- Provee una interfaz web y una CLI para la gestión de las VMs.
- Es de código abierto. Tiene versión gratis y paga (soporte).
- Ya que usa KVM, aprovecha sus características como, por ejemplo, la migración en caliente.
- Soporta CEPH, que permite crear un Sistema de archivos distribuido entre todos los hosts.
- Se puede configurar en cluster (varios hosts corriendo proxmox y gestionarlos desde uno de ellos).
- Su instalación es sencilla a través de una “.iso”. El instalador provee el SO y las herramientas.

#### Hyper-V

- Infraestructura de virtualización propietaria de Microsoft.
- Requiere un procesador con extensión para virtualización.
- Virtualiza plataformas x86 de 32 y 64 bits.
- Se lo considera tipo 1, ya que está embebida en el Kernel del SO.
- También conocido como Viridian fue introducido en el año 2008 en Windows 2008 server.
- Es un hipervisor nativo. La funcionalidad se agrega como un Nuevo “Rol” en versions de Windows Server y Windows 10 Pro (no Home Edition).
- Permite correr SO guest Windows, GNU/Linux, BSD y otros.
- Junto con VMWare lideran el Mercado de los sistemas de virtualización empresariales.

#### VMWare vSphere

- Infraestructura de virtualización propietaria.
- Es para servers.
- Virtualiza plataformas x86 de 32 y 64 bits.
- Virtualiza redes, firewalls, almacenamiento.
- Incluye una importante suite de herramientas que permiten, de acuerdo a la licencia adquirida, utilizar un mayor o menos número de funciones.
- ESXi, junto con Hyper-V, lideran el Mercado de los sistemas de virtualización empresariales.

#### Xen

- Infraestructura de virtualización de código abierto desarrollada por la Universidad de Cambridge.
- Requiere un procesador con extensión para virtualización.
- Virtualiza plataformas x86 de 32 y 64 bits.
- Es un hipervisor nativo y también soporta paravirtualización.
- Permite correr SO guest Windows, GNU/Linux, BSD y otros.
- Permite la migración de máquinas virtuales en caliente entre miembros de un cluster XEN.

---

<h1 align="center">Clase 6 - 23 de abril, 2025</h1>

## CGroups y Namespaces

### Chroot

- Forma de aislar procesos del resto del sistema.
- Cambia el directorio raíz aparente de un proceso y todos sus hijos.
- No se puede acceder a archivos y comandos fuera de ese directorio.
- En otras palabras: el proceso "queda encerrado" en el nuevo directorio, y no puede ver ni acceder a archivos fuera de él. Es como si, para ese proceso, el directorio que se le indicó fuera todo el sistema.
- El nuevo directorio raíz es llamado "jail chroot".

### Control Groups

- Característica del kernel que permite agrupar procesos y gestionar sus recursos de forma jerárquica.
- Sirven para limitar, priorizar, aislar y monitorizar el uso de recursos del sistema (CPU, memoria, E/S de disco, red, etc.) entre grupos de procesos.
- Empezó a ser desarrollado en 2006 en Google bajo el nombre "process containers".
- Está disponible desde la versión 2.6.24 del kernel.
- Es **invisible para los procesos**.
- Posee 12 subsistemas
- Permiten un control de grano fino en:
  - Limitación de recursos:
    - Los grupos no pueden excederse en el uso de un recurso (tiempo de CPU, cantidad de cores, cantidad de memoria, etc).
  - Alocación.
  - Priorización:
    - Un grupo puede obtener prioridad en el uso de un
  - Denegación.
  - Monitoreo de los recursos del sistema (accounting):
    - Permite medir el uso de determinados recursos por parte de un grupo y obtener estadísticas y monitoreo.
  - Control:
    - Permite congelar y reiniciar un grupo de procesos.

#### Versiones

- Cgroups posee dos versiones principales, la v1 y la v2.
- Ambos controladores pueden ser montados en el mismo sistema.
- Una jerarquía de un controlador no puede estar en ambos cgroups simultáneamente.

##### v1

- **Grupos (cgroups)**: Contenedores de procesos que comparten las mismas limitaciones de recursos.
- **Jerarquía**: Los cgroups se organizan en una estructura de árbol donde cada grupo puede tener subgrupos.
- **Subsistemas**: Son módulos que proporcionan funcionalidades específicas de control de recursos (CPU, memoria, E/S, etc.).
  - También se los llama controllers o resource controllers.
  - Cada subsistema representa un único recurso: tiempo de CPU, memoria, etc.
- Funcionamiento:
  - Montaje: Los cgroups v1 se acceden a través de un sistema de archivos virtual, montado típicamente en `/sys/fs/cgroup`.
  - Creación de grupos: Se crean directorios dentro de la jerarquía para representar nuevos grupos.
  - Asignación de procesos: Se escriben los PIDs de los procesos en el archivo `tasks` o `cgroup.procs` del grupo.
  - Configuración de límites: Se establecen parámetros escribiendo en los archivos de control del grupo.
- Limitaciones:
  - Solo permite una jerarquía activa por subsistema.
  - La administración de recursos puede ser compleja.
  - Algunas funcionalidades están divididas entre múltiples subsistemas.
- Se usa `cgcreate` o `mkdir` dentro de la estructura para crear un nuevo cgroup.

##### v2

- Simplifica y unifica la administración de recursos, resolviendo varias limitaciones de la versión 1.
- Mejoras:
  - Jerarquía unificada: Solo existe una jerarquía que maneja todos los recursos (a diferencia de v1 con múltiples jerarquías independientes).
  - Diseño más coherente: Elimina inconsistencias y solapamientos entre subsistemas.
  - Nuevas características: Mejor soporte para contenedores, presión de recursos, y más.
- La jerarquía se monta en `/sys/fs/cgroup/` y posee a todos los controllers (CPU, memoria, etc).
- Cada cgroup posee los siguientes archivos:
  - cgroup.procs: Lista de procesos en el grupo.
  - cgroup.controllers: Lista de controladores disponibles.
  - cgroup.subtree_control: Controladores activados para subgrupos.
  - cgroup.events: Notificaciones de eventos.
    - Populated: si es 1, este cgroup o alguno de sus descendientes tiene procesos miembros.
    - Frozen: si es 1, el grupo está congelado y todos los procesos dentro de él pausan su ejecución.
    - Empty: Permite notificar cuando un cgroup está vacío.
- Se usa `mkdir` y `rmdir` para crear y eliminar cgroups.

### Namespace Isolation

- Técnica que separa y aísla ciertos recursos del sistema operativo para que diferentes procesos crean que tienen su propio entorno independiente.
- Permite abstraer un recurso global del sistema para que los procesos dentro de ese namespace crean que tienen su propia instancia aislada de ese recurso global.
- Limita lo que un proceso puede ver y por ende lo que puede usar.
- Las modificaciones a un recurso particular quedan contenidas dentro del namespace.
- Un proceso solo puede estar en un namespace de un tipo a la vez.
- Un namespace es automáticamente eliminado cuando el último proceso en él termina o lo abandona.
- Un proceso puede usar ninguno/algunos/todos de los namespaces de su padre.
- Existen varios tipos de namespaces, cada uno aislando un recurso diferente:
  - PID namespace: aísla los PIDs (cada grupo de procesos puede tener su propio init PID 1).
  - Mount namespace: aísla los puntos de montaje del sistema de archivos.
  - Network namespace: aísla interfaces de red, direcciones IP, tablas de rutas, etc.
  - UTS namespace: permite a un proceso tener su propio nombre de host y nombre de dominio.
  - IPC namespace: aísla mecanismos de comunicación entre procesos (como colas de mensajes o memoria compartida).
  - User namespace: permite que un proceso tenga diferentes IDs de usuario y grupo, incluso ejecutarse como root dentro de su propio namespace pero no fuera de él.
  - Cgroup namespace: aísla la vista de los control groups (cgroups).
- La función `unshare()` agrega al proceso actual a un nuevo namespace. Crea el namespace y hace miembro de él al proceso llamador.
- La función `setns()` agrega al proceso actual a un namespace existente. Desasocia al proceso llamante de una instancia de un tipo de namespace y lo reasocia con otra instancia del mismo tipo de namespace.
- Cada proceso tiene un subdirectorio que contiene todos los nombres de los namespaces a los que está asociado: `/proc/[pid]/ns`.
- Un proceso hijo hereda todos los namespaces de su proceso padre.

## Contenedores

### Breve historia

- 1979: UNIX v7. Implementa la system call chroot.
- 2000:
  - FreeBSD Jail.
  - Extiende el chroot: dirección IP, hostname, usuarios y procesos propios.
- 2004: Solaris Zones. Pueden contener diferentes binarios, toolkits e, inclusive diferente OS.
- 2008: Linux Containers (LXC).
- 2013: Docker entra en escena.
- 2014:
  - Se anuncia el proyecto Kubernetes.
  - Se libera Docker 1.0.
- 2015:
  - Se crea la Open Container Initiative.
  - Se lanza Kubernetes 1.0.
- 2018: Google Kubernetes Engine se vuelve disponible.
- 2019: RedHat lanza la versión 1.0 de Podman.

### Concepto

- Tecnología liviana de virtualización a nivel SO que permite ejecutar varios sistemas aislados (conjuntos de procesos) en un único host.
- La virtualización tradicional (como las VMs) emula hardware completo y requiere levantar un SO entero para cada instancia.
- En cambio, los contenedores **comparten el mismo kernel** del sistema operativo anfitrión.
- Esto los hace mucho más livianos y rápidos para iniciar, consumir menos recursos y ser más fáciles de escalar.
- No es necesario un software de virtualización tipo hypervisor.
- Son procesos normales del SO.
- No es posible ejecutar instancias de SO con kernel diferente al SO base (por ej. Windows sobre Linux).
- Los más populares son Podman y Docker.
- Usualmente, cada contenedor provee un único servicio comúnmente denominado **microservicio**.
- Visión:
  - Desde el lado del nodo host, un contenedor es un **proceso** (o conjunto de procesos) en ejecución.
  - En el nodo host se ven los procesos de todos los contenedores. Esto no es posible a la inversa ni entre distintos contenedores.

### Características clave

- **Autocontenidos**:
  - Tienen todo lo que necesitan para funcionar.
  - Un contenedor contiene un código específico y todas las librerías y dependencias necesarias para ejecutarse.
  - Imágenes.
- **Aislados**:
  - Ejecutan de manera aislada en modo usuario usando un kernel compartido.
  - Mínima influencia en el nodo y en otros contenedores.
- **Independientes**:
  - Administrar a un contenedor no afecta a ningun otro contenedor.
- **Portables**:
  - Están desacoplados del entorno donde se ejecutan.
  - Pueden ejecutarse de igual manera en diferentes entornos.

### Tipos de contenedores

- De SO:
  - Ejecutan un SO completo (excepto el kernel).
  - LXC, BSD Jails, Solaris Zone, etc.
- De aplicaciones:
  - Empaquetan una aplicación o proceso.
  - Docker, Podman, etc.

### Relación con Namespaces y Cgroups

- Un contenedor se basa en estos dos conceptos:
  - **Namespaces**:
    - Proporcionan aislamiento de recursos.
    - Cada contenedor tiene su propio namespace para procesos (PID), sistema de archivos (mount), red (network), usuarios (user), etc.
    - Así, los procesos dentro de un contenedor no ven ni interactúan con los de otros contenedores o del host, asegurando el aislamiento.
  - **Control Groups (cgroups)**:
    - Controlan cuánto CPU, memoria, I/O de disco y otros recursos puede usar un contenedor, evitando que uno monopolice los recursos del sistema.

---

<h1 align="center">Clase 7 - 30 de abril, 2025</h1>

## Docker

### Concepto

- Plataforma open-source que permite empaquetar y ejecutar una aplicación en contenedores livianos.
- Se usa para:
  - Desarrollo/testing.
  - Escalado y deployment.
  - Más servicios en un equipo sin VMs.
- Docker Engine, el runtime que usa Docker, se divide en 3 componentes:
  - **Docker Daemon (dockerd)**:
    - Es el servidor.
    - Responsable por crear, ejecutar y monitorear los contenedores, construir imágenes, etc.
  - **API**:
    - Especifica la interface que los programas pueden usar para interactuar con el servidor.
  - **CLI**:
    - Cliente.
    - Permite a los usuarios interactuar con el servidor mediante comandos.

### Características

- Usa una arquitectura C/S.
- El cliente y el servidor:
  - Se pueden ejecutar en el mismo sistema (vía IPC o socket domain) o en diferentes nodos (vía socket TCP/IP).
  - Se distribuyen como binarios.
  - Ejecutan en espacio de usuario.
- El Docker Daemon escucha por API requests.
- Se comunican usando la REST API de Docker, el cliente envía comandos HTTP.
- Además de la CLI existe una interface gráfica: Docker Desktop.

### Funcionamiento

- Docker usa una serie de características del kernel para poder proveer contenedores:
  - **Namespaces**:
    - Docker los usa para proveer el espacio de trabajo aislado que llamamos contenedor.
    - Por cada contenedor, Docker crea un conjunto de namespaces (entre ellos pid, net, ipc y mnt).
  - **Cgroups**:
    - Para (opcionalmente) limitar los recursos asignados a un contenedor.
  - **Union file systems**:
    - Se usan como filesystem de los contenedores.
    - Docker puede usar overlay2, AUFS, btrfs, vfs, y DeviceMapper.

### Definiciones

- **Imágen**: Template de sólo lectura con todas las instrucciones para construir un contenedor. Una imágen puede basarse en otras.
- **Contenedor**: Instancia de una imágen en ejecución.
- **Registry**: Almacén de imágenes de Docker. Puede ser público o privado. Por defecto, Docker usa Docker Hub.
- **Dockerfile**: Archivo que indica los pasos necesarios para construir la imágen.

### Imágenes y capas

- Cada imágen se compone de una serie de capas que se montan una sobre otra.
- Cada capa es un conjunto de diferencias con la capa previa.
- Solo la última capa es R/W (la capa del contenedor). Las demás son read-only.
- Las capas pueden ser reusadas entre las imágenes.
- Capa escribible permite almacenar datos generados durante la ejecución del contenedor.
- Docker usa "storage drivers" para almacenar capas de una imágen y para almacenar datos en la capa escribible del contenedor.
- Hay distintos tipos de storage drivers:
  - overlay2.
  - btrfs.
  - zfs.
  - etc.
  - En windows, windowsfilter.
- Apilando las capas:
  - Cada capa que se baja, se extrae el contenido en un directorio del filesystem del nodo.
  - Al ejecutar el contenedor desde una imagen, se genera un union-filesystem donde las capas se apilan una sobre otra.
  - Usando chroot, se estable el union-filesystem creado como directorio raíz del contenedor.
  - Por ultimo, se crea un nuevo directorio para el contenedor que permite modificar el filesystem (capa escribible de contenedor).
- El directorio modificable es el que permite correr múltiples contenedores a partir de las mismas capas (uno nuevo por cada contenedor).

### Dockerfile

- Cada imágen se contruye siguiendo las instrucciones de un dockerfile.
- Cada instrucción en el dockerfile añade una nueva capa a la imágen.

### Contenedores

- Un contenedor es una instancia de una imágen.
- La diferencia principal entre contenedor e imagen es la capa escribible.
  - Esta capa se elimina al eliminar al contenedor.
  - Las capas inferiores se mantienen intactas.
- Desde una imágen es posible generar varios contenedores.
- Cada contenedor es autónomo y ejecuta en su propio entorno aislado.
- Todos los contenedores comparten el kernel y ejecutan en sus propios namespaces.
- Los contenedores pueden ser iniciados, detenidos, pausados o destruidos usando la Docker CLI.

### Almacenamiento

- Los archivos creados dentro de un contenedor son almacenados en una capa escribible.
- Los datos escritos en el contenedor no persisten cuando es destruido.
- Docker tiene dos opciones para almacenar datos en el host para que sean persistentes:
  - **Volumes**: almacenados en una parte del filesystem administrada por Docker (por default: /var/lib/docker/volumes)
  - **Bind Mounts**: pueden estar en cualquier parte del filesystem. Pueden ser modificados por procesos que no sean de Docker.
  - Ambos deben ser montados en el contenedor.
  - Volumes permite una mejor portabilidad entre sistemas.

### Networking

- Contenedores tienen el networking habilitado por default, aunque puede desactivarse.
- Es posible realizar conexiones salientes.
- Los usuarios pueden definir nuevas redes.
- Múltiples contenedores pueden conectarse a la misma red y comunicarse usando direcciones IP y/o nombre.
- Un contenedor se puede conectar a varias redes a la vez.
- Para hacer disponible un servicio el contenedor debe publicar el correspondiente puerto.
- Dos contenedores en el mismo host no pueden publicar el mismo puerto.
- Los contenedores usan los mismos servidores DNS que el nodo host, pero se pueden modificar.

### Arquitectura

![Arquitectura Docker](https://i.imgur.com/BzzpW2B.png)

### Comandos básicos

- `docker pull httpd`: Descarga la imágen de Apache de DockerHUB.
- `docker run httpd`: Ejecuta la imágen.
- `docker image build -t NOMBRE_TAG`: Crea una imágen a partir de un dockerfile.
- `docker run NOMBRE_TAG`: Ejecuta la imágen creada.
- `docker push NOMBRE_TAG\USUARIO_DOCKERHGUB/REPOSITORIO`: Sube la imágen a DockerHUB (hay que ejecutar `docker login` previamente).
- `docker info`: Información general y configuración.
- `docker ps`: Contenedores en ejecución.
- `docker image ls`: Imágenes y contenedores
- `docker container ls -a`: Imágenes y contenedores
- `docker pull ubuntu && \docker run -v ./dir_comp:/mnt -it ubuntu`: Ejecuta el container de Ubuntu en modo interactivo (bash).
- `docker commit CONTAINER REPOSITORY:TAG`: Crea una nueva imágen con los cambios del contenedor.

---

<h1 align="center">Clase 8 - 7 de mayo, 2025</h1>

##

---

<h1 align="center">Clase 8 - 14 de mayo, 2025</h1>

##

---

<h1 align="center">Clase 8 - 21 de mayo, 2025</h1>

##

---
