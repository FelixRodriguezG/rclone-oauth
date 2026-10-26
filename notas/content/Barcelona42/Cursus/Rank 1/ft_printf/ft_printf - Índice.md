---
tags: [proyecto, ft_printf, indice, rank-1, variadicas]
aliases: ["ft_printf", "libftprintf"]
---
# ft_printf - Índice

[[Inicio|← Inicio]] · Proyectos · Rank 1

> **ft_printf** consiste en reprogramar una versión reducida de [[printf]]: una función que recibe una cadena de formato y un número variable de argumentos, y escribe el resultado con `write`. Sirve para aprender [[stdarg y funciones variádicas|funciones variádicas]], conversiones de números a texto en distintas bases y a organizar un proyecto en varios ficheros.

> [!warning] Comprueba el subject
> Resumen basado en versiones públicas del subject; los detalles cambian entre versiones. Lo que manda es el PDF de tu intra.

## Qué se pide

Lo comúnmente conocido del subject (no lo tienes delante en este vault):

| Aspecto | Lo habitual |
|---|---|
| Prototipo | `int ft_printf(const char *, ...);` |
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
> En 42 el bonus solo se evalúa si la parte obligatoria está **perfecta**. Si tu diseño no está pensado para banderas desde el principio, añadirlas después cuesta mucho. Decide antes de empezar si vas a por el bonus.

## Conceptos que necesitas

| Concepto | Para qué en ft_printf | Nota |
|---|---|---|
| Funciones variádicas | Leer los argumentos de `...` | [[stdarg y funciones variádicas]] |
| `printf` original | Saber qué salida debes imitar | [[printf]] |
| Tipos enteros | `int` vs `unsigned int` vs `unsigned long`, desbordamiento de `INT_MIN` | [[Tipos enteros y conversiones]] |
| Recursión | Imprimir un número en cualquier base | [[Recursión]] |
| Punteros | Qué es una dirección y cómo convertirla a número | [[Punteros en C]] |
| `write` y descriptores | Escribir en la salida estándar (fd 1) | [[Descriptores de fichero y read]] |
| Librería estática | Generar `libftprintf.a` e incluir libft | [[Makefile y librerías estáticas]] |
| Headers | Un `ft_printf.h` limpio con prototipos | [[Preprocesador y headers]] |
| Errores | Propagar el `-1` de `write` | [[Control de errores en C]] |

Funciones de libft que reutilizarás: [[ft_putchar_fd]], [[ft_putstr_fd]], [[ft_strlen]] y la idea de [[ft_putnbr_fd]] (pero ojo: esas no devuelven cuántos caracteres han escrito, y tú lo necesitas).

## Cómo organizar el código

Idea general: recorres `format` carácter a carácter. Si no es `%`, lo escribes. Si es `%`, miras la letra siguiente y llamas a la función que toca. **Cada función devuelve cuántos caracteres ha escrito** (o `-1` si falló `write`), y el bucle principal los va sumando.

```text
ft_printf("Hola %s, tienes %d años\n", "Ana", 20)

format:  H o l a ␣ % s , ␣ t i e n e s ␣ % d ␣ a ñ o s \n
         └─ write ─┘ │                     │
                     └→ print_str("Ana")    └→ print_nbr(20)
cuenta:  5         + 3 + 9               + 2 + ...
```

Una estructura de ficheros típica:

```text
ft_printf/
├── Makefile
├── ft_printf.h
├── ft_printf.c          ← ft_printf() y el reparto por conversión
├── ft_print_chars.c     ← %c, %s, %%
├── ft_print_nums.c      ← %d, %i, %u, %x, %X
├── ft_print_ptr.c       ← %p
└── libft/               ← si usas tu libft
```

### Bucle principal y reparto

Ejemplo orientativo que cumple la [[La Norma|Norma]] (máx. 25 líneas, declaraciones arriba, sin `for`):

```c
static int	ft_convert(char c, va_list *args)
{
	if (c == 'c')
		return (ft_print_char(va_arg(*args, int)));
	if (c == 's')
		return (ft_print_str(va_arg(*args, char *)));
	if (c == 'p')
		return (ft_print_ptr(va_arg(*args, void *)));
	if (c == 'd' || c == 'i')
		return (ft_print_nbr(va_arg(*args, int)));
	if (c == 'u')
		return (ft_print_base(va_arg(*args, unsigned int), "0123456789"));
	if (c == 'x')
		return (ft_print_base(va_arg(*args, unsigned int), HEX_LOW));
	if (c == 'X')
		return (ft_print_base(va_arg(*args, unsigned int), HEX_UP));
	if (c == '%')
		return (ft_print_char('%'));
	return (0);
}

int	ft_printf(const char *format, ...)
{
	va_list	args;
	int		count;
	int		ret;

	if (!format)
		return (-1);
	va_start(args, format);
	count = 0;
	while (*format)
	{
		if (*format == '%' && *(format + 1))
			ret = ft_convert(*(++format), &args);
		else
			ret = ft_print_char(*format);
		if (ret == -1)
		{
			va_end(args);
			return (-1);
		}
		count += ret;
		format++;
	}
	va_end(args);
	return (count);
}
```

