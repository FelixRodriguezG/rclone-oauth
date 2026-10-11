---
tags: [libft, libft/parte3]
funcion: ft_lstmap
original: ninguna
externas: "malloc, free"
estado: pendiente
---
# ft_lstmap

[[libft - Índice|← Índice]] · Parte 3 · Listas enlazadas

> Nueva lista aplicando f a cada contenido

## Prototipo

```c
t_list *ft_lstmap(t_list *lst, void *(*f)(void *), void (*del)(void *));
```

> [!info] Sin función original
> No existe en la libc, así que no tiene página `man 3`. El comportamiento lo define el **subject** de libft.
> Las citas de `man` de abajo son de las funciones externas que usa.

## Comportamiento según el subject

> [!quote] Subject v19.3 — Descripción
> Itera la lista ‘lst’ y aplica la función ‘f’ al contenido de cada nodo. Crea una lista resultante de aplicar sucesivamente la función ‘f’ a cada nodo. La función ‘del’ se utiliza para eliminar el contenido de un nodo si es necesario.

| | |
|---|---|
| **Parámetros** | `lst`: un puntero a un nodo.<br>`f`: la dirección de un puntero a una función usada en la iteración de cada elemento de la lista.<br>`del`: un puntero a función utilizado para eliminar el contenido de un nodo, si es necesario. |
| **Valor devuelto** | La nueva lista.<br>NULL si falla la reserva de memoria. |
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

- Si falla un `ft_lstnew`, libera el contenido recién creado con `del` y toda la lista nueva con `ft_lstclear`, y devuelve NULL.
- La lista original no se modifica.

## Relacionadas

[[ft_lstnew]] · [[ft_lstclear]] · [[ft_lstiter]]

---
*Descripción, parámetros y valor devuelto copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). Citas de `man`: Linux man-pages 5.10 (Ubuntu 22.04), con traducción propia.*
