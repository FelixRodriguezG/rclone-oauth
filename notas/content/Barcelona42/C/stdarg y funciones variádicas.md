---
tags: [c, stdarg, funciones-variadicas, ft_printf]
aliases: ["stdarg.h y funciones variádicas", "va_arg", "Funciones variádicas"]
---
# stdarg y funciones variádicas

[[Inicio|← Inicio]] · C

> Una **función variádica** acepta un **número variable de argumentos**. El ejemplo más conocido es [[printf]], que recibe desde un argumento hasta decenas. La cabecera `<stdarg.h>` trae las herramientas para leer esos argumentos extra uno a uno.

| Término | Qué significa |
|---|---|
| **Variádica** | Que acepta un número variable de argumentos |
| **Parámetro fijo** | Los parámetros con nombre que van antes de `...` |
| **Macro** | Código que el preprocesador sustituye antes de compilar; parece una función pero no lo es (ver [[Preprocesador y headers]]) |
| **Promoción de tipos** | Conversión automática de un tipo pequeño a otro más grande (`char` → `int`), ver [[Tipos enteros y conversiones]] |

## Prototipo

Se indica con tres puntos (`...`) al final de la lista de parámetros:

```c
int	ft_printf(const char *format, ...);
```

> [!danger] Reglas de sintaxis
> 1. Debe haber **al menos un parámetro fijo** antes de `...` (obligatorio hasta C17; C23 lo relaja, pero en 42 se compila con estándares anteriores).
> 2. Los `...` van **siempre al final** de la lista de parámetros.
> 3. La función **no sabe cuántos argumentos** ha recibido ni de qué tipo: tiene que deducirlo de los parámetros fijos (un contador, o la cadena de formato en `printf`).

## Las herramientas de `<stdarg.h>`

Para leer los argumentos que hay detrás de `...` se usa un tipo y tres macros obligatorias (más una opcional):

| Macro / tipo | Qué hace |
|---|---|
| `va_list` | Tipo de la variable que guarda por dónde vas leyendo los argumentos |
| `va_start(args, ultimo_fijo)` | Inicializa `args` justo después del último parámetro fijo |
| `va_arg(args, TIPO)` | Devuelve el **siguiente** argumento interpretado como `TIPO` y avanza |
| `va_end(args)` | Termina la lectura. Obligatoria por cada `va_start` (y por cada `va_copy`) |
| `va_copy(dst, src)` | *(Opcional)* Copia el estado de una `va_list` en otra para recorrerla dos veces |

## Ejemplo: sumar N enteros

```c
#include <stdarg.h>
#include <stdio.h>

int	sum_all(int count, ...)
{
	va_list	args;
	int		total;
	int		i;

	total = 0;
	i = 0;
	va_start(args, count);
	while (i < count)
	{
		total += va_arg(args, int);
		i++;
	}
	va_end(args);
	return (total);
}

int	main(void)
{
	printf("Suma 1: %d\n", sum_all(3, 10, 20, 30));
	printf("Suma 2: %d\n", sum_all(5, 1, 2, 3, 4, 5));
	return (0);
}
```

```text
Suma 1: 60
Suma 2: 15
```

Paso a paso:
1. `va_list args;` declara el «cursor» de argumentos.
2. `va_start(args, count);` lo coloca justo después de `count`, el último parámetro fijo.
3. `va_arg(args, int)` lee cada argumento como `int` y avanza al siguiente. `count` dice cuántas veces hay que hacerlo.
4. `va_end(args);` cierra la lectura antes de volver.

> [!note] Norma
> El ejemplo cumple la [[La Norma|Norma]]: sin `for`, declaraciones arriba y sin comentarios dentro de la función (por eso las explicaciones van fuera).

## Promoción de tipos por defecto

Los argumentos que pasan por `...` sufren una **promoción automática** antes de llegar a la función:

> [!warning] Regla de promoción
> - `char` y `short` → se convierten en **`int`**.
> - `float` → se convierte en **`double`**.

Por eso **nunca** debes pedir `char`, `short` o `float` a `va_arg`: es comportamiento indefinido.

