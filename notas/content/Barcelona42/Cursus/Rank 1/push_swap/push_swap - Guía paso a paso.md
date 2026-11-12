---
tags: [proyecto, push-swap, rank-1, guia]
aliases: ["Guía de push_swap", "push_swap paso a paso"]
---
# push_swap - Guía paso a paso

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · Guía

> Esta guía explica **cómo organizar** push_swap y en qué orden construirlo, de forma que en cada etapa tengas algo que funciona y se puede probar. No es una solución para copiar: los fragmentos de código ilustran las partes delicadas (overflow, normalización, operaciones). El diseño y los algoritmos tenéis que escribirlos y entenderlos vosotros dos (subject, capítulo III).

## 1. La arquitectura

```text
 argv
  │
  ▼
┌──────────────┐  flags: --simple … --bench          ┌──────────────┐
│ parse_args   │────────────────────────────────────►│  t_config    │
└──────┬───────┘                                     └──────┬───────┘
       │ números (int)                                      │
       ▼                                                    │
┌──────────────┐   Error → liberar todo → "Error\n" a fd 2  │
│ validar      │   (no número, overflow, duplicado)         │
└──────┬───────┘                                            │
       ▼                                                    │
┌──────────────┐                                            │
│ normalizar   │  valores → índices 0…n-1                   │
└──────┬───────┘                                            │
       ▼                                                    ▼
┌──────────────┐   ┌───────────────────────────────────────────┐
│ desorden     │──►│ elegir estrategia (flag o adaptativa)      │
└──────────────┘   └──────────────────┬────────────────────────┘
                                      ▼
                 ┌──────────────────────────────────────┐
                 │ estrategia → op(s, PB) → op(s, RA)…  │  cada op: mover + contar + imprimir
                 └──────────────────┬───────────────────┘
                                    ▼
                 ┌──────────────────────────────────────┐
                 │ --bench → métricas a fd 2            │
                 └──────────────────┬───────────────────┘
                                    ▼
                               liberar todo
```

### Reparto de ficheros (orientativo)

| Fichero | Contenido |
|---|---|
| `push_swap.h` | Structs, `enum`, prototipos |
| `main.c` | `main`: une las piezas |
| `parse.c` | Flags, conversión, duplicados |
| `stack_ops.c` / `stack_ops2.c` | Las operaciones básicas (máx. 5 funciones por fichero, ver [[La Norma]]) |
| `op.c` | `op()`: ejecutar + contar + imprimir |
| `disorder.c` | Índice de desorden |
| `small_sort.c` | 2, 3, 4 y 5 números |
| `strategy_simple.c`, `strategy_medium.c`, `strategy_complex.c`, `strategy_adaptive.c` | Una estrategia por fichero |
| `bench.c` | Salida de `--bench` |
| `libft/` | Vuestra libft con su Makefile (más `ft_printf` si la usáis) |

> [!tip] Reparto en pareja
> Una partición natural: una persona hace parseo + operaciones + bench, y la otra, estrategias. Pero **las dos tenéis que entenderlo todo**: cruzad revisiones y explicaos el código mutuamente antes de la evaluación. Anotad quién hizo qué para el README.

## 2. Las estructuras

Una opción sencilla: dos arrays de tamaño n, con el top en la posición 0.

```c
typedef enum e_strategy
{
	ADAPTIVE,
	SIMPLE,
	MEDIUM,
	COMPLEX
}	t_strategy;

typedef struct s_stacks
{
	int		*a;
	int		*b;
	int		size_a;
	int		size_b;
	long	counts[OP_COUNT];
}	t_stacks;

typedef struct s_config
{
	t_strategy	strategy;
	int			bench;
}	t_config;
```

