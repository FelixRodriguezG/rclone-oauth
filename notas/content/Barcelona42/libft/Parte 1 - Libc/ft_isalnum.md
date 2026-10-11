---
tags: [libft, libft/parte1, libft/caracteres]
funcion: ft_isalnum
original: "isalnum(3)"
cabecera: "<ctype.h>"
estado: hecho
---
# ft_isalnum

[[libft - Índice|← Índice]] · Parte 1 · Caracteres

> ¿Es letra o dígito?

## Prototipo

```c
// Original — isalnum(3)
#include <ctype.h>
int isalnum(int c);

// libft
int ft_isalnum(int c);
```

Consultar: `man 3 isalnum`

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

> [!quote] man 3 isalnum — DESCRIPTION
> These functions check whether c, which must have the value of an unsigned char or EOF, falls into a certain character class according to the specified locale.  The functions without the "_l" suffix perform the check based on the current locale.

> [!note] Traducción
> Estas funciones comprueban si c, que debe tener el valor de un unsigned char o EOF, pertenece a una determinada clase de caracteres según el locale especificado. Las funciones sin el sufijo "_l" hacen la comprobación según el locale actual.

> [!quote] man 3 isalnum — DESCRIPTION
> isalnum()
>        checks for an alphanumeric character; it is equivalent to (isalpha(c) || isdigit(c)).

> [!note] Traducción
> isalnum()
>        comprueba si es un carácter alfanumérico; equivale a (isalpha(c) || isdigit(c)).

> [!quote] man 3 isalnum — RETURN VALUE
> The values returned are nonzero if the character c falls into the tested class, and zero if not.

> [!note] Traducción
> Los valores devueltos son distintos de cero si el carácter c pertenece a la clase comprobada, y cero si no.

> [!quote] man 3 isalnum — NOTES
> The standards require that the argument c for these functions is either EOF or a value that is representable in the type unsigned char.  If the argument c is of type char, it must be cast to unsigned char

> [!note] Traducción
> Los estándares exigen que el argumento c de estas funciones sea EOF o un valor representable en el tipo unsigned char. Si el argumento c es de tipo char, debe convertirse (cast) a unsigned char

## Puntos clave y casos límite

- Se puede implementar reutilizando `ft_isalpha` y `ft_isdigit`.

## Relacionadas

[[ft_isalpha]] · [[ft_isdigit]]

---
*Fuente de las citas: `man 3 isalnum` — Linux man-pages 5.10 (Ubuntu 22.04). Texto en inglés copiado literalmente del manual; la traducción es propia. Requisitos copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). En los Mac de 42 (man de BSD/macOS) la redacción puede variar.*
