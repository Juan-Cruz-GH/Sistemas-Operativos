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
