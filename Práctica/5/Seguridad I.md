<h1 align="center">Práctica 5</h1>

## Seguridad I

### Notas:

### Utilizar un kernel completo (no el compilado en las prácticas 1 y 2).

### Compilar el código C usando el Makefile provisto a fin de deshabilitar algunas medidas de seguridad del compilador y generar un código assembler más simple.

### Acceda al código necesario para la práctica en el repositorio de la materia.

### A - Introducción

#### 1. Defina política y mecanismo.

- Una **política** define qué se permite y qué no: reglas de seguridad.
- Un **mecanismo** define cómo se implementa y se hace cumplir cada regla de seguridad. Son las aplicaciones de las políticas.

#### 2. Defina objeto, dominio y right.

- Un **objeto** es lo que se quiere proteger. Puede ser de hardware o software. Cada objeto tiene un ID.
- Un **dominio** es un conjunto de pares (objeto, derecho). Cada par especifica un objeto y un conjunto de operaciones que se pueden realizar con él.
- Un **derecho** implica autorización para efectuar esas operaciones.

#### 3. Defina POLA (Principle of least authority).

El POLA (Principle Of Least Authority) dice que cada componente (usuario, proceso, programa) debe tener **solo los privilegios mínimos necesarios para realizar su tarea**.

En otras palabras, define que los procesos accedan sólo a los objetos que necesitan (con los derechos que necesiten) para completar su tarea.

Este principio es fundamental por varias razones:

- **Reduce riesgos**: menos privilegios = menos daño si hay un fallo o una intrusión.
- **Limita el impacto de errores o ataques**: un proceso comprometido no puede afectar más de lo necesario.
- **Mejora la seguridad global del sistema**.

#### 4. ¿Qué valores definen el dominio en UNIX?

En Unix el dominio está definido por el UID y el GID:

- Dado un par **(UID, GID)** hay un conjunto de objetos a los cuales se puede acceder con ciertos permisos.
- Dos procesos con igual **(UID, GID)** pertenecen al mismo dominio, y por ende pueden acceder al mismo conjunto de archivos.

#### 5. ¿Qué es ASLR (Address Space Layout Randomization)? ¿Linux provee ASLR para los procesos de usuario? ¿Y para el kernel?

ASLR es una técnica de seguridad informática que busca prevenir la explotación de vulnerabilidades de corrupción de memoria, como los buffer overflows.

ASLR funciona aleatorizando las posiciones en la memoria de áreas clave de un programa, incluyendo:

- **La base del ejecutable**: La dirección donde comienza el código del programa.
- **La pila**: Donde se almacenan las variables locales y las llamadas a funciones.
- **La heap**: La memoria dinámica que el programa solicita durante su ejecución.
- **Las librerías compartidas**: Archivos de código reutilizable que los programas cargan en tiempo de ejecución (ej. libc).

Al hacer que estas direcciones de memoria sean diferentes en cada ejecución, ASLR dificulta que un atacante pueda predecir dónde se encuentran los datos o el código que necesita para ejecutar un ataque. Si un atacante intenta saltar a una dirección de memoria fija, es muy probable que el programa falle en lugar de ejecutar el código malicioso, ya que la dirección habrá cambiado.

Linux provee ASLR para los procesos de usuario vía el archivo `/proc/sys/kernel/randomize_va_space`.

**Para que la aleatorización sea efectiva, los programas también deben estar compilados como "Position Independent Executables" (PIE).**

Linux también provee ASLR para el kernel, conocido como KASLR.

El objetivo de KASLR es el mismo que el de ASLR para el espacio de usuario: aleatorizar las ubicaciones de los componentes del kernel en la memoria para dificultar los ataques que intentan explotar vulnerabilidades en el propio kernel. Cuando KASLR está habilitado, el kernel carga sus propios códigos y datos en direcciones aleatorias en cada arranque del sistema.

**Para que KASLR funcione, el kernel de Linux debe estar compilado con la opción CONFIG_RANDOMIZE_BASE habilitada.**

