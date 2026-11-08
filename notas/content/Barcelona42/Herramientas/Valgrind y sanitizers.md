---
tags: [herramientas, valgrind, sanitizers, memoria-dinamica, debug, c]
aliases: ["Valgrind", "AddressSanitizer", "fsanitize"]
---
# Valgrind y sanitizers

[[Inicio|← Inicio]] · Herramientas

> **Valgrind** y los **sanitizers** (`-fsanitize=address`, `-fsanitize=undefined`) son detectores de errores de memoria. Ejecutan tu programa y te avisan, con fichero y línea, de **leaks** (memoria que reservaste y nunca liberaste), lecturas o escrituras fuera de un bloque, uso de memoria ya liberada y otros fallos que muchas veces **no crashean**… hasta el día de la evaluación.

> [!info] Por qué importa en 42
> En casi todos los proyectos del Common Core un leak o un acceso inválido es motivo de suspenso. Que tu programa «funcione» no significa que esté bien: un `s[5] = 'x'` sobre un bloque de 5 bytes suele no hacer nada visible en tu ordenador y crashear en el del evaluador.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Heap** (montón) | Zona de memoria de donde salen los bloques de `malloc`. Ver [[Segmentos de memoria en C]] |
| **Bloque** | Cada trozo que devuelve un `malloc`/`calloc` |
| **Leak** (fuga) | Un bloque que sigue reservado al terminar el programa |
| **Valor no inicializado** | Una variable o un byte de `malloc` que lees antes de haberle dado valor |
| **Use after free** | Usar un puntero después de haber hecho `free` sobre él |
| **Double free** | Hacer `free` dos veces del mismo bloque |
| **Instrumentar** | Añadir comprobaciones extra al programa (lo hace el compilador con `-fsanitize`) |

## Valgrind vs. sanitizers en una tabla

| | Valgrind (Memcheck) | `-fsanitize=address` (ASan) |
|---|---|---|
| ¿Hay que recompilar? | No (aunque conviene `-g`) | Sí, con la bandera |
| Velocidad | Muy lento (unas 20–30 veces más lento) | Rápido (unas 2 veces más lento) |
| Leaks | ✅ con 4 categorías | ✅ en Linux (LeakSanitizer). En macOS no viene activado por defecto |
| Desbordar un bloque de `malloc` | ✅ | ✅ |
| Desbordar un array **local** (pila) | ❌ | ✅ |
| Leer memoria no inicializada | ✅ | ❌ |
| Disponible en los Mac de 42 | ❌ | ✅ (para leaks usa `leaks`) |

> [!summary] Regla práctica
> Programa con `-fsanitize=address -g` mientras desarrollas, y **antes de entregar** pasa Valgrind con las opciones completas. Se complementan: cada uno ve cosas que el otro no.

## Paso 1: compilar con `-g`

Igual que con [[Depurar con gdb|gdb]], sin `-g` Valgrind no sabe qué línea de tu código provocó el error y solo enseña direcciones.

```bash
cc -Wall -Wextra -Werror -g main.c -o prog
```

Si usas tu libft, compílala también con `-g` (añade `-g` a `CFLAGS` en su [[Makefile y librerías estáticas|Makefile]]).

## Paso 2: ejecutar con Valgrind

```bash
valgrind --leak-check=full --show-leak-kinds=all --track-origins=yes ./prog arg1 arg2
```

| Opción | Para qué |
|---|---|
| `--leak-check=full` | Detalla **cada** leak con la línea del `malloc` que lo creó |
| `--show-leak-kinds=all` | Enseña también los «still reachable» (por defecto se ocultan) |
| `--track-origins=yes` | En errores de valores no inicializados, dice **de dónde salió** ese valor. Más lento, pero muy útil |
| `-q` | Modo silencioso: solo imprime los errores |

> [!tip] Alias
> Ponte un alias en `~/.zshrc`: `alias vg='valgrind --leak-check=full --show-leak-kinds=all --track-origins=yes'`. Así lanzas `vg ./prog`.

Cada línea de Valgrind empieza por `==PID==` (el número del proceso). Lo que imprime tu programa sale sin ese prefijo.

## Cómo leer los leaks

Programa de ejemplo con tres errores a propósito (`leak.c`):

