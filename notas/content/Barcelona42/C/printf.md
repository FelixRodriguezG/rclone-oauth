---
tags: [c, printf, ft_printf, stdio]
aliases: ["printf — formato e impresión en C"]
---
# printf

[[Inicio|← Inicio]] · C

> `printf` (*print formatted*) imprime texto con formato en la salida estándar (`stdout`). Es la función que tendrás que reimplementar en el proyecto **[[ft_printf - Índice|ft_printf]]**.

| Término | Qué significa |
|---|---|
| **Cadena de formato** | El primer argumento: texto normal mezclado con huecos que empiezan por `%` |
| **Especificador** (o conversión) | Cada hueco `%algo`: indica qué tipo de dato va ahí y cómo escribirlo |
| **`stdout`** | Salida estándar, el *file descriptor* 1: normalmente la terminal |
| **Función variádica** | Función que acepta un número variable de argumentos, como `printf` |

## Prototipo

```c
#include <stdio.h>

int	printf(const char *format, ...);
```

- `format`: la cadena con el texto y los **especificadores** (`%d`, `%s`…).
- `...`: un número variable de argumentos, uno por especificador. Ver [[stdarg y funciones variádicas]].

> [!info] Valor de retorno
> Devuelve el **número de caracteres impresos** (sin contar el `'\0'`), o un **valor negativo** si hay un error de escritura.
> En realidad cuenta **bytes**: con UTF-8, una letra con tilde ocupa 2. `printf("Carácter\n")` devuelve **10**, no 9.

## Especificadores de formato

| Especificador | Tipo esperado | Imprime | Ejemplo |
|---|---|---|---|
| `%c` | `int` (el `char` se promociona) | Un carácter | `A` |
| `%s` | `char *` | Una cadena | `Hola 42` |
| `%p` | `void *` | Dirección de memoria en hexadecimal (ver [[Punteros en C]]) | `0x7ffd5fbff7c0` |
| `%d` | `int` | Entero con signo, base 10 | `-42` |
| `%i` | `int` | Igual que `%d` en `printf` (solo cambia en `scanf`) | `42` |
| `%u` | `unsigned int` | Entero sin signo, base 10 (ver [[Tipos enteros y conversiones]]) | `4200000000` |
| `%x` | `unsigned int` | Hexadecimal en minúsculas | `2a` |
| `%X` | `unsigned int` | Hexadecimal en mayúsculas | `2A` |
| `%%` | — | El símbolo `%` | `%` |

> [!tip] ft_printf
> Las conversiones obligatorias de ft_printf son estas nueve: `cspdiuxX%`.

> [!warning] Comprueba el subject
> Resumen basado en versiones públicas del subject; los detalles cambian entre versiones. Lo que manda es el PDF de tu intra.

## Ejemplos

### Impresión básica

```c
#include <stdio.h>

int	main(void)
{
	char	letter;
	char	*name;
	int		num;

	letter = 'Z';
	name = "42 Barcelona";
	num = 42;
	printf("Carácter: %c\n", letter);
	printf("String: %s\n", name);
	printf("Número: %d\n", num);
	return (0);
}
```

### Usar el valor de retorno

```c
int	len;

len = printf("Hola Mundo\n");
printf("Se han impreso %d caracteres.\n", len);
```

```text
Hola Mundo
Se han impreso 11 caracteres.
```

El `'\n'` cuenta como carácter: `Hola Mundo` (10) + `'\n'` (1) = 11.

### Direcciones de memoria (`%p`)

```c
int	var;
int	*ptr;

var = 10;
ptr = &var;
printf("Dirección de var: %p\n", (void *)&var);
printf("Dirección en ptr: %p\n", (void *)ptr);
```

`%p` espera un `void *`, así que lo correcto es hacer *cast* (conversión explícita de tipo) a `(void *)`.

## Ancho, precisión y banderas

> [!note] Sintaxis general
> `%[banderas][ancho][.precisión]especificador`

