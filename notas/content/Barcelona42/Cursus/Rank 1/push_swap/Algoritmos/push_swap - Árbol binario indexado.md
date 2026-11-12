---
tags: [proyecto, push-swap, rank-1, algoritmos, o-n-log-n, estructuras-de-datos, bits]
aliases: ["Árbol binario indexado", "Fenwick tree", "BIT", "Binary indexed tree"]
---
# push_swap - Árbol binario indexado

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · [[push_swap - Catálogo de algoritmos|Algoritmos]] · O(n log n)

> Un **árbol binario indexado** (*Binary Indexed Tree*, BIT o *Fenwick tree*) es un array que permite dos cosas en O(log n): **sumar 1** en una posición y preguntar **cuántos hay hasta** una posición. No ordena nada por sí mismo. En push_swap sirve para **calcular rápido** cosas sobre la pila, sobre todo el [[push_swap - Índice de desorden|índice de desorden]] en O(n log n) en vez de O(n²).

## Ficha

| | |
|---|---|
| Familia | Nombrado por el subject entre los O(n log n): «algoritmos de árbol binario indexado» |
| Qué es | Una estructura de datos de **conteo**, no una estrategia de ordenación |
| Coste | O(log n) por consulta o actualización, en **CPU** |
| Memoria extra | O(n) |
| Operaciones de push_swap | Ninguna por sí mismo |
| Uso recomendado | Calcular el desorden y, en general, «cuántos menores que x hay antes de la posición i» |

## La idea

Cada casilla `tree[i]` guarda la suma de un **tramo** que acaba en `i`. El tamaño del tramo es el bit más bajo a 1 de `i`, que se obtiene con `i & -i` (ver [[Operadores de bits]]):

```text
i (base 1)   binario   i & -i   tree[i] cubre
    1          0001       1      [1]
    2          0010       2      [1..2]
    3          0011       1      [3]
    4          0100       4      [1..4]
    6          0110       2      [5..6]
    8          1000       8      [1..8]

contar hasta 7:  tree[7] + tree[6] + tree[4]      (7 → 6 → 4 → 0: quitar el bit bajo)
sumar en 5:      tree[5], tree[6], tree[8]         (5 → 6 → 8: sumar el bit bajo)
```

Cada operación recorre como mucho log₂ n casillas.

## Aplicación: el índice de desorden en O(n log n)

Recorre la pila de izquierda a derecha. Antes de añadir `x`, cuenta cuántos de los ya vistos son **≤ x**. Los que no lo son (los mayores) forman inversiones con `x`:

```c
#include <stdio.h>

static void	bit_add(int *tree, int n, int i)
{
	i++;
	while (i <= n)
	{
		tree[i]++;
		i += i & -i;
	}
}

static int	bit_count_le(int *tree, int i)
{
	int	sum;

	sum = 0;
	i++;
	while (i > 0)
	{
		sum += tree[i];
		i -= i & -i;
	}
	return (sum);
}

static long	inversions_bit(int *idx, int n, int *tree)
{
	long	mistakes;
	int		i;

	i = 0;
	while (i <= n)
		tree[i++] = 0;
	mistakes = 0;
	i = 0;
	while (i < n)
	{
		mistakes += i - bit_count_le(tree, idx[i]);
		bit_add(tree, n, idx[i]);
		i++;
	}
	return (mistakes);
}

int	main(void)
{
	int	idx[6];
	int	tree[7];

	idx[0] = 1;
	idx[1] = 0;
	idx[2] = 2;
	idx[3] = 4;
	idx[4] = 3;
	idx[5] = 5;
	printf("inversiones: %ld de %d pares\n", inversions_bit(idx, 6, tree), 15);
	return (0);
}
```

```text
$ cc -Wall -Wextra -Werror bit.c -o bit && ./bit
inversiones: 2 de 15 pares
```

Los índices `1 0 2 4 3 5` son la normalización de `2 1 3 6 5 8`, el ejemplo del subject: 2 inversiones de 15 pares → 13.33 %, igual que con los dos bucles anidados.

| Método | Coste en CPU | 500 números |
|---|---|---|
| Dos bucles (pseudocódigo del subject) | O(n²) | 124 750 comparaciones |
| BIT | O(n log n) | ~500 · 2 · 9 ≈ 9 000 pasos |

Con 500 números, los dos son instantáneos. La ventaja es **teórica**, pero es un buen detalle para el README: «calculamos el desorden en O(n log n) con un árbol binario indexado».

## ¿Y como estrategia de ordenación?

El BIT responde a preguntas de **conteo**. No mueve elementos, así que no puede «ser» la estrategia `--complex`. Lo honesto es usarlo **dentro** de otra estrategia:

| Dentro de… | Pregunta que responde rápido |
|---|---|
| Índice de desorden | ¿Cuántos mayores que `x` hay antes? |
| [[push_swap - Algoritmo de coste (turco)\|Coste]] / [[push_swap - Inserción\|inserción]] | ¿Cuántos menores que `x` hay ya en `b`? (es decir, en qué posición relativa entra) |
| [[push_swap - LIS\|LIS]] | Variante con BIT de máximos para la subsecuencia creciente más larga |

Y siempre con la misma advertencia que el [[push_swap - Heap sort|heap]]: acelera el cálculo en CPU, pero **no cambia el número de operaciones**, que es lo que se mide.

## Errores frecuentes

> [!warning] Índices desde 0
> El BIT trabaja con posiciones desde 1 (`i & -i` con `i = 0` vale 0 y el bucle no avanza nunca). Por eso el código hace `i++` al entrar.

> [!warning] `tree` con tamaño n
> Necesita n + 1 casillas: la 0 no se usa.

## Relacionadas

[[push_swap - Catálogo de algoritmos]] · [[push_swap - Índice de desorden]] · [[push_swap - Heap sort]] · [[Operadores de bits]] · [[Notación Big-O]]

## Referencias

- [Wikipedia: Fenwick tree](https://en.wikipedia.org/wiki/Fenwick_tree) — estructura, `i & -i` y operaciones en O(log n).
- [cp-algorithms: Fenwick Tree](https://cp-algorithms.com/data_structures/fenwick.html) — implementación base 1 y aplicaciones.
- Subject de push_swap v1.0, VI.3.2 (índice de desorden) y VI.3.3 («Algoritmos de árbol binario indexado»).
- Salida del ejemplo: compilado con Ubuntu clang 12.0.1.
