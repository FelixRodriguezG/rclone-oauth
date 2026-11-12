---
tags: [proyecto, push-swap, rank-1, algoritmos, extra, adaptativo]
aliases: ["LIS", "Subsecuencia creciente más larga", "Longest increasing subsequence"]
---
# push_swap - LIS

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · [[push_swap - Catálogo de algoritmos|Algoritmos]] · Extra

> **No lo nombra el subject**, y es el que mejor ha funcionado en estas pruebas. La idea: buscar en `a` la **subsecuencia creciente más larga** (*Longest Increasing Subsequence*, LIS) y **no tocarla**. Todo lo demás se pasa a `b` en una sola vuelta, y luego se reinserta en `a`, cada elemento en su hueco, eligiendo siempre el más barato. Cuanto más ordenada esté la entrada, más larga es la LIS y menos trabajo queda. Por eso encaja de forma natural en el **tramo de desorden bajo** de la adaptativa.

## Ficha

| | |
|---|---|
| Familia | No está en el subject. Adaptativo: inserción por coste a partir de la LIS |
| Operaciones | n + k·O(n) en el peor caso, con k = elementos fuera de la LIS. O(n) si k está acotado |
| Coste en CPU | O(n log n) para calcular la LIS + O(n·k) para los costes (no se mide) |
| Memoria extra | O(n) (marcar qué elementos son de la LIS) |
| 100 aleatorios | **547** (582): excelente |
| 500 aleatorios | **4 950** (5 131): excelente, el mejor medido |
| Al revés (500) | 1 497 |
| Casi ordenada (500) | **909**: el mejor medido |
| Uso recomendado | Tramo bajo (< 0,2) de la adaptativa, y candidato serio para los otros |

## Qué es la LIS

La subsecuencia creciente más larga de una lista: elementos **en orden de aparición**, no necesariamente seguidos, que van creciendo.

```text
5 2 7 0 3 6 1 4
LIS de longitud 3, por ejemplo:  2 3 6  ·  0 3 6  ·  0 1 4  ·  2 3 4
(no existe ninguna de longitud 4)
```

Si `a` tiene una LIS de longitud L, esos L elementos ya están en el orden correcto entre sí: se pueden dejar en `a` y solo hay que meter los **n − L** restantes en sus huecos.

| Entrada | Longitud de la LIS | Elementos que hay que mover |
|---|---|---|
| Ordenada | n | 0 |
| 5 elementos fuera de sitio | ≈ n − 5 | ≈ 5 |
| Aleatoria (n = 500) | ≈ 2√n ≈ 45 | ≈ 455 |

Se calcula en O(n log n) con el método de las pilas de paciencia (*patience sorting*). En CPU, no cuesta operaciones.

## Adaptación a dos pilas

```text
1. calcular la LIS de a (en memoria) y marcar sus elementos
2. una vuelta a a:
       si el top es de la LIS:  ra
       si no:                   pb
   (se puede parar cuando en a solo quedan los de la LIS)
3. mientras b no esté vacía:
       para cada y de b: su hueco en a = encima del menor de a mayor que y
                         (si no hay, encima del mínimo de a)
       coste(y) = las cuatro combinaciones de rotaciones, como en el algoritmo de coste
       mover el más barato (rr/rrr cuando se pueda) y pa
4. subir el mínimo de a por el lado más corto
```

El paso 3 es la fase de devolución del [[push_swap - Algoritmo de coste (turco)|algoritmo de coste]], pero eligiendo entre **todos** los de `b`, no solo el top.

## Traza real

`5 2 7 0 3 6 1 4`, con la LIS `0 1 4`:

```text
pb, pb, pb          a: 0 3 6 1 4        b: 7 2 5      5, 2, 7 no son de la LIS
ra                  a: 3 6 1 4 0                       0 sí: se queda (al fondo)
pb, pb              a: 1 4 0            b: 6 3 7 2 5   en a solo queda la LIS
--- reinsertar, el más barato cada vez ---
rra, pa             a: 6 0 1 4                         6: no hay mayor → encima del mínimo
rra, pa             a: 3 4 6 0 1                       3 encima del 4
rb, pa              a: 2 3 4 6 0 1                     2 encima del 3
rrr, rra, pa        a: 7 0 1 2 3 4 6                   7: encima del mínimo
rrr, pa             a: 5 6 7 0 1 2 3 4                 5 encima del 6
ra × 3              a: 0 1 2 3 4 5 6 7
total 20
```

## Complejidad

- Paso 2: exactamente **n** operaciones (`ra` o `pb`).
- Paso 3: k = n − L inserciones. Cada una cuesta como mucho ~n/2 rotaciones + 1 → O(k·n).
- Paso 4: ≤ n/2.

```text
total ≤ n + k·(n/2 + 1) + n/2   =   O(n + k·n)
```

- Si la entrada tiene **pocos elementos fuera de sitio** (k acotado): **O(n)**. Es lo que pide el subject para el tramo de desorden bajo.
- En general: O(n²). Medido con aleatorios, el comportamiento es mucho mejor (~1,1 · n log₂ n) porque se elige siempre el más barato.

> [!warning] Desorden bajo no siempre implica k pequeño
> Una lista con **bloques de 10 barajados** tiene un desorden de 0,009, pero su LIS solo conserva unos 4 o 5 elementos de cada bloque: hay que mover más de la mitad. Medido con 500: 2 159 operaciones, unas 4·n. Sigue siendo el mejor algoritmo para ese caso, pero no es «O(n)» en sentido estricto. Ver [[push_swap - Estrategias#El tramo O(n) y su trampa]].

## Pros y contras

| ✅ | ❌ |
|---|---|
| El mejor medido: aleatorios, casi ordenados y al revés | Más código: LIS + cálculo de costes |
| Se adapta solo al orden que ya tenga la entrada | Cota teórica general O(n²): hay que explicarla bien |
| Justifica bien el tramo O(n) de la adaptativa | |

## Preguntas de evaluación

> [!question] ¿Por qué la LIS y no simplemente dejar 3 elementos en `a`?
> «Porque los de la LIS ya están en orden entre sí: no hace falta moverlos. Cuantos más dejemos quietos, menos inserciones. En una lista casi ordenada se quedan casi todos.»

> [!question] ¿Cómo calculáis la LIS?
> «Con el método de las pilas de paciencia, en O(n log n) de CPU, guardando de qué elemento viene cada uno para reconstruirla. No genera ninguna operación de push_swap.»

## Relacionadas

[[push_swap - Catálogo de algoritmos]] · [[push_swap - Algoritmo de coste (turco)]] · [[push_swap - Inserción]] · [[push_swap - Estrategias#4. Adaptativa]] · [[push_swap - Índice de desorden]]

## Referencias

- [Wikipedia: Longest increasing subsequence](https://en.wikipedia.org/wiki/Longest_increasing_subsequence) — algoritmo O(n log n) y la longitud esperada ≈ 2√n en una permutación aleatoria.
- [Wikipedia: Patience sorting](https://en.wikipedia.org/wiki/Patience_sorting) — el método de las pilas de paciencia.
- Cifras y traza: simulador propio (ver [[push_swap - Catálogo de algoritmos]]).
