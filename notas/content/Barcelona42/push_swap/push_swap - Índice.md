---
tags: [proyecto, push-swap, indice, rank-1, algoritmos]
aliases: ["push_swap"]
---
# push_swap - Índice

[[Inicio|← Inicio]] · Proyectos · Rank 1

> **push_swap** ordena una lista de enteros usando dos pilas (`a` y `b`) y un juego de 11 operaciones. El programa no imprime los números ordenados: imprime **la lista de operaciones** que los ordena, y cuantas menos, mejor nota.

> [!warning] Comprueba el subject
> Resumen basado en versiones públicas del subject; los detalles cambian entre versiones. Lo que manda es el PDF de tu intra.

> [!info] Rank 1
> En el nuevo Common Core (2025), push_swap es uno de los tres proyectos **obligatorios** del rank 1, junto con [[ft_printf - Índice|ft_printf]] y [[get_next_line - Índice|get_next_line]]. No hay que elegir entre proyectos: hay que hacer los tres.

## Conceptos que necesitas

| Concepto | Para qué en push_swap |
|---|---|
| [[Algoritmos de ordenación y complejidad]] | Elegir algoritmo y entender por qué uno da 700 operaciones y otro 1100 |
| [[Listas enlazadas]] o [[Arrays en C]] | Representar cada pila |
| [[Structs y typedef]] | Guardar las dos pilas y su tamaño en un `t_stack` |
| [[Operadores de bits]] | Radix sort mira los bits de cada número |
| [[Tipos enteros y conversiones]] | Detectar overflow al convertir `"2147483648"` |
| [[ft_atoi]] · [[ft_split]] | Parsear argumentos |
| [[Control de errores en C]] · [[Valgrind y sanitizers]] | Liberar todo al salir con `Error` |

## Vocabulario

| Término | Significado |
|---|---|
| Pila (*stack*) | Estructura en la que solo se trabaja con el elemento de arriba (*top*) |
| Top | Primer elemento de la pila. En `./push_swap 2 1 3`, el top de `a` es `2` |
| Rotar | Mover todos los elementos una posición: el top pasa al fondo (o al revés) |
| Índice normalizado | Posición que tendría cada número una vez ordenado (0, 1, 2…). Simplifica los algoritmos |
| Coste | Número de operaciones necesarias para colocar un elemento en su sitio |

## Las dos pilas y las 11 operaciones

