---
tags: [proyecto, push-swap, rank-1, algoritmos, extra, busqueda, recursion]
aliases: ["BFS en push_swap", "Búsqueda óptima", "IDDFS en push_swap", "Solución óptima para pocos números"]
---
# push_swap - Búsqueda óptima (BFS)

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · [[push_swap - Catálogo de algoritmos|Algoritmos]] · Extra

> **No lo nombra el subject.** Con pocos números (hasta 5 o 6), en lugar de inventar reglas, se pueden **probar todas las secuencias de operaciones**, de las más cortas a las más largas, hasta encontrar una que ordene. La primera que funcione es la **óptima**: nadie puede hacerlo con menos operaciones. Es una **búsqueda en anchura** (*Breadth-First Search*, BFS) sobre los estados de las dos pilas.

## Ficha

| | |
|---|---|
| Familia | No está en el subject. Búsqueda exhaustiva |
| Operaciones | **Las mínimas posibles** |
| Coste en CPU | Exponencial: solo sirve para n pequeño |
| Uso recomendado | Casos de 3 a 5 (o 6) números, y como caso base de algoritmos recursivos |

## Lo que se gana

Peor caso exacto, calculado recorriendo **todos** los estados:

| n | Estados posibles | Óptimo (peor caso) | Óptimo (media) | Método «sacar mínimos» |
|---|---|---|---|---|
| 3 | 24 | **2** | 1,17 | 2 |
| 4 | 120 | **5** | 2,88 | 6 |
| 5 | 720 | **8** | 4,86 | 10 |
| 6 | 5 040 | **10** | 7,06 | 15 |

Con 5 números, el método habitual da 10 en el peor caso y el óptimo es 8. Si la hoja de evaluación pide «≤ 12» con 5, los dos pasan, pero el óptimo es indiscutible en la defensa.

Ejemplo, el peor caso de 5:

```text
3 2 1 0 4   →   sa pb rra sa pa rra rra sa   (8 operaciones)
```

## La idea

Un **estado** es el contenido de las dos pilas. Cada una de las 11 operaciones lleva de un estado a otro. Buscar la secuencia más corta es buscar el camino más corto en ese grafo:

```text
                       (2 1 3 | )
           sa ↙          pb ↓           ra ↘
   (1 2 3 | ) ✅    (1 3 | 2)      (1 3 2 | )
      distancia 1    distancia 1     distancia 1
```

**BFS** explora primero todos los estados a distancia 1, luego a distancia 2… El primero ordenado que encuentra está a la distancia mínima.

## En C: mejor IDDFS que BFS

BFS necesita una cola y guardar los estados visitados: muchos `malloc` y una estructura para buscar. Una alternativa más simple con la misma garantía de mínimo es **IDDFS** (*Iterative Deepening DFS*): una búsqueda en profundidad con un límite que va subiendo.

```text
para límite = 0, 1, 2, …:
    si dfs(estado_inicial, límite) encuentra solución: devolverla

dfs(estado, quedan):
    si estado ordenado y b vacía: éxito
    si quedan == 0: fracaso
    para cada operación op (11):
        si op deshace la anterior (ra tras rra, pa tras pb…): saltarla
        si op no hace nada (pa con b vacía, sa con < 2…):     saltarla
        aplicar op, guardarla en el camino
        si dfs(estado, quedan − 1): éxito
        deshacer op (con su inversa)
    fracaso
```

- Memoria: solo el camino actual (≤ 10 operaciones). Ni colas ni tablas.
- Es [[Recursión|recursiva]] y aplica/deshace operaciones sobre las mismas pilas, con las funciones que ya tienes.
- Las podas («no deshacer la anterior», «no hacer operaciones nulas») reducen muchísimo la búsqueda.

Nodos visitados en el peor caso, con esas podas (medido):

| n | Profundidad | Nodos |
|---|---|---|
| 5 | 8 | ~318 000: instantáneo en C |
| 6 | 10 | ~31 millones: menos de un segundo en C, pero ya se nota |

> [!tip] Dónde usarlo
> - Estrategia propia para **n ≤ 5** (antes de elegir algoritmo).
> - **Caso base** del [[push_swap - Quicksort|quicksort]] o del [[push_swap - Radix MSD|radix MSD]] cuando el segmento es la pila entera.

## Por qué no sirve para n grande

El número de estados es n!·(n + 1) (como en la tabla: 24, 120, 720, 5 040…), y el de secuencias que explora la búsqueda crece exponencialmente con la profundidad. Con 100 números, ni todos los ordenadores del mundo terminarían. Por eso existen los algoritmos de las otras notas: renuncian al óptimo a cambio de ser rápidos.

## Preguntas de evaluación

> [!question] ¿Cómo sabéis que vuestro caso de 5 es óptimo?
> «Porque lo calculamos con una búsqueda por profundidad iterativa: prueba todas las secuencias de longitud 0, luego 1, 2…, y para en la primera que ordena. Ninguna más corta existe porque ya se probaron todas.»

## Relacionadas

[[push_swap - Catálogo de algoritmos]] · [[push_swap - Estrategias#Atajos antes de elegir]] · [[push_swap - Quicksort]] · [[Recursión]] · [[Notación Big-O]]

## Referencias

- [Wikipedia: Breadth-first search](https://en.wikipedia.org/wiki/Breadth-first_search) e [Iterative deepening depth-first search](https://en.wikipedia.org/wiki/Iterative_deepening_depth-first_search).
- Tabla de óptimos: BFS propia sobre todos los estados alcanzables desde la pila ordenada (las 11 operaciones tienen inversa, así que el grafo es simétrico). Nodos de IDDFS: implementación propia en Python con las podas descritas.
