---
tags: [proyecto, push-swap, rank-1, algoritmos, o-n-log-n]
aliases: ["Merge sort en push_swap", "Mezcla con dos pilas"]
---
# push_swap - Merge sort

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · [[push_swap - Catálogo de algoritmos|Algoritmos]] · O(n log n)

> **Merge sort**: ordena trozos pequeños y luego los **mezcla** de dos en dos, cada vez más grandes: 1+1 → 2, 2+2 → 4… Mezclar dos listas ordenadas es fácil: miras la cabeza de cada una y sacas la menor. En push_swap, una lista está en `a` y la otra se aparta a `b`, y el resultado se va construyendo **en el fondo de `a`**. Es Θ(n log n) demostrable, pero la constante es alta.

## Ficha

| | |
|---|---|
| Familia | O(n log n): «adaptación del orden por fusión utilizando dos stacks» del subject |
| Operaciones | **Θ(n log n)**: ≈ 2,5 · n · log₂ n, igual para cualquier entrada |
| Memoria extra | O(1) |
| 100 aleatorios | 1 717 (siempre) |
| 500 aleatorios | 10 829 (siempre): solo pasa el **mínimo** (< 12 000) |
| Al revés / casi ordenada (500) | 10 829 / 10 829 |
| Uso recomendado | Para entender la mezcla con pilas. Para entregar, mejor [[push_swap - Quicksort\|quicksort]] |

## La idea original

```text
            [5 1 4 2 8 0]
   ancho 1: [1 5] [2 4] [0 8]      ← mezclar parejas de 1
   ancho 2: [1 2 4 5] [0 8]        ← mezclar parejas de 2
   ancho 4: [0 1 2 4 5 8]          ← mezclar parejas de 4
```

Esta es la versión **ascendente** (*bottom-up*): sin recursión, duplicando el ancho de los trozos (*runs*) en cada pasada.

## Adaptación a dos pilas

```text
w = 1
mientras w < n:
    recorrer a por parejas de runs (R1, R2) de ancho w:
        pb × |R1|                        R1 pasa a b: queda al revés (su mínimo al fondo de b)
        mientras queden elementos de R1 o de R2:
            si el mínimo de R1 (fondo de b) < top de a (mínimo de R2):
                rrb, pa, ra              el de R1 al fondo de a
            si no:
                ra                       el de R2 al fondo de a
        (si un run se queda sin pareja: ra × |R1|)
    w = w × 2
```

```text
Mezclar R1 = [1 5] con R2 = [2 4] (a = 1 5 2 4 …):

pb pb            a: 2 4 …          b: 5 1          el mínimo de R1 (1) está al fondo de b
1 < 2 → rrb pa ra    a: 2 4 … 1
2 < 5 → ra           a: 4 … 1 2
4 < 5 → ra           a: … 1 2 4
solo R1 → pa ra       a: … 1 2 4 5   ✅ (con un solo elemento en b, el rrb sobra)
```

Tras recorrer todas las parejas, la pila ha dado **una vuelta completa** y vuelve a empezar por el primer run, ahora del doble de ancho.

## Traza real

Índices `2 4 1 3 0`. Cuando `b` tiene un solo elemento, el `rrb` sobra y se omite:

```text
--- w = 1: runs [2] [4] | [1] [3] | [0] ---
pb, pa, ra          a: 4 1 3 0 2         R1 = [2]: 2 < 4 → al fondo
ra                  a: 1 3 0 2 4         R2 = [4] al fondo
pb, pa, ra, ra      a: 0 2 4 1 3         [1] y [3]
ra                  a: 2 4 1 3 0         [0] sin pareja: solo ra
--- w = 2: runs [2 4] [1 3] | [0] ---
pb, pb              a: 1 3 0             b: 4 2      (el mínimo de R1, 2, al fondo de b)
ra                  a: 3 0 1             1 < 2
rrb, pa, ra         a: 3 0 1 2           2 < 3
ra                  a: 0 1 2 3           3 < 4
pa, ra              a: 0 1 2 3 4         solo queda el 4
ra                  a: 1 2 3 4 0         [0] sin pareja
--- w = 4: runs [1 2 3 4] [0] ---
pb × 4              a: 0                 b: 4 3 2 1
ra                  a: 0                 0 < 1
rrb, pa, ra  × 3    a: 0 1 2 3           1, 2, 3
pa, ra              a: 0 1 2 3 4         ✅
total 35
```

Con 5 números son 35 operaciones. Selección hace 13, y radix LSD, 25.

## Complejidad

Hay **⌈log₂ n⌉ pasadas** (el ancho se duplica hasta superar n). En cada pasada:

| Elementos | Operaciones por elemento |
|---|---|
| De R1 | `pb` + `rrb` + `pa` + `ra` = 4 |
| De R2 | `ra` = 1 |

Media: (4 + 1) / 2 = **2,5 operaciones por elemento y pasada**:

```text
operaciones ≈ 2,5 · n · ⌈log₂ n⌉           n = 100: 2,5 · 100 · 7 = 1 750  (medido: 1 717)
```

Es Θ porque el algoritmo no mira si la entrada estaba ordenada: siempre hace todas las pasadas completas.

## Mejoras posibles

| Mejora | Efecto |
|---|---|
| **Runs naturales** | Empezar con los tramos que ya vienen ordenados en vez de con ancho 1. En listas casi ordenadas, muchas menos pasadas |
| **Mezcla en ping-pong** | Alternar: una pasada mezcla hacia `b` en orden descendente y la siguiente hacia `a` en ascendente. Ahorra los `rrb` |
| **Casos base mayores** | Ordenar trozos de 3 a mano antes de empezar a mezclar |

## Pros y contras

| ✅ | ❌ |
|---|---|
| Θ(n log n) muy fácil de justificar | Constante grande: 10 829 con 500 |
| No necesita recursión ni medianas | No aprovecha el orden previo (sin la mejora de runs naturales) |

## Preguntas de evaluación

> [!question] Si es O(n log n) como el quicksort, ¿por qué da el doble de operaciones?
> «Big-O ignora las constantes: aquí cada elemento de R1 cuesta 4 operaciones por pasada, porque hay que sacarlo del fondo de `b`. En quicksort, cada elemento cuesta 1 o 2.»

## Relacionadas

[[push_swap - Catálogo de algoritmos]] · [[push_swap - Quicksort]] · [[Algoritmos de ordenación y complejidad#Merge sort — O(n log n) siempre]] · [[Notación Big-O]]

## Referencias

- [Wikipedia: Merge sort — Bottom-up implementation](https://en.wikipedia.org/wiki/Merge_sort#Bottom-up_implementation).
- [Wikipedia: Timsort](https://en.wikipedia.org/wiki/Timsort) — la idea de aprovechar *runs* naturales.
- Subject de push_swap v1.0, VI.3.3 («Adaptación del orden por fusión utilizando dos stacks»).
- Cifras y traza: simulador propio (ver [[push_swap - Catálogo de algoritmos]]).
