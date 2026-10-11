---
tags: [libft, libft/parte1, libft/cadenas]
funcion: ft_strrchr
original: "strrchr(3)"
cabecera: "<string.h>"
estado: pendiente
---
# ft_strrchr

[[libft - Índice|← Índice]] · Parte 1 · Cadenas

> Última aparición de un carácter

## Prototipo

```c
// Original — strrchr(3)
#include <string.h>
char *strrchr(const char *s, int c);

// libft
char *ft_strrchr(const char *s, int c);
```

Consultar: `man 3 strrchr`

## Requisitos del subject

> [!quote] Subject v19.3 — Parte 1
> Tus funciones tendrán los mismos prototipos y comportamientos que las funciones originales, siguiendo fielmente las definiciones indicadas en la página del man de cada función. La única diferencia será el nombre: todas deberán comenzar con el prefijo “ft_”.
> 
> Deberás escribir tus propias versiones de las siguientes funciones originales. No deben depender de ninguna función externa.

## Comportamiento según el manual

> [!quote] man 3 strrchr — DESCRIPTION
> The strrchr() function returns a pointer to the last occurrence of the character c in the string s.

> [!note] Traducción
> La función strrchr() devuelve un puntero a la última aparición del carácter c en la cadena s.

> [!quote] man 3 strrchr — DESCRIPTION
> Here "character" means "byte"; these functions do not work with wide or multibyte characters.

> [!note] Traducción
> Aquí "carácter" significa "byte"; estas funciones no funcionan con caracteres anchos ni multibyte.

> [!quote] man 3 strrchr — RETURN VALUE
> The strchr() and strrchr() functions return a pointer to the matched character or NULL if the character is not found.  The terminating null byte is considered part of the string, so that if c is specified as '\0', these functions return a pointer to the terminator.

> [!note] Traducción
> strchr() y strrchr() devuelven un puntero al carácter encontrado, o NULL si no se encuentra. El byte nulo final se considera parte de la cadena, así que si c es '\0', estas funciones devuelven un puntero al terminador.

## Puntos clave y casos límite

- Igual que `ft_strchr` pero se queda con la **última** coincidencia (o busca desde el final).
- `ft_strrchr(s, '\0')` devuelve el puntero al terminador.

## Relacionadas

[[ft_strchr]]

---
*Fuente de las citas: `man 3 strrchr` — Linux man-pages 5.10 (Ubuntu 22.04). Texto en inglés copiado literalmente del manual; la traducción es propia. Requisitos copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). En los Mac de 42 (man de BSD/macOS) la redacción puede variar.*
