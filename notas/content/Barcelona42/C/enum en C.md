---
tags: [c, fundamentos, enum, tipos, push-swap]
aliases: ["enum", "Enumeraciones", "Enumeraciones en C"]
---
# enum en C

[[Inicio|← Inicio]] · C · Fundamentos

> Un **`enum`** da nombres a una lista de constantes enteras: `SA` vale 0, `SB` vale 1… En vez de escribir `op(s, 4)` y tener que recordar que 4 es `pb`, escribes `op(s, PB)`. El compilador lo convierte en el mismo número, pero el código se lee solo.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Enumerador** | Cada nombre de la lista (`SA`, `PB`…) |
| **Valor** | El entero que representa. Por defecto empieza en 0 y sube de 1 en 1 |
| **`t_op`** | El tipo creado con `typedef` (la Norma pide prefijo `e_` para el `enum` y `t_` para el `typedef`) |

## Sintaxis

```c
typedef enum e_op
{
	SA,
	SB,
	SS,
	PA,
	PB,
	RA,
	RB,
	RR,
	RRA,
	RRB,
	RRR,
	OP_COUNT
}	t_op;
```

- `SA` = 0, `SB` = 1, …, `RRR` = 10.
- **`OP_COUNT` = 11**: truco clásico. Como el último enumerador vale «cuántos hay antes», sirve como tamaño de arrays.
- Puedes fijar valores: `enum e_x { A = 1, B = 4, C }` → `C` vale 5.

> [!note] La Norma y los `typedef`
> Norminette **no permite** `typedef` en un `.c`: el `enum` va en tu `.h`. También pide una tabulación al declarar un `enum` (ver [[La Norma]]).

## Ejemplo: contar operaciones por tipo

Un `enum` y dos arrays indexados con él sirven para el modo `--bench` de push_swap: uno guarda los nombres y otro los contadores.

```c
#include <stdio.h>

typedef enum e_op
{
	SA,
	SB,
	SS,
	PA,
	PB,
	RA,
	RB,
	RR,
	RRA,
	RRB,
	RRR,
	OP_COUNT
}	t_op;

int	main(void)
{
	static const char	*names[OP_COUNT] = {"sa", "sb", "ss", "pa", "pb",
		"ra", "rb", "rr", "rra", "rrb", "rrr"};
	int					counts[OP_COUNT];
	int					i;

	i = 0;
	while (i < OP_COUNT)
		counts[i++] = 0;
	counts[PB] += 2;
	counts[RA]++;
	counts[PA] += 2;
	printf("OP_COUNT = %d, RRA vale %d\n", OP_COUNT, RRA);
	i = 0;
	while (i < OP_COUNT)
	{
		if (counts[i])
			printf("%s: %d\n", names[i], counts[i]);
		i++;
	}
	return (0);
}
```

```text
$ cc -Wall -Wextra -Werror enum.c -o enum && ./enum
OP_COUNT = 11, RRA vale 8
pa: 2
pb: 2
ra: 1
```

(En este ejemplo el `typedef` está en el `.c` para que quepa en un solo fichero; en el proyecto, al `.h`.)

> [!tip] Una sola función para todas las operaciones
> Con el `enum`, `op(s, PB)` puede hacer tres cosas a la vez: ejecutar el movimiento, sumar `counts[PB]++` e imprimir `names[PB]`. Si todo pasa por ahí, el contador del bench no puede desincronizarse de lo que imprimes.

## Otro uso: la estrategia elegida

```c
typedef enum e_strategy
{
	ADAPTIVE,
	SIMPLE,
	MEDIUM,
	COMPLEX
}	t_strategy;
```

`ADAPTIVE` vale 0: si inicializas tu struct de configuración a cero, ya tienes el comportamiento por defecto que pide el subject.

## `enum` frente a `#define`

| | `enum` | `#define` |
|---|---|---|
| Numeración automática | ✅ | ❌, a mano |
| El depurador muestra el nombre | ✅ (`gdb` imprime `PB`) | ❌ (ve `4`) |
| Tiene tipo | ✅ (entero) | ❌ (sustitución de texto) |
| Sirve para valores no enteros | ❌ | ✅ |

Más sobre `#define` en [[Preprocesador y headers]].

## Errores frecuentes

> [!warning] Desordenar el `enum` y el array de nombres
> Si añades un enumerador en medio y no actualizas `names[]`, `names[RRA]` imprimirá otra operación. Mantén los dos en el mismo orden y en el mismo sitio.

> [!warning] Suponer que un `enum` limita los valores
> C no comprueba nada: `t_op x = 42;` compila. Si recibes un valor de fuera, compruébalo tú (`if (x >= OP_COUNT)`).

## Relacionadas

[[Structs y typedef]] · [[Preprocesador y headers]] · [[Arrays en C]] · [[Tablas de punteros a función]] · [[La Norma]] · [[push_swap - Guía paso a paso]]

## Referencias

- [cppreference: Enumerations](https://en.cppreference.com/w/c/language/enum) — valores por defecto y explícitos.
- [[La Norma]] — apartado «Typedef, struct, enum y union».
- Salida del ejemplo: compilado con Ubuntu clang 12.0.1. Norminette: el `typedef` en un `.c` da `FORBIDDEN_TYPEDEF`, como se indica.
