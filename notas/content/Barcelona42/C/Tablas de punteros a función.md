---
tags: [c, punteros, punteros-a-funcion, structs, push-swap]
aliases: ["Tabla de despacho", "Dispatch table", "Array de punteros a función"]
---
# Tablas de punteros a función

[[Inicio|← Inicio]] · C · Punteros

> Una **tabla de despacho** es un array de structs en el que cada fila une un **nombre** con la **función** que lo ejecuta. En vez de una cadena de 11 `if` para saber qué operación es `"rra"`, recorres la tabla, comparas el nombre y llamas a la función de esa fila. Es la base del `checker` de push_swap. Antes, repasa [[Punteros en C#Punteros a función]].

## Vocabulario

| Término | Qué significa |
|---|---|
| **Puntero a función** | Variable que guarda la dirección de una función y permite llamarla |
| **Despachar** (*dispatch*) | Decidir qué función ejecutar según un dato de entrada |
| **Centinela** | Fila final especial (`{NULL, NULL}`) que marca el fin de la tabla |

## Leer la declaración

```c
void	(*fn)(t_stacks *);
```

Se lee de dentro afuera: `fn` es un **puntero** (`*fn`) a una **función** que recibe un `t_stacks *` y devuelve `void`. Todas las funciones de la tabla tienen que tener **la misma firma**.

## Ejemplo: del texto a la función

```c
#include <stdio.h>

typedef struct s_stacks
{
	int	moves;
}	t_stacks;

typedef struct s_entry
{
	const char	*name;
	void		(*fn)(t_stacks *);
}	t_entry;

static void	do_sa(t_stacks *s)
{
	s->moves++;
	printf("  ejecuto sa\n");
}

static void	do_ra(t_stacks *s)
{
	s->moves++;
	printf("  ejecuto ra\n");
}

static int	str_eq(const char *a, const char *b)
{
	while (*a && *a == *b)
	{
		a++;
		b++;
	}
	return (*a == *b);
}

static int	run_op(t_stacks *s, const char *line)
{
	static const t_entry	table[] = {{"sa", do_sa}, {"ra", do_ra},
	{NULL, NULL}};
	int						i;

	i = 0;
	while (table[i].name)
	{
		if (str_eq(table[i].name, line))
		{
			table[i].fn(s);
			return (1);
		}
		i++;
	}
	return (0);
}

int	main(void)
{
	t_stacks	s;

	s.moves = 0;
	printf("\"ra\"  -> %d\n", run_op(&s, "ra"));
	printf("\"sa\"  -> %d\n", run_op(&s, "sa"));
	printf("\"sa \" -> %d\n", run_op(&s, "sa "));
	printf("\"xx\"  -> %d\n", run_op(&s, "xx"));
	printf("movimientos: %d\n", s.moves);
	return (0);
}
```

```text
$ cc -Wall -Wextra -Werror dispatch.c -o dispatch && ./dispatch
  ejecuto ra
"ra"  -> 1
  ejecuto sa
"sa"  -> 1
"sa " -> 0
"xx"  -> 0
movimientos: 2
```

Fíjate en `"sa "` (con espacio): **no** coincide, y `run_op` devuelve 0. El checker de push_swap tiene que tratarlo como instrucción inválida y responder `Error`.

## Por qué es mejor que una cadena de `if`

```c
/* Sin tabla: 11 ramas casi iguales, y la Norma limita a 25 líneas */
if (str_eq(line, "sa"))
	do_sa(s);
else if (str_eq(line, "sb"))
	do_sb(s);
/* … 9 más … */
```

| | Cadena de `if` | Tabla |
|---|---|---|
| Añadir una operación | Una rama más | Una fila más |
| Líneas de la función | Crece con cada operación | Fija |
| Cabe en 25 líneas con 11 ops | ❌ | ✅ |

> [!tip] Encaja con el `enum`
> Si las operaciones tienen un [[enum en C|`enum`]] (`SA`, `SB`…), la tabla puede indexarse con él (`table[PB].fn(s)`) y el nombre sirve para imprimir y para el bench.

## Errores frecuentes

> [!danger] Olvidar el centinela
> Sin `{NULL, NULL}` al final, el `while (table[i].name)` sigue leyendo fuera del array: [[Comportamiento indefinido]].

> [!warning] Firmas distintas
> Si `do_pa` recibe dos parámetros y las demás uno, no puede ir en la misma tabla. Haz que todas reciban lo mismo (por ejemplo, un `t_stacks *` con las dos pilas dentro).

> [!warning] Comparar con prefijo
> Con `ft_strncmp(line, "rr", 2)`, `"rra"` coincidiría con `"rr"`. Compara la cadena entera.

## Relacionadas

[[Punteros en C]] · [[Structs y typedef]] · [[enum en C]] · [[argc y argv]] · [[push_swap - checker (bonus)]]

## Referencias

- [cppreference: Pointer declaration — pointers to functions](https://en.cppreference.com/w/c/language/pointer#Pointers_to_functions) — sintaxis y llamada.
- [Wikipedia: Dispatch table](https://en.wikipedia.org/wiki/Dispatch_table) — el patrón.
- Salida del ejemplo: compilado con Ubuntu clang 12.0.1.
