---
tags: [libft, libft/parte1, libft/cadenas]
funcion: ft_strnstr
original: "strnstr(3bsd)"
cabecera: "<string.h> / <bsd/string.h>"
estado: pendiente
---
# ft_strnstr

[[libft - Índice|← Índice]] · Parte 1 · Cadenas

> Busca una subcadena en los primeros len caracteres

## Prototipo

```c
// Original — strnstr(3bsd)
#include <string.h>
char *strnstr(const char *big, const char *little, size_t len);

// libft
char *ft_strnstr(const char *big, const char *little, size_t len);
```

Consultar: `man 3bsd strnstr`

## Requisitos del subject

> [!quote] Subject v19.3 — Parte 1
> Tus funciones tendrán los mismos prototipos y comportamientos que las funciones originales, siguiendo fielmente las definiciones indicadas en la página del man de cada función. La única diferencia será el nombre: todas deberán comenzar con el prefijo “ft_”.
> 
> Deberás escribir tus propias versiones de las siguientes funciones originales. No deben depender de ninguna función externa.

## Comportamiento según el manual

> [!quote] man 3bsd strnstr — DESCRIPTION
> The strnstr() function locates the first occurrence of the null-terminated string little in the string big, where not more than len characters are searched.  Characters that appear after a '\0' character are not searched.  Since the strnstr() function is a FreeBSD specific API, it should only be used when portability is not a concern.

> [!note] Traducción
> La función strnstr() localiza la primera aparición de la cadena little (terminada en nulo) en la cadena big, buscando en no más de len caracteres. Los caracteres que aparecen después de un '\0' no se examinan. Como strnstr() es una API específica de FreeBSD, solo debe usarse cuando la portabilidad no importe.

> [!quote] man 3bsd strnstr — RETURN VALUES
> If little is an empty string, big is returned; if little occurs nowhere in big, NULL is returned; otherwise a pointer to the first character of the first occurrence of little is returned.

> [!note] Traducción
> Si little es una cadena vacía, se devuelve big; si little no aparece en big, se devuelve NULL; en otro caso se devuelve un puntero al primer carácter de la primera aparición de little.

## Puntos clave y casos límite

- La coincidencia completa debe caber dentro de los `len` primeros caracteres.
- Ejemplo del man: `strnstr("Foo Bar Baz", "Bar", 4)` → NULL.
- Para probar la original en Linux: `#include <bsd/string.h>` y `-lbsd`.

## Relacionadas

[[ft_strncmp]] · [[ft_strchr]]

---
*Fuente de las citas: `man 3bsd strnstr` — libbsd (libbsd-dev, Ubuntu 22.04). Texto en inglés copiado literalmente del manual; la traducción es propia. Requisitos copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). En los Mac de 42 (man de BSD/macOS) la redacción puede variar.*
