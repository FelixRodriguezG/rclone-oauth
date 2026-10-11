---
tags: [libft, libft/parte1, libft/memoria]
funcion: ft_memcmp
original: "memcmp(3)"
cabecera: "<string.h>"
estado: pendiente
---
# ft_memcmp

[[libft - Índice|← Índice]] · Parte 1 · Memoria

> Compara n bytes de memoria

## Prototipo

```c
// Original — memcmp(3)
#include <string.h>
int memcmp(const void *s1, const void *s2, size_t n);

// libft
int ft_memcmp(const void *s1, const void *s2, size_t n);
```

Consultar: `man 3 memcmp`

## Requisitos del subject

> [!quote] Subject v19.3 — Parte 1
> Tus funciones tendrán los mismos prototipos y comportamientos que las funciones originales, siguiendo fielmente las definiciones indicadas en la página del man de cada función. La única diferencia será el nombre: todas deberán comenzar con el prefijo “ft_”.
> 
> Deberás escribir tus propias versiones de las siguientes funciones originales. No deben depender de ninguna función externa.

## Comportamiento según el manual

> [!quote] man 3 memcmp — DESCRIPTION
> The memcmp() function compares the first n bytes (each interpreted as unsigned char) of the memory areas s1 and s2.

> [!note] Traducción
> La función memcmp() compara los primeros n bytes (cada uno interpretado como unsigned char) de las zonas de memoria s1 y s2.

> [!quote] man 3 memcmp — RETURN VALUE
> The memcmp() function returns an integer less than, equal to, or greater than zero if the first n bytes of s1 is found, respectively, to be less than, to match, or be greater than the first n bytes of s2.
>
> For a nonzero return value, the sign is determined by the sign of the difference between the first pair of bytes (interpreted as unsigned char) that differ in s1 and s2.
>
> If n is zero, the return value is zero.

> [!note] Traducción
> La función memcmp() devuelve un entero menor, igual o mayor que cero si los primeros n bytes de s1 son, respectivamente, menores, iguales o mayores que los primeros n bytes de s2.
>
> Si el valor devuelto no es cero, su signo lo determina el signo de la diferencia entre el primer par de bytes (interpretados como unsigned char) que difieren en s1 y s2.
>
> Si n es cero, el valor devuelto es cero.

## Puntos clave y casos límite

- No se detiene en `'\0'`.
- Devuelve `(unsigned char)a - (unsigned char)b` del primer byte distinto.
- El man solo garantiza el **signo**, no el valor exacto: en los tests compara signos.

## Relacionadas

[[ft_strncmp]] · [[ft_memchr]]

---
*Fuente de las citas: `man 3 memcmp` — Linux man-pages 5.10 (Ubuntu 22.04). Texto en inglés copiado literalmente del manual; la traducción es propia. Requisitos copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). En los Mac de 42 (man de BSD/macOS) la redacción puede variar.*
