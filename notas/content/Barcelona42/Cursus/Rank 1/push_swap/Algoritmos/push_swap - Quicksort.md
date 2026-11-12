---
tags: [proyecto, push-swap, rank-1, algoritmos, o-n-log-n, recursion]
aliases: ["Quicksort en push_swap", "Quick sort con pilas"]
---
# push_swap - Quicksort

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · [[push_swap - Catálogo de algoritmos|Algoritmos]] · O(n log n)

> **Quicksort**: elige un **pivote**, pon a un lado los menores y al otro los mayores, y repite en cada lado. En push_swap, «a un lado» es `b`: los menores que el pivote se pasan con `pb` y los demás se rotan. Luego se ordena recursivamente cada mitad, una en `a` y otra en `b`. Con la **mediana** como pivote, cada partición divide justo por la mitad, y la complejidad O(n log n) se cumple **en el peor caso**.

## Ficha

| | |
|---|---|
| Familia | O(n log n): «adaptación del orden rápido con partición por stacks» del subject |
| Operaciones | O(n log n) en el peor caso **si el pivote es la mediana** |
| Memoria extra | O(log n) de recursión (más el array de la mediana) |
| 100 aleatorios | 721 (748) |
| 500 aleatorios | **5 240** (5 275): excelente |
| Al revés (500) | 5 726 |
| Casi ordenada (500) | 2 284 |
| Uso recomendado | **`--complex`**: buen número de operaciones y una demostración sólida |

## La idea original

```text
[5 1 4 2 8 0 3]  pivote 3 →  [1 2 0] 3 [5 4 8]  → repetir en cada lado
```

Con el pivote malo (siempre el menor o el mayor), un lado queda vacío y el otro con todo: O(n²). Con la mediana, los dos lados tienen la mitad y hay log₂ n niveles.

En push_swap, calcular la mediana **no cuesta operaciones**: copias el segmento a un array, lo ordenas en memoria (o usas los índices normalizados) y tomas el del medio.

## Adaptación a dos pilas

Dos funciones recursivas, que se llaman la una a la otra:

```text
ordenar_a(n):       ordena los n de arriba de a, de menor a mayor
    si n <= 3 o ya están ordenados: caso base (sa, tabla de 3…)
    p = mediana de esos n
    repetir hasta haber pasado todos los < p:
        top < p  →  pb          (a b)
        si no    →  ra          (se queda en a)
    si a tiene más de n elementos:  rra × (los ra hechos)    ← deshacer
    ordenar_a(los que quedaron en a)
    ordenar_b(los que se pasaron a b)

ordenar_b(n):       ordena los n de arriba de b y los coloca encima de a, en orden
    si n <= 2: caso base (sb si hace falta, pa, pa)
    p = mediana de esos n
    repetir hasta haber pasado todos los >= p:
        top >= p →  pa          (a a)
        si no    →  rb
    si b tiene más de n elementos:  rrb × (los rb hechos)
    ordenar_a(los que se pasaron a a)
    ordenar_b(los que quedaron en b)
```

Por qué cuadra: todos los que `ordenar_b` sube a `a` son **menores** que lo que ya hay debajo en `a` (la mitad «alta» se ordenó antes), y **mayores** que lo que queda en `b`. Así, cada bloque se coloca encima del anterior en su sitio.

> [!tip] Dos ahorros importantes
> - **Parar la partición en cuanto has movido todos los del lado que toca**: no hace falta recorrer el resto del segmento.
> - **No deshacer las rotaciones si el segmento es toda la pila**: rotar una pila entera no cambia nada.

## Traza real

Índices `2 4 1 3 0`, mediana 2:

