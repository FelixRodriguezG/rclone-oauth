---
tags: [proyecto, a-maze-ing, indice, rank-2, python, algoritmos, laberintos]
aliases: ["A-Maze-ing", "a_maze_ing", "amazing"]
---
# A-Maze-ing - Índice

[[Inicio|← Inicio]] · Proyectos · Rank 2

> **A-Maze-ing** es el primer proyecto completo en Python del nuevo Common Core: un **generador de laberintos**. Lee un fichero de configuración, genera un laberinto (aleatorio pero reproducible con una semilla), calcula el camino más corto entre la entrada y la salida, lo guarda en un fichero de texto con los muros codificados en **hexadecimal** y lo muestra por pantalla. La lógica de generación se entrega además como un **paquete de Python instalable** con `pip`.

> [!warning] Comprueba el subject
> Resumen basado en versiones públicas del subject (README de repositorios de alumnos que ya lo han entregado); los detalles cambian entre versiones. Lo que manda es el PDF de tu intra.

Antes de empezar conviene haber hecho los [[Python - Índice|módulos de Python]], sobre todo el 01 (clases), el 02 (excepciones), el 06 (paquetes) y el 08 (venv y `pyproject.toml`).

## Qué se pide

Lo que coincide en los README públicos consultados:

| Aspecto | Lo habitual |
|---|---|
| Lenguaje | Python 3 (uno de los repositorios indica 3.10 o superior) |
| Ejecución | `python3 a_maze_ing.py config.txt` |
| Entrada | Fichero de configuración `CLAVE=VALOR`, una por línea; las líneas con `#` se ignoran |
| Generación | Laberinto **perfecto** (un único camino entre dos celdas cualesquiera) y opción de **imperfecto** (con bucles) |
| Semilla | `SEED` opcional: con la misma semilla sale el mismo laberinto |
| Patrón «42» | Un «42» dibujado con celdas cerradas en el centro del laberinto |
| Resolución | Camino **más corto** de la entrada a la salida |
| Salida | Fichero de texto: muros en hexadecimal + entrada + salida + camino |
| Visualización | En terminal (ASCII o `curses`) o en ventana; con opciones como regenerar y mostrar/ocultar el camino |
| Paquete | Módulo reutilizable (`mazegen`, clase `MazeGenerator`) empaquetado como `.whl` e instalable con `pip` |
| Calidad | flake8 y mypy sin avisos |
| `Makefile` | Reglas como `install`, `run`, `debug`, `clean`, `fclean`, `lint`, `lint-strict` |
| README | Explicar algoritmos elegidos, uso, formato del config y reparto de tareas (se hace en equipo) |

### El fichero de configuración

| Clave | Obligatoria | Ejemplo | Significado |
|---|---|---|---|
| `WIDTH` | Sí | `WIDTH=20` | Ancho en celdas |
| `HEIGHT` | Sí | `HEIGHT=15` | Alto en celdas |
| `ENTRY` | Sí | `ENTRY=0,0` | Celda de entrada `x,y` |
| `EXIT` | Sí | `EXIT=19,14` | Celda de salida `x,y` |
| `OUTPUT_FILE` | Sí | `OUTPUT_FILE=maze.txt` | Fichero donde se escribe el resultado |
| `PERFECT` | Sí | `PERFECT=True` | Perfecto o con bucles |
| `SEED` | No | `SEED=42` | Semilla para reproducir el laberinto |

Muchos equipos añaden claves propias (algoritmo, animación, colores…). Lo que **no** debe pasar nunca es un traceback ante un config mal escrito: valida cada clave y da un mensaje claro.

> [!warning] Validaciones del config
> Clave obligatoria ausente o repetida, valor no numérico, ancho o alto demasiado pequeños, entrada o salida fuera del laberinto, entrada igual a salida, entrada o salida dentro del patrón «42», fichero inexistente. Cada caso: mensaje de error y salida limpia.

