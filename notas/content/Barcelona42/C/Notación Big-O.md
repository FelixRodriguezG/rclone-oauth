---
tags: [c, algoritmos, complejidad, big-o, push-swap]
aliases: ["Big-O", "Big O", "O grande", "Notación O grande", "Complejidad algorítmica"]
---
# Notación Big-O

[[Inicio|← Inicio]] · C · Algoritmos

> La **notación Big-O** describe **cómo crece el trabajo** de un algoritmo cuando crece la entrada. No dice cuántos segundos tarda: dice qué pasa con el trabajo si duplicas, o multiplicas por 10, el número de elementos. Es el lenguaje que usa [[push_swap - Índice|push_swap]] para clasificar sus estrategias: O(n²), O(n√n) y O(n log n).

## Vocabulario

| Término | Qué significa |
|---|---|
| **n** | Tamaño de la entrada: cuántos números hay que ordenar |
| **Coste** | Lo que cuentas: comparaciones, pasos de un bucle o, en push_swap, **operaciones impresas** |
| **Cota superior** | «Como mucho crece así». Es lo que dice O(…) |
| **Peor caso** | La entrada más desfavorable para el algoritmo |
| **Caso medio** | Lo esperable con una entrada aleatoria |
| **Constante** | Un factor fijo (×3, +5) que Big-O ignora |

## La idea en una frase

Se queda **solo el término que más crece** y se quitan las constantes:

| Coste exacto | Big-O | Por qué |
|---|---|---|
| `3n + 5` | O(n) | El `+5` y el `×3` no cambian la forma de crecer |
| `n²/2 + n` | O(n²) | Con n grande, `n²` se come a `n` |
| `2n·log₂ n + 4n` | O(n log n) | `n log n` crece más que `n` |
| `7` | O(1) | No depende de n |

## Las clases que vas a ver

| Notación | Si n se multiplica por 4, el trabajo… | Ejemplo |
|---|---|---|
| O(1) | no cambia | `sa`, leer `t[5]` |
| O(log n) | suma 2 pasos | Búsqueda binaria |
| O(n) | ×4 | [[ft_strlen]], recorrer una pila una vez |
| O(n log n) | algo más de ×4 | Merge sort, radix sobre índices |
| O(n√n) | ×8 | Ordenar por chunks en push_swap |
| O(n²) | ×16 | Dos bucles anidados sobre n, selection sort |

`log n` (base 2) responde a «¿cuántas veces puedo partir n por la mitad hasta llegar a 1?». `√n` es la raíz cuadrada. Comparadas:

```text
             n = 100     n = 500     n = 10 000
log₂ n            ~7          ~9           ~13
√n                10         ~22           100
n log n         ~664      ~4 483      ~132 877
n √n           1 000     ~11 180     1 000 000
n²            10 000     250 000   100 000 000
```

Fíjate en que O(n√n) queda **entre** O(n log n) y O(n²). Con 500 números, n√n ≈ 11 000 y n log n ≈ 4 500. Por eso push_swap las trata como clases distintas.

## Cómo calcularla mirando el código

> [!tip] Reglas rápidas
> - Un bucle que recorre `n` elementos: **O(n)**.
> - Un bucle dentro de otro, los dos sobre `n`: **O(n²)**.
> - Bucles uno **detrás** de otro: se suman, y gana el mayor (`O(n) + O(n²)` = O(n²)).
> - Un bucle que divide el problema por 2 en cada vuelta: **O(log n)**.
> - Repetir `k` veces algo que cuesta O(n): **O(k·n)**. Si `k` = log n, el resultado es O(n log n). Si `k` = √n, es O(n√n).

Ejemplo: el [[push_swap - Índice de desorden|índice de desorden]] compara todos los pares (i, j) con dos bucles anidados. Hace `n(n−1)/2` comparaciones, así que es O(n²) en **tiempo de CPU**.

## Lo que cambia en push_swap: contar operaciones, no tiempo

El subject lo dice explícitamente: la complejidad se mide en **número de operaciones de push_swap que el programa imprime** (`sa`, `pb`, `ra`…), no en cuántos cálculos hace tu código.

```text
Lo que cuesta CPU (da igual)          Lo que cuenta (se mide)
──────────────────────────────        ──────────────────────────
buscar el mínimo recorriendo a        cada "ra" / "rra" impreso
calcular el coste de cada número      cada "pb" / "pa" impreso
normalizar con dos bucles: O(n²)      cada "sa", "rr", "rrr"…
```

Por eso puedes calcular muchísimo (por ejemplo, el coste de mover cada elemento) y aun así tener un algoritmo «barato»: lo único que importa es cuántas líneas salen por stdout.

Cómo razonar la clase de una estrategia:

| Estrategia | Razonamiento | Clase |
|---|---|---|
| Selection sort con pilas | n veces: subir el mínimo (hasta n/2 rotaciones) + `pb` | O(n²) |
| Chunks (√n bloques) | Cada elemento cuesta del orden de √n rotaciones para encontrarlo y recolocarlo | O(n√n) |
| Radix binario | log₂ n pasadas × ~2n operaciones cada una | O(n log n) |

Cifras medidas con un simulador de las pilas (media de 40 entradas aleatorias de 100 y 12 de 500):

| n | Selection (O(n²)) | Chunks (O(n√n)) | Radix (O(n log n)) |
|---|---|---|---|
| 100 | ~1 456 | ~776 | 1 084 |
| 500 | ~32 377 | ~7 377 | 6 784 |

> [!warning] Big-O no es «el que menos operaciones da»
> Con 100 números, chunks (O(n√n)) da **menos** operaciones que radix (O(n log n)). Big-O habla de **cómo crece** el coste, no del valor exacto con un n concreto. Las constantes que Big-O esconde importan cuando n es pequeño. Al pasar de 100 a 500, selection multiplica su coste por ~22, chunks por ~9,5 y radix por ~6,3.

## O, Ω y Θ

Big-O es una **cota superior**. Hay dos primas que también aparecen en la defensa:

| Notación | Lee | Significado |
|---|---|---|
| O(f) | «como mucho del orden de f» | Cota superior |
| Ω(f) | «al menos del orden de f» | Cota inferior |
| Θ(f) | «exactamente del orden de f» | Las dos a la vez |

Radix sobre índices hace siempre ⌈log₂ n⌉ pasadas de n operaciones, así que es **Θ(n log n)**: ni mejor ni peor, sea como sea la entrada.

### El límite que nadie puede romper: Ω(n log n)

Hay `n!` formas de desordenar n números. Cada operación de push_swap es una de **11** posibles. Con `k` operaciones solo puedes escribir `11^k` programas distintos, y necesitas uno distinto para cada desorden:

```text
11^k ≥ n!   →   k ≥ log₁₁(n!) ≈ n·log₁₁ n
```

Conclusión: **ningún algoritmo de push_swap puede ordenar cualquier entrada en menos de Ω(n log n) operaciones en el peor caso.** Esto explica por qué el «O(n)» que pide el subject para desorden bajo solo es posible para entradas *casi* ordenadas de una forma concreta (ver [[push_swap - Estrategias#El tramo O(n) y su trampa]]).

## Peor caso y caso medio

Un algoritmo puede tener distinto coste según la entrada:

- Insertion sort: O(n) si ya está ordenado, O(n²) en el peor caso.
- Quicksort: O(n log n) de media, O(n²) si el pivote cae siempre en un extremo.

Cuando el subject pide «O(n log n)», lo seguro es dar una cota del **peor caso**. Si tu algoritmo solo la cumple en media, dilo así en el README.

## Espacio (memoria)

Big-O también mide memoria extra. En push_swap:

- Las dos pilas: O(n).
- Un array de índices normalizados: O(n).
- La tabla de pares del índice de desorden **no hace falta guardarla**: se cuenta sobre la marcha, O(1) extra.

El subject pide indicar **cotas de tiempo y espacio** de la estrategia adaptativa en el README.

## Errores frecuentes

> [!warning] Confundir tiempo de CPU con operaciones
> «Mi algoritmo es O(n²) porque calcula el coste de todos los elementos». En push_swap eso es CPU; lo que se clasifica es el número de operaciones impresas.

> [!warning] Llamar O(n log n) a algo que solo lo es en la práctica
> El «algoritmo turco» da muy pocas operaciones, pero en el peor caso teórico cada inserción puede costar ~n/2 rotaciones. Si lo presentas como O(n log n), prepara el argumento.

> [!warning] Olvidar que O(…) ignora constantes
> «O(n log n) siempre gana a O(n√n)» es falso con n = 100, como se vio en la tabla.

## Relacionadas

[[Algoritmos de ordenación y complejidad]] · [[push_swap - Estrategias]] · [[push_swap - Índice de desorden]] · [[Recursión]] · [[Operadores de bits]]

## Referencias

- Subject de push_swap v1.0 (`es.subject_push_swap.pdf`), capítulos I («Prólogo») y VI.3.1 («Modelo de complejidad y restricciones»).
- [Wikipedia: Big O notation](https://en.wikipedia.org/wiki/Big_O_notation) — definición formal de O, Ω y Θ.
- [Wikipedia: Comparison sort — Number of comparisons required](https://en.wikipedia.org/wiki/Comparison_sort#Number_of_comparisons_required_to_sort_a_list) — el argumento de `log(n!)` como cota inferior.
- Cifras de operaciones: simulador propio de las dos pilas (Python, `random.seed(42)`), contrastado con un prototipo en C compilado con Ubuntu clang 12.0.1 (radix: 1 084 y 6 784 en ambos).