| Formato | Significado | Resultado |
|---|---|---|
| `%5d` | Ancho mínimo 5, alineado a la derecha | `[   42]` |
| `%-5d` | Ancho 5, alineado a la izquierda | `[42   ]` |
| `%05d` | Ancho 5, rellena con ceros | `[00042]` |
| `%.3s` | Imprime como mucho 3 caracteres | `[Hol]` |
| `%+d` | Muestra siempre el signo | `[+42]` |
| `%#x` | Añade el prefijo `0x` | `[0x2a]` |

```c
printf("[%5d]\n", 42);      // [   42]
printf("[%-5d]\n", 42);     // [42   ]
printf("[%05d]\n", 42);     // [00042]
printf("[%.3s]\n", "Hola"); // [Hol]
```

Estas banderas solo se piden en el **bonus** de ft_printf.

## Claves para ft_printf

> [!warning] Casos límite
> 1. **`%s` con `NULL`**: la `printf` de glibc (Linux) imprime `(null)`. Lo habitual es que tu versión haga lo mismo en vez de crashear.
> 2. **`%p` con `NULL`**: Linux imprime `(nil)` y macOS `0x0`. Compara con la `printf` del sistema donde te evalúen.
> 3. **El valor de retorno** tiene que coincidir exactamente con el de `printf`, también en estos casos.
> 4. **Promoción de tipos**: con `va_arg`, `char` y `short` llegan como `int`. Pide siempre `va_arg(args, int)` para `%c`.

> [!danger] `printf("%s\n", NULL)` puede crashear igualmente
> gcc cambia `printf("%s\n", s)` por `puts(s)` (es más rápido), y `puts(NULL)` da segfault. Comprobado en Ubuntu 22.04: con gcc el programa muere (`SIGSEGV`); con clang (`cc` en 42) imprime `(null)`. Si comparas tu ft_printf con la original, no te fíes de un solo caso con `\n` al final.

- ft_printf usa las macros de `<stdarg.h>`: `va_start`, `va_arg` y `va_end` (y `va_copy` si la necesitas). Ver [[stdarg y funciones variádicas]].
- Para escribir se usa `write`, igual que en [[ft_putchar_fd]] y [[ft_putnbr_fd]] de libft.

## Errores frecuentes

| ❌ Error | Qué pasa | ✅ Arreglo |
|---|---|---|
| El tipo no coincide con el especificador (`%d` con un `long`, `%s` con un `char`) | Comportamiento indefinido: basura o crash | Usa el especificador correcto; `-Wall` (`-Wformat`) lo avisa |
| Menos argumentos que especificadores | Lee basura de la memoria | Un argumento por cada `%` (salvo `%%`) |
| `printf(cadena_del_usuario)` | Si contiene `%`, se interpreta como formato | `printf("%s", cadena_del_usuario)` |
| Mezclar `printf` y `write` en el mismo programa | El texto sale desordenado: `printf` guarda en un *buffer* y escribe más tarde | Usa solo uno, o vacía el buffer con `fflush(stdout)` |
| Olvidar `\n` y no ver la salida antes de un crash | El buffer no llega a escribirse | Termina las líneas con `\n` al depurar |

## Relacionadas

[[ft_printf - Índice]] · [[stdarg y funciones variádicas]] · [[Tipos enteros y conversiones]] · [[Punteros en C]] · [[Control de errores en C]] · [[ft_putchar_fd]] · [[ft_putstr_fd]] · [[ft_putnbr_fd]] · [[La Norma]]

## Referencias

- [printf(3) — Linux man-pages](https://man7.org/linux/man-pages/man3/printf.3.html) — especificadores, banderas, ancho, precisión y valor de retorno.
- [printf, fprintf… — cppreference](https://en.cppreference.com/w/c/io/fprintf) — tabla de conversiones y tipo esperado por cada una.
- [Borrador del estándar C11 (N1570)](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf) — §7.21.6.1 (la función `fprintf`).
- [GCC — Warning Options](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html) — `-Wformat`, incluido en `-Wall`.
- Salidas de `(null)`, `(nil)`, el valor de retorno con tildes y el crash de gcc con `%s\n` comprobadas en Ubuntu 22.04 (glibc 2.35, gcc 10.5, clang 12).
