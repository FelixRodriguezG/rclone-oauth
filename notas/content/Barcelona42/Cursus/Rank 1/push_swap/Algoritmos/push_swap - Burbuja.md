---
tags: [proyecto, push-swap, rank-1, algoritmos, o-n2]
aliases: ["Bubble sort en push_swap", "Burbuja con pilas"]
---
# push_swap - Burbuja

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · [[push_swap - Catálogo de algoritmos|Algoritmos]] · O(n²)

> **Bubble sort**: compara cada pareja de vecinos y los intercambia si están al revés. En cada pasada, el mayor «sube» como una burbuja hasta el final. En push_swap solo puedes comparar **los dos de arriba** (`sa`), así que la pila se recorre **girándola** con `ra`. Usa **solo la pila `a`**: no necesita `b`.

## Ficha

| | |
|---|---|
| Familia | O(n²), «algoritmo simple» del subject |
| Operaciones | Θ(n²): ≈ 1,15·n² con datos aleatorios |
| Memoria extra | O(1), y no usa `b` |
| 100 aleatorios | 11 477 (12 721) |
| 500 aleatorios | 299 205 (306 185) |
| Al revés (500) | 374 250 (su peor caso) |
| Uso recomendado | Ninguno en serio. Sirve para **entender** por qué las rotaciones cuestan |

## La idea original

```text
pasada 1:  [5 1] 4 2 → 1 [5 4] 2 → 1 4 [5 2] → 1 4 2 5      ← el 5 ya está en su sitio
pasada 2:  [1 4] 2 5 → 1 [4 2] 5 → 1 2 4 5                   ✅
```

## Adaptación a dos pilas

Una pasada = una **vuelta completa** a la pila:

```text
mientras a no esté ordenada:
    repetir n − 1 veces:
        si a[0] > a[1]:  sa
        ra
    ra                     ← cierra la vuelta: la pila vuelve a su orientación inicial
```

Por qué funciona: el `ra` hace que la pareja «de arriba» avance por la pila. El `sa` deja siempre el mayor de los dos en segunda posición, así que el `ra` siguiente lo lleva **pegado** a la siguiente comparación. Igual que la burbuja en el array, el mayor viaja hasta el final de la vuelta.

## Traza real (primeras dos pasadas)

```text
ini   a: 2 4 1 3 0
ra    a: 4 1 3 0 2         2 < 4 → no hay sa
sa    a: 1 4 3 0 2         4 > 1 → sa
ra    a: 4 3 0 2 1
sa    a: 3 4 0 2 1         4 > 3 → sa
ra    a: 4 0 2 1 3
sa    a: 0 4 2 1 3         4 > 0 → sa
ra    a: 4 2 1 3 0
ra    a: 2 1 3 0 4         ← fin de la pasada 1: el 4 está al fondo
sa    a: 1 2 3 0 4
ra    a: 2 3 0 4 1
…
total 27 operaciones para 5 números (selección: 13)
```

## Complejidad

Cada pasada cuesta exactamente **n `ra`** más hasta n − 1 `sa`. Hacen falta hasta n − 1 pasadas (tantas como posiciones tenga que viajar el elemento más lejano):

```text
operaciones ≤ (n − 1)·(n + n − 1) ≈ 2n²       →  Θ(n²)
```

Es Θ y no solo O porque **cada pasada paga n rotaciones aunque no haya nada que intercambiar**. En un array, la pasada se acorta (los últimos ya están colocados); aquí no, porque para volver al principio hay que dar la vuelta entera.

## Variantes

- **Parar en cuanto esté ordenada**: comprobar después de cada `ra`, no solo al final de la pasada. Ahorra la última vuelta.
- **Cocktail shaker** (ida con `ra`, vuelta con `rra`): aprovecha las dos direcciones, pero sigue siendo Θ(n²).

## Pros y contras

| ✅ | ❌ |
|---|---|
| Solo usa `sa` y `ra` | 10–20 veces peor que selección |
| No necesita `b` | Ni siquiera aprovecha las listas casi ordenadas: cada pasada gira entera |
| Muy fácil de demostrar | |

## Preguntas de evaluación

> [!question] En un array, la burbuja es O(n) si ya está ordenado. ¿Y aquí?
> «Si ya está ordenada, 0 operaciones, porque lo comprobamos antes de empezar. Pero con un solo elemento fuera de sitio ya hay que pagar vueltas enteras de n rotaciones: una por cada posición que tenga que recorrer hacia arriba.»

## Relacionadas

[[push_swap - Catálogo de algoritmos]] · [[push_swap - Selección]] · [[push_swap - Inserción]] · [[Algoritmos de ordenación y complejidad#Burbuja (*bubble sort*) — O(n²)]]

## Referencias

- [Wikipedia: Bubble sort](https://en.wikipedia.org/wiki/Bubble_sort) y [Cocktail shaker sort](https://en.wikipedia.org/wiki/Cocktail_shaker_sort).
- Subject de push_swap v1.0, VI.3.3 («Adaptación del orden burbuja»).
- Cifras y traza: simulador propio (ver [[push_swap - Catálogo de algoritmos]]).
