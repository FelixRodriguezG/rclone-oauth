---
tags: [proyecto, push-swap, rank-1, algoritmos, o-n-log-n, estructuras-de-datos]
aliases: ["Heap sort en push_swap", "Montículo", "Heap"]
---
# push_swap - Heap sort

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · [[push_swap - Catálogo de algoritmos|Algoritmos]] · O(n log n)

> **Heap sort** usa un **montículo** (*heap*): un árbol guardado en un array, en el que cada padre es menor (o mayor) que sus hijos. Sacar el mínimo y reorganizar cuesta O(log n) **comparaciones**. Es O(n log n) sobre un array. En push_swap, sin embargo, ese O(n log n) **no se traslada a operaciones**. Esta nota explica por qué, y qué hacer si queréis usarlo de todas formas.

## Ficha

| | |
|---|---|
| Familia | Nombrado por el subject entre los O(n log n): «adaptación del orden por montículos» |
| Operaciones | Si el montículo vive en memoria y las pilas solo extraen: las de la [[push_swap - Selección\|selección]], O(n²) |
| Memoria extra | O(n) (el montículo) |
| 100 / 500 aleatorios | 1 461 / 32 338: idénticas a la selección |
| Uso recomendado | **No** como estrategia `--complex`. Sí como estructura auxiliar en CPU |

## La idea original

```text
min-heap guardado en array:  [0, 2, 1, 5, 3, 4]

            0               padre de i:   (i − 1) / 2
          /   \             hijos de i:   2i + 1, 2i + 2
         2     1
        / \   /
       5   3 4

extraer mínimo: sacar la raíz, poner el último en su lugar y «hundirlo»
intercambiándolo con el hijo menor → log₂ n intercambios
```

Cada intercambio une dos posiciones **lejanas** del array: `i` con `2i + 1`.

## Por qué no encaja con dos pilas

En push_swap solo puedes tocar los dos de arriba de cada pila (`sa`, `pa`/`pb`) o girar una pila entera. Intercambiar la posición `i` con la `2i + 1` exige rotar hasta allí: del orden de `i` rotaciones, que pueden ser cientos. Un solo «hundir» de log₂ n intercambios pasaría a costar O(n) operaciones, y el algoritmo entero, O(n²) o más.

```text
array:  intercambiar t[3] y t[7]  →  1 operación
pila:   llevar el 3.º y el 7.º a los dos de arriba, sa, y deshacer  →  ~2·(3 + 4) rotaciones
```

La única forma sencilla de «usar un heap» es tenerlo **en memoria** para saber cuál es el siguiente mínimo, y mover las pilas como en la selección:

```text
heap = montículo con los valores de a                ← en memoria, sin coste de operaciones
mientras a no esté vacía:
    m = extraer_minimo(heap)                          ← O(log n) comparaciones en CPU
    subir m al top de a (ra o rra) y pb               ← O(n) operaciones de push_swap
pa × n
```

El heap acelera **la búsqueda del mínimo en CPU** (de O(n) a O(log n)), pero las operaciones impresas son exactamente las de la selección: O(n²). Medido: 1 461 / 32 338, idéntico.

> [!danger] No lo presentéis como O(n log n)
> Si en el README pone «heap sort, O(n log n)» y el evaluador cuenta las operaciones, el argumento se cae: el subject dice que la complejidad se mide en **operaciones de push_swap**. No conocemos ninguna adaptación razonable del heap sort que dé O(n log n) operaciones con estas pilas.

## Para qué sí sirve un heap en push_swap

| Uso | Ventaja |
|---|---|
| Elegir el elemento más barato en el [[push_swap - Algoritmo de coste (turco)\|algoritmo de coste]] | Una cola de prioridad por coste |
| Simular muchas entradas rápido en un tester propio | CPU más rápida con n grande |

Ninguno cambia el número de operaciones; solo aceleran el cálculo.

## Preguntas de evaluación

> [!question] El subject nombra el heap sort como O(n log n). ¿Por qué no lo usasteis?
> «Porque su O(n log n) cuenta comparaciones sobre un array con acceso directo. Con pilas, cada intercambio entre posiciones lejanas cuesta rotaciones, y el número de operaciones deja de ser O(n log n). Elegimos quicksort con mediana, que sí lo cumple en operaciones.»

## Relacionadas

[[push_swap - Catálogo de algoritmos]] · [[push_swap - Selección]] · [[push_swap - Árbol binario indexado]] · [[push_swap - Quicksort]] · [[Notación Big-O#Lo que cambia en push_swap: contar operaciones, no tiempo]]

## Referencias

- [Wikipedia: Heapsort](https://en.wikipedia.org/wiki/Heapsort) y [Binary heap](https://en.wikipedia.org/wiki/Binary_heap) — estructura y coste de extraer el mínimo.
- Subject de push_swap v1.0, VI.3.1 (la complejidad se mide en operaciones) y VI.3.3 («Adaptación del orden por montículos»).
- Cifras: simulador propio con `heapq` de Python como montículo en memoria (ver [[push_swap - Catálogo de algoritmos]]).
