---
tags: [c, estructuras-de-datos, pila, push-swap]
aliases: ["Pila", "Stack", "Cola", "Queue", "LIFO", "FIFO"]
---
# Pilas y colas

[[Inicio|← Inicio]] · C · Estructuras de datos

> Una **pila** (*stack*) es una colección en la que solo se trabaja con el elemento de **arriba**: el último que entra es el primero que sale (LIFO). Una **cola** (*queue*) es al revés: el primero que entra es el primero que sale (FIFO). Las pilas de [[push_swap - Índice|push_swap]] son pilas «con trampa», porque además se pueden **rotar**.

> [!note] No confundir con «la stack»
> La *stack* de memoria (donde viven las variables locales, ver [[Segmentos de memoria en C]]) se llama así porque funciona como una pila, pero es otra cosa. Aquí hablamos de la pila como **estructura de datos** que programas tú.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Top** (cima) | El elemento accesible de la pila |
| **Push** | Meter un elemento encima |
| **Pop** | Sacar el elemento de encima |
| **Peek** | Mirar el de encima sin sacarlo |
| **LIFO** | *Last In, First Out*: el último en entrar es el primero en salir |
| **FIFO** | *First In, First Out*: el primero en entrar es el primero en salir |

## Pila y cola en un dibujo

```text
PILA (LIFO)                       COLA (FIFO)
push 1, push 2, push 3            enqueue 1, 2, 3

   │ 3 │ ← top: sale primero        sale ← [ 1 | 2 | 3 ] ← entra
   │ 2 │                               primero        último
   │ 1 │
   └───┘
pop → 3                           dequeue → 1
```

Ejemplos reales: el «deshacer» de un editor es una pila; la cola de impresión es una cola.

## Las pilas de push_swap

push_swap tiene dos pilas, `a` y `b`, y sus operaciones son las de una pila más dos extras:

| Operación clásica | En push_swap |
|---|---|
| push / pop entre dos pilas | `pa`, `pb` (pop de una + push en la otra) |
| Intercambiar los dos de arriba | `sa`, `sb` |
| **Rotar** (no existe en una pila pura) | `ra`/`rb`: el top pasa al fondo. `rra`/`rrb`: el fondo pasa al top |

Con las rotaciones, la pila se comporta en realidad como un **anillo**: puedes llevar cualquier elemento arriba. El coste de traer el elemento en la posición `i` (contando desde 0) de una pila de tamaño `n` es:

```text
coste(i) = min(i, n − i)      → i veces ra   o   n − i veces rra
```

```text
n = 6,  quiero el elemento de la posición 4

 pos  0  1  2  3  4  5
      7  2  9  1 [5] 3
 ra × 4  →  5 3 7 2 9 1
 rra × 2 →  5 3 7 2 9 1   ← más barato: min(4, 6 − 4) = 2
```

Esta idea está en todos los algoritmos de push_swap (ver [[push_swap - Estrategias]]).

## Cómo se implementa en C

Hay tres opciones habituales:

| Implementación | `push`/`pop` | `ra`/`rra` | Nota |
|---|---|---|---|
| Array con el top en `t[0]` | O(n): desplazar todo | O(n): desplazar todo | La más fácil de depurar ([[Arrays en C]]) |
| [[Array circular]] | O(1) | O(1): mover el índice `head` | Elegante, necesita `%` ([[División entera y módulo]]) |
| [[Listas doblemente enlazadas\|Lista doblemente enlazada]] circular | O(1) | O(1): mover el puntero `top` | Muchos `malloc`, cuidado con los leaks |

> [!tip] El coste de CPU de tus operaciones no puntúa
> En push_swap solo cuenta el número de operaciones **impresas**. Que tu `ra` tarde O(n) en un array normal no penaliza nada; con 500 números, ni se nota. Elige la implementación que mejor entiendas y puedas explicar.

## Errores frecuentes

> [!danger] Operar sobre una pila vacía
> `pb` con `a` vacía, o `sa` con un solo elemento, deben **no hacer nada**. Si lees `t[0]` o `top->next` sin comprobar el tamaño, tendrás un segfault.

> [!warning] Confundir el top con el final del array
> En push_swap el primer argumento es el top. Si guardas el top en `t[size - 1]`, todo funciona igual, pero tienes que ser coherente en todas las operaciones.

## Relacionadas

[[Array circular]] · [[Listas doblemente enlazadas]] · [[Listas enlazadas]] · [[Arrays en C]] · [[Structs y typedef]] · [[push_swap - Guía paso a paso]]

## Referencias

- [Wikipedia: Stack (abstract data type)](https://en.wikipedia.org/wiki/Stack_(abstract_data_type)) — operaciones push, pop y peek.
- [Wikipedia: Queue (abstract data type)](https://en.wikipedia.org/wiki/Queue_(abstract_data_type)) — FIFO.
- Subject de push_swap v1.0, VI.2 («Las reglas») — las 11 operaciones.