#### 6. ¿Cómo se activa/desactiva ASLR para todos los procesos de usuario en Linux?

El ASLR se puede configurar para todos los procesos de usuario a través del archivo `/proc/sys/kernel/randomize_va_space`. Los valores posibles son:

- **0**: No hay aleatorización. Todas las direcciones de memoria son estáticas.
- **1**: Aleatorización conservadora. Las librerías compartidas, la pila, `mmap()` y la página VDSO se aleatorizan.
- **2**: Aleatorización completa. Además de lo anterior, la memoria gestionada a través de `brk()` también se aleatoriza. Este suele ser el valor por defecto en los kernel Linux modernos.

### B - Ejercicio introductorio: Buffer Overflow simple

#### El propósito de este ejercicio es que las y los estudiantes tengan una introducción simple a un stack buffer overflow a fin de poder abordar el siguiente ejercicio. Las y los estudiantes aprenderán a identificar la vulnerabilidad, analizar la disposición de la memoria y construir una entrada que aproveche la vulnerabilidad para obtener acceso no autorizado a una función privilegiada.

#### Nota: Puede ser de ayuda ver el código assembler generado al compilar (`00-stack-overflow.s`) o utilizar `gdb` para depurar el programa pero no es obligatorio.

#### Tip del profe sobre el payload para `01-stack-overflow-ret`: "En la explicación vieron que probé un payload bastante grande (128 bytes de relleno antes del puntero) y no funcionó, bueno el payload en realidad es mucho más chico, el enunciado de la práctica los guía para que lo puedan calcular sin usar el debugger."

#### 1. Usando el makefile provisto, compilar el ejemplo `00-stack-overflow.c` provisto en el repositorio de la cátedra.

```sh
juan@juan-Lenovo-IdeaPad-S145-15AST:~/Desktop/Downloads/codigo-para-practicas/practica5$ make
cc -save-temps -g -fno-stack-protector -z execstack -no-pie -fcf-protection=none -O0    00-stack-overflow.c   -o 00-stack-overflow
00-stack-overflow.c: In function ‘login’:
00-stack-overflow.c:19:5: warning: implicit declaration of function ‘gets’; did you mean ‘fgets’? [-Wimplicit-function-declaration]
   19 |     gets(password);             // Vulnerable function reads without bounds
      |     ^~~~
      |     fgets
/usr/bin/ld: 00-stack-overflow.o: in function `login':
/home/juan/Desktop/Downloads/codigo-para-practicas/practica5/00-stack-overflow.c:19:(.text+0x6d): warning: the `gets' function is dangerous and should not be used.
cc -save-temps -g -fno-stack-protector -z execstack -no-pie -fcf-protection=none -O0    01-stack-overflow-ret.c   -o 01-stack-overflow-ret
/usr/bin/ld: 01-stack-overflow-ret.o: in function `login':
/home/juan/Desktop/Downloads/codigo-para-practicas/practica5/01-stack-overflow-ret.c:33:(.text+0x11a): warning: the `gets' function is dangerous and should not be used.
```

#### 2. Ejecutar el programa y observar las direcciones de las variables access y password, así como la distancia entre ellas.

```sh
juan@juan-Lenovo-IdeaPad-S145-15AST:~/Desktop/Downloads/codigo-para-practicas/practica5$ ./00-stack-overflow
access pointer: 0x7ffcd6b9faff, password pointer: 0x7ffcd6b9fae0, distance: 31
Write password:
```

Las direcciones son:

- **access**: 0x7ffcd6b9faff
- **password**: 0x7ffcd6b9fae0

La distancia entre ellas es de 0x1F o 31 en decimal.

#### 3. Probar el programa con una password cualquiera y con "big secret" para verificar que funciona correctamente.

