---
tags: [proyecto, push-swap, rank-1, algoritmos, extra, greedy]
aliases: ["Algoritmo turco", "Turk algorithm", "Algoritmo de coste", "Mechanical Turk"]
---
# push_swap - Algoritmo de coste (turco)

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · [[push_swap - Catálogo de algoritmos|Algoritmos]] · Extra

> **No lo nombra el subject**, pero es el algoritmo más usado en 42 para conseguir la nota máxima. Es una [[push_swap - Inserción|inserción]] inteligente: en cada paso **calcula cuánto costaría** colocar cada elemento de `a` en su sitio de `b`, y mueve **el más barato**. Además, combina las rotaciones de las dos pilas (`rr`, `rrr`) siempre que puede. Es un algoritmo **voraz** (*greedy*): elige lo mejor ahora, sin planificar el futuro.

## Ficha

| | |
|---|---|
| Familia | No está en el subject. Voraz, basado en inserción |
| Operaciones | Muy pocas en la práctica. Cota teórica del peor caso: O(n²) |
| Coste en CPU | O(n²) (calcula el coste de todos en cada paso; no se mide) |
| Memoria extra | O(1) |
| 100 aleatorios | **560** (604): excelente |
| 500 aleatorios | **5 068** (5 348): excelente |
| Al revés (500) | 1 496 |
| Casi ordenada (500) | 1 006 |
| Uso recomendado | Tramo alto de la adaptativa, o fase 2 de otros algoritmos, **con la justificación bien hecha** |

> [!note] ¿Por qué «turco»?
> Es el nombre con el que circula en la comunidad de 42 (*Turk algorithm*), a raíz de artículos y repositorios de estudiantes. No es un nombre de la literatura académica: en el README es mejor describirlo como «inserción voraz por coste mínimo».

## Adaptación a dos pilas

`b` se mantiene **descendente en circular**, como en la inserción.

```text
pb, pb                                    (si a tiene más de 3)
mientras a tenga más de 3:
    para cada x de a, en la posición i:
        objetivo(x) = el mayor de b menor que x  (si no hay, el máximo de b)
        j = posición de objetivo(x)
        coste(x) = mín de:
            max(i, j)                         ra y rb juntos con rr
            max(len_a − i, len_b − j)         rra y rrb juntos con rrr
            i + (len_b − j)                   ra + rrb
            (len_a − i) + j                   rra + rb
    ejecutar las rotaciones del x más barato (usando rr/rrr) y pb
ordenar los 3 de a (tabla de 3)
mientras b no esté vacía:
    objetivo en a: el menor de a mayor que el top de b (si no hay, el mínimo de a)
    rotar a hasta él y pa
subir el mínimo de a por el lado más corto
```

```text
Ejemplo de coste (len_a = 10, len_b = 6):
x en i = 2, su objetivo en j = 1
   arriba en las dos:  max(2, 1)          = 2   ← rr, ra, pb
   abajo en las dos:   max(8, 5)          = 8
   mixto:              2 + 5 = 7 ,  8 + 1 = 9
coste = 2 (+1 del pb)
```

## Traza real

`5 2 7 0 3 6 1 4`:

```text
pb, pb              a: 7 0 3 6 1 4      b: 2 5
rrb, pb             a: 0 3 6 1 4        b: 7 5 2       7: mayor que todos → encima del máximo (5)
pb                  a: 3 6 1 4          b: 0 7 5 2     0: menor que todos → encima del máximo (7)
rrb, pb             a: 6 1 4            b: 3 2 0 7 5   3: encima del 2
ra                  a: 1 4 6                           ordenar los 3
--- devolver ---
ra, pa              a: 3 4 6 1          b: 2 0 7 5     3 va encima del 4
pa                  a: 2 3 4 6 1                       2 encima del 3
rra, pa             a: 0 1 2 3 4 6                     0 encima del 1
pa                  a: 7 0 1 2 3 4 6                   7: no hay mayor → encima del mínimo
rrr, pa             a: 5 6 7 0 1 2 3 4                 5 encima del 6 (rrr: b tiene 1 elemento, equivale a rra)
ra × 3              a: 0 1 2 3 4 5 6 7
total 19
```

## Complejidad: hay que ser honesto

**En la práctica**, con datos aleatorios, cada elemento cuesta pocas rotaciones porque siempre se elige el más barato: el total medido crece como ~n log n (560 con 100 y 5 068 con 500).

**En teoría**, no hay una demostración sencilla de O(n log n). La cota fácil es:

```text
cada inserción ≤ max(len_a, len_b) / 2 rotaciones + 1  →  total ≤ O(n²)
```

> [!warning] Cómo presentarlo
> - Correcto: «Cota demostrada O(n²); comportamiento medido ~1,1 · n log₂ n con entradas aleatorias».
> - Arriesgado: «Es O(n log n)», sin más. Si el evaluador pregunta por qué, no hay un argumento corto.
>
> Si lo usáis como `--complex`, el subject exige que la clase declarada sea válida en el modelo. Una alternativa limpia: `--complex` = [[push_swap - Quicksort|quicksort con mediana]] (demostrable), y el de coste solo dentro de la adaptativa, justificado con medidas.

## Mejoras habituales

| Mejora | Efecto |
|---|---|
| Al hacer los dos primeros `pb`, mandar primero los «pequeños» (por debajo de la mediana) | `b` empieza mejor repartida |
| Dejar en `a` una [[push_swap - LIS\|subsecuencia creciente]] en vez de solo 3 | Menos elementos que insertar: es el algoritmo LIS |
| Usar la fase 2 del coste en otros algoritmos | Por ejemplo, chunks + inserción por coste |

## Preguntas de evaluación

> [!question] ¿Por qué hay cuatro fórmulas de coste?
> «Porque cada pila se puede rotar hacia arriba o hacia abajo. Si las dos van en el mismo sentido, `rr` o `rrr` mueven las dos a la vez y el coste es el máximo; si van en sentidos distintos, se suman.»

> [!question] ¿Es óptimo?
> «No. Es voraz: elige el más barato ahora, pero eso no garantiza el mínimo total. Para pocos números, el óptimo exacto se puede buscar con [[push_swap - Búsqueda óptima (BFS)|BFS]].»

## Relacionadas

[[push_swap - Catálogo de algoritmos]] · [[push_swap - Inserción]] · [[push_swap - LIS]] · [[push_swap - Quicksort]] · [[Notación Big-O]]

## Referencias

- Búsqueda «push_swap turk algorithm»: artículos y repositorios de estudiantes de 42 que describen esta variante (no hay una fuente canónica).
- [Wikipedia: Greedy algorithm](https://en.wikipedia.org/wiki/Greedy_algorithm) — qué es un algoritmo voraz y por qué no garantiza el óptimo.
- Cifras y traza: simulador propio (ver [[push_swap - Catálogo de algoritmos]]).
