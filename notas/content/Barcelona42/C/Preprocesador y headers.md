---
tags: [c, preprocesador, headers, define, include, norma]
aliases: ["Headers en C", "#define e #include"]
---
# Preprocesador y headers

[[Inicio|← Inicio]] · C

> El **preprocesador** es un programa que pasa por tu código **antes** que el compilador y hace sustituciones de texto: pega el contenido de los `#include`, cambia cada `#define` por su valor y quita o deja trozos según los `#ifdef`. Los **headers** (`.h`) son ficheros pensados para ser pegados así: contienen prototipos, tipos y constantes que comparten varios `.c`.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Directiva** | Línea que empieza por `#`. No es C: son órdenes para el preprocesador |
| **Header** (cabecera) | Fichero `.h` con declaraciones que se incluyen en varios `.c` |
| **Prototipo** | Declaración de una función sin su cuerpo: `size_t ft_strlen(const char *s);` |
| **Macro** | Nombre definido con `#define` que el preprocesador sustituye por su texto |
| **Guarda de inclusión** | El trío `#ifndef` / `#define` / `#endif` que impide pegar dos veces un header |

## Qué hace de verdad el preprocesador

Es **sustitución de texto, sin entender C**. Puedes ver su resultado con `cc -E`:

```c
#define MAX_FD 1024
#define MSG "hola"

int	main(void)
{
	char	*s;

	s = MSG;
	return (MAX_FD + s[0]);
}
```

```text
$ cc -E mini.c | grep -v "^#" | grep -v "^$"
int main(void)
{
 char *s;
 s = "hola";
 return (1024 + s[0]);
}
```

Los nombres `MAX_FD` y `MSG` han desaparecido: el compilador nunca los ve. Esta es la primera de las fases de compilación (ver [[Cómo compila C]]):

```text
 libft.h ─┐
          ▼
 ft_split.c ──[preprocesador]──► código C «puro» ──[compilador]──► ft_split.o ──[enlazador]──► programa
          (#include, #define,      (una unidad de
           #ifndef resueltos)       traducción)
```

## `#include "..."` frente a `#include <...>`

`#include` copia y pega el fichero entero en ese punto. La diferencia entre comillas y ángulos es **dónde lo busca**:

| Forma | Dónde busca | Para qué |
|---|---|---|
| `#include <unistd.h>` | Solo en los directorios del sistema | Headers de la biblioteca estándar y del sistema |
| `#include "libft.h"` | Primero en la carpeta del `.c`, luego en las rutas de `-I`, luego en las del sistema | Tus propios headers |

Las rutas del sistema se pueden ver con `-v`:

```text
$ cc -v -E v.c -o /dev/null
#include "..." search starts here:
#include <...> search starts here:
 /usr/local/include
 /usr/lib/llvm-12/lib/clang/12.0.1/include
 /usr/include/x86_64-linux-gnu
 /usr/include
End of search list.
```

Si tu header está en otra carpeta (típico: `includes/` o la libft dentro de otro proyecto), tienes que decírselo con `-I`:

```text
$ cc -Wall -Wextra -Werror usa2.c
usa2.c:1:10: fatal error: 'ft.h' file not found
$ cc -Wall -Wextra -Werror -I inc2 usa2.c -o u
$ ./u; echo "exit $?"
exit 0
```

En un [[Makefile y librerías estáticas|Makefile]] eso va en `CFLAGS`: `CFLAGS = -Wall -Wextra -Werror -I includes -I libft`.

> [!warning] Nunca `#include "fichero.c"`
> Pegaría el código de las funciones en cada `.c` que lo incluya, y el enlazador se quejaría de `multiple definition`. La Norma lo prohíbe expresamente: «No puedes incluir un archivo C».

## Guardas de inclusión

Si un header se pega dos veces en el mismo `.c` (por ejemplo, porque `libft.h` y `get_next_line.h` lo incluyen ambos), cada `typedef` y cada `struct` aparecen dos veces:

```c
#include "punto.h"
#include "punto.h"
```

```text
$ cc -Wall -Wextra -Werror sin_guarda.c -o a
In file included from sin_guarda.c:2:
./punto.h:1:16: error: redefinition of 's_punto'
typedef struct s_punto
               ^
./punto.h:1:16: note: unguarded header; consider using #ifdef guards or #pragma once
```

La solución es envolver el header en una **guarda**:

```c
#ifndef PUNTO_OK_H
# define PUNTO_OK_H

typedef struct s_punto
{
	int	x;
	int	y;
}	t_punto;

#endif
```

Cómo funciona, inclusión por inclusión:

```text
1ª vez:  ¿PUNTO_OK_H definido? NO  → define PUNTO_OK_H y pega el contenido
2ª vez:  ¿PUNTO_OK_H definido? SÍ  → salta hasta #endif: no pega nada
```

