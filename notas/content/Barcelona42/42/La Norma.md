---
tags: [42, norma, c]
---
# La Norma (versión 4)

[[Inicio|← Inicio]] · 42

> Resumen de la Norma de 42. Se aplica a todos los proyectos de C del Common Core. La comprueba la herramienta `norminette` (salvo el Makefile y lo que debe revisar un humano).

> [!info] Comprobarla
> ```bash
> norminette              # todos los .c y .h de la carpeta actual
> norminette ft_split.c   # un archivo concreto
> ```
> Un archivo que **no compila** no se considera que pase la Norma.

## 1. Límites

| Qué | Límite |
|---|---|
| Líneas por función | **25**, sin contar las llaves de la función |
| Parámetros por función | **4** (sin parámetros → `void`) |
| Variables declaradas por función | **5** |
| Funciones definidas por archivo `.c` | **5** |
| Ancho de línea | **80 columnas**, comentarios incluidos. Un tab cuenta como los espacios que ocupa, no como 1 |
| Instrucciones por línea | **1** |

## 2. Formato e indentación

- **Indentación con tabulaciones reales** (de 4 espacios de ancho), nunca con espacios.
- **Espacios:**
  - Nada de espacios ni tabs al final de una línea. Una línea vacía debe estar realmente vacía.
  - Nunca dos espacios seguidos.
  - Después de `,` y `;` va un espacio (salvo a final de línea).
  - Cada operador y operando separados por **un** espacio: `a + b`, `i = 0`.
  - Después de una palabra clave de C va un espacio (`if (`, `while (`, `return (`), excepto en los tipos (`int`, `char`…) y en `sizeof`.
- **Llaves:** salto de línea antes y después de cada llave de función, declaración o estructura de control. Las estructuras de control llevan llaves, salvo que contengan una sola línea.
- **Líneas largas:** puedes partirlas indentando la continuación; los **operadores van al principio** de la nueva línea.
- **Funciones:**
  - Separadas entre sí por **una línea vacía**.
  - **Un tab** entre el tipo de retorno y el nombre: `int	ft_strlen(...)`.
  - Los parámetros de los prototipos deben tener nombre.
  - El valor de retorno va entre paréntesis: `return (valor);` o `return ;` en una `void`.

## 3. Variables

- **Declaraciones al principio** de la función, **una por línea**, con los nombres **alineados en la misma columna** (con tabs).
- **No se declara e inicializa en la misma línea**, salvo globales (si se permiten), `static` y constantes.
- **Una línea vacía** entre las declaraciones y el resto del código. **No se permite ninguna otra** línea vacía dentro de la función.
- El `*` de los punteros va **pegado al nombre**: `char	*str;`.
- **Prohibidas las asignaciones múltiples**: `a = b = 0;`.

```c
int	ft_count(char const *s, char c)
{
	int		count;
	size_t	i;

	count = 0;
	i = 0;
	while (s[i])
	{
		if (s[i] == c)
			count++;
		i++;
	}
	return (count);
}
```

## 4. Nombres

| Elemento | Prefijo | Ejemplo |
|---|---|---|
| `struct` | `s_` | `struct s_list` |
| `typedef` | `t_` | `t_list` |
| `union` | `u_` | `u_value` |
| `enum` | `e_` | `e_state` |
| Variable global | `g_` | `g_count` |

- Variables, funciones, archivos y directorios en **snake_case**: minúsculas, dígitos y `_`. Nada de mayúsculas.
- Solo caracteres **ASCII estándar**.
- Todos los identificadores **en inglés** y con nombres **explícitos** (nada de `f1`, `a`, `b`…).
- **Variables globales**: prohibidas salvo que sean `const` o `static`, o que el subject las permita expresamente.

## 5. `typedef`, `struct`, `enum` y `union`

- Un tab al declarar un `struct`, `enum` o `union`, y otro antes del nombre del `typedef`.
- Los nombres de los campos, alineados en la misma columna.
- **No se puede declarar un `struct` en un archivo `.c`**: van en el `.h`.

```c
typedef struct s_list
{
	void			*content;
	struct s_list	*next;
}	t_list;
```

## 6. Headers (`.h`)

- Solo pueden contener: **includes, declaraciones, defines, prototipos y macros**.
- Todos los `#include` **al principio** del archivo, y cada uno debe estar **justificado** (prohibido incluir algo que no se usa).
- **Nunca se incluye un `.c`**.
- **Protección contra doble inclusión**: si el archivo es `ft_foo.h`, la macro es `FT_FOO_H`.
- Lo que va dentro de `#ifndef`/`#if` se indenta con un espacio tras el `#`.

```c
#ifndef LIBFT_H
# define LIBFT_H

# include <stdlib.h>

size_t	ft_strlen(const char *s);

#endif
```

## 7. Cabecera de 42

- Todo `.c` y `.h` **empieza** con la cabecera estándar de 42: login, correo, fecha de creación y de última actualización.
- Se inserta con `:Stdheader` o `F1` en Vim, o con `C-c C-h` en Emacs, y se actualiza sola al guardar. Ver [[Vim]].

## 8. Macros y preprocesador

- `#define` **solo para valores literales y constantes**. Prohibidos los que sirven para saltarse la Norma u ofuscar el código.
- Nombres de macros **en MAYÚSCULAS**.
- **Prohibidas las macros de varias líneas**.
- Las directivas del preprocesador solo en el ámbito global (nunca dentro de una función).

## 9. Comentarios

- **Prohibidos dentro del cuerpo de las funciones.** Fuera de ellas pueden ir en su propia línea o al final de una línea.
- En **inglés** y útiles. Un comentario no justifica una función confusa.

## 10. Prohibido

- `for`, `do…while`, `switch`, `case`, `goto`.
- Operador ternario `? :`.
- **VLA** (arrays de longitud variable): `char buf[n];`.
- Tipos implícitos en las declaraciones.

## 11. Makefile

La `norminette` no revisa el Makefile: lo revisa quien te evalúa.

- Reglas obligatorias: `$(NAME)`, `all`, `clean`, `fclean`, `re`.
- **Si hace relink, el proyecto se considera no funcional.**
- Si usas una librería propia (p. ej. libft), el Makefile debe compilarla automáticamente.
- **Todos los archivos fuente nombrados explícitamente** (prohibido `*.c` / `$(wildcard ...)`).
- Proyectos con varios binarios: una regla que compile todos y otra para cada uno.

Más detalle en [[Makefile y librerías estáticas]].

## Relacionadas

[[Arrays en C]] · [[Control de errores en C]] · [[Vim]] · [[Makefile y librerías estáticas]]

---
*Fuente: La Norma, versión 4 (PDF oficial de 42, en español): [[Normativa 42/es.norm-2.pdf|es.norm-2.pdf]]. Repositorio de la norminette: https://github.com/42School/norminette*
