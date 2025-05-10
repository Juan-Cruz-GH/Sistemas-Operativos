<h1 align="center">Práctica 4</h1>

## Conceptos Teóricos

#### 1. Defina virtualización. Investigue cuál fue la primera implementación que se realizó.

La virtualización es una tecnología que permite crear representaciones virtuales de recursos físicos, como servidores, SOs, dispositivos de almacenamiento o redes. En lugar de depender directamente del hardware, las aplicaciones y SOs pueden ejecutarse dentro de entornos virtuales aislados llamados máquinas virtuales, que son administradas por un software llamado hipervisor. Esto permite aprovechar mejor el hardware, facilitar la administración y mejorar la flexibilidad y escalabilidad de los sistemas.

La primera implementación importante de virtualización fue desarrollada por IBM en la década de 1960, específicamente con el proyecto CP/CMS (Control Program/Cambridge Monitor System), diseñado para la línea de mainframes IBM System/360 modelo 67. Este sistema permitía que varios usuarios ejecutaran instancias completamente separadas de un SO, cada una con su propio espacio de memoria y acceso simulado al hardware, lo que constituyó la base del concepto moderno de máquina virtual. Fue una solución pensada para mejorar el aprovechamiento de los costosos mainframes y facilitar el desarrollo y prueba de software.

#### 2. ¿Qué diferencia existe entre virtualización y emulación?

La emulación implica que un sistema (ya sea software o hardware) imite el comportamiento de otro sistema distinto.

| Característica                      | **Virtualización**                                                            | **Emulación**                                                                    |
| ----------------------------------- | ----------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| **Compatibilidad de arquitectura**  | Requiere que el sistema invitado tenga **la misma arquitectura** que el host. | Puede emular una **arquitectura completamente distinta** a la del host.          |
| **Acceso al hardware**              | Acceso **directo o casi directo** al hardware físico mediante el hipervisor.  | Todo el hardware es **simulado por software**.                                   |
| **Velocidad y rendimiento**         | **Alto rendimiento**, cercano al nativo.                                      | **Menor rendimiento**, debido a la traducción constante de instrucciones.        |
| **Complejidad del sistema**         | Menor complejidad, ya que se apoya en hardware real.                          | Mayor complejidad, ya que debe recrear el comportamiento completo del hardware.  |
| **Flexibilidad de sistemas**        | Limitada a sistemas compatibles con la arquitectura del host.                 | Muy flexible: puede ejecutar software de **cualquier sistema** (si se emula).    |
| **Uso típico**                      | Consolidación de servidores, entornos de prueba, contenedores.                | Ejecutar software antiguo o de otras plataformas (consolas, sistemas obsoletos). |
| **Ejemplo práctico**                | VMware, VirtualBox, KVM, Hyper-V.                                             | QEMU, DOSBox, SNES9x, PCSX2.                                                     |
| **Dependencia del hipervisor**      | Necesita un hipervisor para gestionar las máquinas virtuales.                 | No necesita hipervisor; el emulador actúa como todo el sistema.                  |
| **Requisitos del sistema invitado** | El sistema operativo invitado cree que está corriendo sobre hardware real.    | El software invitado **no distingue** que está siendo emulado.                   |
| **Objetivo principal**              | Aislar entornos y mejorar el uso del hardware moderno.                        | Reproducir hardware diferente o antiguo para compatibilidad o preservación.      |

#### 3. Investigue el concepto de hypervisor y responda:

##### (a) ¿Qué es un hypervisor?

Un hipervisor es la pieza clave de software que hace posible la virtualización. Actúa como una capa entre el hardware físico y los sistemas operativos invitados. Su trabajo es gestionar y distribuir los recursos de hardware a las diferentes VMs y aislarlas entre sí.

##### (b) ¿Qué beneficios traen los hypervisors? ¿Cómo se clasifican?

Los beneficios principales de los hipervisores son:

