---
tags: [libft, libft/parte2]
funcion: ft_striteri
original: ninguna
externas: "Ninguna"
estado: pendiente
---
# ft_striteri

[[libft - Índice|← Índice]] · Parte 2 · Funciones adicionales

> Aplica f(índice, &carácter) en el sitio

## Prototipo

```c
void ft_striteri(char *s, void (*f)(unsigned int, char*));
```

> [!info] Sin función original
> No existe en la libc, así que no tiene página `man 3`. El comportamiento lo define el **subject** de libft.
> Las citas de `man` de abajo son de las funciones externas que usa.

## Comportamiento según el subject

> [!quote] Subject v19.3 — Descripción
> Aplica la función ‘f’ a cada carácter de la string ‘s’, pasando como parámetros el índice de cada carácter dentro de ‘s’ y la dirección del propio carácter, que puede modificarse si es necesario.

| | |
|---|---|
| **Parámetros** | `s`: La cadena sobre la que iterar.<br>`f`: La función a aplicar sobre cada carácter. |
| **Valor devuelto** | Nada |
| **Funciones autorizadas** | Ninguna |

## Puntos clave y casos límite

- Diferencia con `ft_strmapi`: modifica `s` directamente (recibe `&s[i]`), sin reservar memoria.

## Relacionadas

[[ft_strmapi]]

---
*Descripción, parámetros y valor devuelto copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). Citas de `man`: Linux man-pages 5.10 (Ubuntu 22.04), con traducción propia.*