```sh
juan@juan-Lenovo-IdeaPad-S145-15AST:~/Desktop/Downloads/codigo-para-practicas/practica5$ ./00-stack-overflow
access pointer: 0x7ffd7ade81cf, password pointer: 0x7ffd7ade81b0, distance: 31
Write password: abc
Access denied
juan@juan-Lenovo-IdeaPad-S145-15AST:~/Desktop/Downloads/codigo-para-practicas/practica5$ ./00-stack-overflow
access pointer: 0x7ffd7e16a76f, password pointer: 0x7ffd7e16a750, distance: 31
Write password: big secret
Now you know the secret
```

#### 4. Volver a ejecutar pero ingresar una password lo suficientemente larga para sobreescribir access. Usar distance como referencia para establecer la longitud de la password.

Probando con password de 31 caracteres: `aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa`

```sh
juan@juan-Lenovo-IdeaPad-S145-15AST:~/Desktop/Downloads/codigo-para-practicas/practica5$ ./00-stack-overflow
access pointer: 0x7ffd44f4fc6f, password pointer: 0x7ffd44f4fc50, distance: 31
Write password: aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
Access denied
```

Probando con password de 32 caracteres: `aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa`

```sh
juan@juan-Lenovo-IdeaPad-S145-15AST:~/Desktop/Downloads/codigo-para-practicas/practica5$ ./00-stack-overflow
access pointer: 0x7ffc8921f0cf, password pointer: 0x7ffc8921f0b0, distance: 31
Write password: aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
Now you know the secret
```

Explicación del problema:

```c
int login() {
    int8_t access = 0;
    char password[16];
    printf("access pointer: %p, password pointer: %p, distance: %lu\n",
           &access, password, (char *) &access - password);
    printf("Write password: ");
    gets(password);             // Vulnerable function reads without bounds
    if (strcmp(password, "big secret") == 0) {
        access = 1;
    }
    return access;
}
```

- La función devuelve 1 si la contraseña es correcta o 0 si es incorrecta.
- Inicialmente, la variable flag que indica si la contraseña es correcta o no se inicializa en 0, es decir incorrecta.
- La variable access ocupa un solo byte.
- La variable password ocupa 16 bytes.
- Sin embargo la diferencia es 31. Esto significa que el compilador está agregando relleno (padding) entre variables para alinearlas correctamente.
- Gráficamente:

```
+---------------------+  ← Dirección más baja (top del stack)
| password[0]         |  ← Comienzo del buffer (16 bytes)
| password[1]         |
| ...                 |
| password[15]        |
+---------------------+
| padding (relleno)   |  ← Espacio extra por alineación (15 bytes)
+---------------------+
| access (1 byte)     |  ← Variable que controla el acceso (0 = denegado)
+---------------------+  ← Dirección más alta (bottom del stack)
```

- La función `gets()` lee entrada del usuario, pero es extremadamente peligrosa debido a su comportamiento no seguro.
- Cuando escribo las 32 'a's:

```
+---------------------+
| 'a' (password[0])   |  ← 1er byte
| 'a' (password[1])   |
| ...                 |
| 'a' (password[15])  |  ← Byte 16 (fin del buffer)
+---------------------+
| 'a' (padding[0])    |  ← Byte 17 (empieza a sobrescribir el padding)
| ...                 |
| 'a' (padding[14])   |  ← Byte 31
+---------------------+
| 'a' (access)        |  ← Byte 32: ¡Sobrescribe `access` con 'a' (0x41)!
+---------------------+
```

- Originalmente, access = 0 (acceso denegado).
- Al escribir 32 bytes, el último byte ('a' = 0x61) sobrescribe access.
- El if da falso, por lo cual no sobreescribe access con 1, pero no importa, porque el overflow ya sobreescribió access con 0x61 que es distinto de 0.
- Entonces, en la función `main()`, `if (login())` da verdadero, porque en C cualquier valor distinto de 0 es "verdadero".

**Nota**: El valor 0x61 ('a') es arbitrario. Cualquier byte ≠ 0 en la posición 32 haría el overflow.