- **Aprovechamiento del hardware**: Permiten ejecutar varias VMs en un solo equipo físico, usando al máximo los recursos disponibles.
- **Aislamiento**: Cada VM es independiente. Si una falla o se infecta con malware, no afecta a las demás.
- **Facilidad para pruebas y desarrollo**: Se pueden crear y descartar entornos rápidamente, ideal para testing de software.
- **Portabilidad**: Las VMs son archivos que se pueden copiar, mover o clonar fácilmente entre distintos sistemas.
- **Escalabilidad**: Es sencillo aumentar o reducir recursos asignados a una VM sin necesidad de modificar el hardware físico.
- **Reducción de costos**: Menos hardware físico implica menos mantenimiento, menor consumo energético y menor espacio requerido.
- **Compatibilidad y legado**: Permiten ejecutar sistemas operativos antiguos o específicos sin necesidad de hardware dedicado.

Existen dos tipos de hipervisores:

- **Tipo 1 (Bare Metal)**:
  - Se instala directamente sobre el hardware físico, sin un SO anfitrión tradicional.
  - Es muy eficiente y se usa comúnmente en centros de datos.
  - Ejemplos: VMware ESXi, Microsoft Hyper-V.
- **Tipo 2 (Hosted)**:
  - Se ejecuta como una aplicación dentro de un SO anfitrión existente.
  - Es más sencillo de instalar y usar en equipos de escritorio.
  - Ejemplos: Oracle VirtualBox, VMware Workstation, Parallels.

#### 4. ¿Qué es la full virtualization? ¿Y la virtualización asistida por hardware?

Full Virtualization y Virtualización asistida por hardware son dos técnicas para que el OS guest, que **cree** que tiene acceso directo al hardware, no cause problemas al intentar ejecutar instrucciones que solo el sistema con mayor privilegio (hipervisor o el OS host en Tipo 2) debería ejecutar.

**Full virtualization**:

- Tiene como objetivo permitir que un SO invitado sin modificar (que no sabe que está virtualizado) se ejecute en una VM.
- El hipervisor simula todo el hardware.
- No necesita soporte especial del procesador.
- Más carga para el sistema, menor rendimiento.
- El hipervisor debe interceptar todas las instrucciones "sensibles" o "privilegiadas" que el OS invitado intente ejecutar y que podrían afectar a otras VMs o al propio hipervisor.
- Las técnicas para interceptar y manejar estas instrucciones son:
  - Trap-And-Emulate.
  - Binary Translation.

**Hardware-assisted virtualization**:

- Evolución y el método dominante hoy en día para lograr Virtualización Completa de manera eficiente.
- Los fabricantes de CPUs (Intel VT-x, AMD-V) han añadido instrucciones y modos de operación especiales al hardware diseñados específicamente para la virtualización.
- Esto permite que el hipervisor configure la CPU de modo que las instrucciones privilegiadas del Guest OS sí generen un trap limpio que el hipervisor puede manejar directamente, sin necesidad de la compleja y lenta Traducción Binaria.
- Esencialmente, el hardware ahora hace el trabajo de Trap And Emulate de forma eficiente para la Virtualización Completa.
- Requiere CPU con soporte (Intel VT-x, AMD-V).
- El hardware gestiona directamente muchas tareas de virtualización.
- Permite ejecución casi directa del SO invitado.
- Mejor rendimiento y menor complejidad del hipervisor.

#### 5. ¿Qué implica la técnica binary translation? ¿Y trap-and-emulate?

Binary Translation y Trap-And-Emulate son técnicas que usan los hipervisores para interceptar y manejar instrucciones privilegiadas que no pueden ser ejecutadas directamente por un SO guest.

**Trap-And-Emulate**:

