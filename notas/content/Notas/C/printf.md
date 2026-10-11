---
tags: [c, printf, ft_printf, stdio]
date: 2026-10-01
---
# printf — formato e impresión en C

[[Inicio|← Inicio]] · C

> `printf` (*print formatted*) imprime texto con formato en la salida estándar (`stdout`). Es la función que tendrás que reimplementar en el proyecto **ft_printf**.

## Prototipo

```c
#include <stdio.h>

int	printf(const char *format, ...);
```

- `format`: la cadena con el texto y los **especificadores** (`%d`, `%s`…).
- `...`: un número variable de argumentos, uno por especificador. Ver [[stdarg y funciones variádicas]].

> [!info] Valor de retorno
> Devuelve el **número de caracteres impresos** (sin contar el `'\0'`), o un **valor negativo** si hay un error de escritura.

## Especificadores de formato

| Especificador | Tipo esperado | Imprime | Ejemplo |
|---|---|---|---|
| `%c` | `int` (el `char` se promociona) | Un carácter | `A` |
| `%s` | `char *` | Una cadena | `Hola 42` |
| `%p` | `void *` | Dirección de memoria en hexadecimal | `0x7ffd5fbff7c0` |
| `%d` | `int` | Entero con signo, base 10 | `-42` |
| `%i` | `int` | Igual que `%d` en `printf` (solo cambia en `scanf`) | `42` |
| `%u` | `unsigned int` | Entero sin signo, base 10 | `4200000000` |
| `%x` | `unsigned int` | Hexadecimal en minúsculas | `2a` |
| `%X` | `unsigned int` | Hexadecimal en mayúsculas | `2A` |
| `%%` | — | El símbolo `%` | `%` |

> [!tip] ft_printf
> Las conversiones obligatorias de ft_printf son exactamente estas nueve: `cspdiuxX%`.

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

`%p` espera un `void *`, así que lo correcto es hacer *cast* a `(void *)`.

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
> 1. **`%s` con `NULL`**: la `printf` de glibc (Linux) imprime `(null)`. Tu versión debe hacer lo mismo en vez de crashear.
> 2. **`%p` con `NULL`**: Linux imprime `(nil)` y macOS `0x0`. Compara con la `printf` del sistema donde te evalúen.
> 3. **El valor de retorno** tiene que coincidir exactamente con el de `printf`, también en estos casos.
> 4. **Promoción de tipos**: con `va_arg`, `char` y `short` llegan como `int`. Pide siempre `va_arg(args, int)` para `%c`.

- ft_printf usa las macros de `<stdarg.h>`: `va_start`, `va_arg` y `va_end` (y `va_copy` si la necesitas). Ver [[stdarg y funciones variádicas]].
- Para escribir se usa `write`, igual que en `ft_putchar_fd` y `ft_putnbr_fd` de libft.

## Relacionadas

[[stdarg y funciones variádicas]] · [[La Norma]]
