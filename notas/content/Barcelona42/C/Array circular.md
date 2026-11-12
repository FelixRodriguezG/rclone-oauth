---
tags: [c, estructuras-de-datos, arrays, modulo, push-swap]
aliases: ["Buffer circular", "Ring buffer", "Anillo"]
---
# Array circular

[[Inicio|← Inicio]] · C · Estructuras de datos

> Un **array circular** (*ring buffer*) es un array normal que se recorre como si el final estuviera pegado al principio. En vez de mover los datos, mueves un índice **`head`** que dice dónde empieza. Así, rotar una [[Pilas y colas|pila]] cuesta **un solo cálculo** en lugar de desplazar todos los elementos.

## Vocabulario

| Término | Qué significa |
|---|---|
| **`cap`** (capacidad) | Tamaño real del array reservado |
| **`head`** | Índice del array donde está el elemento lógico 0 (el top) |
| **`size`** | Cuántos elementos hay guardados ahora |
| **Índice lógico** | La posición «para el usuario»: 0 = top, 1 = el siguiente… |
| **Índice físico** | La casilla real del array: `(head + lógico) % cap` |

## La idea

```text
data (físico):  [ 1 | 2 | 3 | 4 | 5 ]      cap = 5
                  0   1   2   3   4

head = 0  →  lógico: 1 2 3 4 5
head = 2  →  lógico: 3 4 5 1 2     ← el mismo array, empezando en la casilla 2
```

La fórmula que hace la magia usa el **módulo** (`%`, ver [[División entera y módulo]]):

```c
fisico = (head + logico) % cap;
```

Si `head + logico` se pasa del final, `%` lo devuelve al principio: con `cap = 5`, `(3 + 4) % 5 = 2`.

## Ejemplo

```c
#include <stdio.h>

typedef struct s_ring
{
	int	data[8];
	int	cap;
	int	head;
	int	size;
}	t_ring;

static int	ring_get(t_ring *r, int i)
{
	return (r->data[(r->head + i) % r->cap]);
}

static void	ring_rotate(t_ring *r)
{
	r->head = (r->head + 1) % r->cap;
}

static void	ring_reverse_rotate(t_ring *r)
{
	r->head = (r->head - 1 + r->cap) % r->cap;
}

static void	ring_print(t_ring *r, const char *msg)
{
	int	i;

	printf("%-5s head=%d  ", msg, r->head);
	i = 0;
	while (i < r->size)
		printf("%d ", ring_get(r, i++));
	printf("\n");
}

int	main(void)
{
	t_ring	r;
	int		i;

	r.cap = 5;
	r.head = 0;
	r.size = 5;
	i = 0;
	while (i < 5)
	{
		r.data[i] = i + 1;
		i++;
	}
	ring_print(&r, "ini");
	ring_rotate(&r);
	ring_print(&r, "ra");
	ring_rotate(&r);
	ring_print(&r, "ra");
	ring_reverse_rotate(&r);
	ring_reverse_rotate(&r);
	ring_reverse_rotate(&r);
	ring_print(&r, "rra×3");
	return (0);
}
```

```text
$ cc -Wall -Wextra -Werror circ.c -o circ && ./circ
ini   head=0  1 2 3 4 5 
ra    head=1  2 3 4 5 1 
ra    head=2  3 4 5 1 2 
rra×3 head=4  5 1 2 3 4 
```

Los datos del array **nunca se han movido**; solo ha cambiado `head`.

> [!warning] El `+ cap` del `reverse_rotate`
> `(0 - 1) % 5` vale **−1** en C, no 4: el resto conserva el signo del dividendo. Sumar `cap` antes del `%` evita el índice negativo.

## Cuidado: rotar con `size < cap`

Si la pila **no está llena** (`size < cap`), rotar moviendo solo `head` no funciona: el «fondo» lógico está en `head + size − 1`, no en `head + cap − 1`. En push_swap esto pasa siempre, porque `a` y `b` comparten los n elementos. Dos soluciones:

- Darle a **cada pila** su array con `cap = n` y, al rotar con `size < cap`, mover el elemento: el top se copia a la casilla `(head + size) % cap` y luego `head` avanza.
- O usar un array normal (más sencillo) y aceptar que `ra` cueste O(n) en CPU, que en push_swap no puntúa (ver [[Pilas y colas#Cómo se implementa en C]]).

```text
cap = 5, size = 3, head = 1
data: [ _ | 7 | 8 | 9 | _ ]     lógico: 7 8 9

ra  →  copiar data[1] (7) a data[(1 + 3) % 5] = data[4]
       head = 2
data: [ _ | _ | 8 | 9 | 7 ]     lógico: 8 9 7  ✅
```

## Errores frecuentes

> [!danger] Olvidar el `%`
> `data[head + i]` funciona hasta que `head + i ≥ cap`, y entonces lees fuera del array (comportamiento indefinido, ver [[Comportamiento indefinido]]).

> [!warning] Índices negativos
> Antes de `%`, suma `cap` cuando restes: `(head - 1 + cap) % cap`.

## Relacionadas

[[División entera y módulo]] · [[Pilas y colas]] · [[Arrays en C]] · [[Structs y typedef]] · [[Listas doblemente enlazadas]] · [[push_swap - Guía paso a paso]]

## Referencias

- [Wikipedia: Circular buffer](https://en.wikipedia.org/wiki/Circular_buffer) — funcionamiento y uso de los índices de inicio y fin.
- [cppreference: Arithmetic operators — remainder](https://en.cppreference.com/w/c/language/operator_arithmetic) — signo del resultado de `%` con operandos negativos.
- Salida del ejemplo: compilado con Ubuntu clang 12.0.1.
