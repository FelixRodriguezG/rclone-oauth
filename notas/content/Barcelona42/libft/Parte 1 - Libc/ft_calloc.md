---
tags: [libft, libft/parte1, libft/memoria-dinámica]
funcion: ft_calloc
original: "calloc(3)"
cabecera: "<stdlib.h>"
estado: pendiente
---
# ft_calloc

[[libft - Índice|← Índice]] · Parte 1 · Memoria dinámica

> Reserva memoria inicializada a cero

## Prototipo

```c
// Original — calloc(3)
#include <stdlib.h>
void *calloc(size_t nmemb, size_t size);

// libft
void *ft_calloc(size_t nmemb, size_t size);
```

Consultar: `man 3 calloc`

## Requisitos del subject

> [!quote] Subject v19.3 — Parte 1
> Tus funciones tendrán los mismos prototipos y comportamientos que las funciones originales, siguiendo fielmente las definiciones indicadas en la página del man de cada función. La única diferencia será el nombre: todas deberán comenzar con el prefijo “ft_”.
> 
> Para implementar estas otras dos funciones, tendrás que utilizar malloc():
> 
> Dependiendo de tu sistema operativo actual, el comportamiento de la función calloc puede diferir de lo descrito en su página del manual. Debes seguir la siguiente regla: Si nmemb o size es 0, entonces calloc() debe devolver un puntero único que pueda pasarse con éxito a free().

## Comportamiento según el manual

> [!quote] man 3 calloc — DESCRIPTION
> The calloc() function allocates memory for an array of nmemb elements of size bytes each and returns a pointer to the allocated memory.  The memory is set to zero.  If nmemb or size is 0, then calloc() returns either NULL, or a unique pointer value that can later be successfully passed to free().  If the multiplication of nmemb and size would result in integer overflow, then calloc() returns an error.

> [!note] Traducción
> La función calloc() reserva memoria para un array de nmemb elementos de size bytes cada uno y devuelve un puntero a la memoria reservada. La memoria se pone a cero. Si nmemb o size es 0, calloc() devuelve NULL o un puntero único que más tarde se puede pasar a free() sin problemas. Si la multiplicación de nmemb por size produjera un desbordamiento de enteros, calloc() devuelve un error.

> [!quote] man 3 calloc — RETURN VALUE
> The malloc() and calloc() functions return a pointer to the allocated memory, which is suitably aligned for any built-in type.  On error, these functions return NULL.

> [!note] Traducción
> malloc() y calloc() devuelven un puntero a la memoria reservada, alineada adecuadamente para cualquier tipo predefinido. En caso de error, devuelven NULL.

> [!quote] man 3 calloc — ERRORS
> ENOMEM Out of memory.

> [!note] Traducción
> ENOMEM Sin memoria.

## Puntos clave y casos límite

- Comprueba el desbordamiento antes de multiplicar: `if (size && nmemb > SIZE_MAX / size) return (NULL);`.
- Implementación típica: `malloc(nmemb * size)` + `ft_bzero`.
- Con `nmemb` o `size` a 0, **el subject exige** devolver un puntero único liberable con `free()` (no NULL).

## Relacionadas

[[ft_bzero]] · [[ft_strdup]]

---
*Fuente de las citas: `man 3 calloc` — Linux man-pages 5.10 (Ubuntu 22.04). Texto en inglés copiado literalmente del manual; la traducción es propia. Requisitos copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). En los Mac de 42 (man de BSD/macOS) la redacción puede variar.*
