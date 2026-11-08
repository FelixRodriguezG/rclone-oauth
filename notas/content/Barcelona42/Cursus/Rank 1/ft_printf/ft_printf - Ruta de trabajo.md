---
tags: [proyecto, ft_printf, ruta, rank-1, principiante, pruebas]
aliases: ["Ruta ft_printf", "ft_printf función a función"]
---
# ft_printf - Ruta de trabajo

[[Inicio|← Inicio]] · [[ft_printf - Índice|ft_printf]] · [[ft_printf - Guía paso a paso|Guía]] · Ruta de trabajo

> La [[ft_printf - Guía paso a paso|guía]] explica **por qué** el diseño es así. Esta nota es el **orden de trabajo**: una función (o dos) por paso, con un `main` de prueba que compila con lo que llevas escrito hasta ese momento y la salida exacta que debe dar. No pasas al siguiente paso hasta que el actual sale igual.

> [!info] Cómo usar esta ruta
> - Marca cada casilla al terminar el paso.
> - El código de cada función está en la guía (enlace en cada paso). Intenta escribirla tú primero.
> - Todas las salidas de esta nota son **reales**: se han obtenido compilando el código de la guía paso a paso con `cc -Wall -Wextra -Werror`, y cada paso pasa Valgrind y norminette (salvo la cabecera de 42).
> - Los `main` de prueba usan `printf`/`fprintf` de `<stdio.h>`: están permitidos en las pruebas, **no** en la librería.

## Progreso

- [ ] **Paso 0**: carpetas, header y libft
- [ ] **Paso 1**: `ft_out_char`
- [ ] **Paso 2**: `ft_out_nstr`
- [ ] **Paso 3**: `ft_out_repeat` + `ft_out_field`
- [ ] **Paso 4**: `ft_init_flags` + `ft_set_flags` (obligatoria)
- [ ] **Paso 5**: `ft_convert` (solo `%%`) + `ft_printf`
- [ ] **Paso 6**: `ft_print_char` → `%c`
- [ ] **Paso 7**: `ft_print_str` → `%s`
- [ ] **Paso 8**: `ft_utoa_base` + `ft_base` + `ft_print_number`
- [ ] **Paso 9**: `ft_print_int` → `%d` `%i`
- [ ] **Paso 10**: `ft_print_unsigned` → `%u` `%x` `%X`
- [ ] **Paso 11**: `ft_print_ptr` → `%p`
- [ ] **Paso 12**: Makefile y control final de la obligatoria
- [ ] **Paso 13**: `ft_read_number` + `ft_set_one_flag` + `ft_set_flags` (bonus)
- [ ] **Paso 14**: el bonus completo

Así crecen los ficheros (✚ = lo que añades en ese paso):

| Paso | `ft_output.c` | `ft_flags.c` | `ft_printf.c` | `ft_print_chars.c` | `ft_print_nums.c` |
|---|---|---|---|---|---|
| 1 | ✚ `ft_out_char` | | | | |
| 2 | ✚ `ft_out_nstr` | | | | |
| 3 | ✚ `ft_out_repeat`, `ft_out_field` | | | | |
| 4 | | ✚ `ft_set_flags` | ✚ `ft_init_flags` | | |
| 5 | | | ✚ `ft_convert` (`%%`), `ft_printf` | | |
| 6 | | | rama `'c'` | ✚ `ft_print_char` | |
| 7 | | | rama `'s'` | ✚ `ft_print_str` | |
| 8 | | | | | ✚ `ft_utoa_base`, `ft_base`, `ft_print_number` |
| 9 | | | rama `'d'`/`'i'` | | ✚ `ft_print_int` |
| 10 | | | rama `'u'`/`'x'`/`'X'` | | ✚ `ft_print_unsigned` |
| 11 | | | rama `'p'` | ✚ `ft_print_ptr` | |
| 13 | | (bonus: `ft_flags_bonus.c`) | | | |

---

## Paso 0: carpetas, header y libft

```text
ft_printf/
├── includes/ft_printf.h
├── libft/                ← tu libft, con su Makefile
├── srcs/                 ← vacía de momento
└── tests/                ← los main de esta nota (NO se entregan)
```

