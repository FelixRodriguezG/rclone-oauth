---
tags: [libft, libft/parte2]
funcion: ft_split
original: ninguna
externas: "malloc, free"
estado: pendiente
---
# ft_split

[[libft - Índice|← Índice]] · Parte 2 · Funciones adicionales

> Divide una cadena por un delimitador

## Prototipo

```c
char **ft_split(char const *s, char c);
```

> [!info] Sin función original
> No existe en la libc, así que no tiene página `man 3`. El comportamiento lo define el **subject** de libft.
> Las citas de `man` de abajo son de las funciones externas que usa.

## Comportamiento según el subject

> [!quote] Subject v19.3 — Descripción
> Reserva memoria (utilizando malloc(3)) y devuelve un arreglo (array) de cadenas obtenido al dividir la cadena ‘s’ en subcadenas utilizando el carácter ‘c’ como delimitador.
> Cada cadena del arreglo devuelto se reserva de manera independiente.
> El arreglo de punteros también se reserva dinámicamente.
> El arreglo devuelto debe terminar con un puntero a NULL.

| | |
|---|---|
| **Parámetros** | `s`: La cadena que se va a dividir.<br>`c`: El carácter delimitador. |
| **Valor devuelto** | El arreglo de nuevas cadenas resultante de la división.<br>NULL si falla cualquier reserva de memoria.<br>La estructura devuelta debe liberarse utilizando: 1) free() sobre cada cadena del arreglo; 2) free() sobre el arreglo. |
| **Funciones autorizadas** | malloc, free |

## Funciones externas (man)

> [!quote] man 3 malloc — RETURN VALUE
> The malloc() and calloc() functions return a pointer to the allocated memory, which is suitably aligned for any built-in type.  On error, these functions return NULL.

> [!note] Traducción
> malloc() y calloc() devuelven un puntero a la memoria reservada, alineada adecuadamente para cualquier tipo predefinido. En caso de error, devuelven NULL.

> [!quote] man 3 free — DESCRIPTION
> The free() function frees the memory space pointed to by ptr, which must have been returned by a previous call to malloc(), calloc(), or realloc().  Otherwise, or if free(ptr) has already been called before, undefined behavior occurs.  If ptr is NULL, no operation is performed.

> [!note] Traducción
> La función free() libera el espacio de memoria apuntado por ptr, que debe haber sido devuelto por una llamada previa a malloc(), calloc() o realloc(). Si no, o si ya se llamó antes a free(ptr), el comportamiento es indefinido. Si ptr es NULL, no se hace nada.

## Puntos clave y casos límite

- Delimitadores consecutivos, al principio o al final no generan cadenas vacías: `ft_split("  a  b ", ' ')` → `{"a", "b", NULL}`.
- Si un `malloc` intermedio falla, libera **todo** lo reservado antes de devolver NULL (por eso se permite `free`).
- Estructura típica: contar palabras → reservar array → reservar cada palabra con `ft_substr`.

## Relacionadas

[[ft_substr]] · [[ft_strchr]]

---
*Descripción, parámetros y valor devuelto copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). Citas de `man`: Linux man-pages 5.10 (Ubuntu 22.04), con traducción propia.*
