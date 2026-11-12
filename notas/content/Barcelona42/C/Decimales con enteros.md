---
tags: [c, aritmetica, printf, push-swap]
aliases: ["Punto fijo", "Imprimir decimales sin float", "Porcentajes con enteros"]
---
# Decimales con enteros

[[Inicio|← Inicio]] · C · Fundamentos

> Tu `ft_printf` no tiene `%f`, y en push_swap solo puedes usar `write`, `malloc`, `free`, `exit` y tu `ft_printf`. Aun así, el modo `--bench` pide el índice de desorden como porcentaje **con dos decimales** (`13.33%`). El truco es trabajar en **centésimas**: multiplicas por 100 antes de dividir y luego imprimes la parte entera y la decimal por separado. Esto se llama aritmética de **punto fijo**.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Punto fijo** | Guardar un número decimal como entero multiplicado por una potencia de 10 (13,33 → 1333 centésimas) |
| **Centésima** | 1/100. El porcentaje 13,33 % son 1333 centésimas de punto porcentual |
| **Redondeo al más cercano** | Sumar la mitad del divisor antes de dividir |

## La receta

Queremos `mistakes / total` como porcentaje con 2 decimales:

```text
porcentaje = mistakes / total × 100
en centésimas = mistakes / total × 10 000

hundredths = (mistakes * 10000 + total / 2) / total     ← + total/2 redondea
parte entera  = hundredths / 100
dos decimales = hundredths % 100        ← imprimir siempre con 2 cifras
```

Ejemplo con `2 1 3 6 5 8` (2 errores de 15 pares):

```text
2 × 10000 = 20000
20000 + 15/2 = 20000 + 7 = 20007
20007 / 15 = 1333              (1333,8 truncado)
1333 / 100 = 13    1333 % 100 = 33    →  "13.33%"
```

## Ejemplo completo

Este programa calcula el desorden y lo imprime así. Usa `printf` para comprobarlo; en push_swap lo harás con tu `ft_printf` o con [[ft_putnbr_fd]].

```c
#include <stdio.h>

static void	count_pairs(int *a, int n, long *mistakes, long *total)
{
	int	i;
	int	j;

	*mistakes = 0;
	*total = 0;
	i = 0;
	while (i < n)
	{
		j = i + 1;
		while (j < n)
		{
			(*total)++;
			if (a[i] > a[j])
				(*mistakes)++;
			j++;
		}
		i++;
	}
}

static void	print_disorder(int *a, int n)
{
	long	mistakes;
	long	total;
	long	hundredths;

	count_pairs(a, n, &mistakes, &total);
	hundredths = 0;
	if (total > 0)
		hundredths = (mistakes * 10000 + total / 2) / total;
	printf("%ld/%ld errores -> %ld.%02ld%%\n", mistakes, total,
		hundredths / 100, hundredths % 100);
}

int	main(void)
{
	int	ex1[6];
	int	ex2[5];
	int	one[1];

	ex1[0] = 2;
	ex1[1] = 1;
	ex1[2] = 3;
	ex1[3] = 6;
	ex1[4] = 5;
	ex1[5] = 8;
	ex2[0] = 5;
	ex2[1] = 4;
	ex2[2] = 3;
	ex2[3] = 2;
	ex2[4] = 1;
	one[0] = 42;
	print_disorder(ex1, 6);
	print_disorder(ex2, 5);
	print_disorder(one, 1);
	return (0);
}
```

```text
$ cc -Wall -Wextra -Werror disorder.c -o disorder && ./disorder
2/15 errores -> 13.33%
10/10 errores -> 100.00%
0/0 errores -> 0.00%
```

## El cero de relleno

`hundredths % 100` puede valer `5`, y tienes que imprimir `"05"`, no `"5"`: `13.05%`, no `13.5%`. `printf` lo hace con `%02ld`, pero **tu `ft_printf` probablemente no tiene el flag `0`**. Hazlo a mano:

```c
if (hundredths % 100 < 10)
	ft_putchar_fd('0', 2);
ft_putnbr_fd(hundredths % 100, 2);
```

(El `2` es el fd de stderr, que es donde va la salida del bench.)

## ¿Por qué no usar `float` o `double`?

- `printf("%f")` no está permitido en push_swap, y tu `ft_printf` no lo implementa.
- Aunque calcularas con `double`, necesitarías convertirlo a texto, y eso es justo el problema de partida.
- Con enteros, el resultado es **exacto** y reproducible: `49.93%` sale igual en cualquier máquina.

## Errores frecuentes

> [!warning] Dividir antes de multiplicar
> `mistakes / total * 10000` vale 0 si `mistakes < total`. Ver [[División entera y módulo]].

> [!warning] Desbordar con `int`
> Con 500 números hay 124 750 pares. `124750 * 10000` = 1 247 500 000, que todavía cabe en `int`, pero muy justo: con 1 000 números ya no (4 995 000 000). Usa `long` (ver [[Tipos enteros y conversiones]]).

> [!warning] Dividir por cero con un solo número
> Con n = 1 hay 0 pares. Comprueba `total > 0` antes de dividir: el desorden de un solo número es 0.

## Relacionadas

[[División entera y módulo]] · [[Tipos enteros y conversiones]] · [[printf]] · [[ft_putnbr_fd]] · [[push_swap - Índice de desorden]]

## Referencias

- [Wikipedia: Fixed-point arithmetic](https://en.wikipedia.org/wiki/Fixed-point_arithmetic) — representar decimales como enteros escalados.
- Subject de push_swap v1.0, VI.5 — `--bench` muestra «el índice de desorden (% con dos decimales)».
- Salida del ejemplo: compilado con Ubuntu clang 12.0.1.