### El fichero de salida

```text
D5393
93C6A
EC556

0,0
4,2
EESENESS
```

1. Una línea por fila del laberinto, **un dígito hexadecimal por celda** (qué muros tiene cerrados).
2. Una línea vacía.
3. Coordenadas de entrada, coordenadas de salida y el camino más corto como letras `N`, `E`, `S`, `W`.

Es la salida real del código de ejemplo de esta nota con un laberinto de 5 × 3 y `SEED=42`.

## Vocabulario

| Término | Significado |
|---|---|
| **Celda** | Cada casilla de la cuadrícula. Tiene hasta 4 muros: norte, este, sur, oeste |
| **Laberinto perfecto** | Todas las celdas conectadas y **un único** camino entre dos cualesquiera. Sin bucles ni zonas aisladas. En teoría de grafos, un *árbol de expansión* |
| **Laberinto imperfecto** | Tiene bucles: entre dos celdas puede haber varios caminos |
| **Callejón sin salida** (*dead end*) | Celda con un solo muro abierto |
| **Grafo** | Puntos (celdas) unidos por aristas (pasos abiertos entre celdas vecinas) |
| **Semilla** (*seed*) | Número que fija la secuencia de `random`: misma semilla, mismos «aleatorios» |
| **BFS** | Búsqueda en anchura: explora por capas de distancia; encuentra el camino más corto |
| **Wheel** (`.whl`) | Formato de paquete de Python listo para instalar con `pip install fichero.whl` |

## Representación en memoria: un número por celda

Cada celda se guarda como un entero de 4 bits, **un bit por muro**. Bit a `1` = muro cerrado.

| Bit | Valor | Muro |
|---|---|---|
| 0 | `1` | Norte |
| 1 | `2` | Este |
| 2 | `4` | Sur |
| 3 | `8` | Oeste |

```text
Celda 0xD = 1101 en binario
            ││││
            │││└─ bit 0 (N) = 1 → cerrado
            ││└── bit 1 (E) = 0 → abierto
            │└─── bit 2 (S) = 1 → cerrado
            └──── bit 3 (W) = 1 → cerrado

0xF = celda con los 4 muros (así empieza todo)   0x0 = sin muros
```

Las operaciones son las mismas que en C (ver [[Operadores de bits]]):

| Operación | Python | Qué hace |
|---|---|---|
| ¿Está cerrado el muro este? | `cell & E` | Distinto de 0 si el bit está a 1 |
| Abrir el muro este | `cell &= ~E` | Pone el bit a 0 |
| Cerrar el muro este | `cell \|= E` | Pone el bit a 1 |
| Dígito para el fichero | `f"{cell:X}"` | `13` → `"D"` |

> [!danger] Los muros se comparten
> El muro este de una celda **es** el muro oeste de su vecina de la derecha. Al abrir un paso tienes que tocar **las dos** celdas. Si solo abres una, el fichero hexadecimal es incoherente y un evaluador lo detecta enseguida.

```text
+---+---+---+---+---+
|           |       |      Fila 0:  D 5 3 9 3
+---+---+   +   +   +
|       |       |   |      Fila 1:  9 3 C 6 A
+   +   +---+---+   +
|   |               |      Fila 2:  E C 5 5 6
+---+---+---+---+---+
```

Ese mismo laberinto, dibujado a partir de los bits.

## Algoritmos de generación

Los tres generan laberintos **perfectos**: vistos como grafo, construyen un árbol de expansión de la cuadrícula.

| Algoritmo | Idea | Aspecto del laberinto | Estructura que necesitas |
|---|---|---|---|
| **DFS / backtracking recursivo** | Avanza a un vecino no visitado al azar; si no quedan, retrocede | Pasillos largos y sinuosos, pocos callejones | Pila (o [[Recursión]]) + conjunto de visitados |
| **Prim (aleatorio)** | Crece desde una celda: elige al azar una pared de la «frontera» que lleve a una celda nueva | Muchos callejones cortos, ramificado | Lista de paredes frontera |
| **Kruskal (aleatorio)** | Recorre todas las paredes en orden aleatorio; abre una si separa dos zonas aún no conectadas | Muchos callejones cortos, uniforme | *Union-find* (conjuntos disjuntos) |