`OP_COUNT` y la lista de operaciones vienen de un `enum` (ver [[enum en C]]). Alternativas de representación y sus costes: [[Pilas y colas#Cómo se implementa en C]], [[Array circular]], [[Listas doblemente enlazadas]].

> [!note] Sin globales
> El subject las prohíbe. Todo viaja dentro de `t_stacks` y `t_config`, que se pasan por puntero.

## 3. Parsear los números

### Flags y números

Recorre `argv` desde 1 (ver [[argc y argv]]):

- Empieza por `--` → es un flag. Si no es uno de los cinco válidos → `Error`.
- Si no → es un número. Si aceptáis `"3 2 1"`, separadlo con [[ft_split]] y convertid cada trozo.

### Convertir con detección de overflow

[[ft_atoi]] no detecta overflow. Convierte a `long` y comprueba el rango **en cada dígito**, antes de que también el `long` se desborde con cadenas muy largas (ver [[Tipos enteros y conversiones]]):

```c
static int	parse_int(const char *s, int *out)
{
	long	n;
	int		sign;

	n = 0;
	sign = 1;
	if (*s == '-')
		sign = -1;
	if (*s == '+' || *s == '-')
		s++;
	if (*s == '\0')
		return (0);
	while (*s >= '0' && *s <= '9')
	{
		n = n * 10 + (*s++ - '0');
		if ((sign == 1 && n > INT_MAX) || (sign == -1 && -n < INT_MIN))
			return (0);
	}
	if (*s != '\0')
		return (0);
	*out = (int)(n * sign);
	return (1);
}
```

Devuelve `0` si la cadena no es válida: vacía, solo un signo, con letras o fuera de rango. Quien la llama libera todo, escribe `Error\n` en el fd 2 y sale.

### Duplicados

Con todos los números ya convertidos, compara cada par (dos bucles, O(n²) en CPU, sin importancia). `-0` y `0` dan el mismo `int`, así que se detectan solos.

## 4. Normalizar a índices

El índice de cada número = **cuántos números son menores que él**:

```c
static int	*normalize(int *values, int n)
{
	int	*index;
	int	i;
	int	j;

	index = malloc(sizeof(int) * n);
	if (!index)
		return (NULL);
	i = 0;
	while (i < n)
	{
		index[i] = 0;
		j = 0;
		while (j < n)
		{
			if (values[j] < values[i])
				index[i]++;
			j++;
		}
		i++;
	}
	return (index);
}
```

```text
values:  -50  1000   3   42
index:     0     3   1    2      (1000 tiene 3 menores: -50, 3 y 42)
```

Funciona porque no hay duplicados. Además, el desorden se puede calcular sobre los índices o sobre los valores: da lo mismo, porque normalizar conserva el orden.

## 5. Las operaciones

Tres funciones básicas sobre un array cubren las 11 operaciones:

```c
static void	rotate(int *t, int size)
{
	int	first;
	int	i;

	if (size < 2)
		return ;
	first = t[0];
	i = 0;
	while (i < size - 1)
	{
		t[i] = t[i + 1];
		i++;
	}
	t[size - 1] = first;
}
```

| Operación | Con qué |
|---|---|
| `ra` / `rb` | `rotate(a, size_a)` |
| `rra` / `rrb` | `reverse_rotate` (el simétrico: guarda el último y desplaza hacia la derecha) |
| `sa` / `sb` | Intercambiar `t[0]` y `t[1]` si `size ≥ 2` |
| `pa` / `pb` | Si el origen no está vacío: hacer hueco arriba del destino, copiar, cerrar el hueco del origen |
| `ss`, `rr`, `rrr` | Llamar a las dos básicas |

> [!tip] Una sola puerta: `op()`
> Haz que **todas** las estrategias muevan las pilas solo mediante `op(s, PB)`, `op(s, RR)`… Esa función ejecuta el movimiento, suma `s->counts[PB]++` e imprime el nombre. Así el bench no puede desincronizarse, y es trivial cambiar la salida en una modificación en directo.

> [!warning] `rr` imprime una vez
> `op(s, RR)` debe rotar las dos pilas e imprimir **una** línea `rr`, no `ra` y `rb`. Por eso las funciones básicas no imprimen nada: solo `op()` lo hace.

Sobre el buffer de salida: con `write` directo de cada operación, 5 000 llamadas al sistema siguen siendo instantáneas. Si quieres optimizar, mira [[Buffer de stdout]].

## 6. Elegir y ejecutar la estrategia

```text
si a ya está ordenada                 → no imprimir nada
según config.strategy:
    SIMPLE   → simple(s)
    MEDIUM   → medium(s)
    COMPLEX  → complex(s)
    ADAPTIVE → mirar el desorden y elegir el tramo
```

Detalles de cada una en [[push_swap - Estrategias]].

## 7. El modo `--bench`

Al terminar, si `config.bench` está activo, escribe en el **fd 2**:

```text
[bench] disorder:  49.93%
[bench] strategy:  Adaptive / O(n√n)
[bench] total_ops:  7997
[bench] sa:  0  sb:  0  ss:  0  pa:  500  pb:  500
[bench] ra:  4840  rb:  1098  rr:  0  rra:  0  rrb:  1059  rrr:  0
```

- El desorden se calcula **al principio** (antes de mover nada) y se guarda.
- El porcentaje con dos decimales, sin `float`: [[Decimales con enteros]].
- En la adaptativa, muestra el tramo que se usó de verdad (el ejemplo del subject pone `Adaptive / O(n√n)`).
- `√` es un carácter UTF-8 de 3 bytes: con `write` o `ft_printf("%s", "√")` sale bien; si os da problemas, `O(n*sqrt(n))` también se entiende.

> [!warning] Nada del bench a stdout
> Si una sola línea del bench sale por stdout, el checker la leerá como una instrucción y dirá `Error`/`KO`.

## 8. Construirlo por etapas

| Etapa | Qué haces | Cómo lo pruebas |
|---|---|---|
| 1 | Makefile + libft + `main` vacío | `make`, `make` otra vez (sin relink), `make fclean` |
| 2 | Parseo y errores | Toda la tabla de errores de [[push_swap - Índice#Entrada, salida y errores]] y `valgrind` en cada una |
| 3 | Operaciones + `op()` | Un `main` de prueba que imprima las pilas después de cada op |
| 4 | `is_sorted` + 2/3/5 números | Todas las permutaciones con `checker_linux` |
| 5 | Normalizar + desorden | Comparar con los ejemplos: `13.33%`, `40.00%` |
| 6 | Simple | Ordena (aunque sean muchas ops) con 100 y 500 |
| 7 | Complex (radix) | < 1 100 con 100, < 7 000 con 500 |
| 8 | Medium (chunks) | Pasa los límites con 100 y 500 |
| 9 | Adaptativa + flags | Cada flag con entradas de todos los tamaños y desórdenes |
| 10 | `--bench` | Salida por fd 2 y contadores que suman el total |
| 11 | Optimizar (coste) | Peor de 100 pruebas < 700 / < 5 500 |
| 12 | README + checker | [[push_swap - Defensa y evaluación]] · [[push_swap - checker (bonus)]] |

Los scripts para cada etapa están en [[push_swap - Pruebas y depuración]].

## 9. Liberar siempre

| Ruta | Qué hay que liberar |
|---|---|
| Error al parsear | El array de valores (y los resultados de `ft_split` si lo usas) |
| Error en un `malloc` intermedio | Todo lo reservado hasta ese momento |
| Final normal | `a`, `b`, los arrays auxiliares |

Una función `free_all(t_stacks *s)` que acepte punteros `NULL` simplifica todos los casos (ver [[Control de errores en C]]).

## Relacionadas

[[push_swap - Índice]] · [[push_swap - Estrategias]] · [[push_swap - Pruebas y depuración]] · [[argc y argv]] · [[enum en C]] · [[Pilas y colas]] · [[Decimales con enteros]] · [[Makefile y librerías estáticas]] · [[La Norma]]

## Referencias

- Subject de push_swap v1.0, capítulos II, III, VI.5 y VII.
- Fragmentos de código (`parse_int`, `normalize`, `rotate`, `radix_sort`): compilados en un prototipo con `cc -Wall -Wextra -Werror` (Ubuntu clang 12.0.1). El prototipo reproduce la salida del ejemplo `--simple 5 4 3 2 1` del subject y pasa Valgrind sin errores.
- Formato de `--bench`: ejemplo del subject v1.0, VI.5.1.
