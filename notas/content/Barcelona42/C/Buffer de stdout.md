---
tags: [c, stdio, buffer, printf, write, salida]
aliases: ["Buffer de printf", "Buffering de stdio", "fflush", "stdout con buffer"]
---
# Buffer de stdout

[[Inicio|← Inicio]] · C

> `printf` **no escribe cuando la llamas**. Guarda el texto en un buffer (un array en memoria) y lo manda al sistema de golpe, con un solo `write`, cuando le conviene. `write` en cambio escribe **al momento**. Por eso, si mezclas los dos, el orden de la salida puede cambiar según dónde vaya: a la terminal, a un fichero o a una tubería.

## Vocabulario

| Término | Qué significa |
|---|---|
| **`stdout`** | La salida estándar vista desde `<stdio.h>`: un `FILE *` que por debajo escribe en el fd `1` |
| **Buffer** | Array en memoria donde `printf` acumula el texto antes de escribirlo |
| **Vaciar** (*flush*) | Mandar con `write` lo que hay en el buffer y dejarlo vacío |
| **Llamada al sistema** (*syscall*) | Petición al kernel, como `write` o `read`. Es cara: por eso `printf` las agrupa |
| **Buffer de línea** | Se vacía en cada `'\n'` (o si se llena) |
| **Buffer completo** | Se vacía solo cuando se llena o al acabar el programa |
| **Sin buffer** | Cada `printf` se escribe al momento |

## Los tres modos y cuándo se usa cada uno

| Salida | Modo | Se vacía… |
|---|---|---|
| `stdout` hacia una **terminal** | Buffer de línea | En cada `'\n'`, si se llena, o al salir |
| `stdout` hacia un **fichero o tubería** (`> f`, `\| cat`) | Buffer completo | Si se llena (`BUFSIZ`, 8192 bytes aquí) o al salir |
| `stderr` | Sin buffer | Siempre al momento |

La biblioteca decide el modo al arrancar el programa, mirando si el fd 1 es una terminal.

## El experimento: printf y write mezclados

```c
#include <stdio.h>
#include <unistd.h>

int	main(void)
{
	printf("1. printf\n");
	write(1, "2. write\n", 9);
	printf("3. printf\n");
	write(1, "4. write\n", 9);
	return (0);
}
```

Salida real:

```text
$ ./orden               $ ./orden | cat          $ ./orden > f.txt; cat f.txt
1. printf               2. write                 2. write
2. write                4. write                 4. write
3. printf               1. printf                1. printf
4. write                3. printf                3. printf
```

- **En la terminal**, cada `printf` acaba en `'\n'` y se vacía al momento: sale en orden.
- **En una tubería o un fichero**, los `printf` se quedan en el buffer y salen **al final del programa**, después de los `write`.

### Verlo con `strace`

`strace` enseña las llamadas al sistema que hace el programa:

```text
$ strace -e trace=write ./orden > /dev/null
write(1, "2. write\n", 9)               = 9
write(1, "4. write\n", 9)               = 9
write(1, "1. printf\n3. printf\n", 20)  = 20
```

Los dos `printf` se han convertido en **un único `write`** de 20 bytes, al salir. Eso es todo el buffer: menos llamadas al sistema, más rápido.

### Lo mismo con `stderr`

```text
$ ./stderr 2>&1 | cat
2. stderr
4. stderr
1. stdout
3. stdout
```

`stderr` no tiene buffer: sus mensajes salen al momento. Por eso los mensajes de error y de depuración se escriben ahí (`fprintf(stderr, ...)`): aparecen aunque el programa crashee después.

## Cuándo se pierde lo que hay en el buffer

El buffer se vacía al terminar el programa **de forma normal** (`return` desde `main` o `exit`). Si el programa muere de otra forma, lo que había en el buffer **no se escribe nunca**.

### Un crash

```c
p = NULL;
printf("antes del crash");     /* sin '\n': se queda en el buffer */
*p = 'x';                      /* segfault */
```

