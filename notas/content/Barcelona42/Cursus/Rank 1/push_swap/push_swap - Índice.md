---
tags: [proyecto, push-swap, indice, rank-1, algoritmos]
aliases: ["push_swap"]
---
# push_swap - Índice

[[Inicio|← Inicio]] · Proyectos · Rank 1

> **push_swap** ordena una lista de enteros usando dos pilas (`a` y `b`) y 11 operaciones. El programa no imprime los números ordenados: imprime **la lista de operaciones** que los ordena, y cuantas menos, mejor. En la versión nueva del subject es un proyecto **en pareja**, con **cuatro estrategias** (O(n²), O(n√n), O(n log n) y una adaptativa), un **índice de desorden** y un modo `--bench` opcional.

> [!success] Verificado contra el subject v1.0
> Esta nota está contrastada con `es.subject_push_swap.pdf` (versión 1.0, la del cursus 2025). Si en tu intra aparece una versión más nueva, compara los cambios: lo que manda es el PDF. Muchos tutoriales de internet son de la versión **antigua** (sin estrategias ni desorden): úsalos solo para la parte de algoritmos.

> [!tip] ¿Primera vez con push_swap? Sigue este orden
> 1. Esta nota: qué se pide.
> 2. [[push_swap - Índice de desorden]]: el número que decide la estrategia.
> 3. [[push_swap - Estrategias]]: los cuatro algoritmos, con cifras medidas y la justificación de su complejidad.
>    - [[push_swap - Catálogo de algoritmos]]: una nota por cada algoritmo que nombra el subject, más coste, LIS y búsqueda óptima.
> 4. [[push_swap - Guía paso a paso]]: arquitectura y construcción por etapas.
> 5. [[push_swap - Pruebas y depuración]]: scripts, casos de error, Valgrind y el checker.
> 6. [[push_swap - checker (bonus)]].
> 7. [[push_swap - Defensa y evaluación]]: preguntas, modificación en directo y README.

## Qué se pide

| Aspecto | Subject v1.0 |
|---|---|
| Equipo | **2 personas**. Las dos deben entender y saber explicar **todo** el código |
| Programa | `push_swap` |
| Entrega | `Makefile`, `*.h`, `*.c` |
| Makefile | `NAME`, `all`, `clean`, `fclean`, `re` (y `bonus`), con `cc -Wall -Wextra -Werror`, sin relink |
| Funciones autorizadas | `read`, `write`, `malloc`, `free`, `exit`, `ft_printf` (o equivalente propio) |
| libft | Permitida (copiada en una carpeta `libft/` con su Makefile) |
| Prohibido | Variables globales |
| Argumentos | Los números de `a` (el primero es el **top**) + un selector de estrategia opcional |
| Salida | Operaciones separadas por `\n`, nada más, dejando el más pequeño arriba |
| Error | `Error\n` en **stderr** |
| Sin argumentos | No imprime nada |
| README | Obligatorio, con las contribuciones de cada persona y la justificación de los algoritmos |

## Las dos pilas y las 11 operaciones

Al empezar, `a` tiene los números en el orden de los argumentos y `b` está vacía. Objetivo: `a` ordenada de menor (arriba) a mayor, `b` vacía.

```text
./push_swap 2 1 3 6 5 8

 a     b
 2          ← top
 1
 3
 6
 5
 8
---   ---
```

| Op | Qué hace | Si no se puede |
|---|---|---|
| `sa` | Intercambia los dos primeros de `a` | No hace nada (0 o 1 elementos) |
| `sb` | Intercambia los dos primeros de `b` | No hace nada |
| `ss` | `sa` y `sb` a la vez | — |
| `pa` | Quita el top de `b` y lo pone encima de `a` | No hace nada si `b` está vacía |
| `pb` | Quita el top de `a` y lo pone encima de `b` | No hace nada si `a` está vacía |
| `ra` | Rota `a` hacia arriba: el primero pasa a ser el último | — |
| `rb` | Igual con `b` | — |
| `rr` | `ra` y `rb` a la vez | — |
| `rra` | Rota `a` hacia abajo: el último pasa a ser el primero | — |
| `rrb` | Igual con `b` | — |
| `rrr` | `rra` y `rrb` a la vez | — |

```text
Partiendo de a = 2 1 3 6 5 8 (top a la izquierda), b vacía:

sa   →  a = 1 2 3 6 5 8               b = (vacía)
pb   →  a = 2 3 6 5 8                 b = 1
ra   →  a = 3 6 5 8 2                 b = 1
rra  →  a = 2 3 6 5 8                 b = 1
```

