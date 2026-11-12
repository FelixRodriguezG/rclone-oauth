---
tags: [proyecto, push-swap, rank-1, algoritmos, o-n2]
aliases: ["Extracción de mínimo y máximo", "Min-max en push_swap", "Selección doble"]
---
# push_swap - Extracción de mínimo y máximo

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · [[push_swap - Catálogo de algoritmos|Algoritmos]] · O(n²)

> Una mejora de la [[push_swap - Selección|selección]]: en cada paso miras **el mínimo y el máximo** de `a` y sacas **el que esté más cerca** del top. Los mínimos se apilan en `b` como siempre; los máximos se mandan al **fondo** de `b` con `rb`. Al final, un giro junta las dos mitades.

## Ficha

| | |
|---|---|
| Familia | O(n²), «métodos simples de extracción del mínimo/máximo» del subject |
| Operaciones | O(n²), ≈ 25 % menos que la selección con datos aleatorios |
| Memoria extra | O(1) |
| 100 aleatorios | 1 131 (1 215) |
| 500 aleatorios | 22 291 (23 334) |
| Al revés (500) | 1 497 |
| Casi ordenada (500) | 1 989 |
| Uso recomendado | `--simple` si quieres algo mejor que la selección sin complicarte |

## Adaptación a dos pilas

```text
j = 0                                    ← cuántos máximos hay al fondo de b
mientras a no esté vacía (o esté ordenada y j == 0):
    coste_min = rotaciones para subir el mínimo
    coste_max = rotaciones para subir el máximo + 1 (el rb)
    si coste_min <= coste_max:   subir mínimo, pb
    si no:                       subir máximo, pb, rb, j++
rrb × j                                  ← los máximos vuelven arriba
pa × n
```

Por qué cuadra al final:

```text
mínimos con pb:      se apilan arriba    →  b = mn_k … mn_2 mn_1         (descendente)
máximos con pb + rb: se van al fondo     →  b = … m_1 m_2 … m_j           (m_1 el mayor)
b completa:          mn_k … mn_1 | m_1 … m_j
rrb × j:             m_1 … m_j | mn_k … mn_1    ← descendente de arriba abajo ✅
pa × n:              a = mn_1 … mn_k m_j … m_1  ← ascendente ✅
```

> [!warning] No pares antes de tiempo si ya sacaste máximos
> En la selección puedes parar cuando lo que queda en `a` está ordenado. Aquí no, si `j > 0`: los máximos de `b` deben acabar **debajo** de lo que queda en `a`, y los `pa` los pondrían encima. La condición del bucle lo tiene en cuenta.

## Traza real

`0 4 2 1 3`, donde el máximo (4) está más cerca que el mínimo en el segundo paso:

```text
ini   a: 0 4 2 1 3        b:
pb    a: 4 2 1 3          b: 0            mínimo 0 arriba: coste 0
pb    a: 2 1 3            b: 4 0          mínimo 1 a 2 rotaciones; máximo 4 arriba: 0 + 1 → máximo
rb    a: 2 1 3            b: 0 4          el 4 al fondo
ra    a: 1 3 2            b: 0 4
pb    a: 3 2              b: 1 0 4
ra    a: 2 3              b: 1 0 4
pb    a: 3                b: 2 1 0 4
pb    a:                  b: 3 2 1 0 4
rrb   a:                  b: 4 3 2 1 0    j = 1 → un rrb
pa × 5                    a: 0 1 2 3 4
total 14
```

## Complejidad

Igual que la selección: cada extracción cuesta como mucho `tamaño_a / 2` rotaciones más 1 o 2 operaciones, y hay n extracciones → **O(n²)**. El giro final añade como mucho n `rrb`. La mejora es de **constante**: el más cercano de dos candidatos está, de media, más cerca que uno solo.

## Pros y contras

| ✅ | ❌ |
|---|---|
| ~25 % menos operaciones que la selección | Sigue siendo O(n²): 22 000 con 500 |
| Introduce la idea de usar **los dos extremos** de `b`, que aparece en chunks y rangos | Hay que pensar bien el giro final |

## Preguntas de evaluación

> [!question] ¿Por qué sumáis 1 al coste del máximo?
> «Porque el máximo, además de subirlo y hacer `pb`, necesita un `rb` para mandarlo al fondo de `b`.»

## Relacionadas

[[push_swap - Catálogo de algoritmos]] · [[push_swap - Selección]] · [[push_swap - Chunks]] · [[push_swap - Rangos (butterfly)]]

## Referencias

- Subject de push_swap v1.0, VI.3.3 («Métodos simples de extracción del mínimo/máximo»).
- [Wikipedia: Selection sort — Variants](https://en.wikipedia.org/wiki/Selection_sort#Variants) — la selección doble (mínimo y máximo a la vez).
- Cifras y traza: simulador propio (ver [[push_swap - Catálogo de algoritmos]]).
