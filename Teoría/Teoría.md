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
