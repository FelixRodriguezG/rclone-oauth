---
tags: [libft, libft/parte3]
funcion: ft_lstdelone
original: ninguna
externas: "free"
estado: pendiente
---
# ft_lstdelone

[[libft - Índice|← Índice]] · Parte 3 · Listas enlazadas

> Libera un nodo y su contenido

## Prototipo

```c
void ft_lstdelone(t_list *lst, void (*del)(void *));
```

> [!info] Sin función original
> No existe en la libc, así que no tiene página `man 3`. El comportamiento lo define el **subject** de libft.
> Las citas de `man` de abajo son de las funciones externas que usa.

## Comportamiento según el subject

> [!quote] Subject v19.3 — Descripción
> Recibe como parámetro un nodo ‘lst’ y libera la memoria del contenido utilizando la función ‘del’ dada como parámetro. También libera el nodo en sí mismo, pero no libera el siguiente nodo.

| | |
|---|---|
| **Parámetros** | `lst`: el nodo a liberar.<br>`del`: un puntero a la función utilizada para liberar el contenido del nodo. |
| **Valor devuelto** | Nada |
| **Funciones autorizadas** | free |

## Funciones externas (man)

> [!quote] man 3 free — DESCRIPTION
> The free() function frees the memory space pointed to by ptr, which must have been returned by a previous call to malloc(), calloc(), or realloc().  Otherwise, or if free(ptr) has already been called before, undefined behavior occurs.  If ptr is NULL, no operation is performed.

> [!note] Traducción
> La función free() libera el espacio de memoria apuntado por ptr, que debe haber sido devuelto por una llamada previa a malloc(), calloc() o realloc(). Si no, o si ya se llamó antes a free(ptr), el comportamiento es indefinido. Si ptr es NULL, no se hace nada.

## Puntos clave y casos límite

- `del(lst->content); free(lst);` — el siguiente nodo queda intacto.

## Relacionadas

[[ft_lstclear]]

---
*Descripción, parámetros y valor devuelto copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). Citas de `man`: Linux man-pages 5.10 (Ubuntu 22.04), con traducción propia.*