```c
#include <stdlib.h>
#include <string.h>

typedef struct s_node
{
	char			*str;
	struct s_node	*next;
}	t_node;

static char	*g_keep;

int	main(void)
{
	char	*s;
	t_node	*n;

	s = malloc(10);          /* línea 17 */
	strcpy(s, "hola");
	s = NULL;                /* ❌ perdemos el único puntero */
	n = malloc(sizeof(t_node)); /* línea 20 */
	n->str = malloc(20);     /* línea 21 */
	n->next = NULL;
	n = NULL;                /* ❌ perdemos el nodo y, con él, n->str */
	g_keep = malloc(5);      /* línea 24: ❌ nunca se libera */
	return (0);
}
```

Salida real (Valgrind 3.18.1, Ubuntu 22.04):

```text
==2792855== HEAP SUMMARY:
==2792855==     in use at exit: 51 bytes in 4 blocks
==2792855==   total heap usage: 4 allocs, 0 frees, 51 bytes allocated
==2792855==
==2792855== 5 bytes in 1 blocks are still reachable in loss record 1 of 4
==2792855==    at 0x4848899: malloc (in /usr/libexec/valgrind/vgpreload_memcheck-amd64-linux.so)
==2792855==    by 0x4011B2: main (leak.c:24)
==2792855==
==2792855== 10 bytes in 1 blocks are definitely lost in loss record 2 of 4
==2792855==    at 0x4848899: malloc (in /usr/libexec/valgrind/vgpreload_memcheck-amd64-linux.so)
==2792855==    by 0x401158: main (leak.c:17)
==2792855==
==2792855== 20 bytes in 1 blocks are indirectly lost in loss record 3 of 4
==2792855==    at 0x4848899: malloc (...)
==2792855==    by 0x40118A: main (leak.c:21)
==2792855==
==2792855== 36 (16 direct, 20 indirect) bytes in 1 blocks are definitely lost in loss record 4 of 4
==2792855==    at 0x4848899: malloc (...)
==2792855==    by 0x40117C: main (leak.c:20)
==2792855==
==2792855== LEAK SUMMARY:
==2792855==    definitely lost: 26 bytes in 2 blocks
==2792855==    indirectly lost: 20 bytes in 1 blocks
==2792855==      possibly lost: 0 bytes in 0 blocks
==2792855==    still reachable: 5 bytes in 1 blocks
==2792855==         suppressed: 0 bytes in 0 blocks
```

Se lee **de abajo arriba**: la línea `by ... main (leak.c:17)` es **dónde se reservó** el bloque perdido, no dónde se perdió.

