---
tags: [proyecto, get_next_line, utils, cadenas, rank-1, principiante]
aliases: ["get_next_line_utils", "Utils de get_next_line", "Helpers de GNL"]
---
# get_next_line - Funciones auxiliares

[[Inicio|← Inicio]] · [[get_next_line - Índice|get_next_line]] · [[get_next_line - Guía paso a paso|Guía]] · Funciones auxiliares

> En get_next_line **no puedes usar tu libft**, así que las funciones de cadenas que necesites las vuelves a escribir en `get_next_line_utils.c`. No son copias exactas de las de libft: están adaptadas al stash, que **empieza valiendo `NULL`** y que se **sustituye** por uno nuevo en cada vuelta. Esta nota explica cada una, por qué cambia respecto a libft y cómo probarla sola.

> [!info] Antes de leer
> Esta nota es la etapa 2 de [[get_next_line - Guía paso a paso#6. Construirlo por etapas|la guía]]. Conviene tener claro qué es un puntero ([[Punteros en C]]), la diferencia entre `'\0'` y `NULL` ([[Carácter nulo y puntero NULL]]) y quién libera cada bloque de memoria ([[malloc y free]]).

## Resumen

| Función | Prototipo | Diferencia con libft |
|---|---|---|
| `ft_strlen` | `size_t ft_strlen(const char *s);` | Devuelve `0` si `s` es `NULL` (la de libft crashearía) |
| `ft_strchr` | `char *ft_strchr(const char *s, int c);` | Devuelve `NULL` si `s` es `NULL` |
| `ft_strndup` | `char *ft_strndup(const char *s, size_t n);` | No está en libft: copia **solo** los `n` primeros caracteres |
| `ft_strjoin_free` | `char *ft_strjoin_free(char *stash, const char *buf);` | **Libera el primer argumento** y acepta que sea `NULL` |
| `ft_free` | `char *ft_free(char **ptr);` | No está en libft: libera **y** pone el puntero a `NULL` |

Cinco funciones: justo el máximo por fichero que permite [[La Norma]]. Los nombres son tuyos; lo que importa es lo que hace cada una.

> [!note] ¿Por qué tantas funciones que aceptan `NULL`?
> En la primera llamada, el stash es `NULL` (una `static` empieza a `0`). Si tus funciones crashean con `NULL`, tendrías que llenar `get_next_line` de `if (stash == NULL)`. Haciendo que las utilidades lo aguanten, el código principal queda mucho más simple.

---

## `ft_strlen`: longitud, aguantando `NULL`

```c
size_t	ft_strlen(const char *s)
{
	size_t	i;

	i = 0;
	if (!s)
		return (0);
	while (s[i])
		i++;
	return (i);
}
```

- `while (s[i])` es lo mismo que `while (s[i] != '\0')`: avanza hasta el terminador.
- La única novedad es el `if (!s) return (0);`. Un stash `NULL` es como una cadena vacía: mide 0.
- Se usa para saber **cuánta memoria reservar** al pegar o copiar.

```text
s ──► [ h | o | l | a | \0 ]
        0   1   2   3   4     → devuelve 4 (el '\0' no cuenta)

s = NULL                      → devuelve 0 (sin tocar memoria)
```

## `ft_strchr`: ¿hay un `'\n'` en el stash?

```c
char	*ft_strchr(const char *s, int c)
{
	if (!s)
		return (NULL);
	while (*s)
	{
		if (*s == (char)c)
			return ((char *)s);
		s++;
	}
	if ((char)c == '\0')
		return ((char *)s);
	return (NULL);
}
```

Devuelve la **dirección** del primer carácter `c` dentro de `s`, o `NULL` si no está.

```text
s ──► [ a | b | \n | c | \0 ]
                ▲
ft_strchr(s, '\n') devuelve esta dirección (s + 2)
```

En get_next_line se usa para dos cosas:

1. **Como pregunta sí/no**: `while (... && !ft_strchr(stash, '\n'))` → «mientras no haya salto de línea, sigue leyendo». Si el stash es `NULL`, devuelve `NULL`, que es «no hay `'\n'`»: perfecto para la primera llamada.
2. **Para saber dónde cortar**: en `ft_save_rest`, `nl = ft_strchr(stash, '\n')` te da la posición del salto, y `nl + 1` es donde empieza el resto.

- `(char)c`: el parámetro es `int` por compatibilidad con la `strchr` estándar; se convierte a `char` para comparar.
- `(char *)s`: `s` es `const char *` (prometemos no modificarlo), pero la función devuelve `char *`. El *cast* quita el `const`, igual que hace la `strchr` de la biblioteca estándar.
- Buscar `'\0'` devuelve la dirección del terminador. En GNL no lo usas, pero mantiene el comportamiento de `strchr`.

## `ft_strndup`: copiar un trozo

```c
char	*ft_strndup(const char *s, size_t n)
{
	char	*dst;
	size_t	i;

	dst = malloc(n + 1);
	if (!dst)
		return (NULL);
	i = 0;
	while (i < n && s[i])
	{
		dst[i] = s[i];
		i++;
	}
	dst[i] = '\0';
	return (dst);
}
```

Crea una cadena **nueva** con, como mucho, los `n` primeros caracteres de `s`.

- `malloc(n + 1)`: `n` caracteres y uno más para el `'\0'`.
- `i < n && s[i]`: para si ya ha copiado `n` o si `s` se acaba antes.
- **Siempre** termina con `'\0'`.

La usas dos veces:

```text
stash ──► [ a | b | \n | c | \0 ]

ft_strndup(stash, 3)              → nuevo [ a | b | \n | \0 ]    (la línea)
ft_strndup(nl + 1, ft_strlen(nl + 1)) → nuevo [ c | \0 ]          (el resto)
```

> [!tip] Alternativa: `ft_substr`
> Si prefieres reutilizar la idea de [[ft_substr]] (`ft_substr(s, start, len)`), también vale: la línea es `ft_substr(stash, 0, len)` y el resto `ft_substr(stash, len, ft_strlen(stash) - len)`. Elige la que sepas explicar mejor.

## `ft_strjoin_free`: pegar lo leído al stash

Es la función más delicada. Hace lo mismo que [[ft_strjoin]] (crea una cadena nueva `s1 + s2`), con dos cambios:

1. Acepta que el primer argumento sea `NULL` (primera vuelta).
2. **Libera el primer argumento** después de copiarlo.

```c
char	*ft_strjoin_free(char *stash, const char *buf)
{
	char	*res;
	size_t	i;
	size_t	j;

	res = malloc(ft_strlen(stash) + ft_strlen(buf) + 1);
	if (!res)
		return (ft_free(&stash));
	i = 0;
	while (stash && stash[i])
	{
		res[i] = stash[i];
		i++;
	}
	j = 0;
	while (buf[j])
		res[i++] = buf[j++];
	res[i] = '\0';
	free(stash);
	return (res);
}
```

### Por qué tiene que liberar el stash viejo

Se llama así:

```c
stash = ft_strjoin_free(stash, buf);
```

El valor viejo de `stash` se **sobrescribe** con la dirección nueva. Si nadie hizo `free` del bloque viejo antes, ya no queda ningún puntero hacia él: es un **leak** (ver [[malloc y free#El patrón p = f(p)]]).

```text
ANTES                                     DESPUÉS
stash ──► [ c | \0 ]  (bloque A)           bloque A  → liberado dentro de la función ✅
buf   ──► [ d | e | f | \n | \0 ]          stash ──► [ c | d | e | f | \n | \0 ]  (bloque B, nuevo)
```

Con el `ft_strjoin` de libft (que no libera), Valgrind lo ve enseguida. Fichero de 3 líneas, `BUFFER_SIZE=3`:

```text
==779079==    definitely lost: 6 bytes in 2 blocks
==779079== ERROR SUMMARY: 2 errors from 2 contexts (suppressed: 0 from 0)
```

### Paso a paso

```text
stash = "c"   buf = "def\n"

1. malloc(1 + 4 + 1) = 6 bytes      res ──► [ ? | ? | ? | ? | ? | ? ]
2. copia stash                       res ──► [ c | ? | ? | ? | ? | ? ]      i = 1
3. copia buf detrás                  res ──► [ c | d | e | f | \n| ? ]      i = 5
4. res[5] = '\0'                     res ──► [ c | d | e | f | \n| \0 ]
5. free(stash)                       el bloque "c" se devuelve al sistema
6. return (res)
```

- `while (stash && stash[i])`: si `stash` es `NULL`, la condición es falsa a la primera y no se copia nada. El `&&` es importante: **primero** comprueba que no es `NULL` y solo entonces mira `stash[i]`.
- `free(stash)` con `stash == NULL` es legal y no hace nada (lo garantiza el estándar de C).
- Si `malloc` falla, también libera el stash viejo y devuelve `NULL`: así quien llama no tiene que acordarse de hacerlo.

> [!warning] Solo libera el primero
> `buf` **no** se libera aquí: es el buffer de lectura, que se reutiliza en cada vuelta y lo libera `ft_read_to_stash` al terminar.

## `ft_free`: liberar y dejar el puntero a `NULL`

```c
char	*ft_free(char **ptr)
{
	free(*ptr);
	*ptr = NULL;
	return (NULL);
}
```

Hace tres cosas en una línea de código para quien la llama:

1. Libera el bloque.
2. Pone el puntero **original** a `NULL`, para que no quede apuntando a memoria liberada.
3. Devuelve `NULL`, para poder escribir `return (ft_free(&stash));` en los caminos de error.

### ¿Por qué `char **` y no `char *`?

Porque en C los argumentos se pasan **por copia**. Si la función recibe `char *p`, `p` es una copia del puntero: ponerla a `NULL` no cambia el puntero de quien llama. Para cambiar una variable de quien te llama necesitas su **dirección** (ver [[Punteros en C]]).

```c
static void	liberar_mal(char *p)
{
	free(p);
	p = NULL;          /* solo cambia la copia */
}

static void	liberar_bien(char **p)
{
	free(*p);
	*p = NULL;         /* cambia el puntero original */
}
```

```text
$ ./ftfree
a = 0x... (sigue apuntando a memoria liberada)
b = (nil)
```

```text
          main                          liberar_bien
   ┌───────────────┐              ┌──────────────────┐
   │ b  ───────────┼──► bloque    │ p ──► &b         │
   └───────────────┘              └──────────────────┘
          ▲                              │
          └──── *p = NULL escribe aquí ──┘
```

En get_next_line se usa como `ft_free(&stash)`: después de la llamada, el stash (la `static`) vale `NULL`, y la próxima llamada empezará limpia.

## Probar las utilidades solas

Antes de juntar nada, comprueba cada una con un `main` pequeño:

```c
#include <stdio.h>
#include "get_next_line.h"

int	main(void)
{
	char	*s;

	printf("strlen(NULL) = %zu\n", ft_strlen(NULL));
	printf("strchr(NULL) = %p\n", (void *)ft_strchr(NULL, '\n'));
	printf("strchr(\"ab\\ncd\") = \"%s\"\n", ft_strchr("ab\ncd", '\n') + 1);
	s = ft_strndup("ab\ncd", 3);
	printf("strndup(3) = \"%s\"\n", s);
	free(s);
	s = ft_strjoin_free(NULL, "hola");
	s = ft_strjoin_free(s, " mundo");
	printf("join = \"%s\"\n", s);
	ft_free(&s);
	printf("tras ft_free = %p\n", (void *)s);
	return (0);
}
```

```text
$ cc -Wall -Wextra -Werror get_next_line_utils.c test_utils.c -o tu && ./tu
strlen(NULL) = 0
strchr(NULL) = (nil)
strchr("ab\ncd") = "cd"
strndup(3) = "ab
"
join = "hola mundo"
tras ft_free = (nil)
```

Y con Valgrind: debe acabar en `All heap blocks were freed -- no leaks are possible`. Ojo: `ft_strjoin_free(s, " mundo")` libera el `"hola"` anterior; si lo hubieras liberado tú antes, sería un *double free*.

## Errores frecuentes

> [!warning] `ft_strchr` o `ft_strlen` sin comprobar `NULL`
> En la primera llamada el stash es `NULL`: segfault inmediato.

> [!warning] Usar `ft_strjoin` de libft
> El stash viejo nunca se libera: un leak por cada vuelta del bucle de lectura.

> [!warning] Liberar `buf` dentro del join
> `buf` se reutiliza en la siguiente vuelta: usarías memoria liberada (*use after free*).

> [!warning] Olvidar el `'\0'` en `ft_strndup`
> La línea devuelta no es una cadena: el `printf` del `main` imprime basura.

> [!warning] Más de 5 funciones en `get_next_line_utils.c`
> Error de norma. Si te sobran, mueve alguna a `get_next_line.c` como `static`.

## Relacionadas

[[get_next_line - Guía paso a paso]] · [[get_next_line - Pruebas y depuración]] · [[get_next_line - Índice]] · [[malloc y free]] · [[Punteros en C]] · [[Carácter nulo y puntero NULL]] · [[ft_strlen]] · [[ft_strchr]] · [[ft_strjoin]] · [[ft_substr]] · [[ft_strdup]]

## Referencias

- Subject de get_next_line v14.0: prohíbe libft y permite añadir las funciones de ayuda necesarias en `get_next_line_utils.c`.
- [strchr(3), man7.org](https://man7.org/linux/man-pages/man3/strchr.3.html): comportamiento de referencia, incluida la búsqueda de `'\0'`.
- [free(3), man7.org](https://man7.org/linux/man-pages/man3/free.3.html): «If ptr is NULL, no operation is performed».
- La Norma v4: máximo 5 funciones por fichero y 25 líneas por función.
- Código compilado con `cc -Wall -Wextra -Werror` (Ubuntu clang 12.0.1, Ubuntu 22.04) y comprobado con norminette y Valgrind. Las salidas son reales.
