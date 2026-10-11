---
tags: [libft, libft/parte1, libft/caracteres]
funcion: ft_toupper
original: "toupper(3)"
cabecera: "<ctype.h>"
estado: hecho
---
# ft_toupper

[[libft - Índice|← Índice]] · Parte 1 · Caracteres

> Minúscula → mayúscula

## Prototipo

```c
// Original — toupper(3)
#include <ctype.h>
int toupper(int c);

// libft
int ft_toupper(int c);
```

Consultar: `man 3 toupper`

## Requisitos del subject

> [!quote] Subject v19.3 — Parte 1
> Tus funciones tendrán los mismos prototipos y comportamientos que las funciones originales, siguiendo fielmente las definiciones indicadas en la página del man de cada función. La única diferencia será el nombre: todas deberán comenzar con el prefijo “ft_”.
> 
> Deberás escribir tus propias versiones de las siguientes funciones originales. No deben depender de ninguna función externa.

## Comportamiento según el manual

> [!quote] man 3 toupper — DESCRIPTION
> These functions convert lowercase letters to uppercase, and vice versa.

> [!note] Traducción
> Estas funciones convierten letras minúsculas en mayúsculas, y viceversa.

> [!quote] man 3 toupper — DESCRIPTION
> If c is a lowercase letter, toupper() returns its uppercase equivalent, if an uppercase representation exists in the current locale.  Otherwise, it returns c.

> [!note] Traducción
> Si c es una letra minúscula, toupper() devuelve su equivalente en mayúscula, si existe una representación en mayúscula en el locale actual. Si no, devuelve c.

> [!quote] man 3 toupper — DESCRIPTION
> If c is neither an unsigned char value nor EOF, the behavior of these functions is undefined.

> [!note] Traducción
> Si c no es ni un valor unsigned char ni EOF, el comportamiento de estas funciones es indefinido.

> [!quote] man 3 toupper — RETURN VALUE
> The value returned is that of the converted letter, or c if the conversion was not possible.

> [!note] Traducción
> El valor devuelto es el de la letra convertida, o c si la conversión no fue posible.

## Puntos clave y casos límite

- En ASCII la diferencia entre minúscula y mayúscula es 32 (`'a' - 'A'`).
- Cualquier valor que no sea `'a'`–`'z'` se devuelve sin cambios.

## Relacionadas

[[ft_tolower]] · [[ft_isalpha]]

---
*Fuente de las citas: `man 3 toupper` — Linux man-pages 5.10 (Ubuntu 22.04). Texto en inglés copiado literalmente del manual; la traducción es propia. Requisitos copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). En los Mac de 42 (man de BSD/macOS) la redacción puede variar.*
