---
tags: [libft, libft/parte1, libft/caracteres]
funcion: ft_isascii
original: "isascii(3)"
cabecera: "<ctype.h>"
estado: hecho
---
# ft_isascii

[[libft - Índice|← Índice]] · Parte 1 · Caracteres

> ¿Está en la tabla ASCII (0–127)?

## Prototipo

```c
// Original — isascii(3)
#include <ctype.h>
int isascii(int c);

// libft
int ft_isascii(int c);
```

Consultar: `man 3 isascii`

## Requisitos del subject

> [!quote] Subject v19.3 — Parte 1
> Tus funciones tendrán los mismos prototipos y comportamientos que las funciones originales, siguiendo fielmente las definiciones indicadas en la página del man de cada función. La única diferencia será el nombre: todas deberán comenzar con el prefijo “ft_”.
> 
> Deberás escribir tus propias versiones de las siguientes funciones originales. No deben depender de ninguna función externa.
> 
> Para las funciones de clasificación de caracteres (isalpha, isdigit, isalnum, isascii, isprint), el valor de retorno debe ser:
> • 1 si el carácter cumple la condición comprobada.
> • 0 si el carácter no la cumple.

## Comportamiento según el manual

> [!quote] man 3 isascii — DESCRIPTION
> These functions check whether c, which must have the value of an unsigned char or EOF, falls into a certain character class according to the specified locale.  The functions without the "_l" suffix perform the check based on the current locale.

> [!note] Traducción
> Estas funciones comprueban si c, que debe tener el valor de un unsigned char o EOF, pertenece a una determinada clase de caracteres según el locale especificado. Las funciones sin el sufijo "_l" hacen la comprobación según el locale actual.

> [!quote] man 3 isascii — DESCRIPTION
> isascii()
>        checks whether c is a 7-bit unsigned char value that fits into the ASCII character set.

> [!note] Traducción
> isascii()
>        comprueba si c es un valor unsigned char de 7 bits que cabe en el juego de caracteres ASCII.

> [!quote] man 3 isascii — RETURN VALUE
> The values returned are nonzero if the character c falls into the tested class, and zero if not.

> [!note] Traducción
> Los valores devueltos son distintos de cero si el carácter c pertenece a la clase comprobada, y cero si no.

> [!quote] man 3 isascii — CONFORMING TO
> POSIX.1-2008 marks isascii() as obsolete, noting that it cannot be used portably in a localized application.

> [!note] Traducción
> POSIX.1-2008 marca isascii() como obsoleta, porque no puede usarse de forma portable en una aplicación localizada.

## Puntos clave y casos límite

- Rango: 0–127 (incluye `'\0'` y los caracteres de control).
- A diferencia de las otras `is*`, la original acepta cualquier `int`: es solo una comprobación de rango.

## Relacionadas

[[ft_isprint]]

---
*Fuente de las citas: `man 3 isascii` — Linux man-pages 5.10 (Ubuntu 22.04). Texto en inglés copiado literalmente del manual; la traducción es propia. Requisitos copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). En los Mac de 42 (man de BSD/macOS) la redacción puede variar.*
