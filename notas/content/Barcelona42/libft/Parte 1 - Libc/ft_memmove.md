---
tags: [libft, libft/parte1, libft/memoria]
funcion: ft_memmove
original: "memmove(3)"
cabecera: "<string.h>"
estado: hecho
---
# ft_memmove

[[libft - Índice|← Índice]] · Parte 1 · Memoria

> Copia n bytes (permite solapamiento)

## Prototipo

```c
// Original — memmove(3)
#include <string.h>
void *memmove(void *dest, const void *src, size_t n);

// libft
void *ft_memmove(void *dest, const void *src, size_t n);
```

Consultar: `man 3 memmove`

## Requisitos del subject

> [!quote] Subject v19.3 — Parte 1
> Tus funciones tendrán los mismos prototipos y comportamientos que las funciones originales, siguiendo fielmente las definiciones indicadas en la página del man de cada función. La única diferencia será el nombre: todas deberán comenzar con el prefijo “ft_”.
> 
> Deberás escribir tus propias versiones de las siguientes funciones originales. No deben depender de ninguna función externa.

## Comportamiento según el manual

> [!quote] man 3 memmove — DESCRIPTION
> The memmove() function copies n bytes from memory area src to memory area dest.  The memory areas may overlap: copying takes place as though the bytes in src are first copied into a temporary array that does not overlap src or dest, and the bytes are then copied from the temporary array to dest.

> [!note] Traducción
> La función memmove() copia n bytes de la zona de memoria src a la zona de memoria dest. Las zonas pueden solaparse: la copia se hace como si los bytes de src se copiaran primero a un array temporal que no se solapa ni con src ni con dest, y después se copiaran de ese array a dest.

> [!quote] man 3 memmove — RETURN VALUE
> The memmove() function returns a pointer to dest.

> [!note] Traducción
> La función memmove() devuelve un puntero a dest.

## Puntos clave y casos límite

- No se puede usar un buffer temporal con malloc. El truco: si `dest > src`, copiar **de atrás hacia delante**; si no, de delante hacia atrás.
- Caso de prueba típico: `ft_memmove(s + 2, s, 5)` y `ft_memmove(s, s + 2, 5)`.

## Relacionadas

[[ft_memcpy]]

---
*Fuente de las citas: `man 3 memmove` — Linux man-pages 5.10 (Ubuntu 22.04). Texto en inglés copiado literalmente del manual; la traducción es propia. Requisitos copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). En los Mac de 42 (man de BSD/macOS) la redacción puede variar.*