- Técnica ideal en teoría.
- Cuando un Guest OS en un nivel de privilegio bajo intenta ejecutar una instrucción privilegiada, el hardware debería generar un trap, que es una excepción que el hipervisor puede interceptar.
- El hipervisor entonces "emula" (simula) el comportamiento esperado de la instrucción para ese Guest OS particular.
- El problema es que las arquitecturas x86 clásicas no cumplían perfectamente esta regla; algunas instrucciones privilegiadas simplemente fallaban o se comportaban inesperadamente en niveles bajos de privilegio sin generar una trampa útil.

**Binary Translation**:

- Debido a las limitaciones de Trap And Emulate en arquitecturas como x86 sin soporte de hardware para virtualización, se desarrolló la Traducción Binaria.
- El hipervisor escanea dinámicamente el código del Guest OS. Cuando detecta un bloque de código que contiene instrucciones sensibles/privilegiadas, lo "traduce" sobre la marcha, reemplazando esas instrucciones con nuevas secuencias de instrucciones que interactúan con el hipervisor en lugar de intentar interactuar directamente con el hardware. Este código traducido se ejecuta entonces.
- Es un proceso complejo y añade overhead.
- Permite virtualizar CPUs sin soporte de hardware.
- Es costosa en rendimiento. Implica mucha sobrecarga de CPU por la traducción en tiempo de ejecución.

#### 6. Investigue el concepto de paravirtualización y responda:

##### (a) ¿Qué es la paravirtualización?

La paravirtualización es un enfoque diferente a la Virtualización Completa. En lugar de ocultar la virtualización al Guest OS, la Paravirtualización requiere que el Guest OS sea modificado para ser **consciente** de que está corriendo en un entorno virtual. El Guest OS modificado no ejecuta instrucciones privilegiadas directamente; en su lugar, hace "hiperllamadas" explícitas al hipervisor para solicitar servicios (como acceso a dispositivos, gestión de memoria, etc.). **Es como si el Guest OS tuviera una API para hablar directamente con el hipervisor**.

Puede ser más eficiente que la Virtualización Completa sin asistencia de hardware (es decir, más rápida que la Traducción Binaria) porque no hay necesidad de interceptar y traducir instrucciones complejas.

Requiere modificar el kernel del Guest OS, lo que limita qué SOs pueden usarse (a menos que ya existan versiones paravirtualizadas).

##### (b) Mencione algún sistema que implemente paravirtualización.

Los sistemas principales que usan paravirtualización son:

- Hipervisor Xen.
- SOs basados en Linux bajo Xen.
- Sistemas BSD.

##### (c) ¿Qué beneficios trae con respecto al resto de los modos de virtualización?

La paravirtualización presenta varios beneficios:

- **Mejor rendimiento vs virtualización completa por software**:
  - Al no tener que interceptar y traducir dinámicamente bloques de código privilegiado (como hace la Traducción Binaria), las "hiperllamadas" directas de la paravirtualización son significativamente más eficientes.
  - Esto resulta en un rendimiento mucho más cercano al nativo que la Virtualización Completa sin asistencia de hardware.
- **Menor carga para el hipervisor**:
  - La lógica del hipervisor para manejar las solicitudes del Guest OS es más simple que la necesaria para la traducción binaria o la emulación compleja de dispositivos completos.
- **Mejor rendimiento de I/O**:
  - Aunque la asistencia de hardware resuelve el problema del rendimiento de la CPU en Virtualización Completa, emular dispositivos de hardware completos y complejos (como adaptadores de red gigabit o controladores de almacenamiento rápidos) en software por parte del hipervisor puede seguir siendo costoso.
  - Acá es donde los controladores paravirtualizados destacan. Un controlador paravirtualizado instalado en el Guest OS (por ejemplo, un controlador VirtIO para red o disco) se comunica directamente con una "fachada" o modelo de dispositivo simplificado y optimizado que expone el hipervisor. Esto es mucho más eficiente que emular un dispositivo real instrucción por instrucción. Por ello, incluso en entornos con virtualización de CPU asistida por hardware (como KVM o Hyper-V con Integration Services), se usan drivers paravirtualizados para obtener el mejor rendimiento posible en red, disco, etc.

