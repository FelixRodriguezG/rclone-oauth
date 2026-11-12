---
tags: [proyecto, push-swap, rank-1, algoritmos, o-n-sqrt-n]
aliases: ["Partición por bloques", "Bloques", "Block partition"]
---
# push_swap - Partición por bloques

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · [[push_swap - Catálogo de algoritmos|Algoritmos]] · O(n√n)

> Una mejora de los [[push_swap - Chunks|chunks]]: en cada vuelta a `a` se sacan **dos bloques a la vez**, uno por debajo de la mediana y otro por encima. Se empieza por los del centro y se avanza hacia los extremos. El bloque bajo se manda al **fondo** de `b` (`pb` + `rb`) y el alto se queda **arriba** (`pb`). Así hacen falta la mitad de vueltas, y `b` queda **ordenada por bloques** de mayor a menor.

## Ficha

| | |
|---|---|
| Familia | O(n√n): «métodos de partición basados en bloques» del subject |
| Operaciones | O(n√n) |
| Memoria extra | O(1) |
| 100 aleatorios | **650** (687) |
| 500 aleatorios | **5 918** (6 010) |
| Al revés (500) | 1 760 |
| Casi ordenada (500) | 2 134 |
| Uso recomendado | `--medium`: el «excelente» con 100, casi con 500 |

> [!note] «Partición por bloques» no es un algoritmo con nombre único
> El subject lo pone como ejemplo de una idea: partir por bloques de valores. Esta nota describe una forma concreta y eficaz de hacerlo. Si diseñáis otra, documentadla igual: qué bloques, en qué orden, adónde va cada uno.

## La idea

```text
n = 100, √n = 10 bloques:   B0 B1 B2 B3 B4 | B5 B6 B7 B8 B9
                                         mediana
pasada 1:  B4 (bajo) y B5 (alto)
pasada 2:  B3 (bajo) y B6 (alto)
…
pasada 5:  B0 (bajo) y B9 (alto)
```

En cada pasada, los del bloque **bajo** van al fondo de `b` y los del **alto**, arriba:

```text
b al final (de top a fondo):
   B9 B8 B7 B6 B5 │ B4 B3 B2 B1 B0
   └ altos: el último ┘ └ bajos: el último
     sacado arriba        sacado al fondo

leída de arriba abajo: B9 > B8 > … > B1 > B0
```

Aunque los bloques bajos entran por el fondo y los altos por arriba, el resultado es `b` **ordenada por bloques de mayor a menor**. Dentro de cada bloque no hay orden, pero eso solo cuesta unas pocas rotaciones en la fase 2.

## Adaptación a dos pilas

```text
bloques ordenados del centro hacia fuera: (B4, B5), (B3, B6), …
para cada pareja (bajo, alto):
    mientras quede en a algún número de la pareja:
        sube el más cercano (ra o rra) y pb
        si es del bloque bajo y b tiene más de 1:  rb
fase 2:
    mientras b no esté vacía:
        sube el máximo de b (rb o rrb) y pa
```

En la fase 2, el máximo está siempre en el bloque de arriba de `b`.

## Traza real

`5 2 7 0 3 6 1 4` (n = 8 → bloques de 3: 0–2, 3–5, 6–7; pareja 1: (0–2, 3–5), pareja 2: (—, 6–7)):

```text
ini   a: 5 2 7 0 3 6 1 4    b:
pb                          b: 5                  5: bloque alto → arriba
pb, rb                      b: 5 2                2: bloque bajo → fondo
ra, pb, rb                  b: 5 2 0              0: bajo → fondo
pb                          b: 3 5 2 0            3: alto
ra, pb, rb                  b: 3 5 2 0 1          1: bajo
pb                          b: 4 3 5 2 0 1        4: alto
pb, pb                      b: 6 7 4 3 5 2 0 1    pareja 2: solo el bloque alto 6–7
--- fase 2: sacar el máximo ---
rb, pa                      a: 7                  el 7 estaba en la posición 1
rrb, pa                     a: 6 7
rb, rb, pa                  a: 5 6 7
rrb, rrb, pa, pa, pa        a: 2 3 4 5 6 7
rb, pa, pa                  a: 0 1 2 3 4 5 6 7
total 28
```

(Con números tan pocos, el orden exacto de la pareja impar cambia las cifras; la mejora se ve con 100 y 500.)

## Complejidad

- **Fase 1**: hay √n/2 parejas, y cada una necesita como mucho una vuelta a `a`: ≤ (√n/2)·n rotaciones. Es la mitad de vueltas que los chunks.
- **Fase 2**: el máximo está siempre dentro del bloque de arriba (≤ √n elementos): O(√n) rotaciones por elemento.

```text
total = O(n·√n)          memoria extra: O(1)
```

## Comparada con los chunks

| | Chunks | Partición por bloques |
|---|---|---|
| Vueltas a `a` en la fase 1 | √n | √n / 2 |
| Uso de `rb` | Mitad baja de cada chunk | Bloque bajo entero |
| Forma de `b` | Capas: mitades altas arriba, bajas al fondo | Bloques de mayor (arriba) a menor (fondo) |
| 100 / 500 | 778 / 7 439 | 650 / 5 918 |

## Errores frecuentes

> [!warning] Empezar por los extremos
> Si la primera pareja es (B0, B9), el B9 queda debajo del B8, el B8 debajo del B7… Los bloques acaban en orden **inverso** y la fase 2 se encarece. Del centro hacia fuera, cada bloque nuevo se coloca justo donde le toca.

> [!warning] Número impar de bloques
> Uno se queda sin pareja: procésalo solo en la última pasada (como en la traza).

## Preguntas de evaluación

> [!question] ¿Por qué sigue siendo O(n√n) si das la mitad de vueltas?
> «Porque dividir por 2 es una constante: √n/2 vueltas de n rotaciones siguen siendo del orden de n√n.»

## Relacionadas

[[push_swap - Catálogo de algoritmos]] · [[push_swap - Chunks]] · [[push_swap - Buckets]] · [[push_swap - Rangos (butterfly)]] · [[push_swap - Quicksort]]

## Referencias

- Subject de push_swap v1.0, VI.3.3 («Métodos de partición basados en bloques»).
- [cp-algorithms: Sqrt Decomposition](https://cp-algorithms.com/data_structures/sqrt_decomposition.html) — la idea de trabajar con √n bloques.
- Cifras y traza: simulador propio (ver [[push_swap - Catálogo de algoritmos]]).
