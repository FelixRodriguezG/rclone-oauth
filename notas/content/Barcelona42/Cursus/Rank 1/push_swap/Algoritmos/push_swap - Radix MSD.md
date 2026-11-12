---
tags: [proyecto, push-swap, rank-1, algoritmos, o-n-log-n, bits, recursion]
aliases: ["Radix MSD", "Radix por el bit más significativo"]
---
# push_swap - Radix MSD

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · [[push_swap - Catálogo de algoritmos|Algoritmos]] · O(n log n)

> **Radix MSD** (*Most Significant Digit*) empieza por el bit de **más** peso: separa los que lo tienen a 0 de los que lo tienen a 1 y luego ordena cada grupo **por separado**, mirando el siguiente bit. Al contrario que el [[push_swap - Radix LSD|LSD]], no repasa toda la pila en cada bit, sino solo el grupo que toca. En push_swap acaba siendo un [[push_swap - Quicksort|quicksort]] cuyo pivote se decide por bits.

## Ficha

| | |
|---|---|
| Familia | O(n log n): «adaptación del orden radix (LSD o MSD)» del subject |
| Operaciones | O(n log n) |
| Memoria extra | O(log n) de recursión |
| 100 aleatorios | 761 (781) |
| 500 aleatorios | **5 269** (5 311): excelente |
| Al revés (500) | 5 766 |
| Casi ordenada (500) | 2 271 |
| Uso recomendado | `--complex`, si queréis un radix con muchas menos operaciones que el LSD |

## La idea original

```text
índices de 3 bits: 6 1 7 2 5 0
bit 2:   0xx → [1 2 0]          1xx → [6 7 5]
bit 1:   00x → [1 0]  01x → [2]     10x → [5]  11x → [6 7]
bit 0:   [0] [1]                                [6] [7]
resultado: 0 1 2 5 6 7
```

Cada grupo se ordena **de forma independiente**, de manera recursiva.

## Adaptación a dos pilas

Es la misma estructura que el [[push_swap - Quicksort#Adaptación a dos pilas|quicksort con dos pilas]] (`ordenar_a` / `ordenar_b`). Solo cambia **cómo se elige el pivote**:

```text
segmento con índices entre lo y hi
bit = el bit más alto en el que lo y hi difieren
pivote = el primer valor con ese bit a 1 y el mismo prefijo que hi
      = (hi >> bit) << bit
menores que el pivote (bit = 0) → a un lado
mayores o iguales (bit = 1)     → al otro
```

Ejemplo: segmento con índices de 0 a 3 → `0 ^ 3 = 11₂` → bit 1 → pivote `(3 >> 1) << 1 = 2`. Separa {0, 1} de {2, 3}.

> [!note] ¿Por qué no es simplemente quicksort?
> Con índices normalizados de 0 a n−1, el pivote por bits cae **cerca** de la mediana, pero no exactamente en ella. Por ejemplo, con 0…4 el pivote es 4 y separa 4 elementos de 1. La ventaja es que no hay que calcular ninguna mediana: el pivote sale de una operación de bits. El precio es un reparto algo menos equilibrado.

## Traza real

Índices `2 4 1 3 0` (de 0 a 4 → bit 2 → pivote 4):

```text
ini   a: 2 4 1 3 0        b:
pb    a: 4 1 3 0          b: 2           2 < 4
ra    a: 1 3 0 4          b: 2           4 ≥ 4 → se queda
pb    a: 3 0 4            b: 1 2
pb    a: 0 4              b: 3 1 2
pb    a: 4                b: 0 3 1 2     a = {4} ya ordenado
                                         ordenar_b con 0…3 → bit 1 → pivote 2
rb    a: 4                b: 3 1 2 0     0 < 2 → se queda en b
pa    a: 3 4              b: 1 2 0       3 ≥ 2 → a
rb    a: 3 4              b: 2 0 1
pa    a: 2 3 4            b: 0 1         {2, 3} en a: ya ordenados
sb    a: 2 3 4            b: 1 0         {0, 1} en b: caso base
pa    a: 1 2 3 4          b: 0
pa    a: 0 1 2 3 4        b:
total 12
```

## Complejidad

Cada nivel de recursión mira **un bit**, y hay ⌈log₂ n⌉ bits. En cada nivel, cada elemento se mueve una vez (partición) y como mucho otra para deshacer la rotación:

```text
operaciones ≤ 2 · n · ⌈log₂ n⌉ + O(n)  =  O(n log n)       memoria: O(log n)
```

A diferencia del quicksort con pivote arbitrario, aquí la profundidad **nunca** pasa de ⌈log₂ n⌉, porque cada nivel consume un bit. La cota es para el **peor caso**, aunque los repartos no sean exactamente a la mitad.

## LSD frente a MSD

| | LSD | MSD |
|---|---|---|
| Orden de los bits | Del menos al más significativo | Del más al menos |
| Cada pasada mira | Toda la pila | Solo el grupo actual |
| Recursión | No | Sí |
| Operaciones (500) | 6 784 | 5 269 |
| Demostración | Trivial | Fácil (un bit por nivel) |

## Preguntas de evaluación

> [!question] ¿Qué diferencia hay entre vuestro radix MSD y un quicksort?
> «La estructura es la misma; cambia el pivote. En quicksort es la mediana, que hay que calcular. Aquí es una frontera de bits, y garantiza la profundidad log₂ n sin calcular nada.»

## Relacionadas

[[push_swap - Catálogo de algoritmos]] · [[push_swap - Radix LSD]] · [[push_swap - Quicksort]] · [[Operadores de bits]] · [[Recursión]]

## Referencias

- [Wikipedia: Radix sort — Most significant digit](https://en.wikipedia.org/wiki/Radix_sort#Most_significant_digit) — la variante recursiva.
- Subject de push_swap v1.0, VI.3.3 («Adaptación del orden radix (LSD o MSD)»).
- Cifras y traza: simulador propio (ver [[push_swap - Catálogo de algoritmos]]).
