---
tags: [libft, libft/parte3]
funcion: ft_lstsize
original: ninguna
externas: "Ninguna"
estado: pendiente
---
# ft_lstsize

[[libft - Índice|← Índice]] · Parte 3 · Listas enlazadas

> Cuenta los nodos

## Prototipo

```c
unsigned int ft_lstsize(t_list *lst);
```

> [!info] Sin función original
> No existe en la libc, así que no tiene página `man 3`. El comportamiento lo define el **subject** de libft.
> Las citas de `man` de abajo son de las funciones externas que usa.

## Comportamiento según el subject

> [!quote] Subject v19.3 — Descripción
> Cuenta el número de nodos de una lista.

| | |
|---|---|
| **Parámetros** | `lst`: el principio de la lista. |
| **Valor devuelto** | La longitud de la lista. |
| **Funciones autorizadas** | Ninguna |

## Puntos clave y casos límite

- Lista vacía (`NULL`) → 0.

## Relacionadas

[[ft_lstlast]]

---
*Descripción, parámetros y valor devuelto copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). Citas de `man`: Linux man-pages 5.10 (Ubuntu 22.04), con traducción propia.*
