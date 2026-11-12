---
tags: [c, estructuras-de-datos, listas, punteros, push-swap]
aliases: ["Lista doblemente enlazada", "Lista doble", "Doubly linked list", "Lista circular"]
---
# Listas doblemente enlazadas

[[Inicio|← Inicio]] · C · Estructuras de datos

> En una [[Listas enlazadas|lista enlazada]] simple, cada nodo sabe quién va **después**. En una **doblemente enlazada**, cada nodo sabe también quién va **antes** (`prev`). Si además el último apunta al primero, la lista es **circular**, y eso convierte las rotaciones de [[push_swap - Índice|push_swap]] (`ra`, `rra`) en mover un solo puntero.

## Vocabulario

| Término | Qué significa |
|---|---|
| **`next`** | Puntero al nodo siguiente |
| **`prev`** | Puntero al nodo anterior |
| **Circular** | El último nodo apunta al primero (`last->next == top`) y el primero al último (`top->prev == last`) |
| **Top** | El nodo que consideras el primero. En una lista circular es solo «por dónde empiezas a mirar» |

## El nodo

```c
typedef struct s_node
{
	int				value;
	struct s_node	*prev;
	struct s_node	*next;
}	t_node;
```

Dentro del struct se escribe `struct s_node *`, porque el `typedef` `t_node` aún no existe en ese punto (ver [[Structs y typedef]]).

## Simple frente a doble y circular

```text
SIMPLE:        top → [3] → [1] → [2] → NULL
               ¿el último? hay que recorrer toda la lista: O(n)

DOBLE CIRCULAR:
               ┌──────────────────────────────┐
               ▼                              │
        top → [3] ⇄ [1] ⇄ [2] ────────────────┘
               ▲                 │
               └──── prev ───────┘
               el último es top->prev: O(1)
```

| Operación | Lista simple | Doble circular |
|---|---|---|
| Llegar al último | O(n) | O(1): `top->prev` |
| `ra` (top al fondo) | O(n) | O(1): `top = top->next` |
| `rra` (fondo al top) | O(n) | O(1): `top = top->prev` |
| Memoria por nodo | 1 puntero | 2 punteros |

## Ejemplo: crear, rotar y liberar

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct s_node
{
	int				value;
	struct s_node	*prev;
	struct s_node	*next;
}	t_node;

static t_node	*node_new(int value)
{
	t_node	*n;

	n = malloc(sizeof(t_node));
	if (!n)
		return (NULL);
	n->value = value;
	n->prev = n;
	n->next = n;
	return (n);
}

static void	push_back(t_node **top, t_node *n)
{
	t_node	*last;

	if (!*top)
	{
		*top = n;
		return ;
	}
	last = (*top)->prev;
	last->next = n;
	n->prev = last;
	n->next = *top;
	(*top)->prev = n;
}

static void	print_list(t_node *top, const char *msg)
{
	t_node	*cur;

	printf("%-4s", msg);
	if (top)
	{
		cur = top;
		printf("%d ", cur->value);
		cur = cur->next;
		while (cur != top)
		{
			printf("%d ", cur->value);
			cur = cur->next;
		}
	}
	printf("\n");
}

static void	free_list(t_node **top)
{
	t_node	*cur;
	t_node	*next;

	if (!*top)
		return ;
	(*top)->prev->next = NULL;
	cur = *top;
	while (cur)
	{
		next = cur->next;
		free(cur);
		cur = next;
	}
	*top = NULL;
}

int	main(void)
{
	t_node	*a;
	int		i;

	a = NULL;
	i = 1;
	while (i <= 4)
		push_back(&a, node_new(i++));
	print_list(a, "ini");
	a = a->next;
	print_list(a, "ra");
	a = a->prev;
	a = a->prev;
	print_list(a, "rra");
	free_list(&a);
	return (0);
}
```

```text
$ cc -Wall -Wextra -Werror dlist.c -o dlist && ./dlist
ini 1 2 3 4 
ra  2 3 4 1 
rra 4 1 2 3 
$ valgrind -q --leak-check=full ./dlist > /dev/null
$
```

Valgrind no dice nada: no hay leaks. El ejemplo no comprueba el `NULL` de `node_new` para ser corto; en tu proyecto, sí.

Puntos clave:

- **`ra` es `top = top->next`** y **`rra` es `top = top->prev`**. No se toca ningún enlace.
- Recorrer una lista circular: **no** hay `NULL` al final. Te paras cuando vuelves a `top`.
- Para liberar, primero **rompes el círculo** (`last->next = NULL`) y luego liberas como en una lista simple.

## `pb` en una lista doble circular

Mover el top de `a` a `b` sí toca enlaces: hay que **desenganchar** el nodo de `a` y **engancharlo** encima de `b`.

```text
desenganchar x de a:            enganchar x encima de b:
  x->prev->next = x->next         x->next = top_b
  x->next->prev = x->prev         x->prev = top_b->prev
  top_a = x->next                 top_b->prev->next = x
  (si era el único: top_a = NULL) top_b->prev = x
                                  top_b = x
                                  (si b estaba vacía: x apunta a sí mismo)
```

Los dos casos especiales (pila que se queda vacía y pila que estaba vacía) son la fuente de casi todos los segfaults.

## Errores frecuentes

> [!danger] Bucle infinito al recorrer
> `while (cur != NULL)` en una lista circular nunca termina. Usa `cur != top`, o un contador con el tamaño.

> [!danger] Liberar sin romper el círculo
> Si liberas siguiendo `next` sin poner antes `last->next = NULL`, vuelves al primer nodo ya liberado: *double free* o uso tras `free`.

> [!warning] Olvidar actualizar `prev`
> Un enlace `next` correcto con un `prev` antiguo funciona hasta el primer `rra`. Después de cada operación, dibuja los punteros en papel.

## Relacionadas

[[Listas enlazadas]] · [[Pilas y colas]] · [[Array circular]] · [[Structs y typedef]] · [[Puntero a puntero]] · [[malloc y free]] · [[Valgrind y sanitizers]] · [[push_swap - Guía paso a paso]]

## Referencias

- [Wikipedia: Doubly linked list](https://en.wikipedia.org/wiki/Doubly_linked_list) — nodos con `prev`/`next` y variante circular.
- Salida del ejemplo: compilado con Ubuntu clang 12.0.1, comprobado con Valgrind.