```text
ini   a: 2 4 1 3 0        b:
ra    a: 4 1 3 0 2        b:            2 ≥ 2 → se queda
ra    a: 1 3 0 2 4        b:            4 ≥ 2
pb    a: 3 0 2 4          b: 1          1 < 2 → b
ra    a: 0 2 4 3          b: 1
pb    a: 2 4 3            b: 0 1        ya están los dos menores: fin de la partición
                                        (el segmento era toda la pila: no hay que deshacer)
rra   a: 3 2 4            b: 0 1        ordenar_a(3): caso de 3 números
sa    a: 2 3 4            b: 0 1
sb    a: 2 3 4            b: 1 0        ordenar_b(2): el mayor arriba
pa    a: 1 2 3 4          b: 0
pa    a: 0 1 2 3 4        b:
total 10
```

## Complejidad

Con la mediana, cada llamada parte su segmento en dos mitades, así que la recursión tiene **⌈log₂ n⌉ niveles**. En cada nivel, cada elemento recibe como mucho:

- 1 operación en la partición (`pb`/`ra` o `pa`/`rb`),
- 1 operación para deshacer la rotación (`rra`/`rrb`), si la hubo.

```text
operaciones ≤ 2 · n · ⌈log₂ n⌉ + O(n) (casos base)  =  O(n log n)    ← en el PEOR caso
memoria extra: O(log n) de pila de llamadas
```

Medido: 721 con 100 (n·log₂ n ≈ 664) y 5 240 con 500 (≈ 4 483). Gracias a los dos ahorros, el coste real queda en ~1,1–1,2 · n log₂ n.

> [!important] Sin mediana no hay garantía
> Con el top como pivote, una lista ya ordenada o al revés produce particiones de tamaño 1 y n−1: O(n²) operaciones. Si en el README dices O(n log n), **tienes que usar la mediana** (o explicar por qué tu pivote garantiza particiones equilibradas).

## Variantes

| Variante | Idea |
|---|---|
| Pivote por bits | Es el [[push_swap - Radix MSD\|radix MSD]]: con índices normalizados, casi lo mismo |
| Dos pivotes (tres vías) | Tercios: los menores al fondo de `b` (`pb` + `rb`), los medios arriba de `b` y los mayores en `a`. Menos niveles (log₃ n) |
| Caso base mayor | Con ≤ 5 elementos, resolver a mano o con [[push_swap - Búsqueda óptima (BFS)\|búsqueda óptima]] |

## Errores frecuentes

> [!danger] Deshacer rotaciones de más o de menos
> Si cuentas mal los `ra`, el segmento no vuelve arriba y la recursión ordena los elementos equivocados: KO difícil de depurar. Prueba con todas las permutaciones de 4–6 números.

> [!warning] Recursión y la Norma
> La recursión es válida, pero cada función debe caber en 25 líneas y 4 parámetros: agrupa los datos en un struct (ver [[Recursión]] y [[Structs y typedef]]).

## Preguntas de evaluación

> [!question] ¿Cuánto cuesta calcular la mediana?
> «En operaciones de push_swap, nada: se calcula en memoria. En CPU, O(k log k) o O(k²) para un segmento de k elementos, y eso no se mide.»

> [!question] ¿Por qué es O(n log n) en el peor caso y no solo de media?
> «Porque la mediana garantiza que cada partición deja como mucho la mitad en cada lado, y así hay log₂ n niveles. En cada nivel, cada elemento cuesta como mucho 2 operaciones.»

## Relacionadas

[[push_swap - Catálogo de algoritmos]] · [[push_swap - Radix MSD]] · [[push_swap - Merge sort]] · [[Recursión]] · [[Algoritmos de ordenación y complejidad#Quicksort — O(n log n) de media, O(n²) en el peor caso]] · [[Notación Big-O]]

## Referencias

- [Wikipedia: Quicksort](https://en.wikipedia.org/wiki/Quicksort) — elección de pivote y peor caso.
- Subject de push_swap v1.0, VI.3.3 («Adaptación del orden rápido con partición por stacks»).
- Cifras y traza: simulador propio (ver [[push_swap - Catálogo de algoritmos]]).
