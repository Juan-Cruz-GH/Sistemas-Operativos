<h1 align="center">Clase 1 - 13 de marzo, 2025</h1>

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

<h1 align="center">Clase 2 - 20 de marzo, 2025</h1>

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

<h1 align="center">Clase 3 - 27 de marzo, 2025</h1>

##

---

<h1 align="center">Clase 4 - 10 de abril, 2025</h1>

##

---

<h1 align="center">Clase 5 - 17 de abril, 2025</h1>

##

---

<h1 align="center">Clase 6 - 24 de abril, 2025</h1>

##

---

<h1 align="center">Clase 7 - 1 de mayo, 2025</h1>

##
