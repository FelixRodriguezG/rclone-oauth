---
tags: [libft, libft/parte1, libft/cadenas]
funcion: ft_strlcpy
original: "strlcpy(3bsd)"
cabecera: "<string.h> / <bsd/string.h>"
estado: hecho
---
# ft_strlcpy

[[libft - Índice|← Índice]] · Parte 1 · Cadenas

> Copia segura con tamaño de buffer

## Prototipo

```c
// Original — strlcpy(3bsd)
#include <string.h>
size_t strlcpy(char *dst, const char *src, size_t size);

// libft
size_t ft_strlcpy(char *dst, const char *src, size_t size);
```

Consultar: `man 3bsd strlcpy`

## Requisitos del subject

> [!quote] Subject v19.3 — Parte 1
> Tus funciones tendrán los mismos prototipos y comportamientos que las funciones originales, siguiendo fielmente las definiciones indicadas en la página del man de cada función. La única diferencia será el nombre: todas deberán comenzar con el prefijo “ft_”.
> 
> Deberás escribir tus propias versiones de las siguientes funciones originales. No deben depender de ninguna función externa.
> 
> Algunas de las funciones que debes rehacer, como strlcpy, strlcat y bzero, no están incluidas por defecto en la biblioteca GNU C (glibc). Para compararlas con la versión estándar del sistema, puede ser necesario incluir <bsd/string.h> y compilar con la opción -lbsd.

## Comportamiento según el manual

> [!quote] man 3bsd strlcpy — DESCRIPTION
> The strlcpy() and strlcat() functions copy and concatenate strings respectively.  They are designed to be safer, more consistent, and less error prone replacements for strncpy(3) and strncat(3).  Unlike those functions, strlcpy() and strlcat() take the full size of the buffer (not just the length) and guarantee to NUL-terminate the result (as long as size is larger than 0 or, in the case of strlcat(), as long as there is at least one byte free in dst).  Note that a byte for the NUL should be included in size.  Also note that strlcpy() and strlcat() only operate on true "C" strings.  This means that for strlcpy() src must be NUL-terminated and for strlcat() both src and dst must be NUL-terminated.

> [!note] Traducción
> Las funciones strlcpy() y strlcat() copian y concatenan cadenas respectivamente. Están diseñadas como sustitutas más seguras, coherentes y menos propensas a errores de strncpy(3) y strncat(3). A diferencia de esas funciones, strlcpy() y strlcat() reciben el tamaño completo del buffer (no solo la longitud) y garantizan que el resultado termina en NUL (siempre que size sea mayor que 0 o, en el caso de strlcat(), siempre que quede al menos un byte libre en dst). Ten en cuenta que size debe incluir un byte para el NUL. Además, strlcpy() y strlcat() solo operan sobre cadenas "C" auténticas: para strlcpy() src debe terminar en NUL, y para strlcat() tanto src como dst deben terminar en NUL.

> [!quote] man 3bsd strlcpy — DESCRIPTION
> The strlcpy() function copies up to size - 1 characters from the NUL-terminated string src to dst, NUL-terminating the result.

> [!note] Traducción
> La función strlcpy() copia hasta size - 1 caracteres de la cadena src (terminada en NUL) a dst, terminando el resultado en NUL.

> [!quote] man 3bsd strlcpy — RETURN VALUES
> The strlcpy() and strlcat() functions return the total length of the string they tried to create.  For strlcpy() that means the length of src.  For strlcat() that means the initial length of dst plus the length of src.  While this may seem somewhat confusing, it was done to make truncation detection simple.

> [!note] Traducción
> strlcpy() y strlcat() devuelven la longitud total de la cadena que intentaron crear. Para strlcpy() es la longitud de src. Para strlcat() es la longitud inicial de dst más la longitud de src. Aunque pueda parecer confuso, se hizo así para que detectar el truncamiento sea sencillo.

## Puntos clave y casos límite

- Siempre devuelve `ft_strlen(src)`, aunque se trunque.
- Con `size == 0` no escribe nada en `dst` (ni siquiera el `'\0'`).
- Truncamiento: si el valor devuelto `>= size`, la copia se truncó.
- En Linux no está en glibc 2.35: para probar la original usa `#include <bsd/string.h>` y compila con `-lbsd`.

## Relacionadas

[[ft_strlcat]] · [[ft_strlen]] · [[ft_memcpy]]

---
*Fuente de las citas: `man 3bsd strlcpy` — libbsd (libbsd-dev, Ubuntu 22.04). Texto en inglés copiado literalmente del manual; la traducción es propia. Requisitos copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). En los Mac de 42 (man de BSD/macOS) la redacción puede variar.*
