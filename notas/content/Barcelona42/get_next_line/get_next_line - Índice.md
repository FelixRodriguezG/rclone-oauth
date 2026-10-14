---
tags: [proyecto, get_next_line, indice, rank-1, ficheros, static]
aliases: ["get_next_line", "gnl", "GNL"]
---
# get_next_line - Índice

[[Inicio|← Inicio]] · Proyectos · Rank 1

> **get_next_line** es una función que, cada vez que la llamas, devuelve **la siguiente línea** de un descriptor de fichero. Para lograrlo tiene que recordar entre llamadas lo que ya leyó de más, y eso te obliga a entender `read`, las [[Variables static y ámbito|variables `static`]] y la gestión de memoria sin leaks.

> [!warning] Comprueba el subject
> Resumen basado en versiones públicas del subject; los detalles cambian entre versiones. Lo que manda es el PDF de tu intra.

## Qué se pide

Lo comúnmente conocido del subject:

| Aspecto | Lo habitual |
|---|---|
| Prototipo | `char *get_next_line(int fd);` |
| Ficheros | `get_next_line.c`, `get_next_line_utils.c`, `get_next_line.h` |
| Funciones externas | `read`, `malloc`, `free` |
| Prohibido | Usar libft, `lseek` y variables globales |
| Devuelve | La línea leída **incluido el `'\n'`** (salvo si el fichero termina sin él), o `NULL` si no queda nada o hay un error |
| Compilación | Con `-D BUFFER_SIZE=n` y también **sin** esa opción (necesitas un valor por defecto) |
| Indefinido | Que el fichero cambie entre llamadas, o leer un fichero binario |
| Makefile | Normalmente **no** se entrega: el evaluador compila a mano |

> [!example] Uso
> ```c
> int		fd;
> char	*line;
>
> fd = open("poema.txt", O_RDONLY);
> line = get_next_line(fd);
> while (line)
> {
> 	printf("%s", line);
> 	free(line);
> 	line = get_next_line(fd);
> }
> close(fd);
> ```
> Quien llama es responsable de hacer `free` de cada línea.

### Bonus

- Usar **una sola variable `static`**.
- Gestionar **varios fd a la vez**: leer una línea del fd 3, luego del 4, luego del 3 otra vez, sin perder el hilo de cada uno.
- Ficheros con sufijo `_bonus`: `get_next_line_bonus.c`, `get_next_line_utils_bonus.c`, `get_next_line_bonus.h`.

## Conceptos que necesitas

| Concepto | Para qué en GNL | Nota |
|---|---|---|
| `open`, `read`, `close` | Leer bytes del fichero en trozos | [[Descriptores de fichero y read]] |
| Variables `static` | Recordar el resto entre llamadas | [[Variables static y ámbito]] |
| Memoria dinámica | Reservar y liberar líneas y buffers | [[Segmentos de memoria en C]] |
| Fin de cadena | Saber dónde acaba lo acumulado | [[Carácter nulo y puntero NULL]] |
| Macros del preprocesador | `BUFFER_SIZE` con valor por defecto | [[Preprocesador y headers]] |
| Leaks y accesos inválidos | Comprobar que no se pierde memoria | [[Valgrind y sanitizers]] |
| Depuración | Seguir el estado de la `static` paso a paso | [[Depurar con gdb]] |

Funciones de libft que tendrás que **reescribir** dentro de `get_next_line_utils.c` (no puedes enlazar libft): [[ft_strlen]], [[ft_strchr]], [[ft_strjoin]], [[ft_substr]] o equivalentes.

## Vocabulario

| Término | Significado |
|---|---|
| **fd** (*file descriptor*) | Número entero que el sistema te da al abrir un fichero. `0` es la entrada estándar (stdin) |
| **`BUFFER_SIZE`** | Cuántos bytes pides a `read` en cada llamada |
| **stash** (reserva) | Cadena `static` donde acumulas lo leído y aún no devuelto |
| **EOF** (*end of file*) | Fin del fichero: `read` devuelve `0` |

## `BUFFER_SIZE` y por qué no coincide con las líneas

`read(fd, buf, BUFFER_SIZE)` lee **como mucho** `BUFFER_SIZE` bytes, sin saber nada de líneas. Puede traer media línea o tres líneas y pico. Por eso necesitas acumular.

