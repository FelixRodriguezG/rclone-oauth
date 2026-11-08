---
tags: [proyecto, ft_printf, indice, rank-1, variadicas]
aliases: ["ft_printf", "libftprintf"]
---
# ft_printf - Índice

[[Inicio|← Inicio]] · Proyectos · Rank 1

> **ft_printf** consiste en reprogramar una versión reducida de [[printf]]: una función que recibe una cadena de formato y un número variable de argumentos, y escribe el resultado con `write`. Sirve para aprender [[stdarg y funciones variádicas|funciones variádicas]], conversiones de números a texto en distintas bases y a organizar un proyecto en varios ficheros.

> [!warning] Comprueba el subject
> Resumen basado en versiones públicas del subject; los detalles cambian entre versiones. Lo que manda es el PDF de tu intra. Los requisitos exactos de la v12.0 están en [[ft_printf - Guía paso a paso#1. Qué te piden, en una frase|la guía]].

> [!abstract] Notas de esta serie
> 1. [[ft_printf - Guía paso a paso]]: construir ft_printf **con el bonus pensado desde el principio**, etapa a etapa.
> 2. [[ft_printf - Funciones auxiliares]]: `ft_out_field`, `ft_utoa_base` y el resto de piezas explicadas una a una.
> 3. [[ft_printf - Pruebas y depuración]]: tester contra `printf`, revisión del Makefile, gdb y cómo se ve cada error típico.
> 4. [[ft_printf - Ruta de trabajo]]: el orden de trabajo, función a función, con un `main` de prueba y la salida esperada en cada paso.
> 5. **Esta nota**: el resumen del proyecto, para repasar.

## Qué se pide

Lo comúnmente conocido del subject (no lo tienes delante en este vault):

| Aspecto | Lo habitual |
|---|---|
| Prototipo | `int ft_printf(char const *, ...);` |
| Entrega | Una librería estática `libftprintf.a` creada con `ar` (no `libtool`) |
| Funciones externas | `malloc`, `free`, `write`, `va_start`, `va_arg`, `va_copy`, `va_end` |
| libft | Se puede usar: se copia la carpeta `libft` con su Makefile dentro del repo |
| Buffer | **No** hay que reproducir el buffer interno de la `printf` original |
| Makefile | `$(NAME)`, `all`, `clean`, `fclean`, `re` (y `bonus`), sin relink |
| Compilación | `cc -Wall -Wextra -Werror` |

### Conversiones obligatorias

Son nueve: `c s p d i u x X %`. La tabla completa con ejemplos está en [[printf#Especificadores de formato]]; aquí solo lo que importa al implementarlas:

| Conversión | Lees con `va_arg` | Escribes |
|---|---|---|
| `%c` | `int` | Un byte |
| `%s` | `char *` | La cadena (o `(null)` si es `NULL`) |
| `%p` | `void *` | `0x` + dirección en hexadecimal minúscula |
| `%d` `%i` | `int` | Decimal con signo |
| `%u` | `unsigned int` | Decimal sin signo |
| `%x` `%X` | `unsigned int` | Hexadecimal minúscula / mayúscula |
| `%%` | Nada | Un `%` |

Por qué `%c` se lee como `int`: la **promoción de argumentos por defecto** (un `char` que pasa por `...` llega convertido en `int`). Explicado en [[stdarg y funciones variádicas#Promoción de tipos por defecto]].

### Bonus

Lo habitual en el bonus:

- Banderas `-`, `0`, `.` y ancho mínimo de campo, en cualquier combinación.
- Banderas `#`, espacio y `+`.

Ejemplos de lo que hacen en [[printf#Ancho, precisión y banderas]].

> [!tip] Haz el bonus solo si la parte obligatoria es perfecta
> En 42 el bonus solo se evalúa si la parte obligatoria está **perfecta**. Si tu diseño no está pensado para banderas desde el principio, añadirlas después cuesta mucho. Decide antes de empezar si vas a por el bonus. El diseño que lo permite está en [[ft_printf - Guía paso a paso#2. El enfoque ingenuo y por qué se rompe con el bonus|la guía]].

## Conceptos que necesitas

| Concepto | Para qué en ft_printf | Nota |
|---|---|---|
| Funciones variádicas | Leer los argumentos de `...` | [[stdarg y funciones variádicas]] |
| `printf` original | Saber qué salida debes imitar | [[printf]] |
| Tipos enteros | `int` vs `unsigned int` vs `unsigned long`, desbordamiento de `INT_MIN` | [[Tipos enteros y conversiones]] |
| Bases numéricas | Pasar un número a texto en base 10 o 16 (`ft_utoa_base`) | [[Bases numéricas]] |
| Punteros | Qué es una dirección y cómo convertirla a número | [[Punteros en C]] |
| `write` y descriptores | Escribir en la salida estándar (fd 1) | [[Descriptores de fichero y read]] |
| Librería estática | Generar `libftprintf.a` e incluir libft | [[Makefile y librerías estáticas]] |
| Headers | Un `ft_printf.h` limpio con prototipos | [[Preprocesador y headers]] |
| Errores | Que un fallo de `write` acabe en `-1` (`t_out.error`) | [[Control de errores en C]] |

Funciones de libft que reutilizarás: [[ft_strlen]], [[ft_strchr]] y [[ft_isdigit]]. [[ft_putchar_fd]], [[ft_putstr_fd]] y [[ft_putnbr_fd]] no sirven tal cual: no dicen si `write` ha fallado ni cuántos caracteres han escrito (ver [[ft_printf - Funciones auxiliares]]).

## Cómo organizar el código

> [!note] Un solo diseño en toda la serie
> Esta nota usa los **mismos nombres y prototipos** que [[ft_printf - Guía paso a paso|la guía]], [[ft_printf - Funciones auxiliares]] y [[ft_printf - Pruebas y depuración]]. Es un diseño pensado para el bonus desde el principio, pero la parte obligatoria funciona sola. El código completo, etapa a etapa, está en la guía.

La idea general: recorres `format` carácter a carácter. Si no es `%`, lo escribes. Si es `%`, `ft_set_flags` lee lo que viene detrás (en la obligatoria, se salta las banderas y se queda con la letra) y `ft_convert` llama a la función de esa conversión. Ninguna función devuelve cuántos caracteres ha escrito: todas **apuntan en un `t_out`** lo que escriben (`len`) y si `write` ha fallado (`error`). Al final, `ft_printf` devuelve `-1` si hubo error y `len` si no.

```text
ft_printf("Hola %s, tienes %d años\n", "Ana", 20)

format:  H o l a ␣ % s , ␣ t i e n e s ␣ % d ␣ a ñ o s \n
         └ out_char┘ │                     │
                     └→ ft_print_str        └→ ft_print_int
                        → ft_out_field          → ft_print_number → ft_out_field
out.len: 5         + 3 + 9               + 2 + ...
```

La estructura de ficheros:

```text
ft_printf/
├── Makefile
├── README.md
├── includes/
│   └── ft_printf.h          ← las tres struct y todos los prototipos
├── libft/                   ← tu libft con su Makefile
└── srcs/
    ├── ft_printf.c          ← ft_printf, ft_init_flags y ft_convert (el reparto por letra)
    ├── ft_output.c          ← lo ÚNICO que llama a write: ft_out_char, ft_out_nstr, ft_out_field
    ├── ft_print_chars.c     ← %c, %s, %p
    ├── ft_print_nums.c      ← %d %i %u %x %X, ft_utoa_base y ft_base
    ├── ft_flags.c           ← ft_set_flags OBLIGATORIA (salta las banderas, lee la letra)
    └── ft_flags_bonus.c     ← ft_set_flags BONUS (banderas, ancho, precisión)
```

### Los prototipos

Lo que hay en `ft_printf.h` (las `struct` explicadas en [[ft_printf - Guía paso a paso#4. Tres estructuras|la guía]]):

```c
int		ft_printf(char const *format, ...);
t_flags	ft_init_flags(void);
t_flags	ft_set_flags(const char *s, int *i);
void	ft_out_char(t_out *o, char c);
void	ft_out_nstr(t_out *o, const char *s, int n);
void	ft_out_field(t_out *o, t_flags *f, t_field *fl);
void	ft_print_char(t_out *o, t_flags *f, int c);
void	ft_print_str(t_out *o, t_flags *f, const char *s);
void	ft_print_ptr(t_out *o, t_flags *f, void *p);
void	ft_print_int(t_out *o, t_flags *f, int n);
void	ft_print_unsigned(t_out *o, t_flags *f, unsigned int n);
void	ft_print_number(t_out *o, t_flags *f, unsigned long n,
			const char *prefix);
```

Y las `static`, que no van en el header:

| Función | Fichero | Para qué |
|---|---|---|
| `static void ft_convert(t_out *o, t_flags *f, va_list *args)` | `ft_printf.c` | Llama a la `ft_print_*` de la letra |
| `static void ft_out_repeat(t_out *o, char c, int n)` | `ft_output.c` | Relleno: `n` veces el mismo carácter |
| `static int ft_utoa_base(unsigned long n, const char *base, char *buf)` | `ft_print_nums.c` | Número a texto en cualquier base, sin `malloc` |
| `static const char *ft_base(char spec)` | `ft_print_nums.c` | La base según la letra (`x`/`p` → hex minúscula, `X` → mayúscula, resto → decimal) |
| `static int ft_read_number(const char *s, int *i)` | `ft_flags_bonus.c` | Lee el ancho o la precisión |
| `static void ft_set_one_flag(t_flags *f, char c)` | `ft_flags_bonus.c` | Activa una bandera de `-0# +` |

Cada una explicada en [[ft_printf - Funciones auxiliares]].

### El reparto por conversión

| Conversión | Lees con `va_arg` | Función |
|---|---|---|
| `%c` | `int` | `ft_print_char` |
| `%s` | `char *` | `ft_print_str` |
| `%p` | `void *` | `ft_print_ptr` → `ft_print_number(o, f, (unsigned long)p, "0x")` |
| `%d` `%i` | `int` | `ft_print_int` → `ft_print_number` con prefijo `"-"`, `"+"`, `" "` o `""` |
| `%u` `%x` `%X` | `unsigned int` | `ft_print_unsigned` → `ft_print_number` (la base la elige `ft_base`) |
| `%%` | Nada | `ft_out_char(o, '%')` |

La `va_list` se pasa **por puntero** a `ft_convert` por el motivo explicado en [[stdarg y funciones variádicas#Aplicación en ft_printf]]. `ft_print_number` recibe `unsigned long` para que valga también para punteros (64 bits en Linux x86_64); pasar un `unsigned int` a un parámetro `unsigned long` es seguro: el valor se conserva.

## Casos límite

> [!danger] Los que más suspenden
> | Caso | Qué hace la `printf` de glibc (Linux) | Cómo resolverlo |
> |---|---|---|
> | `%s` con `NULL` | Escribe `(null)` | En `ft_print_str`: `if (!s) s = "(null)";` |
> | `%p` con `NULL` | Escribe `(nil)` (macOS: `0x0`) | Caso especial antes de convertir |
> | `%d` con `INT_MIN` | `-2147483648` | No hagas `n = -n` sobre un `int`: pásalo a `long` antes |
> | `%u` con `-1` | `4294967295` | Leerlo como `unsigned int`, nunca como `int` |
> | `%x` con número negativo | `ffffffd6` para `-42` | Igual: `unsigned int` |
> | `%c` con `'\0'` | Escribe un byte nulo y cuenta **1** | No uses funciones de cadena para `%c` |
> | Valor de retorno | Total de bytes escritos | Súmalo en todas las ramas |

### Por qué falla `INT_MIN`

```text
int n = -2147483648;   (INT_MIN)
-n  → +2147483648  no cabe en int (máx. 2147483647) → desbordamiento: comportamiento indefinido
long nb = n; nb = -nb; → 2147483648 sí cabe en long ✅
```

Más detalle en [[Tipos enteros y conversiones]].

### El `-1` de `write`

`write` devuelve `-1` si no puede escribir (por ejemplo, si la salida estándar está cerrada). La `printf` original devuelve entonces un valor negativo. Muchas correcciones prueban esto, así que **comprueba el retorno de cada `write`**. En este diseño solo lo hacen `ft_out_char` y `ft_out_nstr`: anotan `o->error = 1` y, al final, `ft_printf` devuelve `-1` (explicado en [[ft_printf - Funciones auxiliares#`ft_out_char` y `ft_out_nstr`: escribir y contar|Funciones auxiliares]]).

```bash
./test_printf >&-        # ejecuta con stdout cerrado: write falla con EBADF
```

> [!note] Comportamiento indefinido
> `ft_printf("%")` (un `%` solo al final), conversiones desconocidas como `%k` o pasar un argumento de tipo equivocado son **comportamiento indefinido** en la `printf` original. No te pueden exigir un resultado concreto, pero tu función **no debe crashear**.

## Cómo probar contra printf

La idea: llamar a las dos con los mismos argumentos y comparar **salida** y **valor de retorno**.

```c
#include <stdio.h>
#include <limits.h>
#include "ft_printf.h"

int	main(void)
{
	int	a;
	int	b;

	a = printf("[%d %i %u %x %X %%]\n", INT_MIN, INT_MAX, -1, 255, 255);
	b = ft_printf("[%d %i %u %x %X %%]\n", INT_MIN, INT_MAX, -1, 255, 255);
	printf("printf: %d · ft_printf: %d\n", a, b);
	a = printf("[%s] [%p] [%c]\n", (char *)NULL, NULL, 'A');
	b = ft_printf("[%s] [%p] [%c]\n", (char *)NULL, NULL, 'A');
	printf("printf: %d · ft_printf: %d\n", a, b);
	return (0);
}
```

```bash
cc -Wall -Wextra -Werror main.c -L. -lftprintf -o test_printf
./test_printf | cat -e          # cat -e marca el final de línea con $ y deja ver bytes raros
```

> [!warning] Mezclar `printf` y `ft_printf` en el mismo programa
> `printf` usa un buffer y tu `ft_printf` escribe directamente con `write`. Si rediriges a un fichero o a una tubería, el orden de las líneas puede salir mezclado. Para comparar, escribe la salida de cada una en un programa distinto y usa `diff`, o llama a `fflush(stdout)` tras cada `printf`. Explicado en [[Buffer de stdout]].

> [!warning] gcc y `%s` con `NULL`
> Con `printf("%s\n", NULL)` gcc puede sustituir la llamada por `puts(NULL)`, que **crashea**. No es un fallo tuyo: para probar ese caso usa un formato con más texto (`"[%s]\n"`) o compila con `-fno-builtin`. gcc además avisa del `NULL` con `-Wall`; en tu `main` de pruebas quita `-Werror` si hace falta.

Herramientas útiles:

- [[Valgrind y sanitizers]]: `valgrind ./test_printf` o compilar con `-fsanitize=address` para ver accesos fuera de memoria.
- Testers públicos: printfTester y francinette (enlaces en Referencias). Úsalos **después** de tus propias pruebas: no sustituyen entender cada caso.

## Plan de trabajo

1. Lee el subject de tu intra y anota las conversiones y funciones permitidas.
2. Repasa [[stdarg y funciones variádicas]] y escribe el ejemplo de sumar N enteros.
3. Crea el `Makefile` que genera `libftprintf.a` (y compila libft si la usas). Ver [[Makefile y librerías estáticas]].
4. Escribe la salida (`ft_out_char`, `ft_out_nstr`) y `ft_out_field`.
5. Escribe `ft_printf`, `ft_set_flags` obligatoria y `ft_convert` con `%c`, `%s` y `%%`. Prueba el valor de retorno.
6. Añade `ft_utoa_base`, `ft_base` y `ft_print_number`, y con ellas `%d`/`%i` (caso `INT_MIN`), `%u`, `%x`, `%X`.
7. Añade `%p` con el caso `NULL`.
8. Comprueba que un fallo de `write` acaba en `-1` (`./test_printf >&-`).
9. Pasa norminette, valgrind y los testers.
10. Bonus (opcional): escribe `ft_flags_bonus.c` con la `ft_set_flags` que lee banderas, ancho y precisión. El resto no cambia. Detalle en [[ft_printf - Guía paso a paso#Etapa 7: el bonus (`ft_flags_bonus.c`)|la guía]].

## Errores frecuentes que suspenden

> [!danger] Lista de comprobación antes de entregar
> - Valor de retorno distinto del de `printf` en algún caso (sobre todo `%c` con `'\0'` y `NULL`).
> - Crash con `%s` a `NULL`.
> - `INT_MIN` mal impreso o desbordado.
> - `%u`/`%x` leídos como `int`: los negativos salen con signo.
> - `%p` truncado por guardar la dirección en un `int` o `unsigned int`.
> - Falta `va_end` en algún camino de salida (incluido el de error).
> - El Makefile hace relink, o no recompila libft cuando cambia.
> - `ar` sin `libftprintf.a` en la raíz, o con el nombre equivocado.
> - Fallos de norminette (funciones de más de 25 líneas, más de 5 funciones por fichero).

## Preguntas de evaluación

> [!question] ¿Por qué no puedes hacer `va_arg(args, char)`?
> Porque los argumentos variádicos sufren promoción por defecto: un `char` llega como `int`. Pedir `char` es comportamiento indefinido.

> [!question] ¿Cómo sabe ft_printf cuántos argumentos tiene?
> No lo sabe. Lo deduce de la cadena de formato: un argumento por cada conversión (salvo `%%`). Si pasas menos, es comportamiento indefinido.

> [!question] ¿Qué devuelve tu función y qué pasa si `write` falla?
> El número de caracteres escritos. Si `write` devuelve `-1`, se propaga y ft_printf devuelve `-1`.

> [!question] ¿Por qué `%p` usa `unsigned long`?
> Porque en un sistema de 64 bits una dirección ocupa 8 bytes y no cabe en un `int` (4 bytes).

> [!question] ¿Qué diferencia hay entre `%d` y `%i`?
> En `printf`, ninguna. Solo se comportan distinto en `scanf` (`%i` acepta octal y hexadecimal).

> [!question] Cambia algo en directo
> Es habitual que el evaluador pida, por ejemplo, añadir una conversión `%b` (binario). Basta con dos líneas: `if (spec == 'b') return ("01");` en `ft_base`, y añadir `'b'` junto a `u x X` en el `if` de `ft_convert`. El buffer `digits[MAX_DIGITS]` (64) ya tiene sitio para las 32 cifras binarias de un `unsigned int`. Más ejemplos en [[ft_printf - Guía paso a paso#Modificaciones en directo|la guía]].

## Relacionadas

[[ft_printf - Guía paso a paso]] · [[ft_printf - Ruta de trabajo]] · [[ft_printf - Funciones auxiliares]] · [[ft_printf - Pruebas y depuración]] · [[printf]] · [[stdarg y funciones variádicas]] · [[Tipos enteros y conversiones]] · [[Recursión]] · [[Makefile y librerías estáticas]] · [[Valgrind y sanitizers]] · [[libft - Índice]] · [[get_next_line - Índice]] · [[Born2beroot - Índice]] · [[Bases numéricas]] · [[Buffer de stdout]] · [[Comportamiento indefinido]]

## Referencias

- [printf(3) — Linux man-pages](https://man7.org/linux/man-pages/man3/printf.3.html) — conversiones `cspdiuxX%`, valor de retorno negativo en caso de error.
- [stdarg(3) — Linux man-pages](https://man7.org/linux/man-pages/man3/stdarg.3.html) — `va_start`, `va_arg`, `va_copy`, `va_end`.
- [write(2) — Linux man-pages](https://man7.org/linux/man-pages/man2/write.2.html) — retorno `-1` y `EBADF`.
- [cppreference: Variadic arguments](https://en.cppreference.com/w/c/variadic) — promoción de argumentos por defecto.
- [cppreference: printf, fprintf…](https://en.cppreference.com/w/c/io/fprintf) — comportamiento indefinido con conversiones no válidas.
- [printfTester (Tripouille)](https://github.com/Tripouille/printfTester) y [francinette](https://github.com/xicodomingues/francinette) — testers públicos de la comunidad.
- Requisitos del proyecto: conocimiento general de versiones públicas del subject de ft_printf (no verificado contra el PDF). Comportamiento de `(null)` y `(nil)`: glibc en Linux.
