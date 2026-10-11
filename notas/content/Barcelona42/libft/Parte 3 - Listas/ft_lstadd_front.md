---
tags: [libft, libft/parte3]
funcion: ft_lstadd_front
original: ninguna
externas: "Ninguna"
estado: pendiente
---
# ft_lstadd_front

[[libft - Índice|← Índice]] · Parte 3 · Listas enlazadas

> Añade un nodo al principio

## Prototipo

```c
void ft_lstadd_front(t_list **lst, t_list *new);
```

> [!info] Sin función original
> No existe en la libc, así que no tiene página `man 3`. El comportamiento lo define el **subject** de libft.
> Las citas de `man` de abajo son de las funciones externas que usa.

## Comportamiento según el subject

> [!quote] Subject v19.3 — Descripción
> Añade el nodo ‘new’ al principio de la lista ‘lst’.

| | |
|---|---|
| **Parámetros** | `lst`: la dirección de memoria de un puntero al primer nodo de una lista.<br>`new`: un puntero al nodo que se añade al principio de la lista. |
| **Valor devuelto** | Nada |
| **Funciones autorizadas** | Ninguna |

## Puntos clave y casos límite

- `new->next = *lst; *lst = new;`
- Recibe `t_list **` porque cambia la cabeza de la lista.

## Relacionadas

[[ft_lstnew]] · [[ft_lstadd_back]]

---
*Descripción, parámetros y valor devuelto copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). Citas de `man`: Linux man-pages 5.10 (Ubuntu 22.04), con traducción propia.*