Con la guarda, `con_guarda.c` (que incluye el header dos veces) compila sin errores.

> [!quote] La Norma v4, «Headers»
> «Los archivos de cabecera deben estar protegidos de inclusiones dobles. Si el archivo es ft_foo.h, su macro de protección es FT_FOO_H.»

Para `libft.h`, la guarda es `LIBFT_H`. Fíjate en el espacio tras `#` en `# define`: la Norma pide **indentar** lo que va dentro de un `#ifndef`, un espacio por nivel.

> [!note] ¿Y `#pragma once`?
> Hace lo mismo y lo entienden gcc y clang, pero no es C estándar y la Norma pide la guarda con `#ifndef`. Úsala.

## Qué va en un `.h` (y qué no)

> [!quote] La Norma v4, «Headers»
> «Las cosas permitidas en los archivos de cabecera son: inclusiones de cabecera (de sistema o no), declaraciones, defines, prototipos y macros.»

Un `libft.h` típico:

```c
#ifndef LIBFT_H
# define LIBFT_H

# include <stddef.h>
# include <stdlib.h>

typedef struct s_list
{
	void			*content;
	struct s_list	*next;
}	t_list;

size_t	ft_strlen(const char *s);
char	*ft_strdup(const char *s);
t_list	*ft_lstnew(void *content);

#endif
```

| ✅ Va en el `.h` | ❌ No va en el `.h` |
|---|---|
| Prototipos de funciones públicas | Cuerpos de funciones |
| `typedef` y `struct` (ver [[Structs y typedef]]) | Variables (crearían una copia por cada `.c`, ver [[Variables static y ámbito]]) |
| `#define` de constantes | Prototipos de helpers `static` (son privados de su `.c`) |
| `#include` que necesitan esos prototipos (`<stddef.h>` para `size_t`) | `#include` que no se usan (la Norma los prohíbe) |

## `#define`: constantes y macros

```c
#define BUFFER_SIZE 42        /* constante: lo que la Norma permite        */
#define CUADRADO(x) ((x) * (x)) /* macro con parámetros: la Norma la prohíbe */
```

Una macro con parámetros parece una función, pero es **sustitución de texto**. Si olvidas los paréntesis, la precedencia de operadores te traiciona:

```c
#include <stdio.h>

#define MAL_CUADRADO(x) x * x
#define CUADRADO(x) ((x) * (x))

int	main(void)
{
	printf("MAL_CUADRADO(1 + 2) = %d\n", MAL_CUADRADO(1 + 2));
	printf("CUADRADO(1 + 2)     = %d\n", CUADRADO(1 + 2));
	return (0);
}
```

```text
$ cc -Wall -Wextra -Werror macro.c -o m && ./m
MAL_CUADRADO(1 + 2) = 5
CUADRADO(1 + 2)     = 9
```

`MAL_CUADRADO(1 + 2)` se convierte en `1 + 2 * 1 + 2`, que vale 5.

### Qué permite la Norma

