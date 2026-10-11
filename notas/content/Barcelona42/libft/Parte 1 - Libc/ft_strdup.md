---
tags: [libft, libft/parte1, libft/memoria-dinámica]
funcion: ft_strdup
original: "strdup(3)"
cabecera: "<string.h>"
estado: pendiente
---
# ft_strdup

[[libft - Índice|← Índice]] · Parte 1 · Memoria dinámica

> Duplica una cadena en memoria nueva

## Prototipo

```c
// Original — strdup(3)
#include <string.h>
char *strdup(const char *s);

// libft
char *ft_strdup(const char *s);
```

Consultar: `man 3 strdup`

## Requisitos del subject

> [!quote] Subject v19.3 — Parte 1
> Tus funciones tendrán los mismos prototipos y comportamientos que las funciones originales, siguiendo fielmente las definiciones indicadas en la página del man de cada función. La única diferencia será el nombre: todas deberán comenzar con el prefijo “ft_”.
> 
> Para implementar estas otras dos funciones, tendrás que utilizar malloc():

## Comportamiento según el manual

> [!quote] man 3 strdup — DESCRIPTION
> The strdup() function returns a pointer to a new string which is a duplicate of the string s.  Memory for the new string is obtained with malloc(3), and can be freed with free(3).

> [!note] Traducción
> La función strdup() devuelve un puntero a una cadena nueva que es un duplicado de la cadena s. La memoria de la cadena nueva se obtiene con malloc(3) y puede liberarse con free(3).

> [!quote] man 3 strdup — RETURN VALUE
> On success, the strdup() function returns a pointer to the duplicated string.  It returns NULL if insufficient memory was available, with errno set to indicate the cause of the error.

> [!note] Traducción
> Si tiene éxito, strdup() devuelve un puntero a la cadena duplicada. Devuelve NULL si no había memoria suficiente, y pone en errno la causa del error.

> [!quote] man 3 strdup — ERRORS
> ENOMEM Insufficient memory available to allocate duplicate string.

> [!note] Traducción
> ENOMEM No hay memoria suficiente para reservar la cadena duplicada.

## Puntos clave y casos límite

- Reserva `ft_strlen(s) + 1` bytes (el `+ 1` es para el `'\0'`).
- Comprueba siempre el retorno de `malloc`.

## Relacionadas

[[ft_strlen]] · [[ft_calloc]] · [[ft_substr]]

---
*Fuente de las citas: `man 3 strdup` — Linux man-pages 5.10 (Ubuntu 22.04). Texto en inglés copiado literalmente del manual; la traducción es propia. Requisitos copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). En los Mac de 42 (man de BSD/macOS) la redacción puede variar.*
