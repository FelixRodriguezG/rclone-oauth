---
tags: [proyecto, push-swap, rank-1, algoritmos, o-n2]
aliases: ["Selection sort en push_swap", "Selección con pilas"]
---
# push_swap - Selección

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · [[push_swap - Catálogo de algoritmos|Algoritmos]] · O(n²)

> **Selection sort**: busca el mínimo, ponlo en su sitio y repite con lo que queda. En push_swap, «ponerlo en su sitio» es subirlo al top de `a` y mandarlo a `b`. Al final, `b` tiene todo en orden descendente y basta con devolverlo. Es el algoritmo más fácil de explicar y la base de la estrategia `--simple`.

## Ficha

| | |
|---|---|
| Familia | O(n²), «algoritmo simple» del subject |
| Operaciones | O(n²) en el peor caso; ≈ n²/4 rotaciones con datos aleatorios |
| Memoria extra | O(1) |
| 100 aleatorios | 1 461 de media (peor 1 592) |
| 500 aleatorios | 32 338 (33 445) |
| Al revés (500) | 1 497 |
| Casi ordenada (500) | 1 764 |
| Uso recomendado | `--simple` |

## La idea original

Sobre un array: recorre todo, encuentra el mínimo, intercámbialo con la primera posición. Repite desde la segunda. Siempre hace n(n−1)/2 comparaciones, esté como esté la entrada (ver [[Algoritmos de ordenación y complejidad#Selección (*selection sort*) — O(n²) siempre]]).

## Adaptación a dos pilas

```text
mientras a no esté ordenada:
    pos = posición del mínimo de a
    si pos <= tamaño_a / 2:   ra × pos
    si no:                    rra × (tamaño_a − pos)
    pb
pa × tamaño_b
```

Por qué funciona: cada `pb` pone encima de `b` un número **mayor** que todos los anteriores, así que `b` queda descendente desde el top. Cada `pa` devuelve el mayor que queda, y `a` se construye ordenada.

> [!tip] Para antes de tiempo
> Si lo que queda en `a` ya está ordenado, deja de extraer y pasa directamente a los `pa`: ahorras rotaciones y `pb` inútiles. Si quedan 3, usa la [[push_swap - Estrategias#3 números|tabla de 3]].

## Traza real

Índices `2 4 1 3 0` (top a la izquierda):

```text
ini   a: 2 4 1 3 0        b:
rra   a: 0 2 4 1 3        b:            mínimo 0 en pos 4 de 5 → por abajo
pb    a: 2 4 1 3          b: 0
ra    a: 4 1 3 2          b: 0          mínimo 1 en pos 2 de 4 → por arriba
ra    a: 1 3 2 4          b: 0
pb    a: 3 2 4            b: 1 0
ra    a: 2 4 3            b: 1 0        mínimo 2 en pos 1
pb    a: 4 3              b: 2 1 0
ra    a: 3 4              b: 2 1 0
pb    a: 4                b: 3 2 1 0
pa    a: 3 4              b: 2 1 0
pa    a: 2 3 4            b: 1 0
pa    a: 1 2 3 4          b: 0
pa    a: 0 1 2 3 4        b:
total 13
```

## Complejidad

En cada extracción, subir el mínimo cuesta como mucho `tamaño_a / 2` rotaciones, y `tamaño_a` va bajando de n a 1:

```text
rotaciones ≤ n/2 + (n−1)/2 + … + 1/2 ≈ n²/4
pb + pa    = 2n
total      = O(n²)
```

Con datos aleatorios, el mínimo está de media a n/4 posiciones del lado más cercano, así que la suma real es ≈ n²/8 + 2n. Con n = 100: 1 250 + 200 ≈ 1 450, y la media medida es 1 461.

**¿Por qué el revés sale tan barato?** Con `n … 2 1 0`, el mínimo está siempre en el fondo: un `rra` y un `pb` por elemento, unas 3n operaciones. No contradice el O(n²): Big-O es una cota del **peor** caso.

## Pros y contras

| ✅ | ❌ |
|---|---|
| Se explica en una frase | Con 500 números: ~32 000 operaciones (no pasa ni el mínimo de 12 000) |
| Imposible equivocarse en el orden de `b` | No aprovecha el orden que ya tenga la entrada |
| Base de casi todos los demás (extraer, rotar por el lado corto) | |

## Errores frecuentes

> [!warning] Rotar siempre con `ra`
> Si no eliges el lado más corto, el coste medio se duplica.

> [!warning] Buscar el mínimo en los valores originales con negativos
> Funciona, pero es más fácil con índices normalizados: el mínimo de lo que queda es siempre el siguiente índice (0, 1, 2…), así que no hace falta buscarlo por valor.

## Preguntas de evaluación

> [!question] ¿Por qué es O(n²) si cada extracción es «solo» encontrar el mínimo?
> «Porque lo que contamos son operaciones de pila, no comparaciones: para llevar el mínimo arriba hacen falta hasta n/2 rotaciones, y eso se repite n veces.»

## Relacionadas

[[push_swap - Catálogo de algoritmos]] · [[push_swap - Inserción]] · [[push_swap - Extracción de mínimo y máximo]] · [[push_swap - Heap sort]] · [[push_swap - Estrategias#1. Simple O(n²)]] · [[Notación Big-O]]

## Referencias

- [Wikipedia: Selection sort](https://en.wikipedia.org/wiki/Selection_sort).
- Subject de push_swap v1.0, VI.3.3 («Adaptación del orden por selección») y VI.5.1: el ejemplo `--simple 5 4 3 2 1` es exactamente esta estrategia.
- Cifras y traza: simulador propio (ver [[push_swap - Catálogo de algoritmos]]), contrastado con un prototipo en C.