| Categoría | Qué ha pasado | En el ejemplo |
|---|---|---|
| **definitely lost** | Ya no queda ningún puntero al bloque. Leak seguro | `s` (línea 17) y el nodo (línea 20) |
| **indirectly lost** | Solo se puede llegar a él desde otro bloque que está perdido. Arregla el «definitely» y desaparece | `n->str` (línea 21) |
| **possibly lost** | Solo queda un puntero al **interior** del bloque (p. ej. hiciste `s++`) | — (ejemplo en [[#Possibly lost: un puntero al interior del bloque]]) |
| **still reachable** | Al salir aún había un puntero (global o `static`) al bloque, pero nadie hizo `free` | `g_keep` (línea 24) |

> [!warning] «Still reachable» también cuenta
> Mucha gente lo ignora porque Valgrind no lo marca como error. En 42 la mayoría de evaluadores lo consideran leak: si tu memoria sigue reservada al salir, libérala. Excepción habitual: memoria que reserva por dentro una biblioteca del sistema, que no puedes liberar tú.

> [!example] El caso de las listas
> En [[Listas enlazadas]], si liberas solo el primer nodo verás un «definitely lost» del segundo y un montón de «indirectly lost» del resto. La solución es recorrer la lista liberando cada nodo, como hace [[ft_lstclear]].

Cuando todo está bien verás:

```text
==PID== All heap blocks were freed -- no leaks are possible
==PID== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
```

## Cómo leer los errores de acceso

Programa con cuatro errores (`errs.c`):

```c
s = malloc(5);              /* línea 11 */
strcpy(s, "hola");          /* 5 bytes: 'h','o','l','a','\0' */
s[5] = 'x';                 /* línea 13: ❌ escribe fuera */
printf("%c\n", s[6]);       /* línea 14: ❌ lee fuera */
free(s);
arr = malloc(3 * sizeof(int)); /* línea 16 */
if (arr[0] > 0)             /* línea 17: ❌ arr[0] no tiene valor */
	x = 1;
free(arr);                  /* línea 19 */
free(arr);                  /* línea 20: ❌ double free */
```

### Invalid write / Invalid read

```text
==2793042== Invalid write of size 1
==2793042==    at 0x40118F: main (errs.c:13)
==2793042==  Address 0x4a9e045 is 0 bytes after a block of size 5 alloc'd
==2793042==    at 0x4848899: malloc (...)
==2793042==    by 0x401178: main (errs.c:11)
==2793042==
==2793042== Invalid read of size 1
==2793042==    at 0x401197: main (errs.c:14)
==2793042==  Address 0x4a9e046 is 1 bytes after a block of size 5 alloc'd
```

- **`size 1`**: escribiste/leíste 1 byte (un `char`). Con un `int` sería `size 4`, con un puntero `size 8`.
- **`errs.c:13`**: la línea culpable.
- **`0 bytes after a block of size 5`**: te pasaste justo por uno. Es el clásico olvido del `+ 1` para el `'\0'` (ver [[Carácter nulo y puntero NULL]] y [[ft_strdup]]).
- La segunda pila (`alloc'd at ... errs.c:11`) te dice qué `malloc` creó el bloque.

### Uso de memoria liberada

```text
==2794320== Invalid read of size 1
==2794320==    at 0x401188: main (uaf.c:14)
==2794320==  Address 0x4a9e040 is 0 bytes inside a block of size 4 free'd
==2794320==    at 0x484B27F: free (...)
==2794320==    by 0x401183: main (uaf.c:13)
==2794320==  Block was alloc'd at
==2794320==    at 0x4848899: malloc (...)
==2794320==    by 0x40116F: main (uaf.c:11)
```

`inside a block ... free'd` = **use after free**. Valgrind te da tres sitios: dónde lo usaste (14), dónde lo liberaste (13) y dónde se reservó (11).

### Valor no inicializado

```text
==2793042== Conditional jump or move depends on uninitialised value(s)
==2793042==    at 0x4011CA: main (errs.c:17)
==2793042==  Uninitialised value was created by a heap allocation
==2793042==    at 0x4848899: malloc (...)
==2793042==    by 0x4011BE: main (errs.c:16)
```

«Conditional jump» significa que un `if`/`while` decidió algo usando basura. La segunda parte («created by a heap allocation») solo aparece con `--track-origins=yes`. Soluciones: inicializar, usar [[ft_calloc]] o [[ft_bzero]].

### Invalid free

```text
==2793042== Invalid free() / delete / delete[] / realloc()
==2793042==    at 0x484B27F: free (...)
==2793042==    by 0x4011E8: main (errs.c:20)
==2793042==  Address 0x4a9f0d0 is 0 bytes inside a block of size 12 free'd
==2793042==    at 0x484B27F: free (...)
==2793042==    by 0x4011DF: main (errs.c:19)
```

Doble `free`, o `free` de algo que no salió de `malloc` (un array local, un literal `"hola"`). Truco: después de liberar, pon el puntero a `NULL`; `free(NULL)` no hace nada.

> [!tip] Arregla el primer error primero
> Un error suele provocar otros en cascada. Corrige el primero que aparezca, recompila y vuelve a pasar Valgrind.

## Leer una pila de varias funciones

En un proyecto real los errores no salen en `main`, sino varias llamadas más abajo. Valgrind enseña la **pila de llamadas** completa. Programa (`pila.c`) con el clásico olvido del `+ 1` dentro de un helper:

```c
static char	*ft_join(const char *a, const char *b)
{
	char	*r;

	r = malloc(strlen(a) + strlen(b));      /* línea 8: falta + 1 para el '\0' */
	strcpy(r, a);
	strcat(r, b);                           /* línea 10 */
	return (r);
}

static char	*leer_linea(void)
{
	return (ft_join("hola ", "mundo"));     /* línea 16 */
}

int	main(void)
{
	/* ... */
	linea = leer_linea();                   /* línea 23 */
```

```text
==3286074== Invalid write of size 1
==3286074==    at 0x484EA5E: strcat (in /usr/libexec/valgrind/vgpreload_memcheck-amd64-linux.so)
==3286074==    by 0x40120F: ft_join (pila.c:10)
==3286074==    by 0x4011BC: leer_linea (pila.c:16)
==3286074==    by 0x401183: main (pila.c:23)
==3286074==  Address 0x4a9e04a is 0 bytes after a block of size 10 alloc'd
==3286074==    at 0x4848899: malloc (in /usr/libexec/valgrind/vgpreload_memcheck-amd64-linux.so)
==3286074==    by 0x4011F1: ft_join (pila.c:8)
==3286074==    by 0x4011BC: leer_linea (pila.c:16)
==3286074==    by 0x401183: main (pila.c:23)
```

Cómo leerla:

| Parte | Qué dice |
|---|---|
| `at` (la primera línea) | La función donde **ocurrió** el acceso. Aquí es `strcat`, que **no es tuya** (viene de una librería: no tiene `fichero.c:línea` tuyo) |
| `by` (las siguientes) | Quién llamó a quién, **de dentro hacia fuera**: `ft_join` → la llamó `leer_linea` → la llamó `main` |
| La **primera línea con un fichero tuyo** | Es la que tienes que mirar: `ft_join (pila.c:10)` |
| El segundo bloque (`alloc'd`) | La pila **del `malloc`** que creó el bloque: `pila.c:8`. A menudo el fallo está aquí (el tamaño) y no donde se notó (el `strcat`) |

```text
leer de arriba abajo = de dentro afuera

  at  strcat          ← librería: sáltatela
  by  ft_join  :10    ← ⭐ tu código: aquí se escribió fuera
  by  leer_linea :16
  by  main     :23
```

> [!tip] Pregúntate «¿dónde se notó?» y «¿dónde se creó?»
> La línea 10 es donde **se notó** el error; la 8 es donde **se creó** (reservaste 10 bytes para 11 caracteres). Arréglalo en el `malloc`, no en el `strcat`.

> [!note] Sin `-g` no hay números de línea
> Si en lugar de `pila.c:10` ves solo `by 0x40120F: ft_join (in /home/.../pila)`, compilaste sin `-g`. Recuerda que en un proyecto con librería hay que poner `-g` **también** al compilar la librería, no solo el `main` (ver [[Depurar con gdb]]).

## Segfault dentro de Valgrind

Si el programa crashea, Valgrind te dice **dónde** y **a qué dirección** intentó acceder. Programa (`segv.c`) que cuenta los nodos de una lista sin comprobar que esté vacía:

```c
static int	contar(t_list *lst)
{
	/* ... */
	while (lst->next)          /* línea 14: con lst == NULL */
```

```text
==3286078== Invalid read of size 8
==3286078==    at 0x401183: contar (segv.c:14)
==3286078==    by 0x40114F: main (segv.c:27)
==3286078==  Address 0x8 is not stack'd, malloc'd or (recently) free'd
==3286078==
==3286078== Process terminating with default action of signal 11 (SIGSEGV)
==3286078==  Access not within mapped region at address 0x8
==3286078==    at 0x401183: contar (segv.c:14)
==3286078==    by 0x40114F: main (segv.c:27)
```

Sin Valgrind solo verías `Segmentation fault (core dumped)` y código de salida `139`.

- **`signal 11 (SIGSEGV)`**: el programa ha crasheado. Lo que va después es **dónde**.
- **`not stack'd, malloc'd or (recently) free'd`**: la dirección no es de ninguna variable, ni de `malloc`, ni de algo liberado hace poco. Es una dirección inventada: casi siempre un puntero `NULL` o sin inicializar.
- **`Invalid read of size 8`**: intentaba leer 8 bytes, el tamaño de un puntero: `lst->next`.

> [!example] Dirección pequeña = `NULL` + algo
> `Address 0x8` no es casualidad. `lst->next` lee el campo `next`, que está 8 bytes después del principio de la struct (detrás de `content`, ver [[Padding y alineación de memoria]]). Si `lst` es `NULL` (0), `lst->next` lee en `0 + 8 = 0x8`. **Una dirección muy pequeña (`0x0`, `0x8`, `0x10`…) casi siempre es un campo de un puntero `NULL`.** Una dirección enorme y rara suele ser un puntero sin inicializar.

## `Syscall param write(buf) points to uninitialised byte(s)`

Aparece cuando le das a una llamada al sistema (`write`, `read`…) memoria que tiene **bytes sin inicializar**. Programa (`syscall.c`):

```c
buf = malloc(8);              /* línea 8 */
buf[0] = 'h';
buf[1] = 'i';
write(1, buf, 8);             /* línea 11: escribe 8 bytes, solo 2 tienen valor */
```

```text
==3286192== Syscall param write(buf) points to uninitialised byte(s)
==3286192==    at 0x49869D7: write (write.c:26)
==3286192==    by 0x40118E: main (syscall.c:11)
==3286192==  Address 0x4a9e042 is 2 bytes inside a block of size 8 alloc'd
==3286192==    at 0x4848899: malloc (...)
==3286192==    by 0x401168: main (syscall.c:8)
==3286192==  Uninitialised value was created by a heap allocation
==3286192==    at 0x4848899: malloc (...)
==3286192==    by 0x401168: main (syscall.c:8)
```

- **`2 bytes inside a block of size 8`**: el **primer** byte sin inicializar está en la posición 2. Los dos primeros (`h`, `i`) sí tenían valor.
- Casi siempre es un **tamaño mal calculado**: escribes más bytes de los que has rellenado. En ft_printf, pasar a `write` una longitud mayor que el texto. En get_next_line, olvidar `buf[bytes] = '\0'` y luego copiar con un `strlen` que se pasa (ver [[get_next_line - Pruebas y depuración]]).

## Descriptores de fichero abiertos: `--track-fds=yes`

Valgrind también encuentra los `open` sin su `close`. No los busca por defecto: hay que pedírselo. Programa (`fds.c`):

```c
fd1 = open("pila.c", O_RDONLY);
fd2 = open("segv.c", O_RDONLY);     /* línea 10 */
close(fd1);
/* fd2 nunca se cierra */
```

```text
$ valgrind -q --track-fds=yes ./fds
==3286193== FILE DESCRIPTORS: 4 open (3 std) at exit.
==3286193== Open file descriptor 4: segv.c
==3286193==    at 0x498668B: open (open64.c:41)
==3286193==    by 0x401177: main (fds.c:10)
```

- **`4 open (3 std)`**: al salir había 4 fds abiertos, de los cuales 3 son los estándar (`0`, `1`, `2`), que es normal. El que sobra es el que te dice debajo.
- Te da **el fichero** (`segv.c`) y **la línea del `open`** (`fds.c:10`).
- Es **imprescindible en cualquier programa que use `pipe` o `dup2`**: cada uno crea fds que hay que cerrar. Un fd de una tubería que no se cierra puede hacer que un programa se quede **colgado** esperando un final de fichero que nunca llega (ver [[Descriptores de fichero y read]]).

> [!warning] fds heredados de la terminal
> Según desde dónde lances el programa (algunos editores, `script`, tmux), puede aparecer algún fd abierto que **no es tuyo**, heredado del proceso padre. Para saber si es tuyo, mira si tiene una pila con un `open` o `pipe` de tu código. Si dice `<inherited from parent>`, no es tuyo.

## Possibly lost: un puntero al interior del bloque

La tabla de leaks dice que *possibly lost* es «solo queda un puntero al interior del bloque». Así se ve (`possibly.c`):

```c
static char	*g_p;

int	main(void)
{
	g_p = malloc(10);        /* línea 8 */
	strcpy(g_p, "hola");
	g_p += 2;                /* ahora apunta a "la", en mitad del bloque */
	return (0);
}
```

```text
==3286197== 10 bytes in 1 blocks are possibly lost in loss record 1 of 1
==3286197==    at 0x4848899: malloc (...)
==3286197==    by 0x401158: main (possibly.c:8)
==3286197==
==3286197== LEAK SUMMARY:
==3286197==    definitely lost: 0 bytes in 0 blocks
==3286197==    indirectly lost: 0 bytes in 0 blocks
==3286197==      possibly lost: 10 bytes in 1 blocks
==3286197==    still reachable: 0 bytes in 0 blocks
```

- Sin el `g_p += 2` sería *still reachable* (la global sigue apuntando al principio). Sin la global, sería *definitely lost*.
- Valgrind dice «posiblemente» porque no sabe si guardas el puntero desplazado **a propósito** (y luego harás `free(g_p - 2)`). En 42 trátalo como un leak.
- **Además es un error aunque llegues a liberarlo**: `free(g_p)` con el puntero desplazado da `Invalid free()`. `free` solo acepta la dirección **exacta** que devolvió `malloc`. Es el error de [[get_next_line - Guía paso a paso|get_next_line]] de mover el stash (`stash = nl + 1`) en vez de copiar el resto.

## Leer el resumen final

Las últimas líneas son las que hay que mirar primero:

```text
==PID== HEAP SUMMARY:
==PID==     in use at exit: 0 bytes in 0 blocks
==PID==   total heap usage: 1 allocs, 1 frees, 10 bytes allocated
==PID==
==PID== All heap blocks were freed -- no leaks are possible
==PID==
==PID== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)
```

| Línea | Qué mirar |
|---|---|
| `in use at exit` | Memoria sin liberar al salir. Tiene que ser `0 bytes in 0 blocks` |
| `total heap usage: N allocs, M frees` | Si `N != M`, falta algún `free` (o sobra) |
| `All heap blocks were freed` | Sin leaks |
| `ERROR SUMMARY: N errors` | **Errores de acceso**, no leaks. Tiene que ser `0` |

> [!danger] «No leaks» no significa «sin errores»
> El ejemplo de `pila.c` de arriba dice `All heap blocks were freed -- no leaks are possible`… y a la vez `ERROR SUMMARY: 1 errors`. Liberó todo, pero escribió fuera del bloque. **Mira siempre las dos cosas.** Un evaluador que solo mire los leaks se lo salta; uno que mire el `ERROR SUMMARY`, no.

## AddressSanitizer: `-fsanitize=address`

Se añade al compilar **y** al enlazar:

```bash
cc -Wall -Wextra -Werror -g -fsanitize=address errs.c -o errs
./errs
```

Salida real (recortada):

```text
==2795254==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x602000000015 ...
WRITE of size 1 at 0x602000000015 thread T0
    #0 0x4cb227 in main errs.c:13:7
...
0x602000000015 is located 0 bytes to the right of 5-byte region [0x602000000010,0x602000000015)
allocated by thread T0 here:
    #0 0x49a16d in malloc (...)
    #1 0x4cb1d8 in main errs.c:11:6
```

Diferencias con Valgrind:

- **Se para en el primer error** (Valgrind sigue y los acumula).
- Detecta desbordes de arrays locales, que Valgrind no ve. Con `char buf[4]` y un bucle `while (i <= 4)`, Valgrind no dice nada y ASan da `stack-buffer-overflow`.
- Los leaks aparecen al final como `Direct leak` / `Indirect leak`. Los «still reachable» **no** se cuentan:

```text
==2796313==ERROR: LeakSanitizer: detected memory leaks

Direct leak of 16 byte(s) in 1 object(s) allocated from:
    #1 0x4cb1fc in main leak.c:20:6
Direct leak of 10 byte(s) in 1 object(s) allocated from:
    #1 0x4cb1d8 in main leak.c:17:6
Indirect leak of 20 byte(s) in 1 object(s) allocated from:
    #1 0x4cb20a in main leak.c:21:11

SUMMARY: AddressSanitizer: 46 byte(s) leaked in 3 allocation(s).
```

(Los 5 bytes de `g_keep` no salen: ASan no los considera leak).

> [!danger] No mezcles Valgrind y ASan
> Un ejecutable compilado con `-fsanitize=address` no funciona bajo Valgrind:
> ```text
> ==2831767==Shadow memory range interleaves with an existing memory mapping. ASan cannot proceed correctly. ABORTING.
> ```
> Recompila sin la bandera (`make re`) antes de pasar Valgrind. Y **no entregues** con `-fsanitize` en el Makefile: el subject pide las banderas `-Wall -Wextra -Werror`.

## UndefinedBehaviorSanitizer: `-fsanitize=undefined`

Detecta **comportamiento indefinido** que no es de memoria: desbordamiento de `int` con signo, desplazamientos de bits fuera de rango, división por cero… (ver [[Tipos enteros y conversiones]]).

```c
n = INT_MAX;
n = n + argc;     /* línea 16 */
```

```text
$ cc -g -fsanitize=undefined uaf.c -o uaf_ub && ./uaf_ub
uaf.c:16:8: runtime error: signed integer overflow: 2147483647 + 1 cannot be represented in type 'int'
-2147483648
```

UBSan **avisa y continúa**. Puedes combinar los dos: `-fsanitize=address,undefined`. Es especialmente útil en [[ft_atoi]], [[ft_itoa]] y [[ft_printf - Índice|ft_printf]] con `INT_MIN`.

## En los Mac de 42: `leaks`

En macOS no hay Valgrind y ASan no busca leaks por defecto (habría que activarlo con `ASAN_OPTIONS=detect_leaks=1`, y no siempre funciona con el clang de Apple). Se usa la herramienta `leaks`, que viene con Xcode:

```bash
leaks --atExit -- ./prog arg1
```

Al terminar el programa, `leaks` dice cuántos bloques quedan sin liberar (`Process NNNN: 0 leaks for 0 total leaked bytes.` si todo va bien). Si exportas antes `MallocStackLogging=1`, `leaks` puede enseñar además la pila de llamadas del `malloc` de cada bloque perdido.

> [!warning] No dejes `system("leaks ...")` en el código
> Es habitual añadir `system("leaks prog");` al final de `main` para probar. `system` no está entre las funciones permitidas: bórralo antes de entregar.

## Errores típicos

| Síntoma | Causa probable |
|---|---|
| Sale `main (in /ruta/prog)` sin número de línea | Falta `-g` (en tu código o en tu libft) |
| `Invalid read of size 1` justo después de un bloque | Recorres la cadena hasta un `'\0'` que no reservaste |
| `Invalid write of size 8` | Escribes un puntero fuera de un array de punteros (típico en [[ft_split]]: olvidar el hueco del `NULL` final) |
| Leak en [[get_next_line - Índice\|get_next_line]] solo al final | La variable `static` no se libera cuando `read` devuelve 0 o -1 |
| Valgrind tarda muchísimo | Normal; prueba primero con entradas pequeñas |

## Relacionadas

[[Depurar con gdb]] · [[Segmentos de memoria en C]] · [[Cómo compila C]] · [[Makefile y librerías estáticas]] · [[Control de errores en C]] · [[Listas enlazadas]] · [[ft_calloc]] · [[ft_split]] · [[Comportamiento indefinido]] · [[Padding y alineación de memoria]] · [[Descriptores de fichero y read]] · [[Puntero a puntero]]

## Referencias

- [Valgrind User Manual — Memcheck](https://valgrind.org/docs/manual/mc-manual.html) — tipos de error, categorías de leaks (definitely/indirectly/possibly/still reachable) y opciones `--leak-check`, `--show-leak-kinds`, `--track-origins`.
- [Valgrind Quick Start Guide](https://valgrind.org/docs/manual/quick-start.html) — compilar con `-g` y forma básica de uso.
- [Valgrind FAQ](https://valgrind.org/docs/manual/faq.html) — interpretación de «still reachable» y «possibly lost».
- [GCC — Instrumentation Options](https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html) — `-fsanitize=address` y `-fsanitize=undefined`.
- [Clang — AddressSanitizer](https://clang.llvm.org/docs/AddressSanitizer.html) — qué detecta ASan, coste en rendimiento, detección de leaks activa por defecto en Linux y `ASAN_OPTIONS=detect_leaks=1` en macOS.
- [Clang — LeakSanitizer](https://clang.llvm.org/docs/LeakSanitizer.html) — qué es LeakSanitizer y plataformas soportadas.
- [Clang — UndefinedBehaviorSanitizer](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html) — comprobaciones de UBSan.
- [Apple — Finding Memory Leaks](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/ManagingMemory/Articles/FindingLeaks.html) — herramienta `leaks` y `MallocStackLogging`.
- Salidas reales generadas con Valgrind 3.18.1 y clang 12 (`cc`) en Ubuntu 22.04.