En el header define un valor por defecto para que compile sin `-D`:

```c
#ifndef BUFFER_SIZE
# define BUFFER_SIZE 42
#endif
```

```bash
cc -Wall -Wextra -Werror -D BUFFER_SIZE=1 get_next_line.c get_next_line_utils.c main.c
cc -Wall -Wextra -Werror -D BUFFER_SIZE=10000000 get_next_line.c get_next_line_utils.c main.c
cc -Wall -Wextra -Werror get_next_line.c get_next_line_utils.c main.c
```

## Por qué hace falta `static`

Una variable local normal **desaparece** al salir de la función. Una variable local `static` vive en el segmento de datos durante todo el programa y conserva su valor entre llamadas (ver [[Variables static y ámbito]] y [[Segmentos de memoria en C]]).

```text
Fichero: "hola\nmundo\n"     BUFFER_SIZE = 8

1.ª llamada: read → "hola\nmun"   devuelves "hola\n"   te sobra "mun"
2.ª llamada: si no guardaste "mun", lo has perdido y devolverías "do\n" ❌
             con static stash = "mun" → read → "do\n" → "mundo\n" ✅
```

## Algoritmo paso a paso

Cuatro pasos en cada llamada: **leer**, **acumular**, **extraer la línea**, **guardar el resto**.

```text
Fichero: "ab\ncdef\ng"          BUFFER_SIZE = 4

LLAMADA 1
  stash = NULL
  read → "ab\nc"               stash = "ab\nc"         ¿hay '\n'? sí → dejo de leer
  extraer hasta '\n' incluido  línea = "ab\n"          ← se devuelve
  guardar el resto             stash = "c"

LLAMADA 2
  stash = "c"                  ¿hay '\n'? no → leer
  read → "def\n"               stash = "cdef\n"        ¿hay '\n'? sí
  línea = "cdef\n"                                     ← se devuelve
  resto = ""                   stash = NULL (no queda nada, libero)

LLAMADA 3
  stash = NULL
  read → "g"   (1 byte)        stash = "g"
  read → 0     (EOF)           dejo de leer
  línea = "g"  (sin '\n')                              ← se devuelve
  stash = NULL

LLAMADA 4
  read → 0 y stash vacío       → return NULL           ← fin
```

Dibujo de la memoria en la llamada 2:

```text
 static stash ──► [ c | d | e | f |\n |\0 ]   (heap, malloc)
                     └────── línea ──────┘
 line ─────────► [ c | d | e | f |\n |\0 ]   (copia nueva, la libera quien llama)
 stash ────────► NULL                         (el antiguo se ha liberado con free)
```

### Esqueleto orientativo

Cumple la [[La Norma|Norma]]. Las funciones auxiliares (`ft_strchr` que acepte `NULL`, `ft_strjoin_free` que libere el primer argumento, `ft_extract_line`, `ft_save_rest`, `ft_free`) las escribes tú.

```c
static char	*ft_read_to_stash(int fd, char *stash)
{
	char	*buf;
	ssize_t	bytes;

	buf = malloc((size_t)BUFFER_SIZE + 1);
	if (!buf)
		return (ft_free(&stash));
	bytes = 1;
	while (bytes > 0 && !ft_strchr(stash, '\n'))
	{
		bytes = read(fd, buf, BUFFER_SIZE);
		if (bytes == -1)
		{
			free(buf);
			return (ft_free(&stash));
		}
		buf[bytes] = '\0';
		stash = ft_strjoin_free(stash, buf);
		if (!stash)
			break ;
	}
	free(buf);
	if (stash && !*stash)
		return (ft_free(&stash));
	return (stash);
}

char	*get_next_line(int fd)
{
	static char	*stash;
	char		*line;

	if (fd < 0 || BUFFER_SIZE <= 0)
		return (NULL);
	stash = ft_read_to_stash(fd, stash);
	if (!stash)
		return (NULL);
	line = ft_extract_line(stash);
	if (!line)
	{
		free(stash);
		stash = NULL;
		return (NULL);
	}
	stash = ft_save_rest(stash);
	return (line);
}
```

Puntos clave del esqueleto:

