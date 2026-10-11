---
tags: [libft, libft/parte3]
funcion: ft_lstiter
original: ninguna
externas: "Ninguna"
estado: pendiente
---
# ft_lstiter

[[libft - Índice|← Índice]] · Parte 3 · Listas enlazadas

> Aplica f al contenido de cada nodo

## Prototipo

```c
void ft_lstiter(t_list *lst, void (*f)(void *));
```

> [!info] Sin función original
> No existe en la libc, así que no tiene página `man 3`. El comportamiento lo define el **subject** de libft.
> Las citas de `man` de abajo son de las funciones externas que usa.

## Comportamiento según el subject

> [!quote] Subject v19.3 — Descripción
> Itera la lista ‘lst’ y aplica la función ‘f’ en el contenido de cada nodo.

| | |
|---|---|
| **Parámetros** | `lst`: un puntero al primer nodo.<br>`f`: un puntero a la función que utilizará cada nodo. |
| **Valor devuelto** | Nada |
| **Funciones autorizadas** | Ninguna |

## Puntos clave y casos límite

- Se pasa `lst->content`, no el nodo.

## Relacionadas

[[ft_lstmap]]

---
*Descripción, parámetros y valor devuelto copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). Citas de `man`: Linux man-pages 5.10 (Ubuntu 22.04), con traducción propia.*
