---
tags: [proyecto, push-swap, rank-1, algoritmos, o-n-sqrt-n]
aliases: ["Bucket sort en push_swap", "Buckets", "Cubetas"]
---
# push_swap - Buckets

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · [[push_swap - Catálogo de algoritmos|Algoritmos]] · O(n√n)

> **Bucket sort** reparte los números en **cubetas** (*buckets*) según su valor y luego ordena cada cubeta por separado. En push_swap, las √n cubetas son rangos de índices que se pasan a `b` una detrás de otra. Después, cada cubeta se ordena al devolverla, sacando su máximo. Es la versión más simple de la familia de los [[push_swap - Chunks|chunks]]: sin el truco del `rb`.

## Ficha

| | |
|---|---|
| Familia | O(n√n): «adaptaciones del orden por buckets con √n buckets» del subject |
| Operaciones | O(n√n) |
| Memoria extra | O(1) |
| 100 aleatorios | 806 (834) |
| 500 aleatorios | 8 125 (8 246): **no** llega a «bueno» (< 8 000) |
| Al revés (500) | 1 515 |
| Casi ordenada (500) | 1 514 |
| Uso recomendado | Como paso didáctico hacia chunks o rangos |

## La idea original

```text
números: 29 3 17 41 8 35 22 12          4 cubetas de tamaño 12
cubeta 0 [0–11]:   3 8
cubeta 1 [12–23]:  17 22 12        → ordenar cada una y concatenar
cubeta 2 [24–35]:  29 35
cubeta 3 [36–47]:  41
```

Con un array, el reparto es una sola pasada, porque cada número va directo a su cubeta. Con dos pilas no se puede «ir directo»: solo hay dos sitios donde dejar algo (arriba o abajo de `b`). Por eso el reparto se hace **cubeta a cubeta**.

## Adaptación a dos pilas

```text
fase 1, para cada cubeta [lo, hi) de menor a mayor:
    mientras quede en a algún índice de la cubeta:
        sube el más cercano (ra o rra) y pb
fase 2:
    mientras b no esté vacía:
        sube el máximo de b (rb o rrb) y pa
```

Al acabar la fase 1, `b` está ordenada **por cubetas**: la última (la de los mayores) arriba. Dentro de cada cubeta, el orden es el de llegada. En la fase 2, el máximo está siempre en la cubeta de arriba, a ≤ √n posiciones… desde arriba.

## Traza real

`5 2 7 0 3 6 1 4` (3 cubetas: 0–2, 3–5, 6–7):

```text
ini   a: 5 2 7 0 3 6 1 4    b:
ra, pb                      b: 2                  cubeta 0–2
ra, pb                      b: 0 2
ra, ra, pb                  b: 1 0 2
pb, pb                      b: 5 4 1 0 2          cubeta 3–5
ra, pb                      b: 3 5 4 1 0 2
pb, pb                      b: 7 6 3 5 4 1 0 2    cubeta 6–7
--- fase 2 ---
pa, pa                      a: 6 7
rb, pa, pa                  a: 4 5 6 7            el 3 molesta → rb
rrb, pa                     a: 3 4 5 6 7
rrb, pa, pa, pa             a: 0 1 2 3 4 5 6 7
total 24
```

## Complejidad

- **Fase 1**: √n cubetas, como mucho una vuelta a `a` por cubeta → ≤ n√n rotaciones + n `pb`.
- **Fase 2**: el máximo está en la cubeta de arriba (≤ √n elementos), pero los que se saltan con `rb` acaban **al fondo**, mezclados con las cubetas de debajo. Por eso a veces hay que volver con `rrb`. Cada elemento cuesta O(√n) → n√n.

```text
total = O(n·√n)          memoria extra: O(1)
```

## Por qué es peor que chunks

| | Buckets | Chunks |
|---|---|---|
| Fase 1 | `pb` | `pb` + `rb` si es de la mitad baja |
| Cómo queda `b` | Cubetas apiladas, desordenadas por dentro | Cada chunk partido en dos mitades (arriba y abajo) |
| Distancia al máximo en la fase 2 | Hasta toda la cubeta | Hasta media cubeta, por arriba o por abajo |
| 500 números | 8 125 | 7 439 |

El `rb` cuesta una operación más por cada número de la mitad baja (~n/2 en total), pero ahorra bastantes más en la fase 2.

## Preguntas de evaluación

> [!question] ¿En qué se parece a un bucket sort de verdad?
> «En que primero se reparte por rangos de valor y luego se ordena dentro de cada rango. La diferencia es que con pilas el reparto no puede hacerse en una sola pasada: cada cubeta exige recorrer `a`.»

## Relacionadas

[[push_swap - Catálogo de algoritmos]] · [[push_swap - Chunks]] · [[push_swap - Partición por bloques]] · [[push_swap - Rangos (butterfly)]] · [[Notación Big-O]]

## Referencias

- [Wikipedia: Bucket sort](https://en.wikipedia.org/wiki/Bucket_sort).
- Subject de push_swap v1.0, VI.3.3 («Adaptaciones del orden por buckets con √n buckets»).
- Cifras y traza: simulador propio (ver [[push_swap - Catálogo de algoritmos]]).