- `buf[bytes] = '\0'`: `read` no pone el `'\0'`; sin él, `ft_strjoin` leería basura. Por eso se reservan `BUFFER_SIZE + 1` bytes.
- El buffer va en el **heap** (`malloc`), no en la pila: `char buf[BUFFER_SIZE]` con un `BUFFER_SIZE` de 10 millones desborda la pila (normalmente 8 MB en Linux) y el programa crashea.
- El *cast* `(size_t)BUFFER_SIZE + 1` evita desbordar un `int` si alguien compila con `BUFFER_SIZE=2147483647`.
- Si `read` devuelve `-1`, **se libera el stash**: si no, queda memoria perdida y la siguiente llamada devolvería datos viejos.

## Casos límite

| Caso | Qué debe pasar | Cómo provocarlo |
|---|---|---|
| fd inválido (`-1`, `42`, cerrado) | `NULL`, sin crash ni leak | `get_next_line(-1)`; `close(fd)` y volver a llamar |
| `BUFFER_SIZE=1` | Funciona (muy lento, muchas llamadas a `read`) | `-D BUFFER_SIZE=1` |
| `BUFFER_SIZE` enorme | Funciona si el buffer está en el heap | `-D BUFFER_SIZE=10000000` |
| `BUFFER_SIZE <= 0` | `NULL` | `-D BUFFER_SIZE=0` |
| Fichero vacío | `NULL` en la primera llamada | `touch vacio.txt` |
| Sin `'\n'` final | Última línea **sin** `'\n'` y luego `NULL` | `printf 'a\nb' > f` |
| Solo saltos de línea | Devuelve `"\n"` tantas veces como haya | `printf '\n\n' > f` |
| Línea muy larga | Varias lecturas acumuladas en una línea | Una línea de 100 000 caracteres |
| stdin (fd `0`) | Devuelve cada línea al pulsar Intro; `Ctrl+D` es EOF | `./a.out < fichero` o escribir a mano |

> [!tip] Probar con stdin
> ```bash
> ./gnl_test              # el main lee de fd 0; escribe líneas y pulsa Ctrl+D para terminar
> cat poema.txt | ./gnl_test
> ```

## Leaks

> [!danger] Memoria que se suele perder
> - Línea devuelta que el `main` no libera (culpa del `main`, pero el evaluador lo verá).
> - El `stash` antiguo al hacer `stash = ft_strjoin(stash, buf)` sin liberar el anterior.
> - El `stash` cuando `read` devuelve `-1`.
> - El `buf` en cada salida de la función (también en la de error).

Comprueba con [[Valgrind y sanitizers]]:

```bash
cc -g -D BUFFER_SIZE=3 get_next_line.c get_next_line_utils.c main.c -o gnl_test
valgrind --leak-check=full --show-leak-kinds=all ./gnl_test poema.txt
```

> [!note] «still reachable» al cortar la lectura
> Si tu `main` deja de llamar a `get_next_line` antes del EOF, el `stash` sigue reservado al salir. Valgrind lo marca como *still reachable* (no *definitely lost*). Hay debate sobre si cuenta como leak; lo que el subject mira en cada versión, compruébalo en el PDF. Si lees hasta el `NULL`, no debería quedar nada.

## Bonus: varios fd

La idea más habitual es convertir la `static` en un **array de punteros indexado por fd** (sigue siendo una única variable `static`):

```c
char	*get_next_line(int fd)
{
	static char	*stash[FD_MAX];
	char		*line;

	if (fd < 0 || fd >= FD_MAX || BUFFER_SIZE <= 0)
		return (NULL);
	stash[fd] = ft_read_to_stash(fd, stash[fd]);
	/* … igual que antes, usando stash[fd] … */
}
```

```text
stash[3] ──► "resto del fichero A"
stash[4] ──► "resto del fichero B"
stash[5] ──► NULL
```

Es un fragmento incompleto: el comentario dentro de la función lo marcaría norminette.

`FD_MAX` es una macro tuya (por ejemplo `1024`, el límite de fds abiertos por defecto en muchos Linux, que puedes ver con `ulimit -n`). `OPEN_MAX` no siempre está definida en `<limits.h>` de glibc, así que no cuentes con ella. Comprobar `fd >= FD_MAX` evita escribir fuera del array.

## Plan de trabajo

