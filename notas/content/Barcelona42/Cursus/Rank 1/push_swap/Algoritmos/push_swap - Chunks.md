---
tags: [proyecto, push-swap, rank-1, algoritmos, o-n-sqrt-n]
aliases: ["Chunks", "Chunk sort", "Orden por chunks"]
---
# push_swap - Chunks

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · [[push_swap - Catálogo de algoritmos|Algoritmos]] · O(n√n)

> Divide los índices en **√n grupos** (*chunks*) de valores consecutivos: 0–9, 10–19… Pasa a `b` primero todo el chunk más bajo, luego el siguiente, y así. Después, devuelve siempre el **máximo** de `b`. Como `b` queda organizada por capas, el máximo nunca está lejos. Es el algoritmo O(n√n) más conocido de push_swap.

## Ficha

| | |
|---|---|
| Familia | O(n√n): «orden basado en chunks (dividiendo en √n chunks)» del subject |
| Operaciones | O(n√n) |
| Memoria extra | O(1) (solo los límites del chunk) |
| 100 aleatorios | 778 (805) |
| 500 aleatorios | 7 439 (7 493) |
| Al revés (500) | 1 975 |
| Casi ordenada (500) | 1 970 |
| Uso recomendado | `--medium`. Pasa «bueno» con 100 y 500, pero no «excelente» |

## La idea

Es una **ordenación por grupos**: no ordenas número a número, sino que primero separas «los pequeños», luego «los medianos»… y al final ordenas dentro de cada grupo, que es pequeño.

```text
n = 100 → 10 chunks de 10 índices
chunk 0: 0–9   chunk 1: 10–19   …   chunk 9: 90–99
```

## Adaptación a dos pilas

**Fase 1, de `a` a `b`:**

```text
para cada chunk [lo, hi), de menor a mayor:
    mientras quede en a algún índice del chunk:
        f = posición del primero del chunk desde arriba
        r = posición del primero del chunk desde abajo
        si f <= r + 1:  ra × f     si no:  rra × (r + 1)
        pb
        si el número es de la mitad baja del chunk (< lo + tamaño/2):  rb
```

**Fase 2, de `b` a `a`:**

```text
mientras b no esté vacía:
    sube el máximo de b por el lado más corto (rb o rrb)
    pa
```

El `rb` de la fase 1 es el truco: separa cada chunk en dos mitades, la alta arriba de `b` y la baja en el fondo. Así `b` queda en **capas**, y en la fase 2 el máximo siempre está cerca del top o cerca del fondo:

```text
b, de top a fondo, con chunks 0–3, 4–7 y 8–11:

   11 10 │ 7 6 │ 3 2 │ 0 1 │ 4 5 │ 8 9
   └─ mitades altas ─┘     └─ mitades bajas ─┘
   último chunk arriba     último chunk al fondo
```

## Traza real

`5 2 7 0 3 6 1 4` (n = 8 → 3 chunks de 3: 0–2, 3–5, 6–7):

```text
ini   a: 5 2 7 0 3 6 1 4    b:
ra    a: 2 7 0 3 6 1 4 5    b:                   chunk 0–2: el 2 está en pos 1
pb    a: 7 0 3 6 1 4 5      b: 2                 2 es mitad alta (≥ 1) → se queda arriba
ra    a: 0 3 6 1 4 5 7      b: 2
pb    a: 3 6 1 4 5 7        b: 0 2
rb    a: 3 6 1 4 5 7        b: 2 0               0 es mitad baja → al fondo
ra, ra, pb                  b: 1 2 0             el 1 (mitad alta)
pb, pb                      b: 5 4 1 2 0         chunk 3–5: el 4 y el 5 estaban arriba
ra, pb, rb                  b: 5 4 1 2 0 3       3 es mitad baja → al fondo
pb, rb, pb                  b: 7 5 4 1 2 0 3 6   chunk 6–7
--- fase 2 ---
pa                          a: 7                 máximo arriba
rrb, pa                     a: 6 7               el 6 estaba al fondo
pa, pa                      a: 4 5 6 7
rrb, pa                     a: 3 4 5 6 7
rb, pa, rb, pa, pa          a: 0 1 2 3 4 5 6 7
total 28
```

Con 8 números no compensa (la selección hace 24). La ventaja aparece con n grande.

## Complejidad

**Fase 1.** Hay √n chunks. Para sacar todos los elementos de un chunk, basta como mucho **una vuelta** a `a` (≤ n rotaciones): √n · n. Los `pb` y `rb` suman ≤ 2n.

**Fase 2.** Cuando toca sacar el máximo, este pertenece al chunk más alto que queda. Ese chunk está repartido entre el grupo de arriba y el del fondo de `b`, cada uno de unos √n/2 elementos, así que hacen falta del orden de √n rotaciones por elemento: n · √n.

```text
total = O(n·√n)          memoria extra: O(1)
```

## Variantes y ajustes

| Ajuste | Efecto |
|---|---|
| Número de chunks | √n es el del subject. Con un número **fijo** (5 con 100, 11 con 500) se rascan operaciones, pero el argumento O(n√n) deja de valer |
| Sin `rb` | Es la variante [[push_swap - Buckets\|buckets]]: más simple y algo peor |
| Dos chunks a la vez | Es la [[push_swap - Partición por bloques\|partición por bloques]]: menos vueltas a `a` |
| Ventana deslizante en vez de chunks fijos | Es [[push_swap - Rangos (butterfly)\|rangos]]: el mejor de la familia |
| Fase 2 con cálculo de coste | Insertar en `a` el elemento más barato en vez del máximo (como el [[push_swap - Algoritmo de coste (turco)\|algoritmo de coste]]) |

## Errores frecuentes

> [!warning] Buscar solo desde arriba
> Si solo usas `ra`, cada elemento del chunk cuesta la distancia desde el top. Mirar también desde abajo (`rra`) casi la reduce a la mitad.

> [!warning] Último chunk incompleto
> Si n no es múltiplo del número de chunks, el último es más pequeño. Usa `tamaño = (n + chunks − 1) / chunks` (ver [[División entera y módulo]]) y comprueba `x < hi` y no `x <= hi`.

## Preguntas de evaluación

> [!question] ¿Por qué √n chunks y no 2 o n?
> «Con pocos chunks, cada uno es enorme y en la fase 2 el máximo puede estar lejos. Con muchos, la fase 1 da demasiadas vueltas a `a`. Con √n los dos costes quedan equilibrados en n√n.»

## Relacionadas

[[push_swap - Catálogo de algoritmos]] · [[push_swap - Buckets]] · [[push_swap - Partición por bloques]] · [[push_swap - Rangos (butterfly)]] · [[push_swap - Estrategias#2. Intermedia O(n√n)]] · [[Notación Big-O]]

## Referencias

- Subject de push_swap v1.0, VI.3.3 («Orden basado en chunks (dividiendo en √n chunks)»).
- [cp-algorithms: Sqrt Decomposition](https://cp-algorithms.com/data_structures/sqrt_decomposition.html) — la idea general de dividir en √n bloques para equilibrar costes.
- Cifras y traza: simulador propio (ver [[push_swap - Catálogo de algoritmos]]).
