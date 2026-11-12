---
tags: [proyecto, push-swap, rank-1, algoritmos, indice]
aliases: ["Catálogo de algoritmos de push_swap", "Algoritmos de push_swap (catálogo)"]
---
# push_swap - Catálogo de algoritmos

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · [[push_swap - Estrategias|Estrategias]] · Catálogo

> Una nota por cada algoritmo que **nombra el subject** (sección VI.3.3), más tres que no nombra pero que merecen la pena para este ejercicio. Cada nota explica la idea original, cómo se adapta a dos pilas, una **traza real** paso a paso, por qué tiene la complejidad que tiene (en **operaciones** de push_swap) y cifras medidas.

> [!info] Cómo se han obtenido las cifras
> Todos los algoritmos se han implementado en un simulador de las dos pilas y se han comprobado con **todas** las permutaciones de 1 a 6 números y con entradas aleatorias de 60. Las cifras son la **media (peor caso)** de 20 entradas aleatorias de 100 números y 6 de 500 (`random.seed(42)`), y el resultado con la lista **al revés** y con una lista **casi ordenada** (500 números con 5 cambiados de sitio). Son de otra muestra que [[push_swap - Estrategias]], así que puede haber pequeñas diferencias.

## Los del subject

### O(n²): «algoritmo simple»

| Algoritmo | 100 | 500 | Al revés (500) | Casi ordenada (500) |
|---|---|---|---|---|
| [[push_swap - Selección\|Selección]] | 1 461 (1 592) | 32 338 (33 445) | 1 497 | 1 764 |
| [[push_swap - Inserción\|Inserción]] | 1 451 (1 590) | 32 703 (33 865) | 1 499 | 1 162 |
| [[push_swap - Burbuja\|Burbuja]] | 11 477 (12 721) | 299 205 (306 185) | 374 250 | — |
| [[push_swap - Extracción de mínimo y máximo\|Extracción de mínimo y máximo]] | 1 131 (1 215) | 22 291 (23 334) | 1 497 | 1 989 |

### O(n√n): «algoritmo intermedio»

Los cuatro ejemplos del subject son **variantes de una misma idea**: dividir los índices en unos √n grupos y moverlos grupo a grupo (en informática se llama *descomposición en raíz cuadrada*). Cada nota explica qué cambia.

| Algoritmo | 100 | 500 | Al revés (500) | Casi ordenada (500) |
|---|---|---|---|---|
| [[push_swap - Chunks\|Chunks]] | 778 (805) | 7 439 (7 493) | 1 975 | 1 970 |
| [[push_swap - Partición por bloques\|Partición por bloques]] | 650 (687) | 5 918 (6 010) | 1 760 | 2 134 |
| [[push_swap - Buckets\|Buckets]] | 806 (834) | 8 125 (8 246) | 1 515 | 1 514 |
| [[push_swap - Rangos (butterfly)\|Rangos (butterfly)]] | 584 (612) | 5 085 (5 160) | 5 871 | 1 063 |

### O(n log n): «algoritmo complejo»

| Algoritmo | 100 | 500 | Al revés (500) | Casi ordenada (500) |
|---|---|---|---|---|
| [[push_swap - Radix LSD\|Radix LSD]] | 1 084 (1 084) | 6 784 (6 784) | 6 784 | 6 784 |
| [[push_swap - Radix MSD\|Radix MSD]] | 761 (781) | 5 269 (5 311) | 5 766 | 2 271 |
| [[push_swap - Merge sort\|Merge sort]] | 1 717 (1 717) | 10 829 (10 829) | 10 829 | 10 829 |
| [[push_swap - Quicksort\|Quicksort]] | 721 (748) | 5 240 (5 275) | 5 726 | 2 284 |
| [[push_swap - Heap sort\|Heap sort]] | = selección | = selección | | |
| [[push_swap - Árbol binario indexado\|Árbol binario indexado]] | No ordena: sirve para **calcular** | | | |

> [!warning] Heap sort y el árbol binario indexado no encajan como estrategia
> Los dos son estructuras de datos que viven en la memoria del programa. Su O(n log n) es de **comparaciones en CPU**, no de operaciones de push_swap. Sus notas explican por qué, y para qué sí sirven en el proyecto. Si los presentáis como estrategia «compleja», el razonamiento no se sostiene.

