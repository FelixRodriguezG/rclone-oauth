---
tags: [c, stdarg, variadic-functions, ft_printf]
date: 2026-10-01
---
# stdarg.h y funciones variádicas

[[Inicio|← Inicio]] · C

> Una **función variádica** acepta un **número variable de argumentos**. El ejemplo más conocido es [[printf]], que recibe desde un argumento hasta decenas.

## Prototipo

Se indica con tres puntos (`...`) al final de la lista de parámetros:

```c
int	ft_printf(const char *format, ...);
```

> [!danger] Reglas de sintaxis
> 1. Debe haber **al menos un parámetro fijo** antes de `...`.
> 2. Los `...` van **siempre al final** de la lista de parámetros.
> 3. La función **no sabe cuántos argumentos** ha recibido ni de qué tipo: tiene que deducirlo de los parámetros fijos (un contador, o la cadena de formato en `printf`).

## Las herramientas de `<stdarg.h>`

Para leer los argumentos que hay detrás de `...` se usa un tipo y tres macros obligatorias (más una opcional):

| Macro / tipo | Qué hace |
|---|---|
| `va_list` | Tipo de la variable que guarda por dónde vas leyendo los argumentos |
| `va_start(args, ultimo_fijo)` | Inicializa `args` justo después del último parámetro fijo |
| `va_arg(args, TIPO)` | Devuelve el **siguiente** argumento interpretado como `TIPO` y avanza |
| `va_end(args)` | Termina la lectura. Obligatoria por cada `va_start` |
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

## Aplicación en ft_printf

En ft_printf recorres la cadena de formato carácter a carácter. Cada vez que encuentras un `%`, miras la letra siguiente y llamas a `va_arg` con el tipo que le corresponde:

| Conversión | Llamada |
|---|---|
| `%c` | `va_arg(args, int)` |
| `%s` | `va_arg(args, char *)` |
| `%p` | `va_arg(args, void *)` |
| `%d` / `%i` | `va_arg(args, int)` |
| `%u` / `%x` / `%X` | `va_arg(args, unsigned int)` |
| `%%` | Ninguna: no consume argumento |

> [!tip] Pasar la `va_list` a funciones auxiliares
> Para cumplir el límite de 25 líneas repartirás el trabajo en varias funciones. Pásales la lista **por puntero** (`va_list *args`): si la pasas por valor, después de usarla dentro de la función auxiliar ya no puedes volver a usarla en la que la llamó.

## Relacionadas

[[printf]] · [[La Norma]]