`HEX_LOW` y `HEX_UP` serían macros del header (`"0123456789abcdef"` y `"0123456789ABCDEF"`). La `va_list` se pasa **por puntero** a `ft_convert` por el motivo explicado en [[stdarg y funciones variádicas#Aplicación en ft_printf]].

### Imprimir en cualquier base

Una sola función recursiva sirve para `%u`, `%x`, `%X` y `%p`. Primero imprime «todo menos la última cifra» y luego la última:

```c
int	ft_print_base(unsigned long n, char *base)
{
	unsigned long	len;
	int				count;
	int				ret;

	len = ft_strlen(base);
	count = 0;
	if (n >= len)
	{
		count = ft_print_base(n / len, base);
		if (count == -1)
			return (-1);
	}
	ret = ft_print_char(base[n % len]);
	if (ret == -1)
		return (-1);
	return (count + ret);
}
```

```text
ft_print_base(42, hex)
 └ 42 >= 16 → ft_print_base(2, hex)   escribe "2"  (devuelve 1)
 └ escribe base[42 % 16] = base[10] = "a"           (1 + 1 = 2)
salida: 2a
```

Recibe `unsigned long` para que también valga para punteros (64 bits en Linux x86_64). Pasar un `unsigned int` a un parámetro `unsigned long` es seguro: el valor se conserva.

## Casos límite

> [!danger] Los que más suspenden
> | Caso | Qué hace la `printf` de glibc (Linux) | Cómo resolverlo |
> |---|---|---|
> | `%s` con `NULL` | Escribe `(null)` | `if (!s) return (ft_print_str("(null)"));` |
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

`write` devuelve `-1` si no puede escribir (por ejemplo, si la salida estándar está cerrada). La `printf` original devuelve entonces un valor negativo. Muchas correcciones prueban esto, así que **comprueba el retorno de cada `write`** y propaga el `-1` hasta arriba (como en el código de ejemplo).

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
> `printf` usa un buffer y tu `ft_printf` escribe directamente con `write`. Si rediriges a un fichero o a una tubería, el orden de las líneas puede salir mezclado. Para comparar, escribe la salida de cada una en un programa distinto y usa `diff`, o llama a `fflush(stdout)` tras cada `printf`.

> [!warning] gcc y `%s` con `NULL`
> Con `printf("%s\n", NULL)` gcc puede sustituir la llamada por `puts(NULL)`, que **crashea**. No es un fallo tuyo: para probar ese caso usa un formato con más texto (`"[%s]\n"`) o compila con `-fno-builtin`. gcc además avisa del `NULL` con `-Wall`; en tu `main` de pruebas quita `-Werror` si hace falta.

Herramientas útiles:

- [[Valgrind y sanitizers]]: `valgrind ./test_printf` o compilar con `-fsanitize=address` para ver accesos fuera de memoria.
- Testers públicos: printfTester y francinette (enlaces en Referencias). Úsalos **después** de tus propias pruebas: no sustituyen entender cada caso.

## Plan de trabajo

1. Lee el subject de tu intra y anota las conversiones y funciones permitidas.
2. Repasa [[stdarg y funciones variádicas]] y escribe el ejemplo de sumar N enteros.
3. Crea el `Makefile` que genera `libftprintf.a` (y compila libft si la usas). Ver [[Makefile y librerías estáticas]].
4. Escribe `ft_printf` con solo `%c`, `%s` y `%%`. Prueba el valor de retorno.
5. Añade `ft_print_base` y con ella `%u`, `%x`, `%X`.
6. Añade `%d`/`%i` con el caso `INT_MIN`.
7. Añade `%p` con el caso `NULL`.
8. Comprueba el `-1` de `write` en todas las funciones.
9. Pasa norminette, valgrind y los testers.
10. Bonus (opcional): diseña una `struct` con las banderas leídas antes de convertir. Ver [[Structs y typedef]].

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
> Es habitual que el evaluador pida, por ejemplo, añadir una conversión `%b` (binario). Con `ft_print_base` basta una línea: `ft_print_base(va_arg(*args, unsigned int), "01")`.

## Relacionadas

[[printf]] · [[stdarg y funciones variádicas]] · [[Tipos enteros y conversiones]] · [[Recursión]] · [[Makefile y librerías estáticas]] · [[Valgrind y sanitizers]] · [[libft - Índice]] · [[get_next_line - Índice]] · [[Born2beroot - Índice]]

## Referencias

- [printf(3) — Linux man-pages](https://man7.org/linux/man-pages/man3/printf.3.html) — conversiones `cspdiuxX%`, valor de retorno negativo en caso de error.
- [stdarg(3) — Linux man-pages](https://man7.org/linux/man-pages/man3/stdarg.3.html) — `va_start`, `va_arg`, `va_copy`, `va_end`.
- [write(2) — Linux man-pages](https://man7.org/linux/man-pages/man2/write.2.html) — retorno `-1` y `EBADF`.
- [cppreference: Variadic arguments](https://en.cppreference.com/w/c/variadic) — promoción de argumentos por defecto.
- [cppreference: printf, fprintf…](https://en.cppreference.com/w/c/io/fprintf) — comportamiento indefinido con conversiones no válidas.
- [printfTester (Tripouille)](https://github.com/Tripouille/printfTester) y [francinette](https://github.com/xicodomingues/francinette) — testers públicos de la comunidad.
- Requisitos del proyecto: conocimiento general de versiones públicas del subject de ft_printf (no verificado contra el PDF). Comportamiento de `(null)` y `(nil)`: glibc en Linux.