### DFS iterativo, paso a paso

1. Todas las celdas empiezan con los 4 muros (`0xF`). Marca la inicial como visitada y apílala.
2. Mira la celda de la cima de la pila. Busca vecinos **no visitados**.
3. Si hay: elige uno al azar, abre el muro entre los dos, márcalo visitado y apílalo.
4. Si no hay: desapila (*backtracking*).
5. Termina cuando la pila está vacía: todas las celdas se han visitado una vez.

```python
import random

N, E, S, W = 1, 2, 4, 8
DIRS = {N: (0, -1, S), E: (1, 0, W), S: (0, 1, N), W: (-1, 0, E)}


def generate(width: int, height: int, seed: int) -> list[list[int]]:
    """Laberinto perfecto con DFS iterativo (backtracking con pila)."""
    rng = random.Random(seed)
    grid = [[N | E | S | W] * width for _ in range(height)]
    visited = {(0, 0)}
    stack = [(0, 0)]
    while stack:
        x, y = stack[-1]
        options = []
        for wall, (dx, dy, opposite) in DIRS.items():
            nx, ny = x + dx, y + dy
            if 0 <= nx < width and 0 <= ny < height \
                    and (nx, ny) not in visited:
                options.append((wall, nx, ny, opposite))
        if not options:
            stack.pop()
            continue
        wall, nx, ny, opposite = rng.choice(options)
        grid[y][x] &= ~wall
        grid[ny][nx] &= ~opposite
        visited.add((nx, ny))
        stack.append((nx, ny))
    return grid
```

> [!tip] Iterativo mejor que recursivo
> La versión recursiva es más corta, pero Python limita la profundidad de recursión (por defecto unas 1000 llamadas): un laberinto de 50 × 50 tiene 2500 celdas y la versión recursiva puede lanzar `RecursionError`. Con una pila explícita no hay límite.

> [!note] `random.Random(seed)` en vez de `random.seed()`
> Un generador propio con su semilla hace el resultado reproducible aunque otra parte del programa use `random`. Sin `SEED` en el config, crea `random.Random()` sin argumento.

### Prim y Kruskal en una frase

- **Prim**: empieza con una celda dentro; mete sus paredes en una lista; saca una pared al azar; si al otro lado hay una celda que aún no está dentro, ábrela y mete las paredes de esa celda. Repite hasta vaciar la lista.
- **Kruskal**: cada celda es su propio conjunto; baraja todas las paredes interiores; para cada pared, si las dos celdas están en conjuntos distintos, ábrela y une los conjuntos. Necesitas *union-find* para preguntar «¿están ya conectadas?» rápido.

### Laberinto imperfecto

Lo habitual: generar uno perfecto y después **abrir paredes extra** (por ejemplo, en callejones sin salida) para crear bucles. Cuidado con no abrir zonas grandes vacías ni romper el patrón «42».

### El patrón «42»

Marca antes de generar las celdas que forman el «42» como **bloqueadas** (siempre `0xF`, nunca visitables). El algoritmo las trata como si estuvieran fuera de la cuadrícula. Si el laberinto es demasiado pequeño para dibujarlo, avisa en vez de fallar.

> [!warning] Celdas bloqueadas y conectividad
> Si bloqueas celdas, el laberinto sigue teniendo que ser conexo: comprueba con un BFS desde la entrada que llegas a **todas** las celdas no bloqueadas.

## Resolución: BFS

En un laberinto perfecto hay un solo camino, pero en uno imperfecto hay varios y te piden el **más corto**. BFS lo encuentra siempre en una cuadrícula donde cada paso cuesta lo mismo.

