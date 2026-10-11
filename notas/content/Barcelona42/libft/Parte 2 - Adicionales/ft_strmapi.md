---
tags: [libft, libft/parte2]
funcion: ft_strmapi
original: ninguna
externas: "malloc"
estado: pendiente
---
# ft_strmapi

[[libft - Índice|← Índice]] · Parte 2 · Funciones adicionales

> Nueva cadena aplicando f(índice, carácter)

## Prototipo

```c
char *ft_strmapi(char const *s, char (*f)(unsigned int, char));
```

> [!info] Sin función original
> No existe en la libc, así que no tiene página `man 3`. El comportamiento lo define el **subject** de libft.
> Las citas de `man` de abajo son de las funciones externas que usa.

## Comportamiento según el subject

> [!quote] Subject v19.3 — Descripción
> Aplica la función ‘f’ a cada carácter de la cadena ‘s’, pasando su índice como primer argumento y el propio carácter como segundo argumento. Se crea una nueva cadena (utilizando malloc(3)) para almacenar los resultados de las sucesivas aplicaciones de ‘f’.

| | |
|---|---|
| **Parámetros** | `s`: La cadena sobre la que iterar.<br>`f`: La función a aplicar sobre cada carácter. |
| **Valor devuelto** | La cadena creada tras el correcto uso de ‘f’ sobre cada carácter.<br>NULL si falla la reserva de memoria. |
| **Funciones autorizadas** | malloc |

## Funciones externas (man)

> [!quote] man 3 malloc — RETURN VALUE
> The malloc() and calloc() functions return a pointer to the allocated memory, which is suitably aligned for any built-in type.  On error, these functions return NULL.

> [!note] Traducción
> malloc() y calloc() devuelven un puntero a la memoria reservada, alineada adecuadamente para cualquier tipo predefinido. En caso de error, devuelven NULL.

## Puntos clave y casos límite

- `s` no se modifica: el resultado va en memoria nueva.
- No olvides el `'\0'` final.

## Relacionadas

[[ft_striteri]]

---
*Descripción, parámetros y valor devuelto copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). Citas de `man`: Linux man-pages 5.10 (Ubuntu 22.04), con traducción propia.*