#### 7. Investigue sobre containers y responda:

##### (a) ¿Qué son?

Los contenedores son procesos aislados que empaquetan y ejecutan aplicaciones junto con sus dependencias, como si cada contenedor tuviera su propio sistema operativo, pero sin la sobrecarga de una máquina virtual.

##### (b) ¿Dependen del hardware subyacente?

Los contenedores dependen del hardware subyacente, pero de forma más ligera y controlada que una VM.

Dependen de:

- **Kernel**: Todos los contenedores usan el mismo kernel del host, por ende dependen del tipo de SO y arquitectura del host.
- **Arquitectura de la CPU**: Por ejemplo, un contenedor construido para x86_64 no corre en ARM, a menos que se use emulación.
- **Recursos físicos de hardware disponibles**: Aunque se puede limitar cuánta CPU o memoria puede usar un contenedor, al final los contenedores usan los recursos del hardware físico.

##### (c) ¿Qué lo diferencia por sobre el resto de las tecnologías estudiadas?

| Característica        | Contenedores                 | Máquinas Virtuales              |
| --------------------- | ---------------------------- | ------------------------------- |
| Kernel                | Comparten el del host.       | Cada VM tiene su propio kernel. |
| Peso                  | Ligeros (solo lo necesario). | Pesadas (sistema completo).     |
| Velocidad de arranque | Rápida (segundos).           | Lenta (varios segundos o más).  |
| Aislamiento           | A nivel de proceso.          | A nivel de hardware.            |
| Uso de recursos       | Más eficiente.               | Más costoso.                    |

##### (d) Investigue qué funcionalidades son necesarias para poder implementar containers.

Las principales funcionalidades necesarias para poder implementar containers son:

- **Namespaces**:
  - Aíslan distintas partes del sistema para que un contenedor crea que es el único que existe.
  - Permiten el **aislamiento** entre contenedores.
- **Cgroups**:
  - Permiten limitar y monitorear el uso de recursos de hardware por parte de un contenedor.
  - Evitan que un contenedor consuma todo el sistema.
- **chroot**:
  - Cambian el root filesystem de un proceso para que "viva" en su propio mundo.
  - Soportan la **separación** del sistema de archivos.

## Chroot

### Debido a que para la realización de la práctica es necesario tener más de una terminal abierta simultáneamente tenga en cuenta la posibilidad de lograr esto mediante alguna alternativa (ssh, terminales gráficas, etc.)

### En algunos casos suele ser conveniente restringir la cantidad de información a la que un proceso puede acceder. Uno de los métodos más simples para aislar servicios es `chroot`, que consiste simplemente en cambiar lo que un proceso, junto con sus hijos, consideran que es el directorio raíz, limitando de esta forma lo que pueden ver en el sistema de archivos. En esta sección de la práctica se preparará un árbol de directorios que sirva como directorio raíz para la ejecución de una shell.

#### 1. ¿Qué es el comando chroot? ¿Cuál es su finalidad?

El comando `chroot` cambia el directorio raíz aparente de un proceso en ejecución y de todos sus procesos hijos si los tuviera.

Su finalidad es restringir lo que ese proceso y sus hijos pueden ver y acceder, para lograr crear un entorno aislado para ese proceso.

#### 2. Crear un subdirectorio llamado sobash dentro del directorio root. Intente ejecutar el comando `chroot /root/sobash`. ¿Cuál es el resultado? ¿Por qué se obtiene ese resultado?

```bash
root@so:~# mkdir sobash
root@so:~# cd sobash
root@so:~/sobash# chroot /root/sobash/
chroot: failed to run command '/bin/bash': No such file or directory
```

Se obtiene este resultado porque Linux no logra encontrar al archivo `/bin/bash` necesario para poder ejecutar `chroot`.

#### 3. Cree la siguiente jerarquía de directorios dentro de sobash:

```
sobash/
    bin
    lib
        x86_64-linux-gnu
    lib64
```

