---
tags: [libft, libft/parte1, libft/caracteres]
funcion: ft_tolower
original: "tolower(3)"
cabecera: "<ctype.h>"
estado: hecho
---
# ft_tolower

[[libft - Índice|← Índice]] · Parte 1 · Caracteres

> Mayúscula → minúscula

## Prototipo

```c
// Original — tolower(3)
#include <ctype.h>
int tolower(int c);

// libft
int ft_tolower(int c);
```

Consultar: `man 3 tolower`

## Requisitos del subject

> [!quote] Subject v19.3 — Parte 1
> Tus funciones tendrán los mismos prototipos y comportamientos que las funciones originales, siguiendo fielmente las definiciones indicadas en la página del man de cada función. La única diferencia será el nombre: todas deberán comenzar con el prefijo “ft_”.
> 
> Deberás escribir tus propias versiones de las siguientes funciones originales. No deben depender de ninguna función externa.

## Comportamiento según el manual

> [!quote] man 3 tolower — DESCRIPTION
> These functions convert lowercase letters to uppercase, and vice versa.

> [!note] Traducción
> Estas funciones convierten letras minúsculas en mayúsculas, y viceversa.

> [!quote] man 3 tolower — DESCRIPTION
> If c is an uppercase letter, tolower() returns its lowercase equivalent, if a lowercase representation exists in the current locale.  Otherwise, it returns c.

> [!note] Traducción
> Si c es una letra mayúscula, tolower() devuelve su equivalente en minúscula, si existe una representación en minúscula en el locale actual. Si no, devuelve c.

> [!quote] man 3 tolower — DESCRIPTION
> If c is neither an unsigned char value nor EOF, the behavior of these functions is undefined.

> [!note] Traducción
> Si c no es ni un valor unsigned char ni EOF, el comportamiento de estas funciones es indefinido.

> [!quote] man 3 tolower — RETURN VALUE
> The value returned is that of the converted letter, or c if the conversion was not possible.

> [!note] Traducción
> El valor devuelto es el de la letra convertida, o c si la conversión no fue posible.

## Puntos clave y casos límite

- Cualquier valor que no sea `'A'`–`'Z'` se devuelve sin cambios.

## Relacionadas

[[ft_toupper]] · [[ft_isalpha]]

---
*Fuente de las citas: `man 3 tolower` — Linux man-pages 5.10 (Ubuntu 22.04). Texto en inglés copiado literalmente del manual; la traducción es propia. Requisitos copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). En los Mac de 42 (man de BSD/macOS) la redacción puede variar.*
