---
tags: [proyecto, push-swap, rank-1, algoritmos, o-n-sqrt-n]
aliases: ["Butterfly", "Rangos", "K-sort", "Ventana deslizante en push_swap"]
---
# push_swap - Rangos (butterfly)

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · [[push_swap - Catálogo de algoritmos|Algoritmos]] · O(n√n)

> En vez de chunks fijos, usa una **ventana que se desliza**: acepta en `b` cualquier número que no sea mucho mayor que los ya pasados (`x ≤ i + r`). Los muy pequeños (`x ≤ i`) van al fondo de `b` y los demás, arriba. `b` termina casi ordenada, con forma de **mariposa** (*butterfly*): grandes en los dos extremos y pequeños en el centro. Es el mejor de la familia O(n√n) en estas pruebas.

## Ficha

| | |
|---|---|
| Familia | O(n√n): «estrategias de orden basadas en rangos» del subject |
| Operaciones | O(n√n) con r ≈ √n |
| Memoria extra | O(1) (un contador `i` y el ancho `r`) |
| 100 aleatorios | **584** (612): excelente |
| 500 aleatorios | **5 085** (5 160): excelente |
| Al revés (500) | 5 871 (su peor caso) |
| Casi ordenada (500) | 1 063 |
| Uso recomendado | `--medium` y tramo medio de la adaptativa |

## La idea

`i` cuenta cuántos números has pasado ya a `b`. Como trabajas con índices normalizados, si todo fuera perfecto el siguiente en pasar sería el `i`. La ventana `[i, i + r]` admite un margen de error `r`:

```text
top de a = x
    x ≤ i        → pb, rb      demasiado pequeño para la ventana: al fondo de b
    x ≤ i + r    → pb          dentro de la ventana: arriba de b
    x > i + r    → ra          aún es pronto para él
cada pb:  i++
```

## Adaptación a dos pilas

```text
i = 0
mientras a no esté vacía:
    según el top de a (como arriba): pb + rb, pb o ra
fase 2:
    mientras b no esté vacía:
        sube el máximo de b (rb o rrb) y pa
```

Por qué `b` queda casi ordenada: cada número entra cuando `i` está a menos de `r` de él. Los que entran «tarde» (`x ≤ i`) van al fondo; los demás, arriba. Al final, ningún número está a más de unas `r` posiciones de su sitio, contando desde el top o desde el fondo.

```text
b al final (de top a fondo):

   grandes ─► medianos ─► pequeños │ pequeños ─► medianos ─► grandes
   └─────────── pb ──────────────┘ └────────── pb + rb ─────────────┘
                     forma de mariposa: el máximo siempre en un extremo
```

## Traza real

`5 2 7 0 3 6 1 4` con una ventana pequeña, `r = 2`, para que se vean los `ra`:

```text
               a                 b              i   decisión
ini            5 2 7 0 3 6 1 4                   0
ra             2 7 0 3 6 1 4 5                   0   5 > 0+2
pb             7 0 3 6 1 4 5     2               1   2 ≤ 2
ra             0 3 6 1 4 5 7     2               1   7 > 3
pb, rb         3 6 1 4 5 7       2 0             2   0 ≤ 1 → fondo
pb             6 1 4 5 7         3 2 0           3   3 ≤ 4
ra             1 4 5 7 6         3 2 0           3   6 > 5
pb, rb         4 5 7 6           3 2 0 1         4   1 ≤ 3 → fondo
pb, rb         5 7 6             3 2 0 1 4       5   4 ≤ 4 → fondo
pb, rb         7 6               3 2 0 1 4 5     6   5 ≤ 5 → fondo
pb             6                 7 3 2 0 1 4 5   7   7 ≤ 8
pb, rb                           7 3 2 0 1 4 5 6 8   6 ≤ 7 → fondo
--- fase 2 ---
pa                               3 2 0 1 4 5 6        7 arriba
rrb, pa  ×3                      3 2 0 1              6, 5, 4 al fondo
pa, pa, rb, pa, pa               (vacía)              3, 2, (rb) 1, 0
total 28
```

## Complejidad

**Fase 1.** Mientras se da una vuelta completa a `a`, entran en `b` todos los números `≤ i + r`, e `i` avanza al menos `r` (salvo al final). Así que hacen falta como mucho `n / r` vueltas, de ≤ n rotaciones cada una:

```text
rotaciones fase 1 ≤ (n / r) · n
```

**Fase 2.** Cada número quedó a ≲ r posiciones de su sitio, contando desde un extremo de `b`. Sacar el máximo cuesta O(r) rotaciones:

```text
rotaciones fase 2 ≤ n · r
```

Total: `n²/r + n·r`. El mínimo de esa suma está en **r = √n**, y entonces los dos términos valen n√n:

```text
total = O(n·√n)  con r ≈ √n          memoria extra: O(1)
```

## Cómo elegir `r`

Medido (media / peor):

| n | r = √n | r = 1,5·√n | r = 2·√n | Fórmula «popular» |
|---|---|---|---|---|
| 100 | r = 10: 595 / 639 | r = 15: **579 / 621** | r = 20: 598 / 645 | r = 17: 584 / 612 |
| 500 | r = 22: 5 340 / 5 417 | r = 33: **5 076 / 5 175** | r = 44: 5 333 / 5 499 | r = 29: 5 085 / 5 160 |

La «fórmula popular» que circula en 42 es `r = 0,000000053·n² + 0,03·n + 14,5`, una curva ajustada a ojo. **r ≈ 1,5·√n** da lo mismo y se justifica con el análisis de arriba: el óptimo teórico es r ∝ √n, y la constante se ajusta midiendo.

## Pros y contras

| ✅ | ❌ |
|---|---|
| Muy pocas líneas de código | Peor caso: la lista **al revés** (5 871 con 500), porque todo lo que está arriba es «demasiado grande» |
| Excelente con 100 y 500 aleatorios | Hay que elegir `r` y justificarlo |
| Aprovecha el orden previo (1 063 casi ordenada) | |

> [!tip] El revés, por separado
> La adaptativa nunca le dará una lista al revés: tiene desorden 1, así que va al tramo alto. Pero `--medium` debe funcionar con cualquier entrada, y con 500 números al revés gasta 5 871, por encima de 5 500. Dentro de los límites del subject es aceptable, pero conviene saberlo.

## Preguntas de evaluación

> [!question] ¿De dónde sale tu `r`?
> «El coste es n²/r por las vueltas a `a` más n·r por la fase 2. Se minimiza con r del orden de √n. Probamos constantes y 1,5·√n fue la mejor.»

## Relacionadas

[[push_swap - Catálogo de algoritmos]] · [[push_swap - Chunks]] · [[push_swap - Partición por bloques]] · [[push_swap - Extracción de mínimo y máximo]] · [[Notación Big-O]]

## Referencias

- Subject de push_swap v1.0, VI.3.3 («Estrategias de orden basadas en rangos»).
- El nombre *butterfly* y la fórmula de `r` circulan en repositorios y guías de estudiantes de 42 (no es un algoritmo con nombre en la literatura). El análisis n²/r + n·r es propio.
- Cifras y traza: simulador propio (ver [[push_swap - Catálogo de algoritmos]]).
