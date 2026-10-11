---
tags: [libft, libft/parte1, libft/memoria]
funcion: ft_memset
original: "memset(3)"
cabecera: "<string.h>"
estado: hecho
---
# ft_memset

[[libft - Índice|← Índice]] · Parte 1 · Memoria

> Rellena n bytes con un valor

## Prototipo

```c
// Original — memset(3)
#include <string.h>
void *memset(void *s, int c, size_t n);

// libft
void *ft_memset(void *s, int c, size_t n);
```

Consultar: `man 3 memset`

## Requisitos del subject

> [!quote] Subject v19.3 — Parte 1
> Tus funciones tendrán los mismos prototipos y comportamientos que las funciones originales, siguiendo fielmente las definiciones indicadas en la página del man de cada función. La única diferencia será el nombre: todas deberán comenzar con el prefijo “ft_”.
> 
> Deberás escribir tus propias versiones de las siguientes funciones originales. No deben depender de ninguna función externa.

## Comportamiento según el manual

> [!quote] man 3 memset — DESCRIPTION
> The memset() function fills the first n bytes of the memory area pointed to by s with the constant byte c.

> [!note] Traducción
> La función memset() rellena los primeros n bytes de la zona de memoria apuntada por s con el byte constante c.

> [!quote] man 3 memset — RETURN VALUE
> The memset() function returns a pointer to the memory area s.

> [!note] Traducción
> La función memset() devuelve un puntero a la zona de memoria s.

## Puntos clave y casos límite

- `c` se convierte a `unsigned char`: solo se usa el byte bajo (`memset(p, 256 + 'A', 3)` escribe `AAA`).
- Trabaja byte a byte: `memset(int_array, 1, ...)` **no** pone cada `int` a 1.
- Con `n == 0` no toca nada y devuelve `s`.

## Relacionadas

[[ft_bzero]] · [[ft_calloc]]

---
*Fuente de las citas: `man 3 memset` — Linux man-pages 5.10 (Ubuntu 22.04). Texto en inglés copiado literalmente del manual; la traducción es propia. Requisitos copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). En los Mac de 42 (man de BSD/macOS) la redacción puede variar.*
