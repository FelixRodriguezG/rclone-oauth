---
tags: [libft, libft/parte2]
funcion: ft_strtrim
original: ninguna
externas: "malloc"
estado: pendiente
---
# ft_strtrim

[[libft - Índice|← Índice]] · Parte 2 · Funciones adicionales

> Elimina caracteres de set al principio y al final

## Prototipo

```c
char *ft_strtrim(char const *s1, char const *set);
```

> [!info] Sin función original
> No existe en la libc, así que no tiene página `man 3`. El comportamiento lo define el **subject** de libft.
> Las citas de `man` de abajo son de las funciones externas que usa.

## Comportamiento según el subject

> [!quote] Subject v19.3 — Descripción
> Reserva memoria (con malloc(3)) y devuelve una copia de ‘s1’ con los caracteres de ‘set’ eliminados al principio y al final.

| | |
|---|---|
| **Parámetros** | `s1`: La cadena de caracteres que debe ser recortada.<br>`set`: Los caracteres a eliminar de la cadena en cuestión. |
| **Valor devuelto** | La cadena recortada resultante.<br>NULL si falla la reserva de memoria. |
| **Funciones autorizadas** | malloc |

## Funciones externas (man)

> [!quote] man 3 malloc — RETURN VALUE
> The malloc() and calloc() functions return a pointer to the allocated memory, which is suitably aligned for any built-in type.  On error, these functions return NULL.

> [!note] Traducción
> malloc() y calloc() devuelven un puntero a la memoria reservada, alineada adecuadamente para cualquier tipo predefinido. En caso de error, devuelven NULL.

## Puntos clave y casos límite

- Los caracteres del medio **no** se tocan: `ft_strtrim("xxhoxlaxx", "x")` → `"hoxla"`.
- Si todo son caracteres de `set`, devuelve una cadena vacía reservada.
- `ft_strchr(set, c)` sirve para saber si `c` está en `set`.

## Relacionadas

[[ft_strchr]] · [[ft_substr]]

---
*Descripción, parámetros y valor devuelto copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). Citas de `man`: Linux man-pages 5.10 (Ubuntu 22.04), con traducción propia.*
