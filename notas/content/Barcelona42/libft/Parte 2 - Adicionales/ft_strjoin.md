---
tags: [libft, libft/parte2]
funcion: ft_strjoin
original: ninguna
externas: "malloc"
estado: pendiente
---
# ft_strjoin

[[libft - Índice|← Índice]] · Parte 2 · Funciones adicionales

> Concatena dos cadenas en memoria nueva

## Prototipo

```c
char *ft_strjoin(char const *s1, char const *s2);
```

> [!info] Sin función original
> No existe en la libc, así que no tiene página `man 3`. El comportamiento lo define el **subject** de libft.
> Las citas de `man` de abajo son de las funciones externas que usa.

## Comportamiento según el subject

> [!quote] Subject v19.3 — Descripción
> Reserva memoria (con malloc(3)) y devuelve una nueva cadena de caracteres, formada por la concatenación de ‘s1’ y ‘s2’.

| | |
|---|---|
| **Parámetros** | `s1`: La primera cadena.<br>`s2`: La cadena a añadir a ‘s1’. |
| **Valor devuelto** | La nueva cadena de caracteres.<br>NULL si falla la reserva de memoria. |
| **Funciones autorizadas** | malloc |

## Funciones externas (man)

> [!quote] man 3 malloc — RETURN VALUE
> The malloc() and calloc() functions return a pointer to the allocated memory, which is suitably aligned for any built-in type.  On error, these functions return NULL.

> [!note] Traducción
> malloc() y calloc() devuelven un puntero a la memoria reservada, alineada adecuadamente para cualquier tipo predefinido. En caso de error, devuelven NULL.

## Puntos clave y casos límite

- Reserva `ft_strlen(s1) + ft_strlen(s2) + 1`.
- Útil combinar `ft_strlcpy` + `ft_strlcat`.

## Relacionadas

[[ft_strlcat]] · [[ft_strlcpy]]

---
*Descripción, parámetros y valor devuelto copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). Citas de `man`: Linux man-pages 5.10 (Ubuntu 22.04), con traducción propia.*