#### 5. Después de realizar la explotación, reflexiona sobre las siguientes preguntas:

##### a. ¿Por qué el uso de `gets()` es peligroso?

El uso de `gets()` es peligroso porque:

- No verifica el tamaño del buffer donde almacena los datos.
- Si el usuario escribe más caracteres de los que caben en el buffer, `gets()` seguirá escribiendo más allá del espacio asignado, sobrescribiendo otras partes de la memoria (como otras variables, direcciones de retorno, etc.).

##### b. ¿Cómo se puede prevenir este tipo de vulnerabilidad?

Este tipo de vulnerabilidad se puede prevenir de varias formas:

- Usando funciones seguras como `fgets()` en vez de `gets()`.
- Validando la longitud de las entradas del usuario.
- Usando protecciones del compilador.
- Usando ASLR.

##### c. ¿Qué medidas de seguridad ofrecen los compiladores modernos para evitar estas vulnerabilidades?

Los compiladores modernos ofrecen varias medidas de seguridad para evitar el buffer overflow:

- **Stack Canaries**:

  - Inserta un valor secreto ("canario") antes del return address.
  - Si se modifica (por overflow), el programa se cierra.
  - Flag: `-fstack-protector-strong`

- **ASLR**:

  - Aleatoriza direcciones de memoria (stack, librerías).
  - Activación: `echo 2 > /proc/sys/kernel/randomize_va_space`

- **NX Bit**:

  - Hace el stack no-ejecutable (evita ejecutar shellcode).
  - Flag: `-z noexecstack`

- **Fortify Source**:

  - Reemplaza funciones inseguras (strcpy, gets) con versiones que verifican límites.
  - Flag: `-D_FORTIFY_SOURCE=2`

- **RELRO**:
  - Protege tablas de enlazado dinámico.
  - Flag: `-z now` (Full RELRO)

### C - Ejercicio: Buffer Overflow reemplazando dirección de retorno

#### Objetivo: El objetivo de este ejercicio es que las y los estudiantes comprendan cómo una vulnerabilidad de desbordamiento de búfer puede ser explotada para alterar la dirección de retorno de una función, redirigiendo la ejecución del programa a una función privilegiada. Además, se explorará el mecanismo de seguridad ASLR y cómo desactivarlo temporalmente para facilitar la explotación.

#### Nota: Puede ser de ayuda ver el código assembler generado al compilar (`01-stack-overflow-ret.s`) o utilizar `gdb` para depurar el programa pero no es obligatorio.

#### 1. Compilar usando el makefile provisto el ejemplo `01-stack-overflow-ret.c` provisto en el repositorio de la cátedra.

#### 2. Configurar setuid en el programa para que al ejecutarlo, se ejecute como usuario root.

#### 3. Verificar si tiene ASLR activado en el sistema. Si no está, actívelo.

#### 4. Ejecute `01-stack-overflow-ret` al menos 2 veces para verificar que la dirección de memoria de `privileged_fn()` cambia.

#### 5. Apague ASLR y repita el punto 3 para verificar que esta vez el proceso siempre retorna la misma dirección de memoria para `privileged_fn()`.

#### 6. Suponiendo que el compilador no agregó ningún padding en el stack tenemos los siguientes datos:

##### a. El stack crece hacia abajo.

##### b. Si estamos compilando en x86_64 los punteros ocupan 8 bytes.

##### c. x86_64 es little endian.

##### d. Primero se apiló la dirección de retorno (una dirección dentro de la función `main()`). Ocupa 8 bytes.

##### e. Luego se apiló la vieja base de la pila (`rbp`). Ocupa 8 bytes.

##### f. password ocupa 16 bytes.

##### Calcule cuántos bytes de relleno necesita para pisar la dirección de retorno.

#### 7. Ejecute el script `payload_pointer.py` para generar el payload. La ayuda se puede ver con:

```sh
python payload_pointer.py --help
```

#### 8. Pruebe el payload redirigiendo la salida del script a `01-stack-overflow-ret` usando un pipe.

