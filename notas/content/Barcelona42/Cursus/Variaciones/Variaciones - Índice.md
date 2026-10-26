---
tags: [variaciones, indice]
---
# Variaciones - Índice

[[Inicio|← Inicio]] · Variaciones

> Funciones que **no** están en libft ni en Piscine Reloaded, pero que salen en la C Piscine y en el [[Exam Rank 02]]: escribir y leer números en cualquier base, copias de cadenas «a la antigua», potencias, primos y bits. Casi todas son una variación de una función de libft: cada nota dice cuál y en qué se diferencia.

> [!warning] No son parte de libft
> No las metas en tu `libft.a` para la entrega: el subject de libft prohíbe entregar ficheros que no se usan. Si más adelante te sirven (por ejemplo, `ft_putnbr_base` en ft_printf), añádelas en ese proyecto.

> [!info] Enunciados
> No hay un subject local de estas funciones. Cada nota resume el **enunciado habitual**; los detalles (prototipo exacto, qué hacer con una base inválida) cambian entre versiones de la Piscina y del examen. Si tienes el PDF, manda el PDF.

## Bases

| Función | Variación de | Descripción |
|---|---|---|
| [[ft_putnbr_base]] | [[ft_putnbr_fd]] | Escribe un `int` en cualquier base |
| [[ft_atoi_base]] | [[ft_atoi]] | Lee un número escrito en cualquier base |
| [[ft_itoa_base]] | [[ft_itoa]] | `int` → cadena nueva en base 2–16 |
| [[ft_convert_base]] | — | Pasa una cadena de una base a otra |

> [!tip] Las tres reglas de una base válida
> Al menos 2 símbolos, ninguno repetido y sin `'+'` ni `'-'` (en `ft_atoi_base` de la Piscina, tampoco espacios). Las mismas reglas valen para todas las funciones de esta sección.

## Cadenas

| Función | Variación de | Descripción |
|---|---|---|
| [[ft_strcpy]] | [[ft_strlcpy]] | Copia sin límite de tamaño |
| [[ft_strncpy]] | [[ft_strlcpy]] | Copia como mucho n bytes, rellenando con `'\0'` |
| [[ft_strcat]] | [[ft_strlcat]] | Concatena sin límite de tamaño |
| [[ft_strstr]] | [[ft_strnstr]] | Busca una subcadena en toda la cadena |
| [[ft_strrev]] | — | Invierte una cadena en el sitio |

## Matemáticas

| Función | Descripción |
|---|---|
| [[ft_power]] | Potencia, iterativa y recursiva |
| [[ft_fibonacci]] | Término n de Fibonacci, recursivo |
| [[ft_is_prime]] | ¿Es primo? y siguiente primo |

Las de la Piscina que sí están en Piscine Reloaded: [[ft_iterative_factorial]], [[ft_recursive_factorial]] y [[ft_sqrt]].

## Bits

| Función | Descripción |
|---|---|
| [[ft_print_bits]] | Escribe los 8 bits de un byte (`print_bits` del examen) |

## Todos los diagramas

Incrustados desde la sección «Diagrama de flujo» de cada nota.

### ft_putnbr_base

![[ft_putnbr_base#Diagrama de flujo]]

### ft_atoi_base

![[ft_atoi_base#Diagrama de flujo]]

### ft_itoa_base

![[ft_itoa_base#Diagrama de flujo]]

### ft_convert_base

![[ft_convert_base#Diagrama de flujo]]

### ft_strcpy

![[ft_strcpy#Diagrama de flujo]]

### ft_strncpy

![[ft_strncpy#Diagrama de flujo]]

### ft_strcat

![[ft_strcat#Diagrama de flujo]]

### ft_strstr

![[ft_strstr#Diagrama de flujo]]

### ft_strrev

![[ft_strrev#Diagrama de flujo]]

### ft_power

![[ft_power#Diagrama de flujo]]

### ft_fibonacci

![[ft_fibonacci#Diagrama de flujo]]

### ft_is_prime

![[ft_is_prime#Diagrama de flujo]]

### ft_print_bits

![[ft_print_bits#Diagrama de flujo]]

## Relacionadas

[[libft - Índice]] · [[Piscine Reloaded - Índice]] · [[Exam Rank 02]] · [[Recursión]] · [[Tipos enteros y conversiones]] · [[Operadores de bits]]

## Referencias

- Exam Rank 02 (nota [[Exam Rank 02]]) — qué ejercicios de examen usan estas funciones.
- [Positional notation — Wikipedia](https://en.wikipedia.org/wiki/Positional_notation) — sistemas de numeración en base b.
- Cada nota tiene sus propias referencias (`man`, cppreference, ejemplos compilados).