Al empezar, `a` tiene los números en el orden de los argumentos (el primero arriba) y `b` está vacía. El objetivo: `a` ordenada de menor (arriba) a mayor, `b` vacía.

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
pb   →  a = 1 3 6 5 8                 b = 2
ra   →  a = 1 3 6 5 8 2               b = (vacía)
rra  →  a = 8 2 1 3 6 5               b = (vacía)
```

> [!tip] Las dobles valen una
> `rr`, `rrr` y `ss` cuentan como **una** operación. Si en un paso necesitas `ra` y `rb`, imprime `rr`: es la forma más barata de ahorrar operaciones en el algoritmo de coste.

## Entrada, salida y errores

- Entrada: los números como argumentos (`./push_swap 3 2 1`). Muchas versiones también aceptan una sola cadena (`./push_swap "3 2 1"`), que separas con [[ft_split]]; verifica en tu subject si es obligatorio.
- Salida: una operación por línea en `stdout` (`sa\n`, `pb\n`…). Nada más.
- Sin argumentos: no imprime nada y termina.
- Ya ordenado: no imprime nada.
- Error: imprime `Error\n` en **stderr** (fd 2) y termina.

| Entrada | ¿Error? | Por qué |
|---|---|---|
| `./push_swap 1 2 a` | ✅ Error | `a` no es un número |
| `./push_swap 1 2 2` | ✅ Error | Duplicado |
| `./push_swap 2147483648` | ✅ Error | Mayor que `INT_MAX` |
| `./push_swap -2147483649` | ✅ Error | Menor que `INT_MIN` |
| `./push_swap 1 + 2` · `./push_swap 1 - 2` | ✅ Error | Signo sin dígitos |
| `./push_swap ""` | ✅ Error (habitual) | Argumento vacío |
| `./push_swap -0 0` | ✅ Error (habitual) | Ambos valen 0: duplicado |
| `./push_swap +5 -3` | ❌ Válido | Un signo seguido de dígitos es un número |

> [!warning] `ft_atoi` no detecta overflow
> Tu [[ft_atoi]] de libft devuelve basura con `"99999999999"`. Escribe una versión que convierta a `long` y compruebe el rango **en cada dígito**, antes de que el `long` también desborde con cadenas muy largas.

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

Devuelve `0` si la cadena no es válida; quien la llama imprime `Error`, libera todo y sale. Los duplicados se comprueban después, con todos los números ya convertidos.

## Algoritmos según el tamaño

Usa un algoritmo distinto según cuántos números haya. El paso previo común: **normalizar** los valores a índices (`-50 1000 3` → `0 2 1`). Así trabajas siempre con 0…n-1, sin negativos.

### 2 y 3 números

Con 2: `sa` si están al revés. Con 3 hay 6 órdenes posibles y todos se resuelven en **2 operaciones como máximo**:

| Orden (top primero) | Operaciones |
|---|---|
| 1 2 3 | — |
| 2 1 3 | `sa` |
| 3 2 1 | `sa` `rra` |
| 3 1 2 | `ra` |
| 1 3 2 | `sa` `ra` |
| 2 3 1 | `rra` |

Regla práctica: si el mayor está arriba, `ra`; si está en medio, `rra`; después, si los dos primeros están al revés, `sa`.

### 4 y 5 números

1. Lleva el mínimo arriba (con `ra` o `rra`, lo que sea más corto) y `pb`.
2. Con 5, repite con el siguiente mínimo.
3. Ordena los 3 que quedan en `a`.
4. `pa` (y otro `pa` con 5).

Así 5 números salen en unas 8–12 operaciones.

### Muchos números: radix sort

[Radix sort](https://en.wikipedia.org/wiki/Radix_sort) en base 2 sobre los índices normalizados. Para cada bit, del menos significativo al más:

- Recorre `a` entera una vez: si el bit es `0`, `pb`; si es `1`, `ra`.
- Devuelve todo con `pa`.

```c
static void	radix_sort(t_stacks *s)
{
	int	bit;
	int	i;
	int	size;

	bit = 0;
	size = s->size_a;
	while (!is_sorted(s) && (size - 1) >> bit != 0)
	{
		i = 0;
		while (i++ < size)
		{
			if (((s->a[0] >> bit) & 1) == 0)
				pb(s);
			else
				ra(s);
		}
		while (s->size_b > 0)
			pa(s);
		bit++;
	}
}
```

`(x >> bit) & 1` saca el bit número `bit` de `x` (ver [[Operadores de bits]]). Con 100 números hay 7 bits (99 = `1100011`), y cada pasada cuesta unas `100 + nº de pb` operaciones.

> [!summary] Radix: fácil pero no óptimo
> Es corto, fácil de defender y siempre funciona. Pero ronda **~1100 operaciones con 100 números** y **~6800 con 500**, lo que suele quedar por debajo de la nota máxima. Úsalo para aprobar y pasa a un algoritmo de coste si buscas el 100.

### Muchos números: algoritmo de coste («turk»)

Es el que suele dar la nota máxima. Idea:

1. `pb` dos veces.
2. Mientras `a` tenga más de 3 elementos: para **cada** número de `a`, calcula cuántas operaciones costaría subirlo al top de `a` y subir a la vez a su «hueco» en `b` (`b` se mantiene en orden descendente). El coste combina rotaciones de las dos pilas; si ambas van en el mismo sentido usas `rr`/`rrr`, que cuentan una vez. Mueve el más barato con `pb`.
3. Ordena los 3 que quedan en `a`.
4. Devuelve cada elemento de `b` a `a` colocándolo encima de su sucesor (el menor de `a` que sea mayor que él).
5. Rota `a` hasta que el mínimo quede arriba.

```text
Coste de mover x (posición i en a, hueco j en b):
  arriba en las dos:   max(i, j)                (con rr)
  abajo en las dos:    max(len_a - i, len_b - j) (con rrr)
  mixto:               i + (len_b - j)   o   (len_a - i) + j
Coste(x) = el mínimo de los cuatro
```

Es O(n²) en cálculo, pero eso no importa: lo que se evalúa es el **número de operaciones impresas**, no el tiempo. Más contexto en [[Algoritmos de ordenación y complejidad]].

## Número de operaciones que suele pedirse

> [!warning] No verificado con tu subject
> Estos límites vienen de la hoja de evaluación que circula públicamente. Compruébalos en la intra.

| Tamaño | Límite habitual |
|---|---|
| 3 números | ≤ 3 (se puede en ≤ 2) |
| 5 números | ≤ 12 |
| 100 números | < 700 = nota máxima · < 900 · < 1100 · < 1300 · < 1500 = mínima |
| 500 números | < 5500 = nota máxima · < 7000 · < 8500 · < 10000 · < 11500 = mínima |

## Bonus: checker

Programa `checker` que recibe los mismos números como argumentos y **lee operaciones de stdin** (una por línea, por ejemplo con [[get_next_line - Índice|get_next_line]]). Las ejecuta y escribe:

- `OK` si al final `a` está ordenada y `b` vacía.
- `KO` si no.
- `Error` en stderr si los argumentos son inválidos o llega una operación que no existe (o mal formateada, por ejemplo `sa ` con espacio).

En la intra también te dan un `checker_linux` (y `checker_Mac`) ya compilado para probar tu push_swap.

## Cómo probar

```bash
# 100 números aleatorios distintos entre 1 y 1000
ARG=$(shuf -i 1-1000 -n 100 | tr '\n' ' ')