## Los que no nombra el subject y merecen la pena

| Algoritmo | 100 | 500 | Al revés (500) | Casi ordenada (500) | Por qué |
|---|---|---|---|---|---|
| [[push_swap - Algoritmo de coste (turco)\|Coste («turco»)]] | 560 (604) | 5 068 (5 348) | 1 496 | 1 006 | El más popular en 42 para la nota máxima |
| [[push_swap - LIS\|LIS + inserción por coste]] | 547 (582) | 4 950 (5 131) | 1 497 | **909** | El mejor de todos en estas pruebas, e ideal para desorden bajo |
| [[push_swap - Búsqueda óptima (BFS)\|Búsqueda óptima (BFS)]] | — | — | — | — | El mínimo **exacto** con 3–6 números |
| *Casos pequeños a mano* | | | | | En [[push_swap - Estrategias#Atajos antes de elegir]] |

## Límites de rendimiento

| n | Mínimo | Bueno | Excelente |
|---|---|---|---|
| 100 | < 2000 | < 1500 | < 700 |
| 500 | < 12000 | < 8000 | < 5500 |

```text
Media con 500 números aleatorios (█ = 500 operaciones; burbuja y O(n²) fuera de escala)
LIS               ██████████                4 950  ✅ excelente
Coste             ██████████▏               5 068  ✅ excelente
Rangos            ██████████▏               5 085  ✅ excelente
Quicksort         ██████████▍               5 240  ✅ excelente
Radix MSD         ██████████▌               5 269  ✅ excelente
Bloques           ███████████▊              5 918  bueno
Radix LSD         █████████████▌            6 784  bueno
Chunks            ██████████████▉           7 439  bueno
Buckets           ████████████████▎         8 125  mínimo
Merge sort        █████████████████████▋   10 829  mínimo
                                   5 500 ─┘ (excelente)
```

## Qué combinación elegir

> [!summary] Recomendación
> - **`--simple`**: [[push_swap - Inserción|inserción]] o [[push_swap - Selección|selección]]. Son fáciles de explicar, y la inserción es buena con listas casi ordenadas.
> - **`--medium`**: [[push_swap - Rangos (butterfly)|rangos]] (excelente con 100 y 500) o [[push_swap - Partición por bloques|bloques]]. [[push_swap - Chunks|Chunks]] es lo más sencillo de entender.
> - **`--complex`**: [[push_swap - Quicksort|quicksort con mediana]]. Su O(n log n) en el peor caso **se demuestra**, y da nivel excelente con 500. [[push_swap - Radix LSD|Radix LSD]] es aún más fácil de demostrar, pero da más operaciones.
> - **Adaptativa**: [[push_swap - LIS|LIS]] en el tramo bajo; en los otros dos, los de arriba.
>
> Recuerda que una entrada aleatoria tiene un desorden ≈ 0,5: el tramo medio y el alto se usan los dos en las pruebas de rendimiento (ver [[push_swap - Índice de desorden]]).

## Plantilla de cada nota

Todas siguen el mismo orden, para poder compararlas:

1. **Ficha**: clase, memoria, cifras y dónde encaja.
2. **La idea original** sobre un array.
3. **Adaptación a dos pilas** en pseudocódigo.
4. **Traza real** generada con el simulador.
5. **Complejidad**: el razonamiento para el README y la defensa.
6. **Variantes**, **pros y contras**, **errores frecuentes** y **preguntas de evaluación**.

## Relacionadas

[[push_swap - Estrategias]] · [[push_swap - Índice]] · [[Notación Big-O]] · [[Algoritmos de ordenación y complejidad]] · [[push_swap - Defensa y evaluación]]

## Referencias

- Subject de push_swap v1.0, VI.3.3 («Estrategias requeridas») y VI.6 («Prueba de rendimiento»).
- Cifras y trazas: simulador propio de las dos pilas en Python. Todos los algoritmos se verificaron con todas las permutaciones de 1 a 6 elementos; selección y radix LSD, además, con un prototipo en C (Ubuntu clang 12.0.1).
