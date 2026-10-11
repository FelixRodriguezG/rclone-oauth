---
tags: [herramientas, gdb, lldb, debug, c]
---
# Depurar con gdb

[[Inicio|← Inicio]] · Herramientas

> `gdb` (*GNU Debugger*) ejecuta tu programa **bajo control**: lo puedes parar en cualquier línea, avanzar instrucción a instrucción, mirar el valor de cada variable y ver exactamente dónde y por qué crashea. Es mucho más rápido que llenar el código de `printf`.

> [!info] En los Mac de 42
> En macOS se usa **`lldb`**, que funciona igual con otros nombres de comandos. Al final hay una [[#Equivalencias gdb ↔ lldb|tabla de equivalencias]].

## 1. Compilar para depurar

```bash
cc -Wall -Wextra -Werror -g -O0 main.c -o prog
```

- **`-g`** guarda en el ejecutable la información de depuración: nombres de variables y números de línea. **Sin `-g`, gdb no puede mostrarte tu código.**
- **`-O0`** desactiva las optimizaciones, para que el código ejecutado coincida con el que escribiste. Si optimizas, gdb muestra variables como `<optimized out>`.

> [!warning] No entregues con `-g`
> Úsalo solo mientras depuras. El Makefile que entregas debe tener las banderas que pide el subject (ver [[Makefile y librerías estáticas]]).

## 2. Arrancar

| Comando | Qué hace |
|---|---|
| `gdb ./prog` | Abre el programa sin ejecutarlo todavía |
| `gdb --args ./prog arg1 arg2` | Igual, pero ya con argumentos |
| `run` (`r`) | Ejecuta el programa |
| `run arg1 arg2` | Ejecuta con esos argumentos |
| `quit` (`q`) | Sale de gdb |
| `gdb -tui ./prog` | Modo visual: código fuente arriba y consola abajo |

> [!tip] Atajos
> Casi todos los comandos se pueden abreviar (`b`, `r`, `n`, `s`, `c`, `p`, `bt`), y pulsar **Enter sin escribir nada repite el último comando**: muy útil para ir con `next` línea a línea.

## 3. Caso real: encontrar un segfault

Este programa cuenta cuántas `'a'` hay en el primer argumento:

```c
#include <stddef.h>

static size_t	count_char(const char *s, char c)
{
	size_t	n;

	n = 0;
	while (*s)
	{
		if (*s == c)
			n++;
		s++;
	}
	return (n);
}

int	main(int argc, char **argv)
{
	size_t	n;

	(void)argc;
	n = count_char(argv[1], 'a');
	return ((int)n);
}
```

Con `./count banana` funciona, pero sin argumentos:

```text
$ ./count
zsh: segmentation fault  ./count
```

El sistema solo dice que ha crasheado. Con gdb se ve dónde y por qué:

```text
$ gdb -q ./count
(gdb) run
Program received signal SIGSEGV, Segmentation fault.
0x000000000040116a in count_char (s=0x0, c=97 'a') at count.c:8
8		while (*s)
```

**1. ¿Dónde?** En la línea 8, `while (*s)`, y además `s=0x0`: es `NULL`. Al hacer `*s` se lee la dirección 0.

**2. ¿Quién llamó con `NULL`?** `backtrace` (`bt`) muestra la pila de llamadas, de la más reciente (`#0`) a la más antigua:

```text
(gdb) bt
#0  0x000000000040116a in count_char (s=0x0, c=97 'a') at count.c:8
#1  0x0000000000401138 in main (argc=1, argv=0x7fffffffdf48) at count.c:22
```

**3. Subir a `main` para investigar.** Con `frame 1` (o `up`) cambias al marco de `main` y puedes mirar sus variables:

```text
(gdb) frame 1
#1  0x0000000000401138 in main (argc=1, argv=0x7fffffffdf48) at count.c:22
22		n = count_char(argv[1], 'a');
(gdb) p argc
$1 = 1
(gdb) p argv[1]
$2 = 0x0
(gdb) info locals
n = 100
```

**Diagnóstico:** `argc` vale 1 (solo el nombre del programa), así que `argv[1]` es `NULL` y nadie lo comprobó. La solución es validar `argc` (ver [[Control de errores en C]]).

> [!note] ¿Y ese `n = 100`?
> `n` todavía no se ha asignado, así que contiene **basura**: lo que hubiera antes en esa posición del stack. Por eso nunca hay que usar una variable sin inicializar (ver [[Segmentos de memoria en C]]).

## 4. Breakpoints: parar donde quieras

| Comando | Qué hace |
|---|---|
| `break count_char` (`b`) | Para al entrar en esa función |
| `break count.c:11` | Para en esa línea de ese archivo |
| `break 11 if n == 2` | Para solo si se cumple la condición |
| `info breakpoints` | Lista los breakpoints |
| `delete 2` / `delete` | Borra el breakpoint 2 / todos |
| `disable 2` / `enable 2` | Lo desactiva o reactiva sin borrarlo |

```text
(gdb) break count_char
Breakpoint 1 at 0x40115e: file count.c, line 7.
(gdb) run banana
Breakpoint 1, count_char (s=0x7fffffffe39e "banana", c=97 'a') at count.c:7
7		n = 0;
```

Un **breakpoint condicional** evita pulsar `continue` cien veces en un bucle:

```text
(gdb) break 11 if n == 2
(gdb) run banana
Breakpoint 1, count_char (s=0x7fffffffe3a3 "a", c=97 'a') at count.c:11
11				n++;
(gdb) p n
$1 = 2
```

## 5. Avanzar paso a paso

| Comando | Qué hace |
|---|---|
| `next` (`n`) | Ejecuta la línea actual **sin entrar** en las funciones que llama |
| `step` (`s`) | Ejecuta la línea actual **entrando** en la función que llama |
| `finish` | Termina la función actual, vuelve a quien la llamó y muestra lo que devolvió |
| `continue` (`c`) | Sigue hasta el siguiente breakpoint (o el final) |
| `until 15` | Sigue hasta llegar a la línea 15 (útil para salir de un bucle) |

```text
(gdb) finish
Run till exit from #0  count_char (s=0x7fffffffe3a3 "a", c=97 'a') at count.c:11
0x0000000000401138 in main (argc=2, argv=0x7fffffffdf48) at count.c:22
22		n = count_char(argv[1], 'a');
Value returned is $3 = 3
```

> [!tip] `next` o `step`
> Usa `next` por defecto y `step` solo cuando quieras entrar en una función **tuya**. Si haces `step` en `printf` o `malloc`, acabas dentro de la libc.

## 6. Ver variables y memoria

| Comando | Qué muestra |
|---|---|
| `print n` (`p`) | El valor de `n` |
| `p *ptr` | Lo que hay donde apunta `ptr` |
| `p s[0]@3` | 3 elementos seguidos desde `s[0]` (para arrays y buffers) |
| `p/x c` · `p/d` · `p/c` | El valor en hexadecimal, decimal o como carácter |
| `ptype s` | El tipo de `s` |
| `info locals` · `info args` | Todas las variables locales / los parámetros |
| `display n` | Muestra `n` **automáticamente** cada vez que el programa se para |
| `x/8xb ptr` | Examina 8 bytes de memoria en hex desde `ptr` |
| `set var n = 0` | Cambia el valor de una variable en plena ejecución |

```text
(gdb) display n
1: n = 0
(gdb) display *s
2: *s = 98 'b'
(gdb) next
1: n = 0
2: *s = 97 'a'
(gdb) p s
$1 = 0x7fffffffe39f "anana"
(gdb) p s[0]@3
$2 = "ana"
(gdb) p/x c
$3 = 0x61
(gdb) ptype s
type = const char *
```

## 7. Watchpoints: ¿quién cambia esta variable?

`watch n` para el programa **cada vez que `n` cambia de valor** y muestra el valor anterior y el nuevo. Es ideal cuando una variable acaba con un valor raro y no sabes en qué línea pasa.

```text
(gdb) watch n
Hardware watchpoint 2: n
(gdb) continue
Old value = 0
New value = 1
count_char (s=0x7fffffffe39f "anana", c=97 'a') at count.c:12
12			s++;
```

## 8. Depurar tu libft con tus tests

Tu Makefile de libft tiene un `test_runner`. Para depurarlo hay que recompilar **todo** con `-g`:

```bash
make fclean
make test_runner CFLAGS="-Wall -Wextra -Werror -I. -g"
gdb --args ./test_runner ft_strdup
```

```text
(gdb) break ft_strdup
(gdb) run
Breakpoint 1, ft_strdup (s=0x407bf6 "hola mundo") at ft_strdup.c:20
(gdb) bt
#0  ft_strdup (s=0x407bf6 "hola mundo") at ft_strdup.c:20
#1  0x0000000000402dff in test_ft_strdup () at tests/ft_strdup_test.c:21
```

Cuando termines, vuelve a compilar normal con `make re`.

## 9. Método para depurar

1. **Reproduce** el fallo siempre con la misma entrada.
2. Si crashea: `run` → `bt` → `frame N` → `p` de las variables sospechosas.
3. Si da un resultado incorrecto: pon un `break` **antes** de donde crees que está el fallo, y avanza con `next` y `display` hasta ver la primera variable con un valor que no esperabas.
4. Si una variable acaba con un valor raro: `watch`.
5. Si sospechas de la memoria (leaks, lecturas fuera de rango): usa también `valgrind` o `-fsanitize=address` (ver [[Control de errores en C]]).

## Equivalencias gdb ↔ lldb

| Acción | gdb | lldb (macOS) |
|---|---|---|
| Arrancar con argumentos | `gdb --args ./prog a b` | `lldb -- ./prog a b` |
| Ejecutar | `run` | `run` |
| Breakpoint en función | `b count_char` | `b count_char` |
| Breakpoint en línea | `b count.c:11` | `b count.c:11` |
| Breakpoint condicional | `b 11 if n == 2` | `br set -l 11 -c 'n == 2'` |
| Listar breakpoints | `info breakpoints` | `br list` |
| Siguiente / entrar / salir | `n` / `s` / `finish` | `n` / `s` / `finish` |
| Continuar | `c` | `c` |
| Pila de llamadas | `bt` | `bt` |
| Cambiar de marco | `frame 1` / `up` | `frame select 1` / `up` |
| Variables locales | `info locals` | `frame variable` (`v`) |
| Imprimir | `p n` | `p n` |
| Watchpoint | `watch n` | `watchpoint set variable n` |
| Cambiar una variable | `set var n = 0` | `expr n = 0` |
| Modo visual | `gdb -tui` / `layout src` | `gui` |

## Relacionadas

[[Control de errores en C]] · [[Segmentos de memoria en C]] · [[Makefile y librerías estáticas]] · [[Vim]]