1. Escribe un programa que abra un fichero y lo imprima entero con `read` en un bucle. Ver [[Descriptores de fichero y read]].
2. Prueba un contador `static` en una función para ver que conserva el valor ([[Variables static y ámbito]]).
3. Escribe las utilidades: `ft_strlen`, `ft_strchr` (que acepte `NULL`), `ft_strjoin` que libere el primero.
4. Implementa los cuatro pasos y prueba con `BUFFER_SIZE=1`, `42` y `10000000`.
5. Prueba los casos límite de la tabla, uno a uno.
6. Pasa valgrind en todos ellos.
7. Pasa norminette y un tester público.
8. Bonus: convierte la `static` en array y prueba alternando dos ficheros.

## Errores frecuentes que suspenden

> [!danger] Lista de comprobación antes de entregar
> - Usar libft, `lseek` o una variable global.
> - No compilar sin `-D BUFFER_SIZE` (falta el `#ifndef`).
> - Buffer en la pila: crash con `BUFFER_SIZE` grande.
> - Olvidar `buf[bytes] = '\0'`.
> - Devolver la línea sin `'\n'` cuando sí lo había, o devolver `""` en vez de `NULL` al final.
> - No liberar el `stash` en caso de error de `read`.
> - Leer el fichero entero en la primera llamada (va en contra del espíritu del proyecto y del subject).
> - En el bonus, usar más de una `static` o no comprobar el rango del fd.

## Preguntas de evaluación

> [!question] ¿Qué es una variable `static` local y dónde vive?
> Una variable cuyo valor se conserva entre llamadas. No está en la pila: vive en el segmento de datos (`.data` o `.bss`) durante todo el programa. Solo es visible dentro de la función.

> [!question] ¿Qué pasa con `BUFFER_SIZE=1`? ¿Y con 10 000 000?
> Con 1 funciona pero hace una llamada a `read` por byte (lento). Con 10 millones funciona si el buffer se reserva con `malloc`; en la pila desbordaría.

> [!question] ¿Qué devuelve `read`?
> El número de bytes leídos (puede ser menor que el pedido), `0` en EOF y `-1` en error.

> [!question] ¿Por qué tu función devuelve la línea con el `'\n'`?
> Porque así se distingue una última línea que no termina en `'\n'` de una que sí. Lo pide el subject.

> [!question] ¿Cómo gestionas varios fd en el bonus?
> Con un array `static` de punteros indexado por el número de fd. Cada fd tiene su propio resto.

> [!question] Enséñame que no hay leaks
> Ten preparado un `main` y el comando de valgrind. Prueba también con un fd inválido y con un fichero vacío.

## Relacionadas

[[Descriptores de fichero y read]] · [[Variables static y ámbito]] · [[Segmentos de memoria en C]] · [[Valgrind y sanitizers]] · [[Preprocesador y headers]] · [[ft_strjoin]] · [[ft_printf - Índice]] · [[Born2beroot - Índice]] · [[libft - Índice]]

## Referencias

- [read(2) — Linux man-pages](https://man7.org/linux/man-pages/man2/read.2.html) — valores de retorno (bytes leídos, `0` en EOF, `-1` en error) y lecturas parciales.
- [open(2) — Linux man-pages](https://man7.org/linux/man-pages/man2/open.2.html) — `O_RDONLY` y descriptores de fichero.
- [close(2) — Linux man-pages](https://man7.org/linux/man-pages/man2/close.2.html) — cerrar un fd; usar un fd cerrado da `EBADF`.
- [malloc(3) — Linux man-pages](https://man7.org/linux/man-pages/man3/malloc.3.html) y [free(3)](https://man7.org/linux/man-pages/man3/free.3.html) — memoria dinámica.
- [Valgrind: Memcheck manual](https://valgrind.org/docs/manual/mc-manual.html) — tipos de leak (*definitely lost*, *still reachable*).
- [gnlTester (Tripouille)](https://github.com/Tripouille/gnlTester) — tester público de la comunidad.
- Requisitos del proyecto: conocimiento general de versiones públicas del subject de get_next_line (no verificado contra el PDF). El esqueleto de código se ha compilado y probado con `BUFFER_SIZE` 1, 3, 42 y 10 000 000 y valgrind sin errores.
