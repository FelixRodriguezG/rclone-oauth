---
tags: [libft, libft/parte1, libft/memoria]
funcion: ft_memchr
original: "memchr(3)"
cabecera: "<string.h>"
estado: pendiente
---
# ft_memchr

[[libft - Índice|← Índice]] · Parte 1 · Memoria

> Busca un byte en n bytes de memoria

## Prototipo

```c
// Original — memchr(3)
#include <string.h>
void *memchr(const void *s, int c, size_t n);

// libft
void *ft_memchr(const void *s, int c, size_t n);
```

Consultar: `man 3 memchr`

## Requisitos del subject

> [!quote] Subject v19.3 — Parte 1
> Tus funciones tendrán los mismos prototipos y comportamientos que las funciones originales, siguiendo fielmente las definiciones indicadas en la página del man de cada función. La única diferencia será el nombre: todas deberán comenzar con el prefijo “ft_”.
> 
> Deberás escribir tus propias versiones de las siguientes funciones originales. No deben depender de ninguna función externa.

## Comportamiento según el manual

> [!quote] man 3 memchr — DESCRIPTION
> The memchr() function scans the initial n bytes of the memory area pointed to by s for the first instance of c.  Both c and the bytes of the memory area pointed to by s are interpreted as unsigned char.

> [!note] Traducción
> La función memchr() examina los n primeros bytes de la zona de memoria apuntada por s buscando la primera aparición de c. Tanto c como los bytes de la zona apuntada por s se interpretan como unsigned char.

> [!quote] man 3 memchr — RETURN VALUE
> The memchr() and memrchr() functions return a pointer to the matching byte or NULL if the character does not occur in the given memory area.

> [!note] Traducción
> memchr() y memrchr() devuelven un puntero al byte coincidente, o NULL si el carácter no aparece en la zona de memoria indicada.

## Puntos clave y casos límite

- A diferencia de `strchr`, **no** se detiene en `'\0'`: recorre exactamente `n` bytes.
- Compara usando `unsigned char` tanto para `c` como para los bytes.

## Relacionadas

[[ft_strchr]] · [[ft_memcmp]]

---
*Fuente de las citas: `man 3 memchr` — Linux man-pages 5.10 (Ubuntu 22.04). Texto en inglés copiado literalmente del manual; la traducción es propia. Requisitos copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). En los Mac de 42 (man de BSD/macOS) la redacción puede variar.*
