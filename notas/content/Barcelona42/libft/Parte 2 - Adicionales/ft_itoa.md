---
tags: [libft, libft/parte2]
funcion: ft_itoa
original: ninguna
externas: "malloc"
estado: pendiente
---
# ft_itoa

[[libft - Índice|← Índice]] · Parte 2 · Funciones adicionales

> int → cadena

## Prototipo

```c
char *ft_itoa(int n);
```

> [!info] Sin función original
> No existe en la libc, así que no tiene página `man 3`. El comportamiento lo define el **subject** de libft.
> Las citas de `man` de abajo son de las funciones externas que usa.

## Comportamiento según el subject

> [!quote] Subject v19.3 — Descripción
> Reserva memoria (utilizando malloc(3)) y devuelve una cadena que represente el valor del número entero recibido como argumento. Debe ser capaz de manejar números negativos.

| | |
|---|---|
| **Parámetros** | `n`: el entero a convertir. |
| **Valor devuelto** | La cadena que represente el número.<br>NULL si falla la reserva de memoria. |
| **Funciones autorizadas** | malloc |

## Funciones externas (man)

> [!quote] man 3 malloc — RETURN VALUE
> The malloc() and calloc() functions return a pointer to the allocated memory, which is suitably aligned for any built-in type.  On error, these functions return NULL.

> [!note] Traducción
> malloc() y calloc() devuelven un puntero a la memoria reservada, alineada adecuadamente para cualquier tipo predefinido. En caso de error, devuelven NULL.

## Puntos clave y casos límite

- `INT_MIN` (-2147483648) no se puede negar en `int`: usa `long` o trátalo aparte.
- `0` → `"0"` (cuidado con el bucle que cuenta dígitos).

## Relacionadas

[[ft_atoi]] · [[ft_putnbr_fd]]

---
*Descripción, parámetros y valor devuelto copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). Citas de `man`: Linux man-pages 5.10 (Ubuntu 22.04), con traducción propia.*
