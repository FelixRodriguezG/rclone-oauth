---
tags: [libft, libft/parte3]
funcion: ft_lstnew
original: ninguna
externas: "malloc"
estado: pendiente
---
# ft_lstnew

[[libft - Índice|← Índice]] · Parte 3 · Listas enlazadas

> Crea un nodo nuevo

## Prototipo

```c
t_list *ft_lstnew(void *content);
```

> [!info] Sin función original
> No existe en la libc, así que no tiene página `man 3`. El comportamiento lo define el **subject** de libft.
> Las citas de `man` de abajo son de las funciones externas que usa.

## Comportamiento según el subject

> [!quote] Subject v19.3 — Descripción
> Reserva memoria (usando malloc(3)) y devuelve un nuevo nodo. La variable ‘content’ se inicializa con el contenido del parámetro ‘content’. Mientras que la variable ‘next’ se inicializa con NULL.

| | |
|---|---|
| **Parámetros** | `content`: el contenido con el que se crea el nodo. |
| **Valor devuelto** | Un puntero al nuevo nodo |
| **Funciones autorizadas** | malloc |

## Funciones externas (man)

> [!quote] man 3 malloc — RETURN VALUE
> The malloc() and calloc() functions return a pointer to the allocated memory, which is suitably aligned for any built-in type.  On error, these functions return NULL.

> [!note] Traducción
> malloc() y calloc() devuelven un puntero a la memoria reservada, alineada adecuadamente para cualquier tipo predefinido. En caso de error, devuelven NULL.

## Puntos clave y casos límite

- No copia el contenido: guarda el puntero tal cual.
- Devuelve NULL si `malloc` falla.

## Relacionadas

[[ft_lstadd_front]] · [[ft_lstadd_back]]

---
*Descripción, parámetros y valor devuelto copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). Citas de `man`: Linux man-pages 5.10 (Ubuntu 22.04), con traducción propia.*