1. Mete la entrada en una cola (`collections.deque`).
2. Saca la primera celda; para cada muro **abierto**, si el vecino no se ha visto, apúntalo como visto, guarda de dónde vienes y mételo en la cola.
3. Al llegar a la salida, reconstruye el camino hacia atrás siguiendo «de dónde vengo».

```python
from collections import deque

LETTER = {N: "N", E: "E", S: "S", W: "W"}


def solve(grid: list[list[int]], start: tuple[int, int],
          end: tuple[int, int]) -> str:
    """Camino más corto con BFS, como letras N/E/S/W."""
    came: dict[tuple[int, int], tuple[tuple[int, int], int]] = {}
    queue = deque([start])
    seen = {start}
    while queue:
        x, y = queue.popleft()
        if (x, y) == end:
            break
        for wall, (dx, dy, _) in DIRS.items():
            nxt = (x + dx, y + dy)
            if not grid[y][x] & wall and nxt not in seen:
                seen.add(nxt)
                came[nxt] = ((x, y), wall)
                queue.append(nxt)
    path = []
    cur = end
    while cur != start:
        cur, wall = came[cur]
        path.append(LETTER[wall])
    return "".join(reversed(path))
```

> [!question] ¿Por qué BFS y no DFS para resolver?
> DFS encuentra **un** camino, no necesariamente el más corto. BFS visita las celdas por orden de distancia a la entrada, así que la primera vez que llega a la salida lo hace por el camino mínimo.

## Plan por pasos

### Paso 1: entorno y esqueleto

Repositorio, `.gitignore` (`.venv/`, `__pycache__/`), venv, `Makefile` con `install`, `run`, `lint` y `clean`. flake8 y mypy pasando desde el primer commit.

### Paso 2: parseo del config

Una función que devuelve un objeto con los valores ya convertidos y validados, o lanza una excepción propia con un mensaje claro. Prueba todos los casos de error antes de seguir.

### Paso 3: la cuadrícula y los bits

Clase del laberinto con la cuadrícula de enteros, abrir muro (en las dos celdas) y escribir el fichero hexadecimal. Comprueba a mano un 2 × 2.

### Paso 4: generación perfecta con semilla

DFS iterativo. Comprueba que con la misma semilla sale el mismo fichero y que todas las celdas son alcanzables.

### Paso 5: resolución

BFS y escritura del camino en el fichero. Verifica el camino recorriéndolo sobre los bits.

### Paso 6: visualización

ASCII primero (`+---+` y `|`), después `curses` u otra interfaz si quieres: regenerar, mostrar camino, colores.

### Paso 7: patrón «42» e imperfecto

Celdas bloqueadas y apertura de paredes extra.

### Paso 8: paquete

Mueve la lógica a `src/mazegen/` con su `__init__.py`, un `pyproject.toml`, y genera el `.whl`. Pruébalo instalándolo en un **venv nuevo**.

### Paso 9: README y bonus

Algoritmos (y por qué los elegiste), uso, formato del config, reparto del trabajo. Bonus habituales en los repositorios públicos: animación de la generación, varios algoritmos a elegir, más patrones, colores.

## Errores frecuentes

> [!danger] Lo que suele fallar en evaluación
> - Abrir el muro solo en una de las dos celdas: fichero incoherente.
> - Traceback con un config mal escrito en vez de un mensaje de error.
> - Misma semilla, laberintos distintos (por usar `random` global en varios sitios o iterar un `set` para elegir).
> - `RecursionError` con laberintos grandes por usar DFS recursivo.
> - Entrada o salida dentro del «42» o en una celda inalcanzable.
> - El camino del fichero no coincide con los muros (atraviesa una pared).
> - flake8 o mypy con avisos; funciones sin anotar.
> - El `.whl` solo funciona desde la carpeta del proyecto (imports relativos mal resueltos).
> - Subir `.venv/` o `__pycache__/` al repositorio.

