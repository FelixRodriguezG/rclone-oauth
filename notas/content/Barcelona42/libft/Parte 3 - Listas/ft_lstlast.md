---
tags: [libft, libft/parte3]
funcion: ft_lstlast
original: ninguna
externas: "Ninguna"
estado: pendiente
---
# ft_lstlast

[[libft - Índice|← Índice]] · Parte 3 · Listas enlazadas

> Devuelve el último nodo

## Prototipo

```c
t_list *ft_lstlast(t_list *lst);
```

> [!info] Sin función original
> No existe en la libc, así que no tiene página `man 3`. El comportamiento lo define el **subject** de libft.
> Las citas de `man` de abajo son de las funciones externas que usa.

## Comportamiento según el subject

> [!quote] Subject v19.3 — Descripción
> Devuelve el último nodo de la lista.

| | |
|---|---|
| **Parámetros** | `lst`: el principio de la lista. |
| **Valor devuelto** | Último nodo de la lista. |
| **Funciones autorizadas** | Ninguna |

## Puntos clave y casos límite

- Lista vacía → NULL.
- Avanza mientras `lst->next != NULL`.

## Relacionadas

[[ft_lstadd_back]] · [[ft_lstsize]]

---
*Descripción, parámetros y valor devuelto copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). Citas de `man`: Linux man-pages 5.10 (Ubuntu 22.04), con traducción propia.*