#### 9. Para poder interactuar con el shell invoque el programa usando el argumento `--program` del script `payload_pointer`. Por ejemplo:

```sh
python payload_pointer.py --padding <padding> --pointer <pointer> --program ./01-stack-overflow-ret
```

#### 10. Pruebe algunos comandos para verificar que realmente tiene acceso a un shell con UID 0.

#### 11. Conteste:

##### a. ¿Qué efecto tiene setear el bit setuid en un programa si el propietario del archivo es `root`? ¿Qué efecto tiene si el usuario es por ejemplo `nobody`?

##### b. Compare el resultado del siguiente comando con la dirección de memoria de `privileged_fn()`. ¿Qué puede notar respecto a los octetos? ¿A qué se debe esto?

```sh
python payload_pointer.py --padding <padding> --pointer <pointer> | hd
```

##### c. ¿Cómo ASLR ayuda a evitar este tipo de ataques en un escenario real donde el programa no imprime en pantalla el puntero de la función objetivo?

##### d. ¿Cómo podría evitar este tipo de ataques en un módulo del kernel de Linux? ¿Qué mecanismo debería estar habilitado?

### C - Ejercicio SystemD

#### Objetivo: Aprender algunas restricciones de seguridad que se pueden aplicar a un servicio en SystemD.

#### https://www.redhat.com/en/blog/cgroups-part-four

#### https://www.redhat.com/en/blog/mastering-systemd

#### 1. Investigue los comandos:

##### a. `systemctl enable`

##### b. `systemctl disable`

##### c. `systemctl daemon-reload`

##### d. `systemctl start`

##### e. `systemctl stop`

##### f. `systemctl status`

##### g. `systemd-cgls`

##### h. `journalctl -u [unit]`

#### 2. Investigue las siguientes opciones que se pueden configurar en una unit service de `systemd`:

##### a. IPAddressDeny e IPAddressAllow

##### b. User y Group

##### c. ProtectHome

##### d. PrivateTmp

##### e. ProtectProc

##### f. MemoryAccounting, MemoryHigh y MemoryMax

#### 3. Tenga en cuenta para los siguientes puntos:

##### a. La configuración del servicio se instala en: `/etc/systemd/system/insecure_service.service`

##### b. Cada vez que modifique la configuración será necesario recargar el demonio de systemd y recargar el servicio:

###### i. `systemctl daemon-reload`

###### ii. `systemctl restart insecure_service.service`

#### 4. En el directorio insecure_service del repositorio de la cátedra encontrará el binario `insecure_service`, el archivo de configuración `insecure_service.service` y el script `install.sh`.

##### a. Instale el servicio usando el script `install.sh`.

##### b. Verifique que el servicio se está ejecutando con `systemctl status`.

##### c. Verifique con qué UID se ejecuta el servicio usando `psaux | grep insecure_service`.

##### d. Abra localhost:8080 en el navegador y explore los links provistos por este servicio.

#### 5. Configure el servicio para que se ejecute con usuario y grupo no privilegiados (en Debian y derivados se llaman nouser y nogroup). Verifique con qué UID se ejecuta el servicio usando `psaux | grep insecure_service`.

#### 6. Limite las IPs que pueden acceder al servicio para denegar todo por defecto y permitir solo conexiones de localhost (127.0.0.0/8).

#### 7. Explore el directorio `/home` y el directorio `/tmp` usando el servicio y luego:

##### a. Reconfigurelo para que no pueda visualizar el contenido de `/home` y tenga su propio `/tmp` privado.

##### b. Recargue el servicio y verifique que estas restricciones surgieron efecto.

#### 8. Limite el acceso a información de otros procesos por parte del servicio.

#### 9. Establezca un límite de 16M al uso de memoria del servicio e intente alocar más de esa memoria en la sección "Memoria" usando el link [Aumentar Reserva de Memoria](http://localhost:8080/mem/alloc).