## Preguntas de evaluación

> [!question] ¿Qué es un laberinto perfecto y cómo demuestras que el tuyo lo es?
> Uno en el que entre dos celdas cualesquiera hay exactamente un camino. Con C celdas libres (sin contar las del «42») tiene exactamente C − 1 pasos abiertos (un árbol) y todas son alcanzables desde la entrada (compruébalo con un BFS).

> [!question] ¿Cómo se codifica una celda en el fichero?
> 4 bits, uno por muro (N = 1, E = 2, S = 4, W = 8), bit a 1 = cerrado. Se escribe como un dígito hexadecimal: `0xF` es una celda cerrada.

> [!question] Explica tu algoritmo de generación
> Ten preparado un dibujo de 3 × 3 y recórrelo en voz alta: pila, visitados, elección aleatoria, *backtracking*.

> [!question] ¿Por qué con la misma semilla sale el mismo laberinto?
> Porque el generador pseudoaleatorio produce siempre la misma secuencia a partir de la misma semilla, y el algoritmo es determinista salvo por esas elecciones.

> [!question] ¿Cómo instalo tu paquete en otro proyecto?
> `python3 -m pip install mazegen-<versión>-py3-none-any.whl` dentro de un venv, y `from mazegen import MazeGenerator`.

## Relacionadas

[[Python - Índice]] · [[Operadores de bits]] · [[Recursión]] · [[push_swap - Índice]] · [[Algoritmos de ordenación y complejidad]] · [[Git para 42]] · [[Exam Rank 02]]

## Referencias

- [martinnsanzz/A-Maze-Ing](https://github.com/martinnsanzz/A-Maze-Ing) — requisitos del proyecto, claves obligatorias del config, reglas del `Makefile`, paquete `mazegen` en `.whl`, DFS iterativo con pila y Kruskal.
- [creespie/maze](https://github.com/creespie/maze) — codificación de bits (bit 0 = N … bit 3 = W), formato del fichero de salida, Python 3.10+, flake8 y mypy estricto.
- [twennyfour/a_maze_ing (Codeberg)](https://codeberg.org/twennyfour/a_maze_ing) — confirma N = 1, E = 2, S = 4, W = 8, el formato de salida en tres bloques y las reglas del `Makefile`.
- [Diogo-Serra/42_School](https://github.com/Diogo-Serra/42_School) — A-Maze-ing en el tronco común del nuevo cursus.
- [Maze generation algorithm — Wikipedia](https://en.wikipedia.org/wiki/Maze_generation_algorithm) — DFS aleatorio, Prim y Kruskal aleatorios.
- [Breadth-first search — Wikipedia](https://en.wikipedia.org/wiki/Breadth-first_search), [Kruskal's algorithm](https://en.wikipedia.org/wiki/Kruskal%27s_algorithm), [Prim's algorithm](https://en.wikipedia.org/wiki/Prim%27s_algorithm) y [Disjoint-set data structure](https://en.wikipedia.org/wiki/Disjoint-set_data_structure) — teoría de los algoritmos.
- Documentación de Python (es): [random](https://docs.python.org/es/3/library/random.html) (semillas, `random.Random`), [collections](https://docs.python.org/es/3/library/collections.html) (`deque`), [curses](https://docs.python.org/es/3/library/curses.html).
- [Packaging Python Projects](https://packaging.python.org/en/latest/tutorials/packaging-projects/) — `pyproject.toml` y generar un wheel.
- [flake8](https://flake8.pycqa.org/en/latest/) y [mypy](https://mypy.readthedocs.io/en/stable/).
- El código de ejemplo se ha ejecutado con Python 3.13.1 y pasa `flake8` y `mypy --strict`; la salida y el dibujo de 5 × 3 son reales (`SEED=42`).