```text
$ ./crash
Segmentation fault (core dumped)
```

El texto «antes del crash» no aparece, aunque el `printf` se ejecutó. Si depuras con `printf` y no ves un mensaje, **no significa que esa línea no se ejecutara**. Usa `fprintf(stderr, ...)`, añade `'\n'` o, mejor, [[Depurar con gdb|gdb]].

### `exit` frente a `_exit`

```c
printf("hola sin salto de línea");
if (argc > 1)
	_exit(0);       /* sale sin vaciar el buffer */
exit(0);            /* vacía el buffer y sale */
```

```text
$ ./exit
hola sin salto de línea
$ ./exit x
$
```

`_exit` es la llamada al sistema directa: termina sin pasar por `<stdio.h>`. Te lo encontrarás en `pipex` y `minishell`, donde tras un `fork` el hijo debe salir con `_exit` para no escribir dos veces lo que había en el buffer del padre.

## Cómo controlarlo

| Qué | Cómo |
|---|---|
| Vaciar ahora | `fflush(stdout);` |
| Quitar el buffer para todo el programa | `setvbuf(stdout, NULL, _IONBF, 0);` al principio del `main` |
| Que algo salga siempre al momento | Escribirlo en `stderr` |

Con `setvbuf(stdout, NULL, _IONBF, 0)` al principio, el experimento de arriba sale en orden **incluso** con `| cat`:

```text
$ ./nobuf | cat
1. printf
2. write
3. printf
4. write
```

## Por qué importa en 42

- **ft_printf**: el subject dice que **no** hay que implementar el buffer de `printf`. Tu `ft_printf` escribe con `write` al momento, así que si lo comparas con `printf` en el mismo programa la salida sale desordenada. No es un fallo tuyo: usa `fflush(stdout)` tras cada `printf` o compara en ficheros separados (ver [[ft_printf - Pruebas y depuración#2. No mezcles printf y ft_printf en la terminal]]).
- **get_next_line**: la idea es la misma, pero al leer. `read` trae bloques de `BUFFER_SIZE` y tú guardas lo que sobra (ver [[get_next_line - Guía paso a paso]]).
- **Depurar con `printf`**: un mensaje sin `'\n'` antes de un crash no sale nunca.
- **`fork`** (pipex, minishell): si haces `printf` sin vaciar y luego `fork`, el hijo hereda **una copia del buffer** y el texto puede salir dos veces.

## Errores frecuentes

> [!danger] Lo que confunde
> - Pensar que tu `ft_printf` está mal porque sale desordenado junto a `printf` en una tubería.
> - Depurar un crash con `printf("aquí")` sin `'\n'` y concluir que esa línea no se ejecuta.
> - Creer que `'\n'` siempre vacía el buffer: solo en una terminal.
> - Salir de un hijo con `exit` tras un `fork` y ver mensajes repetidos.

## Relacionadas

[[printf]] · [[Descriptores de fichero y read]] · [[Control de errores en C]] · [[Depurar con gdb]] · [[ft_printf - Pruebas y depuración]] · [[get_next_line - Guía paso a paso]]

## Referencias

- [setbuf(3), man7.org](https://man7.org/linux/man-pages/man3/setbuf.3.html): los tres modos, `setvbuf` y que `stdout` va con buffer de línea en una terminal.
- [stdout(3), man7.org](https://man7.org/linux/man-pages/man3/stdout.3.html): `stderr` sin buffer.
- [fflush(3), man7.org](https://man7.org/linux/man-pages/man3/fflush.3.html).
- [exit(3)](https://man7.org/linux/man-pages/man3/exit.3.html) y [_exit(2)](https://man7.org/linux/man-pages/man2/_exit.2.html): cuál vacía los buffers y cuál no.
- Salidas reales con Ubuntu clang 12.0.1 y glibc en Ubuntu 22.04 (`BUFSIZ` = 8192).
