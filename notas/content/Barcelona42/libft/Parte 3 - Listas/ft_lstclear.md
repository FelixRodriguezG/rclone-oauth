---
tags: [libft, libft/parte3]
funcion: ft_lstclear
original: ninguna
externas: "free"
estado: pendiente
---
# ft_lstclear

[[libft - Índice|← Índice]] · Parte 3 · Listas enlazadas

> Libera toda la lista

## Prototipo

```c
void ft_lstclear(t_list **lst, void (*del)(void *));
```

> [!info] Sin función original
> No existe en la libc, así que no tiene página `man 3`. El comportamiento lo define el **subject** de libft.
> Las citas de `man` de abajo son de las funciones externas que usa.

## Comportamiento según el subject

> [!quote] Subject v19.3 — Descripción
> Elimina y libera el nodo ‘lst’ dado y todos los consecutivos del mismo, utilizando la función ‘del’ y free(3).
> Al final, el puntero a la lista debe ser NULL.

| | |
|---|---|
| **Parámetros** | `lst`: la dirección de un puntero a un nodo.<br>`del`: un puntero a la función utilizado para eliminar el contenido de un nodo. |
| **Valor devuelto** | Nada |
| **Funciones autorizadas** | free |

## Funciones externas (man)

> [!quote] man 3 free — DESCRIPTION
> The free() function frees the memory space pointed to by ptr, which must have been returned by a previous call to malloc(), calloc(), or realloc().  Otherwise, or if free(ptr) has already been called before, undefined behavior occurs.  If ptr is NULL, no operation is performed.

> [!note] Traducción
> La función free() libera el espacio de memoria apuntado por ptr, que debe haber sido devuelto por una llamada previa a malloc(), calloc() o realloc(). Si no, o si ya se llamó antes a free(ptr), el comportamiento es indefinido. Si ptr es NULL, no se hace nada.

## Puntos clave y casos límite

- Guarda `next` **antes** de liberar el nodo actual.
- Termina con `*lst = NULL;`.

## Relacionadas

[[ft_lstdelone]] · [[ft_lstmap]]

---
*Descripción, parámetros y valor devuelto copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). Citas de `man`: Linux man-pages 5.10 (Ubuntu 22.04), con traducción propia.*
