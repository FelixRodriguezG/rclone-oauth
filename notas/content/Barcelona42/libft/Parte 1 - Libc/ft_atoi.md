---
tags: [libft, libft/parte1, libft/conversion]
funcion: ft_atoi
original: "atoi(3)"
cabecera: "<stdlib.h>"
estado: pendiente
---
# ft_atoi

[[libft - Índice|← Índice]] · Parte 1 · Conversión

> Cadena → int

## Prototipo

```c
// Original — atoi(3)
#include <stdlib.h>
int atoi(const char *nptr);

// libft
int ft_atoi(const char *nptr);
```

Consultar: `man 3 atoi`

## Requisitos del subject

> [!quote] Subject v19.3 — Parte 1
> Tus funciones tendrán los mismos prototipos y comportamientos que las funciones originales, siguiendo fielmente las definiciones indicadas en la página del man de cada función. La única diferencia será el nombre: todas deberán comenzar con el prefijo “ft_”.
> 
> Deberás escribir tus propias versiones de las siguientes funciones originales. No deben depender de ninguna función externa.

## Comportamiento según el manual

> [!quote] man 3 atoi — DESCRIPTION
> The atoi() function converts the initial portion of the string pointed to by nptr to int.  The behavior is the same as
>
>     strtol(nptr, NULL, 10);
>
> except that atoi() does not detect errors.

> [!note] Traducción
> La función atoi() convierte la parte inicial de la cadena apuntada por nptr a int. El comportamiento es el mismo que
>
>     strtol(nptr, NULL, 10);
>
> salvo que atoi() no detecta errores.

> [!quote] man 3 atoi — RETURN VALUE
> The converted value or 0 on error.

> [!note] Traducción
> El valor convertido, o 0 en caso de error.

> [!quote] DESCRIPTION — man 3 strtol
> The string may begin with an arbitrary amount of white space (as determined by isspace(3)) followed by a single optional '+' or '-' sign.
>
> The remainder of the string is converted to a long value in the obvious manner, stopping at the first character which is not a valid digit in the given base.

> [!note] Traducción
> La cadena puede empezar con cualquier cantidad de espacios en blanco (según isspace(3)) seguida de un único signo opcional '+' o '-'.
>
> El resto de la cadena se convierte a long de la forma obvia, deteniéndose en el primer carácter que no sea un dígito válido en la base indicada.

## Puntos clave y casos límite

- Espacios válidos (isspace en locale "C"): `' '`, `'\t'`, `'\n'`, `'\v'`, `'\f'`, `'\r'` (9–13 y 32).
- Un **único** signo: `"+-42"` y `"--42"` → 0.
- Se detiene en el primer no-dígito: `"42abc"` → 42, `"abc42"` → 0.
- El desbordamiento es comportamiento indefinido en la original; `INT_MIN` (`"-2147483648"`) debe funcionar.

## Relacionadas

[[ft_itoa]] · [[ft_isdigit]]

---
*Fuente de las citas: `man 3 atoi` — Linux man-pages 5.10 (Ubuntu 22.04). Texto en inglés copiado literalmente del manual; la traducción es propia. Requisitos copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). En los Mac de 42 (man de BSD/macOS) la redacción puede variar.*
