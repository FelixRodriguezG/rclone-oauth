---
tags: [libft, libft/parte3]
funcion: ft_lstadd_back
original: ninguna
externas: "Ninguna"
estado: pendiente
---
# ft_lstadd_back

[[libft - Índice|← Índice]] · Parte 3 · Listas enlazadas

> Añade un nodo al final

## Prototipo

```c
void ft_lstadd_back(t_list **lst, t_list *new);
```

> [!info] Sin función original
> No existe en la libc, así que no tiene página `man 3`. El comportamiento lo define el **subject** de libft.
> Las citas de `man` de abajo son de las funciones externas que usa.

## Comportamiento según el subject

> [!quote] Subject v19.3 — Descripción
> Añade el nodo ‘new’ al final de la lista ‘lst’.

| | |
|---|---|
| **Parámetros** | `lst`: el puntero al primer nodo de una lista.<br>`new`: el puntero a un nodo que se añade a la lista. |
| **Valor devuelto** | Nada |
| **Funciones autorizadas** | Ninguna |

## Puntos clave y casos límite

- Si `*lst == NULL`, `new` pasa a ser la cabeza.
- Si no, `ft_lstlast(*lst)->next = new;`.

## Relacionadas

[[ft_lstlast]] · [[ft_lstadd_front]]

---
*Descripción, parámetros y valor devuelto copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). Citas de `man`: Linux man-pages 5.10 (Ubuntu 22.04), con traducción propia.*
