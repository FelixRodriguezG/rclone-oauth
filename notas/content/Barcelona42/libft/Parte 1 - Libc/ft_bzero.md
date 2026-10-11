---
tags: [libft, libft/parte1, libft/memoria]
funcion: ft_bzero
original: "bzero(3)"
cabecera: "<strings.h>"
estado: hecho
---
# ft_bzero

[[libft - Índice|← Índice]] · Parte 1 · Memoria

> Pone n bytes a cero

## Prototipo

```c
// Original — bzero(3)
#include <strings.h>
void bzero(void *s, size_t n);

// libft
void ft_bzero(void *s, size_t n);
```

Consultar: `man 3 bzero`

## Requisitos del subject

> [!quote] Subject v19.3 — Parte 1
> Tus funciones tendrán los mismos prototipos y comportamientos que las funciones originales, siguiendo fielmente las definiciones indicadas en la página del man de cada función. La única diferencia será el nombre: todas deberán comenzar con el prefijo “ft_”.
> 
> Deberás escribir tus propias versiones de las siguientes funciones originales. No deben depender de ninguna función externa.
> 
> Algunas de las funciones que debes rehacer, como strlcpy, strlcat y bzero, no están incluidas por defecto en la biblioteca GNU C (glibc). Para compararlas con la versión estándar del sistema, puede ser necesario incluir <bsd/string.h> y compilar con la opción -lbsd.

## Comportamiento según el manual

> [!quote] man 3 bzero — DESCRIPTION
> The bzero() function erases the data in the n bytes of the memory starting at the location pointed to by s, by writing zeros (bytes containing '\0') to that area.

> [!note] Traducción
> La función bzero() borra los datos de los n bytes de memoria que empiezan en la posición apuntada por s, escribiendo ceros (bytes que contienen '\0') en esa zona.

> [!quote] man 3 bzero — RETURN VALUE
> None.

> [!note] Traducción
> Ninguno.

> [!quote] man 3 bzero — CONFORMING TO
> The bzero() function is deprecated (marked as LEGACY in POSIX.1-2001); use memset(3) in new programs.  POSIX.1-2008 removes the specification of bzero().

> [!note] Traducción
> La función bzero() está obsoleta (marcada como LEGACY en POSIX.1-2001); usa memset(3) en programas nuevos. POSIX.1-2008 elimina la especificación de bzero().

## Puntos clave y casos límite

- Equivale a `ft_memset(s, 0, n)`.
- Cabecera `<strings.h>` (con **s**), no `<string.h>`.

## Relacionadas

[[ft_memset]] · [[ft_calloc]]

---
*Fuente de las citas: `man 3 bzero` — Linux man-pages 5.10 (Ubuntu 22.04). Texto en inglés copiado literalmente del manual; la traducción es propia. Requisitos copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). En los Mac de 42 (man de BSD/macOS) la redacción puede variar.*