> [!tip] Las dobles valen una
> `ss`, `rr` y `rrr` cuentan como **una** operación. Si en un paso necesitas `ra` y `rb`, imprime `rr`.

Con las rotaciones, cada pila es en realidad un **anillo**: puedes subir cualquier elemento con `min(pos, tamaño − pos)` rotaciones (ver [[Pilas y colas]]). Casi todos los algoritmos buenos consisten en **rotar lo menos posible**.

## Uso y flags

```text
./push_swap [--simple | --medium | --complex | --adaptive] [--bench] números…
```

| Flag | Estrategia | Clase |
|---|---|---|
| `--simple` | La de [[push_swap - Estrategias#1. Simple O(n²)\|O(n²)]] | O(n²) |
| `--medium` | La de [[push_swap - Estrategias#2. Intermedia O(n√n)\|O(n√n)]] | O(n√n) |
| `--complex` | La de [[push_swap - Estrategias#3. Compleja O(n log n)\|O(n log n)]] | O(n log n) |
| `--adaptive` | [[push_swap - Estrategias#4. Adaptativa\|Elige según el desorden]] | Depende del tramo |
| *(ninguna)* | Igual que `--adaptive` | |
| `--bench` | Métricas por **stderr** al terminar (opcional) | |

> [!warning] Cualquier flag con cualquier entrada
> El subject exige que **cada** selector funcione para **todos** los inputs válidos, sea cual sea el tamaño o el desorden. `--simple` con 500 números tiene que ordenar (aunque use muchas operaciones), y `--complex` con 2 números también.

### El modo `--bench`

Formato del ejemplo del subject (todo por stderr):

```text
$ ARG="4 67 3 87 23"; ./push_swap --bench --adaptive $ARG 2> bench.txt | ./checker_linux $ARG
OK
$ cat bench.txt
[bench] disorder:  40.00%
[bench] strategy:  Adaptive / O(n√n)
[bench] total_ops:  13
[bench] sa:  0  sb:  0  ss:  0  pa:  5  pb:  5
[bench] ra:  2  rb:  1  rr:  0  rra:  0  rrb:  0  rrr:  0
```

Cómo imprimir `40.00%` sin `float`: [[Decimales con enteros]]. Cómo contar cada operación: [[enum en C]].

## Entrada, salida y errores

| Entrada | Resultado | Por qué |
|---|---|---|
| `./push_swap` | *(nada)* | Sin parámetros |
| `./push_swap 1 2 3` | *(nada)* | Ya ordenado: 0 operaciones |
| `./push_swap 0 one 2 3` | `Error` | `one` no es un número (ejemplo del subject) |
| `./push_swap --simple 3 2 3` | `Error` | Duplicado (ejemplo del subject) |
| `./push_swap 2147483648` | `Error` | Mayor que `INT_MAX` |
| `./push_swap -2147483649` | `Error` | Menor que `INT_MIN` |
| `./push_swap 1 + 2` · `./push_swap 1 - 2` | `Error` | Signo sin dígitos |
| `./push_swap "" 1` | `Error` | Argumento vacío (así responde el checker del subject) |
| `./push_swap -0 0` | `Error` | Ambos valen 0: duplicado |
| `./push_swap +5 -3` | Válido | Un signo seguido de dígitos es un número |
| `./push_swap --fast 3 1` | `Error` (decisión vuestra) | Flag que no existe |
| `./push_swap "3 2 1"` | Decisión vuestra | El subject no lo exige; muchos evaluadores lo prueban |

El código para convertir con detección de overflow está en [[push_swap - Guía paso a paso#3. Parsear los números]].

## Rendimiento exigido

Se mide con números **aleatorios** y el `checker_linux` de la intra:

| n | Mínimo para aprobar | Bueno | Excelente |
|---|---|---|---|
| 100 | < 2000 | < 1500 | < 700 |
| 500 | < 12000 | < 8000 | < 5500 |

> [!important] Una entrada aleatoria tiene un desorden de ≈ 0,5
> Por eso, en la configuración por defecto (`--adaptive`), **más o menos la mitad** de los tests aleatorios caen en el tramo medio (O(n√n)) y la otra mitad en el alto (O(n log n)). Los algoritmos de **los dos** tramos tienen que pasar estos límites. Ver [[push_swap - Índice de desorden#Lo que vale con números aleatorios]].

Cifras medidas de algoritmos típicos (media de entradas aleatorias; detalle en [[push_swap - Estrategias#Comparativa medida]]):

| Algoritmo | 100 números | 500 números | Veredicto |
|---|---|---|---|
| Selection (O(n²)) | ~1 456 | ~32 377 | Solo vale para `--simple` |
| Chunks √n (O(n√n)) | ~776 | ~7 377 | Bueno en los dos |
| Radix binario (O(n log n)) | 1 084 | 6 784 | Bueno en los dos |
| Coste / «turco» | ~558 | ~5 084 | Excelente en los dos |

## README obligatorio

Además de lo general (primera línea en cursiva *Este proyecto ha sido creado como parte del currículo de 42 por \<login1\>, \<login2\>*, y las secciones **Descripción**, **Instrucciones** y **Recursos**, con el uso de IA), push_swap pide:

- **Explicación detallada y justificación de los algoritmos** elegidos.
- Para la adaptativa: **justificación de los umbrales**, las **técnicas** de cada tramo y un **análisis de complejidad** con cotas de **tiempo y espacio** en el modelo de push_swap.
- Las **contribuciones de cada persona**.

Plantilla general en [[README de un proyecto]]. Material para justificar en [[push_swap - Estrategias]] y [[push_swap - Defensa y evaluación]].

## Bonus: checker

Programa `checker` que recibe los mismos números, lee operaciones por stdin y responde `OK`/`KO` (o `Error`). Todo en [[push_swap - checker (bonus)]].

> [!warning] El bonus solo cuenta si la obligatoria es perfecta
> En push_swap «perfecta» incluye **pasar todas las pruebas de rendimiento**.

## Conceptos que necesitas

| Concepto | Para qué en push_swap | Nota |
|---|---|---|
| Complejidad | Clasificar y justificar cada estrategia | [[Notación Big-O]] · [[Algoritmos de ordenación y complejidad]] |
| Pilas | Qué es una pila y cómo rotarla | [[Pilas y colas]] |
| Representar las pilas | Array, anillo o lista | [[Arrays en C]] · [[Array circular]] · [[Listas doblemente enlazadas]] |
| Argumentos | Flags y números | [[argc y argv]] · [[ft_split]] |
| Conversión segura | Detectar `2147483648` | [[Tipos enteros y conversiones]] · [[ft_atoi]] |
| Agrupar datos | `t_stacks`, configuración | [[Structs y typedef]] · [[enum en C]] |
| Aritmética | Chunks, anillos, porcentajes | [[División entera y módulo]] · [[Decimales con enteros]] |
| Bits | Radix | [[Operadores de bits]] |
| Checker | Del texto `"rra"` a la función | [[Tablas de punteros a función]] · [[get_next_line - Índice\|get_next_line]] |
| Memoria y errores | Liberar todo, también al salir con `Error` | [[malloc y free]] · [[Control de errores en C]] · [[Valgrind y sanitizers]] |
| Entrega | Compilar libft primero, sin relink | [[Makefile y librerías estáticas]] · [[La Norma]] |

## Vocabulario

| Término | Significado |
|---|---|
| Top | Primer elemento de la pila. En `./push_swap 2 1 3`, el top de `a` es `2` |
| Rotar | Mover todos los elementos una posición: el top pasa al fondo (o al revés) |
| Inversión | Par de números en el que el mayor aparece antes que el menor |
| Índice de desorden | Inversiones / pares totales: 0 = ordenado, 1 = al revés |
| Índice normalizado | Posición que tendría cada número una vez ordenado (0, 1, 2…) |
| Chunk | Grupo de índices consecutivos que se mueven juntos |
| Coste | Número de operaciones necesarias para colocar un elemento |

## Relacionadas

[[push_swap - Índice de desorden]] · [[push_swap - Estrategias]] · [[push_swap - Guía paso a paso]] · [[push_swap - Pruebas y depuración]] · [[push_swap - checker (bonus)]] · [[push_swap - Defensa y evaluación]] · [[Notación Big-O]] · [[ft_printf - Índice]] · [[get_next_line - Índice]]

## Referencias

- Subject de push_swap v1.0 (`es.subject_push_swap.pdf`): capítulos VI («Parte obligatoria»), VII («Requisitos del Readme»), VIII («Parte bonus») y IX («Entrega y evaluación»). Los ejemplos de `--bench`, de errores y la tabla de rendimiento se han copiado de ahí.
- Cifras de operaciones: simulador propio de las dos pilas (Python, `random.seed(42)`, 40 entradas de 100 y 12 de 500), contrastado con un prototipo en C (radix 1 084 / 6 784 en ambos).