```c
c = va_arg(args, char);          // ❌ incorrecto
c = (char)va_arg(args, int);     // ✅ correcto
```

Qué pasa de verdad si lo haces (Ubuntu 22.04, `-Wall -Wextra` sin `-Werror`):

```text
# gcc: avisa y el programa se aborta al llegar a esa línea (Illegal instruction)
warning: 'char' is promoted to 'int' when passed through '...'
note: if this code is reached, the program will abort

# clang: avisa y «funciona», pero sigue siendo comportamiento indefinido
warning: second argument to 'va_arg' is of promotable type 'char'; this va_arg
         has undefined behavior because arguments will be promoted to 'int' [-Wvarargs]
```

Con `-Werror` ninguno de los dos compila: mejor así.

## Aplicación en ft_printf

En ft_printf recorres la cadena de formato carácter a carácter. Cada vez que encuentras un `%`, miras la letra siguiente y llamas a `va_arg` con el tipo que le corresponde:

| Conversión | Llamada |
|---|---|
| `%c` | `va_arg(args, int)` |
| `%s` | `va_arg(args, char *)` |
| `%p` | `va_arg(args, void *)` (muchos lo leen como `unsigned long` para imprimirlo en hexadecimal; en Linux de 64 bits ocupan lo mismo) |
| `%d` / `%i` | `va_arg(args, int)` |
| `%u` / `%x` / `%X` | `va_arg(args, unsigned int)` |
| `%%` | Ninguna: no consume argumento |

> [!tip] Pasar la `va_list` a funciones auxiliares
> Para cumplir el límite de 25 líneas repartirás el trabajo en varias funciones. Pásales la lista **por puntero** (`va_list *args`): si la pasas por valor, después de usarla dentro de la función auxiliar ya no puedes volver a usarla en la que la llamó.

## Errores frecuentes

| ❌ Error | Qué pasa | ✅ Arreglo |
|---|---|---|
| Llamar a `va_arg` más veces que argumentos hay | Lee basura: comportamiento indefinido, nada lo detecta | Que el parámetro fijo (contador o formato) diga exactamente cuántos hay |
| Pedir a `va_arg` un tipo distinto del que se pasó | Valores absurdos o crash | El tipo de `va_arg` debe coincidir con el argumento (ya promocionado) |
| `va_arg(args, char)` | UB; con gcc, aborta | `(char)va_arg(args, int)` |
| Olvidar `va_end` | Funciona en x86-64, pero es incorrecto y no portable | Un `va_end` por cada `va_start` antes de cada `return` |
| Pasar `va_list` por valor a una auxiliar y seguir usándola | La posición queda indeterminada en la función que llama | Pasa `va_list *` |

## Relacionadas

[[printf]] · [[ft_printf - Índice]] · [[Tipos enteros y conversiones]] · [[Preprocesador y headers]] · [[Depurar con gdb]] · [[La Norma]]

## Referencias

- [stdarg(3) — Linux man-pages](https://man7.org/linux/man-pages/man3/stdarg.3.html) — `va_start`, `va_arg`, `va_end`, `va_copy`.
- [Variadic functions — cppreference](https://en.cppreference.com/w/c/variadic) y [Variadic arguments — cppreference](https://en.cppreference.com/w/c/language/variadic) — sintaxis de `...` y promociones por defecto.
- [Implicit conversions — cppreference](https://en.cppreference.com/w/c/language/conversion) — *default argument promotions* (`char`/`short` → `int`, `float` → `double`).
- [Borrador del estándar C11 (N1570)](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf) — §7.16 (`<stdarg.h>`; §7.16p3: una `va_list` pasada a otra función que usa `va_arg` queda indeterminada en la que llama), §6.5.2.2p6–7 (promociones).
- [Beej's Guide to C — Variadic Functions](https://beej.us/guide/bgc/html/split/variadic-functions.html) — explicación introductoria.
- Avisos de gcc 10.5 y clang 12 con `va_arg(args, char)` comprobados en Ubuntu 22.04.
