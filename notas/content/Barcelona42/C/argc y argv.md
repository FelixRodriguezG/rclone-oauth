---
tags: [c, fundamentos, argumentos, main, push-swap]
aliases: ["argc", "argv", "Argumentos de main", "Argumentos de la línea de comandos"]
---
# argc y argv

[[Inicio|← Inicio]] · C · Fundamentos

> Cuando ejecutas `./push_swap --simple 3 2 1`, el sistema le pasa a `main` las palabras que has escrito. **`argc`** dice cuántas hay y **`argv`** es un array de cadenas con cada una. Es la puerta de entrada de cualquier programa de 42 que reciba datos por la línea de comandos.

## Vocabulario

| Término | Qué significa |
|---|---|
| **`argc`** (*argument count*) | Número de argumentos, **incluido el nombre del programa** |
| **`argv`** (*argument vector*) | Array de `char *`: cada elemento es una cadena terminada en `'\0'` |
| **`argv[0]`** | El nombre con el que se ejecutó el programa (`"./push_swap"`) |
| **`argv[argc]`** | Siempre `NULL`: marca el final del array |
| **Flag** | Argumento que empieza por `-` o `--` y cambia el comportamiento (`--bench`) |

## La firma de `main`

```c
int	main(int argc, char **argv)
```

`char **argv` es un [[Puntero a puntero]]: apunta a un array de punteros, y cada uno apunta a una cadena.

```text
./push_swap --simple 3 "-1 2"

argc = 4

argv ──► [0] ──► "./push_swap\0"
         [1] ──► "--simple\0"
         [2] ──► "3\0"
         [3] ──► "-1 2\0"        ← la shell respeta las comillas: UN argumento
         [4] ──► NULL
```

## Ejemplo: recorrer y detectar flags

Este programa imprime cada argumento y marca los que son flags. `str_eq` compara dos cadenas completas (como `strcmp(...) == 0`).

```c
#include <stdio.h>

static int	str_eq(const char *a, const char *b)
{
	while (*a && *a == *b)
	{
		a++;
		b++;
	}
	return (*a == *b);
}

int	main(int argc, char **argv)
{
	int	i;

	printf("argc = %d\n", argc);
	i = 0;
	while (i < argc)
	{
		if (str_eq(argv[i], "--simple") || str_eq(argv[i], "--bench"))
			printf("argv[%d] = \"%s\"  <- flag\n", i, argv[i]);
		else
			printf("argv[%d] = \"%s\"\n", i, argv[i]);
		i++;
	}
	printf("argv[%d] = %p\n", argc, (void *)argv[argc]);
	return (0);
}
```

```text
$ cc -Wall -Wextra -Werror argv.c -o argv && ./argv --simple 3 "-1 2" --bench
argc = 5
argv[0] = "./argv"
argv[1] = "--simple"  <- flag
argv[2] = "3"
argv[3] = "-1 2"
argv[4] = "--bench"  <- flag
argv[5] = (nil)
```

> [!tip] Compara la cadena entera
> Con [[ft_strncmp]]`(argv[i], "--simple", 8)`, `"--simplezzz"` también pasaría, porque solo mira 8 caracteres. Compara hasta el `'\0'`: con una función como `str_eq` o con `ft_strncmp(a, b, ft_strlen(b) + 1)`.

## Lo que hace la shell antes que tu programa

Tu programa **no** ve lo que escribes tal cual; ve lo que queda después de que la shell lo procese:

| Escribes | `argv` recibe | Motivo |
|---|---|---|
| `./p 1 2 3` | `"1"`, `"2"`, `"3"` | Separa por espacios |
| `./p "1 2 3"` | `"1 2 3"` | Las comillas lo agrupan en uno |
| `./p ""` | `""` | Un argumento **vacío**, pero existe |
| `./p $ARG` (bash) | Un argumento por palabra | Bash separa la variable |
| `./p $ARG` (zsh) | **Uno** solo | zsh no separa variables: usa `${=ARG}` |

> [!warning] zsh y `$ARG`
> Los ejemplos de los subjects (`ARG="4 67 3"; ./push_swap $ARG`) asumen bash. En zsh, `$ARG` llega como **un solo** argumento `"4 67 3"`. Si tu programa no separa por espacios, dará `Error`. Prueba con `bash` o con `${=ARG}`.

## Patrón para push_swap

En [[push_swap - Índice|push_swap]] los argumentos mezclan flags y números. Una forma limpia de hacerlo:

1. Recorre `argv` desde `1`.
2. Si el argumento empieza por `--`, es un flag: compáralo con `--simple`, `--medium`, `--complex`, `--adaptive` y `--bench`. Si no es ninguno, es un error.
3. Si no, es un número (o varios, si aceptas `"3 2 1"`): sepáralo con [[ft_split]] y conviértelo.

> [!question] ¿Y si llegan dos flags de estrategia (`--simple --complex`)?
> El subject no lo dice. Decidid una regla (error, o gana la última) y dejadla escrita en el README.

## Errores frecuentes

> [!danger] Leer `argv[1]` sin mirar `argc`
> Si el programa se ejecuta sin argumentos, `argv[1]` es `NULL`, y leer `argv[1][0]` provoca un segfault. Comprueba `argc` primero.

> [!warning] Tratar `argv[0]` como un dato
> El primer número de push_swap está en `argv[1]`, no en `argv[0]`.

> [!warning] Modificar las cadenas de `argv`
> Puedes leerlas, pero si necesitas cambiarlas, haz una copia con [[ft_strdup]].

## Relacionadas

[[Puntero a puntero]] · [[Arrays en C]] · [[Carácter nulo y puntero NULL]] · [[ft_split]] · [[ft_strncmp]] · [[Control de errores en C]] · [[Segmentos de memoria en C]] · [[push_swap - Guía paso a paso]]

## Referencias

- [cppreference: Main function](https://en.cppreference.com/w/c/language/main_function) — `argc`, `argv` y que `argv[argc]` es un puntero nulo.
- [Bash Reference Manual: Word Splitting](https://www.gnu.org/software/bash/manual/html_node/Word-Splitting.html) — por qué `$ARG` se separa en bash.
- [zsh FAQ 3.1: Why does `$var` where `var="foo bar"` not do what I expect?](https://zsh.sourceforge.io/FAQ/zshfaq03.html) — por qué en zsh no se separa.
- Salida del ejemplo: compilado con Ubuntu clang 12.0.1.