1. Copia tu libft en `libft/` y compílala: `make -C libft` (debe crear `libft/libft.a`).
2. Escribe `includes/ft_printf.h` **completo**: las tres `struct`, la macro `MAX_DIGITS` y todos los prototipos. Está en [[ft_printf - Guía paso a paso#Etapa 0: los ficheros|la guía, etapa 0]]. Que el header tenga prototipos de funciones que aún no existen no es un problema: solo falla si alguien las **llama**.
3. Añade `tests/` a tu `.gitignore`. El subject entrega `*/*.c`, así que `tests/t01.c` se entregaría si lo subes.

**Comprueba:** `cc -Wall -Wextra -Werror -Iincludes -Ilibft -fsyntax-only includes/ft_printf.h` no imprime nada.

## Paso 1: `ft_out_char`

**Escribe** en `srcs/ft_output.c` (con `#include "ft_printf.h"` arriba): `ft_out_char`, que escribe 1 byte, suma 1 a `o->len` y pone `o->error = 1` si `write` falla. Código en [[ft_printf - Guía paso a paso#Etapa 1: la salida (`ft_out_char`, `ft_out_nstr`)|etapa 1]].

**Prueba** `tests/t01.c`:

```c
#include <stdio.h>
#include "ft_printf.h"

int	main(void)
{
	t_out	o;

	o.len = 0;
	o.error = 0;
	ft_out_char(&o, 'h');
	ft_out_char(&o, 'i');
	ft_out_char(&o, '\n');
	fprintf(stderr, "len=%d error=%d\n", o.len, o.error);
	return (0);
}
```

```bash
cc -Wall -Wextra -Werror -Iincludes -Ilibft tests/t01.c srcs/ft_output.c libft/libft.a -o test && ./test
```

**Debe salir:**

```text
hi
len=3 error=0
```

Y con la salida cerrada, `./test >&-`, debe salir `len=0 error=1`: `write` falla, no se cuenta nada y se anota el error. Los resultados van a `stderr` (`fprintf(stderr, …)`) precisamente para poder verlos aunque cierres `stdout`.

> [!failure] Si falla
> - `len=3` con `>&-`: no compruebas el `-1` de `write`.
> - `undefined reference to ft_strlen`: falta `libft/libft.a` al final del comando.

## Paso 2: `ft_out_nstr`

**Escribe**, debajo, `ft_out_nstr(o, s, n)`: igual que `ft_out_char` pero con los `n` primeros bytes de `s`. Si `n <= 0`, no hace nada.

**Prueba** `tests/t02.c`:

```c
#include <stdio.h>
#include "ft_printf.h"

int	main(void)
{
	t_out	o;

	o.len = 0;
	o.error = 0;
	ft_out_nstr(&o, "hola mundo\n", 5);
	ft_out_nstr(&o, "nada", 0);
	ft_out_char(&o, '\n');
	fprintf(stderr, "len=%d error=%d\n", o.len, o.error);
	return (0);
}
```

```bash
cc -Wall -Wextra -Werror -Iincludes -Ilibft tests/t02.c srcs/ft_output.c libft/libft.a -o test && ./test
```

**Debe salir** (fíjate en el espacio tras `hola`: son los 5 primeros bytes, `"hola "`):

```text
hola 
len=6 error=0
```

> [!failure] Si falla
> - Sale `hola mundo`: usas `ft_strlen(s)` en vez de `n`.
> - `len=10`: no ignoras `n = 0` (o lo cuentas aunque no escribas).

## Paso 3: `ft_out_repeat` + `ft_out_field`

**Escribe**, debajo, la `static` `ft_out_repeat` (escribe `n` veces un carácter; nada si `n <= 0`) y `ft_out_field`, que escribe las cinco piezas: espacios, prefijo, ceros, cuerpo, espacios. Código en [[ft_printf - Guía paso a paso#Etapa 2: el campo (`ft_out_field`)|etapa 2]]; las cuentas, en [[ft_printf - Guía paso a paso#Las dos únicas cuentas del proyecto|la sección 3]].

`ft_out_repeat` es `static`: **no se puede llamar desde el `main`**. Se prueba a través de `ft_out_field`, que es quien la usa.

**Prueba** `tests/t03.c`. Construye a mano las piezas de `-42` y pide distintos anchos:

```c
#include <stdio.h>
#include "ft_printf.h"

static void	prueba(t_flags f, const char *prefix, const char *body, int zeros)
{
	t_out	o;
	t_field	fl;

	o.len = 0;
	o.error = 0;
	fl.prefix = prefix;
	fl.body = body;
	fl.body_len = ft_strlen(body);
	fl.zeros = zeros;
	ft_out_char(&o, '[');
	ft_out_field(&o, &f, &fl);
	ft_out_char(&o, ']');
	fprintf(stderr, "  len=%d\n", o.len);
}

int	main(void)
{
	t_flags	f;

	f.minus = 0;
	f.zero = 0;
	f.width = 0;
	prueba(f, "-", "42", 0);
	f.width = 8;
	prueba(f, "-", "42", 0);
	prueba(f, "-", "42", 3);
	f.minus = 1;
	prueba(f, "-", "42", 0);
	f.minus = 0;
	f.zero = 1;
	prueba(f, "-", "42", 0);
	f.zero = 0;
	f.width = 1;
	prueba(f, "", "hola", 0);
	return (0);
}
```

```bash
cc -Wall -Wextra -Werror -Iincludes -Ilibft tests/t03.c srcs/ft_output.c libft/libft.a -o test && ./test
```

**Debe salir:**

```text
[-42]  len=5            ← sin ancho: solo prefijo y cuerpo (así es siempre la obligatoria)
[     -42]  len=10      ← ancho 8: 5 espacios delante
[  -00042]  len=10      ← ancho 8 y 3 ceros: quedan 2 espacios
[-42     ]  len=10      ← con '-': los espacios detrás
[-0000042]  len=10      ← con '0': los espacios se convierten en ceros, detrás del signo
[hola]  len=6           ← ancho menor que el contenido: no recorta
```

> [!failure] Si falla
> - `[00000-42]`: escribes los ceros antes que el prefijo.
> - La última línea cuelga o escribe basura: `ft_out_repeat` con `n` negativo; usa `while (n-- > 0)`.

## Paso 4: `ft_init_flags` + `ft_set_flags` (obligatoria)

**Escribe:**
- En `srcs/ft_printf.c`: `ft_init_flags`, que devuelve un `t_flags` todo a `0` salvo `precision = -1`.
- En `srcs/ft_flags.c`: la `ft_set_flags` **obligatoria**, que se salta `-+ #.0123456789` y lee la letra. Avanza `*i` justo detrás de la letra, pero nunca más allá del `'\0'`.

Código en [[ft_printf - Guía paso a paso#Etapa 3: el bucle principal y el reparto|etapa 3]].

**Prueba** `tests/t04.c`. `i = 1` porque `s[0]` es el `%`:

```c
#include <stdio.h>
#include "ft_printf.h"

static void	prueba(const char *s)
{
	t_flags	f;
	int		i;

	i = 1;
	f = ft_set_flags(s, &i);
	if (f.spec)
		printf("%-9s spec=%c i=%d", s, f.spec, i);
	else
		printf("%-9s spec=\\0 i=%d", s, i);
	printf(" width=%d precision=%d minus=%d zero=%d\n",
		f.width, f.precision, f.minus, f.zero);
}

int	main(void)
{
	prueba("%d");
	prueba("%5d");
	prueba("%-08.3d");
	prueba("%");
	return (0);
}
```

```bash
cc -Wall -Wextra -Werror -Iincludes -Ilibft tests/t04.c srcs/ft_output.c srcs/ft_printf.c srcs/ft_flags.c libft/libft.a -o test && ./test
```

**Debe salir:**

```text
%d        spec=d i=2 width=0 precision=-1 minus=0 zero=0
%5d       spec=d i=3 width=0 precision=-1 minus=0 zero=0
%-08.3d   spec=d i=7 width=0 precision=-1 minus=0 zero=0
%         spec=\0 i=1 width=0 precision=-1 minus=0 zero=0
```

- La letra siempre es `d` y las banderas **nunca** se guardan: en la obligatoria se saltan.
- `i` queda justo detrás de la letra. Con `"%"` se queda en `1`: apunta al `'\0'` y no se pasa.

> [!failure] Si falla
> - `spec=5` o `spec=-`: no te saltas las banderas. Con eso, `ft_printf("[%5d] [%s]", 42, "hola")` crashea (ver [[ft_printf - Guía paso a paso#Etapa 3: el bucle principal y el reparto|etapa 3]]).
> - `i=2` con `"%"`: avanzas más allá del `'\0'`.
> - `precision=0`: `ft_init_flags` debe ponerla a `-1`.

## Paso 5: `ft_convert` (solo `%%`) + `ft_printf`

**Escribe**, en `srcs/ft_printf.c` debajo de `ft_init_flags`:
- La `static` `ft_convert`, que de momento solo sabe `%%`, escribe tal cual las letras desconocidas y lleva `(void)args;`.
- `ft_printf`.

Código en [[ft_printf - Guía paso a paso#Etapa 3: el bucle principal y el reparto|etapa 3]].

**Prueba** `tests/t05.c`:

```c
#include <stdio.h>
#include "ft_printf.h"

int	main(void)
{
	int	r;

	r = ft_printf("hola %% %k\n");
	fprintf(stderr, "r=%d\n", r);
	r = ft_printf("[%5%]\n");
	fprintf(stderr, "r=%d\n", r);
	r = ft_printf("fin %");
	fprintf(stderr, "|r=%d\n", r);
	r = ft_printf(NULL);
	fprintf(stderr, "NULL r=%d\n", r);
	return (0);
}
```

```bash
cc -Wall -Wextra -Werror -Iincludes -Ilibft tests/t05.c srcs/ft_output.c srcs/ft_printf.c srcs/ft_flags.c libft/libft.a -o test && ./test
```

**Debe salir:**

```text
hola % %k
r=10
[%]
r=4
fin |r=4
NULL r=-1
```

Y con `./test >&-` o `./test > /dev/full`, los cuatro `r` valen `-1`: el error de `write` llega hasta el valor de retorno.

**A partir de aquí ya usas `ft_printf` de verdad**: los siguientes pasos solo añaden conversiones.

> [!failure] Si falla
> - `unused parameter 'args'`: falta `(void)args;` (lo quitas en el paso 6).
> - `r=5` en `fin %`: avanzas `i` más allá del `'\0'` (paso 4).
> - Crash con `NULL`: falta `if (!format) return (-1);`.

## Paso 6: `ft_print_char` → `%c`

**Escribe:**
- En `srcs/ft_print_chars.c` (nuevo, con el `#include`): `ft_print_char`, que guarda el `int` en un `char` local, prepara el campo (`body_len = 1`) y llama a `ft_out_field`.
- En `ft_convert`: **quita** `(void)args;` y añade la rama `'c'` delante de la de `'%'`. Recuerda que la de `'%'` pasa de `if` a `else if`.

Código en [[ft_printf - Guía paso a paso#Etapa 4: `%c` y `%s`|etapa 4]].

**Prueba** `tests/t06.c`:

```c
#include <stdio.h>
#include "ft_printf.h"

int	main(void)
{
	int	r;

	r = ft_printf("[%c%c%c]\n", 'o', 'k', '!');
	fprintf(stderr, "r=%d\n", r);
	r = ft_printf("[%c]\n", '\0');
	fprintf(stderr, "r=%d\n", r);
	r = ft_printf("[%5c]\n", 'a');
	fprintf(stderr, "r=%d\n", r);
	return (0);
}
```

```bash
cc -Wall -Wextra -Werror -Iincludes -Ilibft tests/t06.c srcs/ft_output.c srcs/ft_printf.c srcs/ft_flags.c srcs/ft_print_chars.c libft/libft.a -o test && ./test 2>&1 | cat -A
```

**Debe salir** (`cat -A` marca el final de línea con `$` y el byte nulo como `^@`):

```text
[ok!]$
r=6$
[^@]$
r=4$
[a]$
r=4$
```

- `'\0'` se escribe y **cuenta** (`r=4`: `[`, el byte nulo, `]` y `\n`).
- `[%5c]` da `[a]`: en la obligatoria el ancho se salta. Con el bonus dará `[    a]`.

> [!failure] Si falla
> - `[]` y `r=3` con `'\0'`: usas `ft_strlen` sobre el carácter; `body_len` debe ser `1` siempre.
> - Letras raras: lees `va_arg(*args, char)` en vez de `int`.

## Paso 7: `ft_print_str` → `%s`

**Escribe:**
- En `ft_print_chars.c`, debajo: `ft_print_str`, con `NULL` → `"(null)"` y la precisión como máximo de caracteres (no se usa hasta el bonus).
- En `ft_convert`: la rama `'s'` después de la de `'c'`.

Código en [[ft_printf - Guía paso a paso#Etapa 4: `%c` y `%s`|etapa 4]].

**Prueba** `tests/t07.c`:

```c
#include <stdio.h>
#include "ft_printf.h"

int	main(void)
{
	int	r;

	r = ft_printf("[%s] [%s]\n", "hola", "");
	fprintf(stderr, "r=%d\n", r);
	r = ft_printf("[%s]\n", (char *)NULL);
	fprintf(stderr, "r=%d\n", r);
	r = ft_printf("[%c|%s|%c]\n", 'a', "bc", 'd');
	fprintf(stderr, "r=%d\n", r);
	return (0);
}
```

```bash
cc -Wall -Wextra -Werror -Iincludes -Ilibft tests/t07.c srcs/ft_output.c srcs/ft_printf.c srcs/ft_flags.c srcs/ft_print_chars.c libft/libft.a -o test && ./test
```

**Debe salir:**

```text
[hola] []
r=10
[(null)]
r=9
[a|bc|d]
r=9
```

La tercera línea comprueba que los argumentos no se desplazan al mezclar conversiones.

> [!failure] Si falla
> - *Segmentation fault* con `NULL`: haces `ft_strlen(s)` antes de sustituir `NULL` por `"(null)"`.

## Paso 8: `ft_utoa_base` + `ft_base` + `ft_print_number`

**Escribe** en `srcs/ft_print_nums.c` (nuevo):
- La `static` `ft_utoa_base`: un número en cualquier base → cifras en un buffer, y devuelve cuántas son.
- La `static` `ft_base`: la base según la letra.
- `ft_print_number`, que prepara el campo con `char digits[MAX_DIGITS]`, la precisión y el prefijo que le pasen.

Código en [[ft_printf - Guía paso a paso#Etapa 5: los números (`ft_utoa_base`, `ft_print_number`)|etapa 5]]; explicación pieza a pieza en [[ft_printf - Funciones auxiliares#`ft_utoa_base`: número a texto en cualquier base|Funciones auxiliares]].

Este paso **no toca `ft_convert`**: `ft_print_number` no es una conversión, la usarán las de los pasos 9, 10 y 11. Las dos `static` no se pueden llamar desde el `main`, así que se prueban a través de `ft_print_number`, eligiendo la base con `f.spec`.

**Prueba** `tests/t08.c`:

```c
#include <stdio.h>
#include "ft_printf.h"

static void	prueba(char spec, unsigned long n, const char *prefix)
{
	t_out	o;
	t_flags	f;

	o.len = 0;
	o.error = 0;
	f = ft_init_flags();
	f.spec = spec;
	ft_out_char(&o, '[');
	ft_print_number(&o, &f, n, prefix);
	ft_out_char(&o, ']');
	fprintf(stderr, "  len=%d\n", o.len);
}

int	main(void)
{
	prueba('d', 0, "");
	prueba('d', 42, "-");
	prueba('u', 4294967295UL, "");
	prueba('x', 255, "");
	prueba('X', 255, "0X");
	prueba('x', 18446744073709551615UL, "0x");
	return (0);
}
```

```bash
cc -Wall -Wextra -Werror -Iincludes -Ilibft tests/t08.c srcs/ft_output.c srcs/ft_printf.c srcs/ft_flags.c srcs/ft_print_chars.c srcs/ft_print_nums.c libft/libft.a -o test && ./test
```

**Debe salir:**

```text
[0]  len=3
[-42]  len=5
[4294967295]  len=12
[ff]  len=4
[0XFF]  len=6
[0xffffffffffffffff]  len=20
```

> [!failure] Si falla
> - `[]` con `0`: `ft_utoa_base` debe devolver al menos 1 cifra (`len = 1` al empezar).
> - Cifras al revés (`[24-]`): rellenas el buffer de izquierda a derecha; va de derecha a izquierda.
> - Basura al final: `body_len` no es lo que devuelve `ft_utoa_base`.

## Paso 9: `ft_print_int` → `%d` `%i`

**Escribe:**
- En `ft_print_nums.c`, debajo: `ft_print_int`. Si `n < 0`, llama a `ft_print_number` con `-(long)n` y prefijo `"-"`. Si no, con prefijo `"+"`, `" "` o `""`.
- En `ft_convert`: la rama `'d' || 'i'` después de la de `'s'`.

Código en [[ft_printf - Guía paso a paso#Etapa 5: los números (`ft_utoa_base`, `ft_print_number`)|etapa 5]].

**Prueba** `tests/t09.c`:

```c
#include <stdio.h>
#include <limits.h>
#include "ft_printf.h"

int	main(void)
{
	int	r;

	r = ft_printf("[%d] [%i] [%d]\n", 0, 42, -42);
	fprintf(stderr, "r=%d\n", r);
	r = ft_printf("[%d] [%d]\n", INT_MIN, INT_MAX);
	fprintf(stderr, "r=%d\n", r);
	return (0);
}
```

```bash
cc -Wall -Wextra -Werror -Iincludes -Ilibft tests/t09.c srcs/ft_output.c srcs/ft_printf.c srcs/ft_flags.c srcs/ft_print_chars.c srcs/ft_print_nums.c libft/libft.a -o test && ./test
```

**Debe salir:**

```text
[0] [42] [-42]
r=15
[-2147483648] [2147483647]
r=27
```

> [!failure] Si falla
> - `[-18446744071562067968]` con `INT_MIN`: haces `-n` sobre el `int` antes de convertir. Debe ser `-(long)n`.

## Paso 10: `ft_print_unsigned` → `%u` `%x` `%X`

**Escribe:**
- En `ft_print_nums.c`, debajo: `ft_print_unsigned`, que con `#` y `n != 0` pone el prefijo `"0x"` o `"0X"`. En la obligatoria siempre es `""`.
- En `ft_convert`: la rama `'u' || 'x' || 'X'` después de la de `'d'`. Las tres usan la misma función; la base la decide `ft_base`.

Código en [[ft_printf - Guía paso a paso#Etapa 5: los números (`ft_utoa_base`, `ft_print_number`)|etapa 5]].

**Prueba** `tests/t10.c`:

```c
#include <stdio.h>
#include "ft_printf.h"

int	main(void)
{
	int	r;

	r = ft_printf("[%u] [%u]\n", 42, -1);
	fprintf(stderr, "r=%d\n", r);
	r = ft_printf("[%x] [%X] [%x] [%x]\n", 255, 255, 0, -1);
	fprintf(stderr, "r=%d\n", r);
	return (0);
}
```

```bash
cc -Wall -Wextra -Werror -Iincludes -Ilibft tests/t10.c srcs/ft_output.c srcs/ft_printf.c srcs/ft_flags.c srcs/ft_print_chars.c srcs/ft_print_nums.c libft/libft.a -o test && ./test
```

**Debe salir:**

```text
[42] [4294967295]
r=18
[ff] [FF] [0] [ffffffff]
r=25
```

> [!failure] Si falla
> - `[-1]`: lees `va_arg(*args, int)`; debe ser `unsigned int`.
> - `[ffffffffffffffff]` con `-1`: lees `unsigned long`; `%x` es `unsigned int` (32 bits).

## Paso 11: `ft_print_ptr` → `%p`

**Escribe:**
- En `ft_print_chars.c`, debajo de `ft_print_str`: `ft_print_ptr`. Con `NULL` escribe el campo `(nil)`; si no, `ft_print_number` con `(unsigned long)p` y `"0x"`.
- En `ft_convert`: la rama `'p'` después de la de `'s'`. `ft_convert` queda ya **completa**.

Código en [[ft_printf - Guía paso a paso#Etapa 6: `%p`|etapa 6]].

**Prueba** `tests/t11.c`:

```c
#include <stdio.h>
#include "ft_printf.h"

int	main(void)
{
	int	r;
	int	x;

	r = ft_printf("[%p]\n", (void *)0x2a);
	fprintf(stderr, "r=%d\n", r);
	r = ft_printf("[%p]\n", NULL);
	fprintf(stderr, "r=%d\n", r);
	r = ft_printf("[%p]\n", (void *)-1);
	fprintf(stderr, "r=%d\n", r);
	r = ft_printf("%p\n", &x);
	printf("%p   <- printf, debe coincidir\n", &x);
	return (0);
}
```

```bash
cc -Wall -Wextra -Werror -Iincludes -Ilibft tests/t11.c srcs/ft_output.c srcs/ft_printf.c srcs/ft_flags.c srcs/ft_print_chars.c srcs/ft_print_nums.c libft/libft.a -o test && ./test
```

**Debe salir** (la dirección de `x` cambia en cada ejecución, pero las dos líneas deben ser iguales):

```text
[0x2a]
r=7
[(nil)]
r=8
[0xffffffffffffffff]
r=21
0x7ffca37173b4
0x7ffca37173b4   <- printf, debe coincidir
```

> [!failure] Si falla
> - `[0xffffffff]` con `(void *)-1`: guardas el puntero en un `unsigned int`; debe ser `unsigned long`.
> - `[0x0]` con `NULL`: falta el caso especial; glibc (Linux) imprime `(nil)`.

## Paso 12: Makefile y control final de la obligatoria

La obligatoria está completa. Antes de tocar el bonus:

- [ ] **Makefile** con las tres listas (comunes, `ft_flags.c`, `ft_flags_bonus.c`): [[ft_printf - Guía paso a paso#En el Makefile|guía, sección 6]]. `make` debe crear `libftprintf.a`, y `make` dos veces seguidas debe decir `Nothing to be done for 'all'`.
- [ ] **Tester** de la [[ft_printf - Guía paso a paso#Comparar contra `printf` automáticamente|guía, sección 11]], sin `-DBONUS`. Debe dar `obligatoria: 9 OK, 0 KO`.
- [ ] **Valgrind**: `valgrind -q --leak-check=full ./t` no imprime nada.
- [ ] **Error de `write`**: con `./t >&-`, `ft_printf` devuelve `-1`.
- [ ] **Norminette**: `norminette includes srcs`, solo con el error de la cabecera de 42 si aún no la has puesto.
- [ ] **Argumentos alineados**: `ft_printf("[%5d] [%s]\n", 42, "hola")` da `[42] [hola]`, sin crash.

Más pruebas y cómo depurar cada fallo en [[ft_printf - Pruebas y depuración]].

> [!warning] Si algo de esta lista falla, no sigas
> El bonus solo se evalúa si la obligatoria está **perfecta**.

## Paso 13: `ft_set_flags` bonus

**Escribe** `srcs/ft_flags_bonus.c` (nuevo):
- La `static` `ft_read_number`, que lee el ancho o la precisión.
- La `static` `ft_set_one_flag`, que activa una bandera.
- Otra versión de `ft_set_flags`, con el **mismo prototipo**, que lee banderas, ancho, `.precisión` y la letra.

**No tocas ningún otro fichero.** Código en [[ft_printf - Guía paso a paso#Etapa 7: el bonus (`ft_flags_bonus.c`)|etapa 7]].

**Prueba** `tests/t12.c`, compilando `ft_flags_bonus.c` **en lugar de** `ft_flags.c`:

```c
#include <stdio.h>
#include "ft_printf.h"

static void	prueba(const char *s)
{
	t_flags	f;
	int		i;

	i = 1;
	f = ft_set_flags(s, &i);
	printf("%-9s spec=%c i=%d minus=%d zero=%d width=%d prec=%d"
		" sharp=%d space=%d plus=%d\n", s, f.spec, i, f.minus, f.zero,
		f.width, f.precision, f.sharp, f.space, f.plus);
}

int	main(void)
{
	prueba("%d");
	prueba("%-08.3d");
	prueba("%#x");
	prueba("%+ 5d");
	prueba("%.d");
	prueba("%50d");
	return (0);
}
```

```bash
cc -Wall -Wextra -Werror -Iincludes -Ilibft tests/t12.c srcs/ft_output.c srcs/ft_printf.c srcs/ft_flags_bonus.c srcs/ft_print_chars.c srcs/ft_print_nums.c libft/libft.a -o test && ./test
```

**Debe salir:**

```text
%d        spec=d i=2 minus=0 zero=0 width=0 prec=-1 sharp=0 space=0 plus=0
%-08.3d   spec=d i=7 minus=1 zero=0 width=8 prec=3 sharp=0 space=0 plus=0
%#x       spec=x i=3 minus=0 zero=0 width=0 prec=-1 sharp=1 space=0 plus=0
%+ 5d     spec=d i=5 minus=0 zero=0 width=5 prec=-1 sharp=0 space=0 plus=1
%.d       spec=d i=3 minus=0 zero=0 width=0 prec=0 sharp=0 space=0 plus=0
%50d      spec=d i=4 minus=0 zero=0 width=50 prec=-1 sharp=0 space=0 plus=0
```

- `%-08.3d`: `zero=0` aunque el formato tenga `0`, porque el `-` lo anula.
- `%+ 5d`: `space=0` aunque tenga espacio, porque el `+` lo anula.
- `%.d`: un `.` sin número es precisión `0`, no `-1`.
- `%50d`: el `0` de `50` es parte del ancho, no una bandera (`zero=0`).

> [!failure] Si falla
> - `multiple definition of 'ft_set_flags'`: has compilado `ft_flags.c` y `ft_flags_bonus.c` a la vez.
> - `zero=1` en `%50d`: lees el `0` como bandera después de haber empezado el ancho.

## Paso 14: el bonus completo

No hay que escribir nada más: `ft_out_field`, `ft_print_number` y compañía ya sabían rellenar desde el paso 3. Ahora simplemente reciben banderas.

**Prueba** `tests/t13.c`:

```c
#include <stdio.h>
#include "ft_printf.h"

int	main(void)
{
	int	r;

	r = ft_printf("[%8d] [%-8d] [%08d] [%08.3d]\n", -42, -42, -42, -42);
	fprintf(stderr, "r=%d\n", r);
	r = ft_printf("[%+d] [% d] [%#x] [%.0d] [%-7.3s]\n", 42, 42, 255, 0, "hola");
	fprintf(stderr, "r=%d\n", r);
	return (0);
}
```

```bash
cc -Wall -Wextra -Werror -Iincludes -Ilibft tests/t13.c srcs/ft_output.c srcs/ft_printf.c srcs/ft_flags_bonus.c srcs/ft_print_chars.c srcs/ft_print_nums.c libft/libft.a -o test && ./test
```

**Debe salir** (igual que `printf`):

```text
[     -42] [-42     ] [-0000042] [    -042]
r=44
[+42] [ 42] [0xff] [] [hol    ]
r=32
```

Para terminar:

- [ ] `make bonus` y el tester con `-DBONUS`: `bonus: 26 OK, 0 KO`.
- [ ] `make bonus` dos veces: la segunda no ejecuta ni `cc` ni `ar`.
- [ ] `ar t libftprintf.a` después de `make bonus` contiene `ft_flags_bonus.o` y **no** `ft_flags.o`.
- [ ] Valgrind y norminette, como en el paso 12.
- [ ] `tests/` no está en el repositorio (`git status` no la muestra).
- [ ] El README ([[ft_printf - Guía paso a paso#12. El README|guía, sección 12]]).

## Relacionadas

[[ft_printf - Guía paso a paso]] · [[ft_printf - Funciones auxiliares]] · [[ft_printf - Pruebas y depuración]] · [[ft_printf - Índice]] · [[Valgrind y sanitizers]] · [[Makefile y librerías estáticas]] · [[Buffer de stdout]]

## Referencias

- Todo el código de los pasos es el de [[ft_printf - Guía paso a paso]], compilado de forma incremental (cada paso solo con los ficheros y funciones escritos hasta ese momento) con `cc -Wall -Wextra -Werror` (Ubuntu clang 12, Ubuntu 22.04). Cada paso pasa Valgrind sin errores y norminette (salvo la cabecera de 42). Las salidas de esta nota son las obtenidas en esa prueba; las del paso 14 coinciden con el `printf` de glibc.
