---
tags: [c, fundamentos, operadores, aritmetica, push-swap]
aliases: ["Módulo", "Operador %", "División entera", "Resto"]
---
# División entera y módulo

[[Inicio|← Inicio]] · C · Fundamentos

> En C, `/` entre dos enteros da un **entero**: se tira la parte decimal (`7 / 2` vale 3). El operador `%` da el **resto** de esa división (`7 % 2` vale 1). Parecen detalles, pero de ellos dependen el [[Array circular|array circular]], el tamaño de los chunks y el porcentaje del modo `--bench` de [[push_swap - Índice|push_swap]].

## Vocabulario

| Término | Qué significa |
|---|---|
| **Dividendo / divisor** | En `a / b`, `a` es el dividendo y `b` el divisor |
| **Cociente** | El resultado entero de `a / b` |
| **Resto** (módulo) | Lo que sobra: `a % b`. Siempre `a == (a / b) * b + a % b` |
| **Truncar** | Quitar los decimales, redondeando **hacia cero** |

## Las reglas, comprobadas

```c
#include <stdio.h>

int	main(void)
{
	int	n;

	n = 5;
	printf("7 / 2 = %d   7 %% 2 = %d\n", 7 / 2, 7 % 2);
	printf("-7 / 2 = %d  -7 %% 2 = %d\n", -7 / 2, -7 % 2);
	printf("(4 + 1) %% %d = %d\n", n, (4 + 1) % n);
	printf("(0 - 1 + %d) %% %d = %d\n", n, n, (0 - 1 + n) % n);
	printf("100 / 10 redondeando arriba: %d\n", (100 + 10 - 1) / 10);
	printf("101 / 10 redondeando arriba: %d\n", (101 + 10 - 1) / 10);
	printf("2 / 15 * 100 = %d\n", 2 / 15 * 100);
	printf("2 * 100 / 15 = %d\n", 2 * 100 / 15);
	return (0);
}
```

```text
$ cc -Wall -Wextra -Werror divmod.c -o divmod && ./divmod
7 / 2 = 3   7 % 2 = 1
-7 / 2 = -3  -7 % 2 = -1
(4 + 1) % 5 = 0
(0 - 1 + 5) % 5 = 4
100 / 10 redondeando arriba: 10
101 / 10 redondeando arriba: 11
2 / 15 * 100 = 0
2 * 100 / 15 = 13
```

Lo que enseña cada línea:

| Línea | Lección |
|---|---|
| `7 / 2 = 3` | Se tiran los decimales |
| `-7 / 2 = -3`, `-7 % 2 = -1` | Se trunca **hacia cero**, y el resto lleva el **signo del dividendo** |
| `(4 + 1) % 5 = 0` | Avanzar dando la vuelta: del último índice al primero |
| `(0 - 1 + 5) % 5 = 4` | Retroceder dando la vuelta: suma el divisor **antes** del `%` |
| `(a + b - 1) / b` | División **redondeando hacia arriba** sin decimales |
| `2 / 15 * 100 = 0` | **El orden importa**: `2 / 15` ya es 0 |
| `2 * 100 / 15 = 13` | Multiplica primero, divide al final |

## Usos en push_swap

| Necesidad | Fórmula |
|---|---|
| Índice siguiente en un anillo | `(i + 1) % size` |
| Índice anterior en un anillo | `(i - 1 + size) % size` |
| Tamaño de cada chunk (√n chunks) | `(n + chunks - 1) / chunks` |
| ¿A qué chunk pertenece el índice `x`? | `x / chunk_size` |
| Porcentaje con 2 decimales | `mistakes * 10000 / total` (ver [[Decimales con enteros]]) |
| ¿Mitad de arriba o de abajo de la pila? | `pos <= size / 2` |
| Bit `k` de un número | `(x >> k) & 1`, que es lo mismo que `(x / 2^k) % 2` (ver [[Operadores de bits]]) |

## Errores frecuentes

> [!danger] Dividir o hacer módulo por cero
> `x / 0` y `x % 0` son [[Comportamiento indefinido]] (normalmente el programa muere con `SIGFPE`). En push_swap, el índice de desorden con **un solo número** tiene 0 pares: compruébalo antes de dividir.

> [!warning] Dividir antes de multiplicar
> `mistakes / total * 100` da 0 siempre que `mistakes < total`. Multiplica primero, y vigila que la multiplicación no desborde (usa `long`, ver [[Tipos enteros y conversiones]]).

> [!warning] Módulo de un negativo
> `-1 % 5` es `-1`, no `4`. Como índice de array, lee fuera de rango.

## Relacionadas

[[Decimales con enteros]] · [[Array circular]] · [[Operadores de bits]] · [[Tipos enteros y conversiones]] · [[Comportamiento indefinido]] · [[ft_div_mod]]

## Referencias

- [cppreference: Arithmetic operators](https://en.cppreference.com/w/c/language/operator_arithmetic) — truncamiento hacia cero y signo del resto (desde C99).
- Salida del ejemplo: compilado con Ubuntu clang 12.0.1.