> [!quote] La Norma v4, «Macros y Preprocesadores»
> «Las constantes del preprocesador (o #define) que crees, deben usarse solo para valores literales y constantes.»
> «Las macros de varias líneas están prohibidas.»
> «Los nombres de las macros deben estar en mayúsculas.»
> «Debes indentar los caracteres que siguen a #if, #ifdef o #ifndef.»
> «Las instrucciones del preprocesador están prohibidas fuera del alcance global.»

La norminette lo comprueba. Un header con macros-función y sin indentar:

```text
$ norminette mal.h
Error: PREPROC_BAD_INDENT   (line:   2, col:   1):	Bad preprocessor indentation
Error: MACRO_FUNC_FORBIDDEN (line:   3, col:  18):	Macro functions are forbidden
Error: PREPROC_CONSTANT     (line:   3, col:  22):	Preprocessor statement must only contain constant defines
Error: MACRO_FUNC_FORBIDDEN (line:   4, col:  13):	Macro functions are forbidden
Error: TERNARY_FBIDDEN      (line:   4, col:  26):	Ternaries are forbidden
```

Resumen: en tus proyectos, `#define` solo para **constantes con nombre en mayúsculas** (`# define HEX_LOWER "0123456789abcdef"`, `# define WIN_WIDTH 1920`). Nada de `#define` dentro de una función.

## `-D BUFFER_SIZE=42` con `#ifndef`

La opción `-D` del compilador **define una macro desde la línea de comandos**, como si escribieras `#define` al principio del fichero. [[get_next_line - Índice|get_next_line]] la usa para que el corrector pruebe distintos tamaños de buffer sin tocar tu código. Tu header debe dar un valor por defecto **solo si no viene ya definido**:

```c
#include <stdio.h>

#ifndef BUFFER_SIZE
# define BUFFER_SIZE 42
#endif

int	main(void)
{
	printf("BUFFER_SIZE = %d\n", BUFFER_SIZE);
	return (0);
}
```

```text
$ cc -Wall -Wextra -Werror buffer.c -o buf && ./buf
BUFFER_SIZE = 42
$ cc -Wall -Wextra -Werror -D BUFFER_SIZE=1 buffer.c -o buf && ./buf
BUFFER_SIZE = 1
$ cc -Wall -Wextra -Werror -D BUFFER_SIZE=10000000 buffer.c -o buf && ./buf
BUFFER_SIZE = 10000000
```

> [!danger] `BUFFER_SIZE` enorme o cero
> Con `-D BUFFER_SIZE=10000000`, `char buf[BUFFER_SIZE + 1];` como local ocupa ~10 MB en la **pila**, cuyo límite en Linux suele ser 8 MB (`ulimit -s` → `8192` KB). Probado con el `leer.c` de [[Descriptores de fichero y read]]: `Segmentation fault (core dumped)`, código de salida 139. Reserva el buffer con `malloc`. Y comprueba `if (BUFFER_SIZE <= 0) return (NULL);`: el corrector suele probar `BUFFER_SIZE=0`.

> [!warning] Comprueba el subject
> Resumen basado en versiones públicas del subject; los detalles cambian entre versiones. Lo que manda es el PDF de tu intra.

## Errores frecuentes

> [!warning] Guarda con nombre equivocado
> Copiar `libft.h` para hacer `get_next_line.h` y dejar `#ifndef LIBFT_H`. Al incluir ambos, el segundo **no se pega** (su guarda ya está definida) y aparece un error que despista mucho, porque el prototipo «está ahí»:
> ```text
> $ cc -Wall -Wextra -Werror -c g.c
> g.c:6:2: error: implicit declaration of function 'get_next_line' is invalid in C99 [-Werror,-Wimplicit-function-declaration]
> ```

> [!warning] `file not found` en tu propio header
> Te falta `-I carpeta`, o usaste `<libft.h>` en vez de `"libft.h"` sin `-I`.

> [!warning] `#include` dentro del `.h` olvidado
> Si `libft.h` usa `size_t` pero no incluye `<stddef.h>` (o `<stdlib.h>`), funcionará o no según qué `.c` lo incluya antes. El header debe compilar por sí solo.

> [!warning] Punto y coma en un `#define`
> `#define MAX 10;` hace que `x = MAX + 1;` se convierta en `x = 10; + 1;`. Las directivas no llevan `;`.

> [!question] Pregunta típica de evaluación
> **«¿Para qué sirve el `#ifndef LIBFT_H` de tu header? ¿Qué pasa si lo quitas?»**
> Es la guarda de inclusión. La primera vez que se incluye el header, `LIBFT_H` no existe: se define y se pega el contenido. Las siguientes veces ya existe y el preprocesador salta hasta `#endif`. Sin ella, si un `.c` incluye `libft.h` dos veces (directa o indirectamente), el `typedef t_list` aparece dos veces y la compilación falla con `redefinition`.

## Relacionadas

[[Cómo compila C]] · [[Makefile y librerías estáticas]] · [[Variables static y ámbito]] · [[Structs y typedef]] · [[Descriptores de fichero y read]] · [[La Norma]] · [[get_next_line - Índice|get_next_line]] · [[libft - Índice]]

## Referencias

- [cppreference: Preprocessor](https://en.cppreference.com/w/c/preprocessor) — fases y directivas.
- [cppreference: Source file inclusion](https://en.cppreference.com/w/c/preprocessor/include) — `#include "..."` frente a `<...>`.
- [cppreference: Replacing text macros](https://en.cppreference.com/w/c/preprocessor/replace) — `#define` con y sin parámetros.
- [cppreference: Conditional inclusion](https://en.cppreference.com/w/c/preprocessor/conditional) — `#ifndef`, `#ifdef`, `#endif`.
- [GNU CPP: Once-Only Headers](https://gcc.gnu.org/onlinedocs/cpp/Once-Only-Headers.html) — guardas de inclusión.
- [GNU CPP: Macro Pitfalls](https://gcc.gnu.org/onlinedocs/cpp/Macro-Pitfalls.html) — paréntesis y precedencia en macros.
- [GCC: Preprocessor Options](https://gcc.gnu.org/onlinedocs/gcc/Preprocessor-Options.html) — `-D`, `-I`, `-E`.
- [norminette (42School)](https://github.com/42School/norminette) — errores `PREPROC_BAD_INDENT`, `MACRO_FUNC_FORBIDDEN`, `PREPROC_CONSTANT`.
- La Norma v4 (`Normativa 42/es.norm-2.pdf`) — secciones «Headers» y «Macros y Preprocesadores».
- Salidas obtenidas con Ubuntu clang 12.0.1 en Ubuntu 22.04.
