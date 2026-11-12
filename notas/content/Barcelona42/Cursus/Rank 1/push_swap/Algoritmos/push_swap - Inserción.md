---
tags: [proyecto, push-swap, rank-1, algoritmos, o-n2]
aliases: ["Insertion sort en push_swap", "Inserción con pilas"]
---
# push_swap - Inserción

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · [[push_swap - Catálogo de algoritmos|Algoritmos]] · O(n²)

> **Insertion sort**: como ordenar cartas en la mano. Tomas la siguiente carta y la metes **en su sitio** entre las que ya tienes ordenadas. En push_swap, la «mano» es la pila `b`: cada número de `a` se inserta en `b` en la posición correcta, rotando `b` antes del `pb`.

## Ficha

| | |
|---|---|
| Familia | O(n²), «algoritmo simple» del subject |
| Operaciones | O(n²) en el peor caso; ≈ 2n con entradas ordenadas o al revés |
| Memoria extra | O(1) |
| 100 aleatorios | 1 451 (1 590) |
| 500 aleatorios | 32 703 (33 865) |
| Al revés (500) | 1 499 |
| Casi ordenada (500) | **1 162** (mejor que selección: 1 764) |
| Uso recomendado | `--simple`, sobre todo si quieres que aproveche el orden previo |

## La idea original

```text
[5] 1 4 2      → [1 5] 4 2      → [1 4 5] 2      → [1 2 4 5]
     ▲ entra 1        ▲ entra 4         ▲ entra 2
```

Sobre un array, cada elemento se desplaza hacia la izquierda hasta su sitio. Si el array ya está casi ordenado, cada uno se mueve muy poco: O(n) en el mejor caso.

## Adaptación a dos pilas

`b` se mantiene **descendente en circular**: si la recorres desde el máximo, los números bajan. Para insertar `x`, rota `b` hasta que el top sea **el mayor de los menores que `x`** (si no hay ninguno menor, el máximo de `b`) y haz `pb`.

```text
mientras a no esté vacía:
    x = top de a
    j = posición en b de: el mayor número < x   (si no hay, el máximo de b)
    rota b hasta j por el lado más corto (rb o rrb)
    pb
rota b hasta que el máximo esté arriba
pa × n
```

```text
b (circular) = 7 5 2 | 9 8         ← 9 es el máximo; descendente desde él: 9 8 7 5 2
insertar x = 6:  el mayor menor que 6 es 5 (pos 1) → rb → b = 5 2 9 8 7 → pb
                 b = 6 5 2 9 8 7   → desde el 9: 9 8 7 6 5 2 ✅
```

## Traza real

```text
ini   a: 2 4 1 3 0        b:
pb    a: 4 1 3 0          b: 2
pb    a: 1 3 0            b: 4 2         con 1 elemento, cualquier sitio vale
pb    a: 3 0              b: 1 4 2       1: no hay menores → encima del máximo (4)… ya está
rrb   a: 3 0              b: 2 1 4       3: el mayor menor es 2 (pos 2 de 3) → rrb
pb    a: 0                b: 3 2 1 4
rrb   a: 0                b: 4 3 2 1     0: no hay menores → encima del máximo
pb    a:                  b: 0 4 3 2 1
rb    a:                  b: 4 3 2 1 0   máximo arriba
pa    a: 4                b: 3 2 1 0
pa × 4  …                 a: 0 1 2 3 4
total 13
```

## Complejidad

Cada inserción rota `b` como mucho `tamaño_b / 2` veces, y `tamaño_b` crece de 0 a n−1:

```text
rotaciones ≤ 0/2 + 1/2 + … + (n−1)/2 ≈ n²/4      +  n (pb)  +  n/2 (giro final)  +  n (pa)
total = O(n²)
```

Mejor caso: si la entrada está ordenada (o al revés), cada número va justo encima o justo debajo del anterior: 0 rotaciones por inserción, ≈ 2n operaciones.

## Comparada con selección

| | Selección | Inserción |
|---|---|---|
| Qué rota | `a` (para encontrar el mínimo) | `b` (para encontrar el hueco) |
| Aleatorio | ≈ igual | ≈ igual |
| Casi ordenada | 1 764 | **1 162** |
| Idea que reutilizan otros | Extraer el mínimo | «Insertar en su sitio en la otra pila»: es el corazón del [[push_swap - Algoritmo de coste (turco)\|algoritmo de coste]] |

## Errores frecuentes

> [!warning] Mantener `b` lineal en vez de circular
> Si exiges que `b` esté ordenada del top al fondo en todo momento, tienes que deshacer las rotaciones después de cada `pb`, y el coste se duplica. Con el orden circular, solo hace falta un giro al final.

> [!warning] Olvidar el caso «no hay ningún menor»
> El número más pequeño hasta el momento va **encima del máximo** (entre el mínimo y el máximo del círculo).

## Preguntas de evaluación

> [!question] ¿Por qué es mejor que selección con listas casi ordenadas?
> «Porque cada número va cerca de donde fue el anterior, así que la rotación de `b` es corta. Selección siempre tiene que buscar el mínimo por toda la pila `a`.»

## Relacionadas

[[push_swap - Catálogo de algoritmos]] · [[push_swap - Selección]] · [[push_swap - Algoritmo de coste (turco)]] · [[push_swap - LIS]] · [[Pilas y colas]]

## Referencias

- [Wikipedia: Insertion sort](https://en.wikipedia.org/wiki/Insertion_sort).
- Subject de push_swap v1.0, VI.3.3 («Adaptación del orden por inserción»).
- Cifras y traza: simulador propio (ver [[push_swap - Catálogo de algoritmos]]).