#### 4. Verifique qué bibliotecas compartidas utiliza el binario `/bin/bash` usando el comando `ldd /bin/bash`. ¿En qué directorio se encuentra linux-vdso.so.1? ¿Por qué?

```bash
root@so:~/sobash/lib# ldd /bin/bash
    linux-vdso.so.1 (0x00007f53517fe000)
    libtinfo.so.6 => /lib/x86_64-linux-gnu/libtinfo.so.6 (0x0...)
    libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x0...)
    /lib64/ld-linux-x86-64.so.2 (0x0...)
```

- Las bibliotecas compartidas que usa `/bin/bash` son:
  - **linux-vdso.so.1**
  - **libtinfo.so.6**
  - **libc.so.6**
  - **/lib64/ld-linux-x86-64.so.2**
- linux-vdso.so.1 no se encuentra en un directorio tradicional del sistema de archivos, porque es una biblioteca virtual dinámica proporcionada directamente por el kernel de Linux: es decir, en lugar de ser un archivo en el disco, el kernel mapea esta biblioteca directamente al espacio de direcciones de cada proceso que la utiliza.

#### 5. Copie en `/root/sobash` el programa `/bin/bash` y todas las librerías utilizadas por el programa bash en los directorios correspondientes. Ejecute nuevamente el comando chroot ¿Qué sucede ahora?

```bash
cp /bin/bash /root/sobash/bin
cp /lib/x86_64-linux-gnu/libtinfo.so.6 /root/sobash/lib/x86_64-linux-gnu
cp /lib/x86_64-linux-gnu/libc.so.6 /root/sobash/lib/x86_64-linux-gnu
cp /lib64/ld-linux-x86-64.so.2 /root/sobash/lib64
```

```bash
root@so:~/sobash# chroot /root/sobash/
bash-5.2#
```

Ahora el comando se ejecuta exitosamente.

#### 6. ¿Puede ejecutar los comandos `cd "directorio"` o `echo`? ¿Y el comando `ls`? ¿A qué se debe esto?

Puedo ejecutar los comandos `cd` y `echo`, pero no `ls` debido a que éste es un ejecutable externo al cual ya no tenemos acceso luego de cambiar el directorio raíz.

#### 7. ¿Qué muestra el comando `pwd`? ¿A qué se debe esto?

```bash
bash-5.2# pwd
/
```

Muestra al directorio `/`. Esto se debe a que toma como directorio raíz el directorio donde se ejecutó chroot, creando una especie de jaula, haciendo que no se pueda acceder ni ver archivos y comandos fuera del directorio.

#### 8. Salir del entorno chroot usando `exit`

```bash
bash-5.2# exit
exit
root@so:~/sobash#
```

#### 9. Usando el repositorio de la cátedra acceda a los materiales en `practica4/02-chroot`

##### a. Verifique que tiene instalado busybox en `/bin/busybox`

```bash
root@so:/home/so/codigo-para-practicas/practica4/02-chroot# which /bin/busybox
/bin/busybox
```

##### b. Cree un chroot con busybox usando `./buildbusyboxroot.sh`

```bash
root@so:/home/so/codigo-para-practicas/practica4/02-chroot# ./buildbusyboxroot.sh
  linux-vdso.so.1 (0x0...)
  libresolv.so.2 => /lib/x86_64-linux-gnu/libresolv.so.2 (0x0...)
  libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x0...)
  /lib64/ld-linux-x86-64.s0.2 (0x0...)
BusyBox root filesystem created in /home/so/codigo-para-practicas/practica4/02-chroot/busyboxroot
You can now chroot into it with:
chroot /home/so/codigo-para-practicas/practica4/02-chroot/busyboxroot /bin/sh
```

##### c. Entre en el chroot

```bash
chroot /home/so/codigo-para-practicas/practica4/02-chroot/busyboxroot /bin/sh

BusyBox v1.35.0 (Debian 1:1.35.0-4+b3) built-in shell (ash)
Enter 'help' for a list of built-in commands.

/ #
```

