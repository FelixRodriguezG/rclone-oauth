---
tags: [libft, libft/parte1, libft/cadenas]
funcion: ft_strncmp
original: "strncmp(3)"
cabecera: "<string.h>"
estado: pendiente
---
# ft_strncmp

[[libft - Índice|← Índice]] · Parte 1 · Cadenas

> Compara hasta n caracteres

## Prototipo

```c
// Original — strncmp(3)
#include <string.h>
int strncmp(const char *s1, const char *s2, size_t n);

// libft
int ft_strncmp(const char *s1, const char *s2, size_t n);
```

Consultar: `man 3 strncmp`

## Requisitos del subject

> [!quote] Subject v19.3 — Parte 1
> Tus funciones tendrán los mismos prototipos y comportamientos que las funciones originales, siguiendo fielmente las definiciones indicadas en la página del man de cada función. La única diferencia será el nombre: todas deberán comenzar con el prefijo “ft_”.
> 
> Deberás escribir tus propias versiones de las siguientes funciones originales. No deben depender de ninguna función externa.

## Comportamiento según el manual

> [!quote] man 3 strncmp — DESCRIPTION
> The strcmp() function compares the two strings s1 and s2.  The locale is not taken into account (for a locale-aware comparison, see strcoll(3)).  The comparison is done using unsigned characters.

> [!note] Traducción
> La función strcmp() compara las dos cadenas s1 y s2. No se tiene en cuenta el locale (para una comparación que lo tenga en cuenta, ver strcoll(3)). La comparación se hace con caracteres sin signo.

> [!quote] man 3 strncmp — DESCRIPTION
> The strncmp() function is similar, except it compares only the first (at most) n bytes of s1 and s2.

> [!note] Traducción
> La función strncmp() es similar, salvo que compara solo los primeros n bytes (como máximo) de s1 y s2.

> [!quote] man 3 strncmp — RETURN VALUE
> The strcmp() and strncmp() functions return an integer less than, equal to, or greater than zero if s1 (or the first n bytes thereof) is found, respectively, to be less than, to match, or be greater than s2.

> [!note] Traducción
> strcmp() y strncmp() devuelven un entero menor, igual o mayor que cero si s1 (o sus primeros n bytes) es, respectivamente, menor, igual o mayor que s2.

## Puntos clave y casos límite

- Se detiene en el primer byte distinto, en `'\0'` o al llegar a `n`.
- Compara como `unsigned char`: `ft_strncmp("\200", "\0", 1)` debe ser **positivo**.
- Con `n == 0` devuelve 0.

## Relacionadas

[[ft_memcmp]] · [[ft_strnstr]]

---
*Fuente de las citas: `man 3 strncmp` — Linux man-pages 5.10 (Ubuntu 22.04). Texto en inglés copiado literalmente del manual; la traducción es propia. Requisitos copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). En los Mac de 42 (man de BSD/macOS) la redacción puede variar.*
