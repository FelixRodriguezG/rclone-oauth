---
tags: [proyecto, push-swap, rank-1, evaluacion, defensa, readme]
aliases: ["Evaluación de push_swap", "Defensa de push_swap"]
---
# push_swap - Defensa y evaluación

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · Evaluación

> En la evaluación de push_swap estáis **los dos**, y cualquiera puede tener que explicar **cualquier** parte del código. Además puede pedirse una **modificación pequeña en directo**. Esta nota reúne las preguntas probables con respuestas modelo, ejemplos de modificaciones y la checklist del README.

## Antes de entrar

> [!check] Checklist
> - [ ] `make`, `make` otra vez (sin relink), `make clean`, `make fclean`, `make re`, `make bonus`.
> - [ ] Norminette sin errores en **todos** los ficheros, también los de libft.
> - [ ] Ninguna variable global.
> - [ ] Tabla de errores completa ([[push_swap - Pruebas y depuración#2. Errores]]).
> - [ ] Peor caso de 50+ pruebas aleatorias con 100 y 500 números dentro de los límites.
> - [ ] Las 4 flags funcionan con 1, 2, 3, 5, 100 y 500 números.
> - [ ] Valgrind limpio, también en las rutas de error.
> - [ ] README con los dos logins, las contribuciones y la justificación de los algoritmos.
> - [ ] Los dos habéis leído **todo** el código del otro.

## Preguntas probables

> [!question] ¿Qué es el índice de desorden y cómo lo calculáis?
> «La proporción de pares (i, j), con i antes que j, en los que el primero es mayor. Contamos con dos bucles anidados los pares invertidos y los dividimos entre n(n−1)/2. Con un solo número no hay pares y devolvemos 0. Lo calculamos antes de mover nada.» → [[push_swap - Índice de desorden]]

> [!question] ¿Por qué el desorden de una lista aleatoria es 0,5?
> «Para cualquier par, que el primero sea mayor o menor es igual de probable, así que la mitad de los pares, de media, están invertidos.»

> [!question] ¿Por qué vuestra estrategia simple es O(n²)?
> «Para cada uno de los n elementos subimos el mínimo con, como mucho, n/2 rotaciones. n · n/2 es O(n²) operaciones.»

> [!question] ¿Y la intermedia O(n√n)?
> «Dividimos los índices en √n chunks. Para pasar cada chunk a `b` basta como mucho una vuelta a `a`: √n · n. Al devolver, el máximo siempre está en el grupo de arriba o en el del fondo del chunk más alto, a unas √n posiciones: n · √n.» → [[push_swap - Estrategias#2. Intermedia O(n√n)]]

> [!question] ¿Y la compleja?
> «Radix binario sobre índices 0…n−1: ⌈log₂ n⌉ pasadas (7 con 100, 9 con 500), cada una de n operaciones `pb`/`ra` más los `pa` de vuelta. Como mucho 2n·log₂ n operaciones.»

> [!question] Si la complejidad se mide en operaciones, ¿cuánto tarda vuestro cálculo de coste?
> «En CPU es O(n²), pero el subject mide operaciones de push_swap impresas, no tiempo de CPU.» → [[Notación Big-O#Lo que cambia en push_swap: contar operaciones, no tiempo]]

> [!question] Justificad los umbrales 0,2 y 0,5
> «0,5 es el desorden esperado de una entrada aleatoria: por encima no hay estructura que aprovechar y usamos el algoritmo general. Por debajo de 0,2 la mayoría de los pares ya están en orden, y conservamos esa parte ordenada (LIS) moviendo solo lo que sobra.» → [[push_swap - Estrategias#Por qué esos umbrales]]

> [!question] ¿Es realmente O(n) vuestro método para desorden bajo?
> Respuesta honesta: «Es O(n) cuando el número de elementos fuera de sitio está acotado. En general es O(n + k·n). Ningún método puede ser O(n) para todas las entradas con desorden < 0,2: por conteo de permutaciones, hacen falta Ω(n log n) operaciones en el peor caso.» → [[push_swap - Estrategias#El tramo O(n) y su trampa]]

> [!question] ¿Por qué `3 4 5 1 2` no usa el tramo alto si su desorden es 0,6?
> «Antes de mirar el desorden comprobamos si es una lista ordenada y rotada; en ese caso solo rotamos.» (Si no lo hacéis, ¡explicad qué pasa!)

> [!question] ¿Cómo detectáis el overflow?
> «Convertimos a `long` y comparamos con `INT_MAX`/`INT_MIN` en cada dígito, para no desbordar ni el `long` con cadenas muy largas.»

> [!question] ¿Cómo imprimís 13.33% sin `float`?
> «Calculamos en centésimas: `(errores * 10000 + total / 2) / total`. Luego imprimimos `/ 100`, un punto y `% 100` con dos cifras.» → [[Decimales con enteros]]

> [!question] ¿Por qué normalizáis?
> «Para trabajar con 0…n−1: sin negativos, radix necesita solo log₂ n bits y los chunks son rangos de índices.»

> [!question] ¿Cuántas operaciones como máximo con 3 y con 5 números?
> 2 y (con nuestro método) 10.

> [!question] ¿Por qué el bench va a stderr?
> «Porque stdout tiene que contener solo operaciones: el checker lee stdout.»

## Modificaciones en directo (ejemplos)

Practicad estas modificaciones; cada una debería llevar pocos minutos si el código está bien organizado:

| Petición | Dónde se toca |
|---|---|
| Cambiar el umbral 0,2 por 0,3 | Una constante en la adaptativa (mejor si es un `#define`) |
| Usar 8 chunks fijos en vez de √n | El cálculo del tamaño del chunk |
| Que `--bench` muestre también el número de elementos | `bench.c` |
| Añadir un flag `--quiet` que no imprima operaciones | Parseo de flags + `op()` |
| Imprimir las operaciones en mayúsculas | La tabla de nombres que usa `op()` |
| Que el checker muestre cuántas instrucciones ha leído | El bucle de lectura |
| Que no se use `ss` | Las estrategias / `op()` |

> [!tip] Por qué `op()` como única puerta ayuda aquí
> La mitad de estas peticiones se resuelven tocando **una** función. Ver [[push_swap - Guía paso a paso#5. Las operaciones]].

## README: checklist específica

- [ ] Primera línea en cursiva: *Este proyecto ha sido creado como parte del currículo de 42 por \<login1\>, \<login2\>.*
- [ ] **Descripción**: el problema, las dos pilas, las 11 operaciones, las 4 estrategias.
- [ ] **Instrucciones**: `make`, ejemplos de uso con cada flag y con `--bench`, `make bonus`.
- [ ] **Recursos**: referencias (Big-O, radix, LIS…) y **uso de IA**: para qué tareas y en qué partes.
- [ ] **Algoritmos**: explicación y justificación de cada estrategia, con su clase de complejidad.
- [ ] **Adaptativa**: umbrales justificados, técnica por tramo, cotas de **tiempo y espacio**.
- [ ] **Contribuciones** de cada persona.
- [ ] Tabla de resultados medidos (media y peor caso con 100 y 500).

Plantilla general en [[README de un proyecto]].

## Errores que suspenden

> [!danger] Lo que más falla
> - `Error` en stdout en vez de stderr, o sin `\n`.
> - Algo del bench o de depuración en stdout.
> - Imprimir algo con la lista ya ordenada o sin argumentos.
> - No detectar `2147483648` o `-0 0`.
> - Una flag que no funciona con algún tamaño (por ejemplo, `--medium` con 2 números).
> - Superar el límite de operaciones en **alguna** prueba aleatoria.
> - Leaks o segfault en las rutas de error.
> - Que una de las dos personas no sepa explicar una parte.
> - Norma o relink ([[La Norma]], [[Makefile y librerías estáticas]]).

## Relacionadas

[[push_swap - Índice]] · [[push_swap - Estrategias]] · [[push_swap - Índice de desorden]] · [[push_swap - Pruebas y depuración]] · [[Notación Big-O]] · [[README de un proyecto]] · [[Git para 42]]

## Referencias

- Subject de push_swap v1.0, capítulos III («Instrucciones sobre la IA»), VI.1 («Requisitos de proyectos en grupo»), VII («Requisitos del Readme») y IX («Entrega y evaluación»).
