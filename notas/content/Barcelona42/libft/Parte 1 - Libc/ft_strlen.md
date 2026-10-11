---
tags: [libft, libft/parte1, libft/cadenas]
funcion: ft_strlen
original: "strlen(3)"
cabecera: "<string.h>"
estado: hecho
---
# ft_strlen

[[libft - Índice|← Índice]] · Parte 1 · Cadenas

> Longitud de una cadena

## Prototipo

```c
// Original — strlen(3)
#include <string.h>
size_t strlen(const char *s);

// libft
size_t ft_strlen(const char *s);
```

Consultar: `man 3 strlen`

## Requisitos del subject

> [!quote] Subject v19.3 — Parte 1
> Tus funciones tendrán los mismos prototipos y comportamientos que las funciones originales, siguiendo fielmente las definiciones indicadas en la página del man de cada función. La única diferencia será el nombre: todas deberán comenzar con el prefijo “ft_”.
> 
> Deberás escribir tus propias versiones de las siguientes funciones originales. No deben depender de ninguna función externa.

## Comportamiento según el manual

> [!quote] man 3 strlen — DESCRIPTION
> The strlen() function calculates the length of the string pointed to by s, excluding the terminating null byte ('\0').

> [!note] Traducción
> La función strlen() calcula la longitud de la cadena apuntada por s, sin contar el byte nulo final ('\0').

> [!quote] man 3 strlen — RETURN VALUE
> The strlen() function returns the number of bytes in the string pointed to by s.

> [!note] Traducción
> La función strlen() devuelve el número de bytes de la cadena apuntada por s.

## Puntos clave y casos límite

- Devuelve `size_t` (sin signo), no `int`.
- `strlen(NULL)` es comportamiento indefinido (segfault).

## Relacionadas

[[ft_strlcpy]] · [[ft_strlcat]] · [[ft_strdup]]

---
*Fuente de las citas: `man 3 strlen` — Linux man-pages 5.10 (Ubuntu 22.04). Texto en inglés copiado literalmente del manual; la traducción es propia. Requisitos copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). En los Mac de 42 (man de BSD/macOS) la redacción puede variar.*