##### d. Busque el directorio `/home/so` ¿Qué sucede? ¿Por qué?

Este directorio "no existe" luego de hacer el chroot. Está inaccesible.

##### e. Ejecute el comando `ps aux` ¿Qué procesos ve? ¿Por qué (pista: ver el contenido de `/proc`)?

Al ejecutar el comando, no se ve ningún proceso.

El directorio `/proc` está vacío.

##### f. Monte `/proc` con `mount -t proc proc /proc` y vuelva a ejecutar `ps aux` ¿Qué procesos ve? ¿Por qué?

Ahora vemos todos los procesos de nuestro sistema original.

##### g. Acceda a `/proc/1/root/home/so` ¿Qué sucede?

```bash
/ # cd /proc/1/root/home/so
sh: getcwd: No such file or directory
```

##### h. ¿Qué conclusiones puede sacar sobre el nivel de aislamiento provisto por chroot?

Concluyo que si bien `chroot` tiene su utilidad, es bastante limitado y puede ser eludido con facilidad.

## Control Groups

### Se aconseja realizar esta parte de la práctica en una máquina virtual (por ejemplo en la provista por la práctica) ya que es necesario cambiar la configuración de CGroups.

### Preparación

#### Actualmente Debian y la mayoría de las distribuciones usan CGroups 2 por defecto, pero para esta práctica usaremos CGroups 1. Para esto es necesario cambiar un parámetro de arranque del sistema en grub:

#### 1. Editar `/etc/default/grub`: Cambiar `GRUB_CMDLINE_LINUX_DEFAULT="quiet"` por `GRUB_CMDLINE_LINUX_DEFAULT="quiet systemd.unified_cgroup_hierarchy=0"`.

#### 2. Actualizar la configuración de GRUB: `sudo update-grub`.

#### 3. Reiniciar la máquina

#### 4. Verificar que se esté usando CGroups 1. Para esto basta con hacer `ls /sys/fs/cgroup/`. Se deberían ver varios subdirectorios como cpu, memory, blkio, etc. (en vez de todo montado de forma unificada).

#### A continuación se probará el uso de cgroups. Para eso se crearán dos procesos que compartirán una misma CPU y cada uno la tendrá asignada un tiempo determinado. Nota: es posible que para ejecutar `xterm` tenga que instalar un gestor de ventanas. Esto puede hacerse con `apt-get install xterm`.

### Ejercicios

#### 1. ¿Dónde se encuentran montados los cgroups? ¿Qué versiones están disponibles?

#### 2. ¿Existe algún controlador disponible en cgroups v2? ¿Cómo puede determinarlo?

#### 3. Analice qué sucede si se remueve un controlador de cgroups v1 (por ej. `Umount /sys/fs/cgroup/rdma`).

#### 4. Crear dos cgroups dentro del subsistema cpu llamados cpualta y cpubaja. Controlar que se hayan creado tales directorios y ver si tienen algún contenido `mkdir /sys/fs/cgroup/cpu/"nombre_cgroup"`

#### 5. En base a lo realizado, ¿qué versión de cgroup se está utilizando?

#### 6. Indicar a cada uno de los cgroups creados en el paso anterior el porcentaje máximo de CPU que cada uno puede utilizar. El valor de cpu.shares en cada cgroup es 1024. El cgroup cpualta recibirá el 70 % de CPU y cpubaja el 30 %.

##### `echo 717 > /sys/fs/cgroup/cpu/cpualta/cpu.shares`

##### `echo 307 > /sys/fs/cgroup/cpu/cpubaja/cpu.shares`

#### 7. Iniciar dos sesiones por ssh a la VM.(Se necesitan dos terminales, por lo cual, también podría ser realizado con dos terminales en un entorno gráfico). Referenciaremos a una terminal como termalta y a la otra, termbaja.