./push_swap $ARG | wc -l              # cuántas operaciones
./push_swap $ARG | ./checker_linux $ARG   # OK / KO
```

`shuf -i 1-1000 -n 100` saca 100 números **sin repetir** del rango; `tr` cambia los saltos de línea por espacios. `wc -l` cuenta líneas, es decir, operaciones.

Para ver el peor caso, repite muchas veces y quédate con el máximo:

```bash
for i in $(seq 1 50); do
	ARG=$(shuf -i 1-10000 -n 500 | tr '\n' ' ')
	./push_swap $ARG | wc -l
done | sort -n | tail -1
```

Prueba también negativos (`shuf -i 0-2000 -n 100 | awk '{print $1 - 1000}'`), números ya ordenados, uno solo, y todos los casos de error de la tabla. Pasa [[Valgrind y sanitizers|valgrind]] también en los errores.

## Plan por pasos

1. **Estructura**: decide array o [[Listas enlazadas|lista]] y escribe un `t_stacks`.
2. **Parseo y errores**: convertir, comprobar duplicados y rango, liberar todo al fallar. Prueba toda la tabla de errores antes de seguir.
3. **Las 11 operaciones**, cada una imprimiendo su nombre. Haz que las dobles (`rr`) llamen a versiones que no imprimen.
4. **`is_sorted`** y salida inmediata si ya está ordenado.
5. **Casos 2, 3 y 5**.
6. **Normalización** a índices.
7. **Radix** → ya puedes entregar y aprobar.
8. **Algoritmo de coste** si quieres la máxima nota.
9. **Checker** (bonus).

## Errores que suspenden

> [!danger] Lo que más falla en evaluación
> - `Error` en stdout en vez de stderr, o sin `\n`.
> - No detectar overflow (`2147483648`) o `-0 0` como duplicado.
> - Imprimir algo si la lista ya está ordenada o si no hay argumentos.
> - Leaks o segfault en las rutas de error.
> - `sa`/`pa` sobre una pila vacía que hace crash en vez de no hacer nada.
> - Superar el límite de operaciones en algún caso aleatorio (prueba muchos, no uno).
> - No cumplir [[La Norma]] o que el `Makefile` haga relink ([[Makefile y librerías estáticas]]).

## Preguntas de evaluación

> [!question] ¿Por qué has elegido este algoritmo y cuál es su complejidad?
> Radix: O(n·log n) operaciones (n por cada uno de los log₂ n bits). Coste/turk: menos operaciones porque elige siempre el movimiento más barato y aprovecha `rr`/`rrr`.

> [!question] ¿Qué pasa con `./push_swap 1 2 3`? ¿Y sin argumentos?
> No imprime nada en ninguno de los dos casos.

> [!question] ¿Cómo detectas el overflow?
> Convirtiendo a `long` y comparando con `INT_MAX`/`INT_MIN` en cada dígito.

> [!question] ¿Cuántas operaciones necesitas como máximo para 3 números? ¿Y para 5?
> 2 para 3. Para 5, el corrector suele pedir 12 o menos.

> [!question] ¿Por qué normalizas los números?
> Para trabajar con índices 0…n-1: sin negativos, y radix solo necesita log₂ n bits.

## Relacionadas

[[Algoritmos de ordenación y complejidad]] · [[Operadores de bits]] · [[Listas enlazadas]] · [[Tipos enteros y conversiones]] · [[ft_atoi]] · [[ft_split]] · [[get_next_line - Índice]] · [[Valgrind y sanitizers]] · [[libft - Índice]]

## Referencias

- [Radix sort — Wikipedia](https://en.wikipedia.org/wiki/Radix_sort) — idea de ordenar por dígitos (aquí, bits) del menos al más significativo.
- [shuf(1) — man7.org](https://man7.org/linux/man-pages/man1/shuf.1.html) — `-i` (rango) y `-n` (cantidad) para generar números sin repetir.
- [wc(1) — man7.org](https://man7.org/linux/man-pages/man1/wc.1.html) — `-l` para contar líneas.
- Operaciones, formato de salida y errores: versiones públicas del subject de push_swap (no se ha consultado el PDF de la intra).
- Límites de operaciones: hoja de evaluación pública de push_swap (no verificada).
