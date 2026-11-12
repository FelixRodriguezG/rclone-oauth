---
tags: [proyecto, push-swap, rank-1, algoritmos, desorden]
aliases: ["Índice de desorden", "Disorder", "Inversiones"]
---
# push_swap - Índice de desorden

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · Desorden

> El **índice de desorden** es un número entre 0 y 1 que dice **cuánto de desordenada** está la pila `a` al empezar. 0 significa ordenada y 1, completamente al revés. Es obligatorio calcularlo, porque la estrategia adaptativa lo usa para decidir qué algoritmo aplicar, y `--bench` lo muestra.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Par** | Dos posiciones `i < j` de la pila. Con n números hay `n(n−1)/2` |
| **Inversión** («error» en el subject) | Un par en el que el de delante es **mayor**: `a[i] > a[j]` |
| **Desorden** | `inversiones / pares` |

## La definición del subject

```text
function compute_disorder(stack a):
    mistakes = 0
    total_pairs = 0
    for i from 0 to size(a)-1:
        for j from i+1 to size(a)-1:
            total_pairs += 1
            if a[i] > a[j]:
                mistakes += 1
    return mistakes / total_pairs
```

Se calcula **antes de hacer ningún movimiento**.

## A mano: `2 1 3 6 5 8`

```text
         j→  1   3   6   5   8
 i=0 (2)    ✗   ·   ·   ·   ·      2 > 1  → inversión
 i=1 (1)        ·   ·   ·   ·
 i=2 (3)            ·   ·   ·
 i=3 (6)                ✗   ·      6 > 5  → inversión
 i=4 (5)                    ·

pares = 6·5/2 = 15     inversiones = 2     desorden = 2/15 = 0,1333 → 13.33%
```

| Entrada | Inversiones / pares | Desorden | Tramo adaptativo |
|---|---|---|---|
| `1 2 3 4 5` | 0 / 10 | 0,00 | bajo |
| `2 1 3 6 5 8` | 2 / 15 | 0,13 | bajo |
| `4 67 3 87 23` | 4 / 10 | 0,40 | medio (el subject muestra 40.00%) |
| `3 4 5 1 2` | 6 / 10 | 0,60 | **alto** |
| `5 4 3 2 1` | 10 / 10 | 1,00 | alto |

> [!warning] El desorden no ve las rotaciones
> `3 4 5 1 2` es una lista ordenada **rotada**: se arregla con `rra rra` (2 operaciones). Pero tiene un desorden de 0,60, el tramo «alto». Una buena adaptativa comprueba **antes** de mirar el desorden si la pila ya está ordenada o solo rotada. Ver [[push_swap - Estrategias#Atajos antes de elegir]].

## En C

Dos bucles anidados que **cuentan** (no hace falta guardar los pares). Se usa `long` porque el número de pares crece como n²/2:

```c
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
```

Es O(n²) en **tiempo de CPU** (~125 000 comparaciones con 500 números: instantáneo) y O(1) en memoria extra. No genera operaciones de push_swap, así que no cuenta en el total.

> [!danger] n = 1 (o n = 0)
> Hay 0 pares, y `0 / 0` es una división por cero. Si `total == 0`, el desorden es 0.

### Sin decimales

El subject habla de un número entre 0 y 1, pero no necesitas `float`. Para comparar con los umbrales, multiplica en vez de dividir:

```text
desorden < 0,2    ⟺   mistakes * 10 < total * 2     ⟺   5 * mistakes < total
desorden < 0,5    ⟺   2 * mistakes < total
```

Para mostrarlo en `--bench` con dos decimales, mira [[Decimales con enteros]] (el ejemplo completo calcula exactamente este índice).

## Lo que vale con números aleatorios

En una permutación al azar, cada par está invertido con probabilidad ½, así que el desorden **esperado es 0,5**. Medido con un simulador:

| n | Mínimo | Mediana | Máximo | Veces ≥ 0,5 |
|---|---|---|---|---|
| 100 (40 pruebas) | 0,421 | 0,506 | 0,591 | 23 de 40 |
| 500 (12 pruebas) | 0,483 | 0,503 | 0,536 | 8 de 12 |

> [!important] Consecuencia para el rendimiento
> Los tests de rendimiento usan números aleatorios y, por defecto, la estrategia adaptativa. Como el desorden cae a ambos lados de 0,5, **tanto el método del tramo medio como el del alto** se usan en evaluación y los dos deben pasar los límites de 100 y 500 números. El propio subject muestra un caso de 500 números con desorden de **49.93%** resuelto por el tramo O(n√n) en 7 997 operaciones.

## Relación con la complejidad

El número de inversiones es exactamente cuántos `swap` de vecinos necesitaría el algoritmo de la burbuja para ordenar (ver [[Algoritmos de ordenación y complejidad]]). Por eso es una medida natural del «trabajo pendiente». Pero en push_swap se puede rotar, y unas pocas rotaciones deshacen muchas inversiones de golpe; de ahí el aviso sobre `3 4 5 1 2`.

## Relacionadas

[[push_swap - Índice]] · [[push_swap - Estrategias]] · [[Decimales con enteros]] · [[División entera y módulo]] · [[Notación Big-O]] · [[Tipos enteros y conversiones]]

## Referencias

- Subject de push_swap v1.0, VI.3.2 («Índice de desorden (obligatorio)») — definición y pseudocódigo; VI.5.1 — ejemplos `40.00%` y `49.93%`.
- [Wikipedia: Inversion (discrete mathematics)](https://en.wikipedia.org/wiki/Inversion_(discrete_mathematics)) — inversiones y su relación con el número de intercambios de vecinos.
- Tabla de desorden aleatorio: simulador propio (Python, `random.seed(42)`).
