---
tags:
  - libft
  - libft/parte2
funcion: ft_substr
original: ninguna
externas: malloc
estado: pendiente
---
# ft_substr

[[libft - Índice|← Índice]] · Parte 2 · Funciones adicionales

> Subcadena desde start de longitud máxima len

## Prototipo

```c
char *ft_substr(char const *s, unsigned int start, size_t len);
```

> [!info] Sin función original
> No existe en la libc, así que no tiene página `man 3`. El comportamiento lo define el **subject** de libft.
> Las citas de `man` de abajo son de las funciones externas que usa.

## Comportamiento según el subject

> [!quote] Subject v19.3 — Descripción
> Reserva memoria (con malloc(3)) y devuelve una subcadena de caracteres de la cadena ‘s’.
> La subcadena comienza en el índice ‘start’ y tiene una longitud máxima ‘len’.

| | |
|---|---|
| **Parámetros** | `s`: La cadena original desde la que se crea la subcadena.<br>`start`: El índice del carácter en ‘s’ desde el que empieza la subcadena.<br>`len`: La longitud máxima de la subcadena. |
| **Valor devuelto** | La subcadena de caracteres resultante.<br>NULL si falla la reserva de memoria. |
| **Funciones autorizadas** | malloc |

## Funciones externas (man)

> [!quote] man 3 malloc — RETURN VALUE
> The malloc() and calloc() functions return a pointer to the allocated memory, which is suitably aligned for any built-in type.  On error, these functions return NULL.

> [!note] Traducción
> malloc() y calloc() devuelven un puntero a la memoria reservada, alineada adecuadamente para cualquier tipo predefinido. En caso de error, devuelven NULL.

## Puntos clave y casos límite

- Si `start >= ft_strlen(s)`: devuelve una cadena vacía reservada (`ft_strdup("")`), no NULL.
- Reserva solo lo necesario: `min(len, ft_strlen(s) - start) + 1`.

## Relacionadas

[[ft_strdup]] · [[ft_strlcpy]]

---
*Descripción, parámetros y valor devuelto copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). Citas de `man`: Linux man-pages 5.10 (Ubuntu 22.04), con traducción propia.*
