---
tags: [libft, libft/parte1, libft/memoria]
funcion: ft_memcpy
original: "memcpy(3)"
cabecera: "<string.h>"
estado: hecho
---
# ft_memcpy

[[libft - Índice|← Índice]] · Parte 1 · Memoria

> Copia n bytes (sin solapamiento)

## Prototipo

```c
// Original — memcpy(3)
#include <string.h>
void *memcpy(void *dest, const void *src, size_t n);

// libft
void *ft_memcpy(void *dest, const void *src, size_t n);
```

Consultar: `man 3 memcpy`

## Requisitos del subject

> [!quote] Subject v19.3 — Parte 1
> Tus funciones tendrán los mismos prototipos y comportamientos que las funciones originales, siguiendo fielmente las definiciones indicadas en la página del man de cada función. La única diferencia será el nombre: todas deberán comenzar con el prefijo “ft_”.
> 
> Deberás escribir tus propias versiones de las siguientes funciones originales. No deben depender de ninguna función externa.

## Comportamiento según el manual

> [!quote] man 3 memcpy — DESCRIPTION
> The memcpy() function copies n bytes from memory area src to memory area dest.  The memory areas must not overlap.  Use memmove(3) if the memory areas do overlap.

> [!note] Traducción
> La función memcpy() copia n bytes de la zona de memoria src a la zona de memoria dest. Las zonas de memoria no deben solaparse. Usa memmove(3) si se solapan.

> [!quote] man 3 memcpy — RETURN VALUE
> The memcpy() function returns a pointer to dest.

> [!note] Traducción
> La función memcpy() devuelve un puntero a dest.

> [!quote] man 3 memcpy — NOTES
> (POSIX and the C standards are explicit that employing memcpy() with overlapping areas produces undefined behavior.)

> [!note] Traducción
> (POSIX y los estándares de C dicen explícitamente que usar memcpy() con zonas solapadas produce comportamiento indefinido.)

## Puntos clave y casos límite

- Con `n == 0` devuelve `dest` sin acceder a memoria.
- `dest` y `src` NULL con `n > 0` es comportamiento indefinido en la original; algunos testers lo prueban, así que decide qué hace tu versión.

## Relacionadas

[[ft_memmove]] · [[ft_strlcpy]]

---
*Fuente de las citas: `man 3 memcpy` — Linux man-pages 5.10 (Ubuntu 22.04). Texto en inglés copiado literalmente del manual; la traducción es propia. Requisitos copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). En los Mac de 42 (man de BSD/macOS) la redacción puede variar.*