#### 8. Usando el comando `taskset`, que permite ligar un proceso a un core en particular, se iniciará el siguiente proceso en background. Uno en cada terminal. Observar el PID asignado al proceso que es el valor de la columna 2 de la salida del comando.

##### `taskset -c 0 md5sum /dev/urandom &`

#### 9. Observar el uso de la CPU por cada uno de los procesos generados (con el comando `top` en otra terminal). ¿Qué porcentaje de CPU obtiene cada uno aproximadamente?

#### 10. En cada una de las terminales agregar el proceso generado en el paso anterior a uno de los cgroup (termalta agregarla en el cgroup cpualta, termbaja en cpubaja. El process_pid es el que obtuvieron después de ejecutar el comando taskset)

##### `echo "process_pid" > /sys/fs/cgroup/cpu/cpualta/cgroup.procs`

#### 11. Desde otra terminal observar cómo se comporta el uso de la CPU. ¿Qué porcentaje de CPU recibe cada uno de los procesos?

#### 12. En termalta, eliminar el job creado (con el comando jobs ven los trabajos, con kill %1 lo eliminan. No se olviden del %.). ¿Qué sucede con el uso de la CPU?

#### 13. Finalizar el otro proceso md5sum.

#### 14. En este paso se agregarán a los cgroups creados los PIDs de las terminales (Importante: si se tienen que agregar los PID desde afuera de la terminal ejecute el comando `echo $$` dentro de la terminal para conocer el PID a agregar. Se debe agregar el PID del shell ejecutando en la terminal).

##### `echo $$ > /sys/fs/cgroup/cpu/cpualta/cgroup.procs` (termalta)

##### `echo $$ > /sys/fs/cgroup/cpu/cpubaja/cgroup.procs` (termbaja)

#### 15. Ejecutar nuevamente el comando `taskset -c 0 md5sum /dev/urandom &` en cada una de las terminales. ¿Qué sucede con el uso de la CPU? ¿Por qué?

#### 16. Si en termbaja ejecuta el comando: `taskset -c 0md5sum /dev/urandom &` (deben quedar 3 comandos md5 ejecutando a la vez, 2 en el termbaja). ¿Qué sucede con el uso de la CPU? ¿Por qué?

## Namespaces

#### 1. Explique el concepto de namespaces.

#### 2. ¿Cuáles son los posibles namespaces disponibles?

#### 3. ¿Cuáles son los namespaces de tipo Net, IPC y UTS una vez que inicie el sistema (los que se iniciaron la ejecutar la VM de la cátedra)?

#### 4. ¿Cuáles son los namespaces del proceso cron? Compare los namespaces net, ipc y uts con los del punto anterior, ¿son iguales o diferentes?

#### 5. Usando el comando `unshare` crear un nuevo namespace de tipo UTS.

##### a. `unshare --uts sh`

##### b. ¿Cuál es el nombre del host en el nuevo namespace? (comando hostname)

##### c. Ejecutar el comando `lsns`. ¿Qué puede ver con respecto a los namespace?.

##### d. Modificar el nombre del host en el nuevo hostname.

##### e. Abrir otra sesión, ¿cuál es el nombre del host anfitrión?

##### f. Salir del namespace (`exit`). ¿Qué sucedió con el nombre del host anfitrión?

#### 6. Usando el comando `unshare` crear un nuevo namespace de tipo Net.

##### a. `unshare --pid sh`

##### b. ¿Cuál es el PID del proceso sh en el namespace? ¿Y en el host anfitrión?

##### c. Ayuda: los PIDs son iguales. Esto se debe a que en el nuevo namespace se sigue viendo el comando `ps` sigue viendo el /proc del host anfitrión. Para evitar esto (y lograr un comportamiento como los contenedores), ejecutar: `unshare --pid --fork --mount-proc`

##### d. En el nuevo namespace ejecutar `ps -ef`. ¿Qué sucede ahora?

##### e. Salir del namespace
