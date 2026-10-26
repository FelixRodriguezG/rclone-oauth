---
tags: [piscine-reloaded, indice]
---
# Piscine Reloaded - Índice

[[Inicio|← Inicio]] · Piscine Reloaded

> **Piscine Reloaded** es un repaso de la C Piscine antes de empezar el Common Core: 6 ejercicios de shell y 22 de C (funciones, bucles, punteros, structs, `malloc`, Makefile y ficheros). La corrige solo la **Moulinette**. Esta nota es el índice de todos los ejercicios. Los que se repiten en libft enlazan directamente a la nota de libft.

> [!tip] Diagramas
> [[Piscine Reloaded - Diagramas de flujo]] reúne el diagrama de flujo de cada función en una sola página.
> Para variantes de la Piscina que no están aquí (`ft_putnbr_base`, `ft_atoi_base`, `ft_strcpy`…), ver [[Variaciones - Índice]].

## Reglas del subject (v2.0)

> [!quote] Subject Piscine Reloaded v2.0 — Instrucciones
> • Tus ejercicios serán corregidos únicamente por la Moulinette.
> • La Moulinette utiliza el programa norminette para comprobar La Norma en sus archivos.
> • Los ejercicios en Shell deberán ser ejecutados con /bin/sh
> • No se tendrá en cuenta un ejercicio complejo si no se ha conseguido realizar perfectamente un ejercicio más sencillo.
> • Solamente hay que entregar una función main() si lo que se pide es un programa.
> • La Moulinette compila con los flags -Wall -Wextra -Werror y utiliza cc.
> • No puedes dejar en tu directorio ningún archivo que no se haya indicado de forma explícita en los enunciados de los ejercicios.
> • Si ft_putchar() es una función autorizada, la Moulinette compilará el código usando su propio ft_putchar.c

Consecuencias prácticas:

- **En orden**: si falla `ex06`, todo lo que va después no cuenta, aunque esté bien.
- **Sin `main`** en las funciones. Para probarlas, usa un `main.c` aparte que no entregas (ver [[Git para 42]]).
- **`ft_putchar` no se entrega**: la Moulinette pone el suyo. En local, escribe un `ft_putchar.c` aparte y compílalo junto al ejercicio.
- Todo el código pasa por la [[La Norma|Norma]], salvo la macro de [[ft_abs]], que el propio subject autoriza.

## Shell (ex00–ex05)

| Ej. | Nombre | Qué piden | Ver |
|---|---|---|---|
| ex00 | Dame más… | Ficheros con permisos, tamaños, enlace simbólico y fechas exactos en `ls -l`, empaquetados con `tar` | `man chmod`, `touch -t`, `ln -s` |
| ex01 | Z | Un fichero `z` que muestre `Z` y un salto de línea con `cat` | — |
| ex02 | clean | Un solo comando `find` que muestre y borre los ficheros acabados en `~` o con forma `#…#` | `man find` (`-name`, `-o`, `-print`, `-delete`) |
| ex03 | find_sh | Nombres de los `.sh` del árbol, sin la extensión | `man find`, [[grep, sed y awk lo justo]] |
| ex04 | MAC | Las direcciones MAC del ordenador, una por línea | [[Redirecciones y tuberías]] · `ip link` en [[Born2beroot - Índice]] |
| ex05 | ¿Puedes crearlo? | Un fichero llamado `"\?$*'MaRViN'*$?\"` que contenga `42` sin salto de línea | [[Variables y comillas en Bash]] |

## C (ex06–ex27)

### Salida

| Ej. | Función | Autorizadas | Descripción | En libft |
|---|---|---|---|---|
| ex06 | [[ft_print_alphabet]] | ft_putchar | Escribe `a`…`z` | — |
| ex07 | [[ft_print_numbers]] | ft_putchar | Escribe `0`…`9` | — |
| ex08 | [[ft_is_negative]] | ft_putchar | Escribe `N` si n < 0, si no `P` | — |
| ex15 | `ft_putstr` | ft_putchar | Escribe una cadena | [[ft_putstr_fd]] con `fd = 1` |

### Punteros

| Ej. | Función | Autorizadas | Descripción | En libft |
|---|---|---|---|---|
| ex09 | [[ft_ft]] | Ninguna | Pone a 42 el entero apuntado | — |
| ex10 | [[ft_swap]] | Ninguna | Intercambia dos enteros | — |
| ex11 | [[ft_div_mod]] | Ninguna | Cociente y resto por punteros | — |

### Matemáticas

| Ej. | Función | Autorizadas | Descripción | En libft |
|---|---|---|---|---|
| ex12 | [[ft_iterative_factorial]] | Ninguna | n! con un bucle | — |
| ex13 | [[ft_recursive_factorial]] | Ninguna | n! con recursión | — |
| ex14 | [[ft_sqrt]] | Ninguna | Raíz cuadrada exacta o 0 | — |

### Cadenas y memoria

| Ej. | Función | Autorizadas | Descripción | En libft |
|---|---|---|---|---|
| ex16 | `ft_strlen` | Ninguna | Longitud de una cadena | [[ft_strlen]] |
| ex17 | [[ft_strcmp]] | Ninguna | Compara dos cadenas enteras | Hermana de [[ft_strncmp]] |
| ex20 | `ft_strdup` | malloc | Duplica una cadena | [[ft_strdup]] |
| ex21 | [[ft_range]] | malloc | Array de enteros de min a max − 1 | — |

### Programas

| Ej. | Programa | Autorizadas | Descripción |
|---|---|---|---|
| ex18 | [[ft_print_params]] | ft_putchar | Escribe sus argumentos, uno por línea |
| ex19 | [[ft_sort_params]] | ft_putchar | Escribe sus argumentos ordenados por ASCII |
| ex27 | [[ft_display_file]] | open, read, write, close | Muestra un fichero, con Makefile propio |

### Headers, punteros a función y Makefile

| Ej. | Entrega | Autorizadas | Descripción | En libft |
|---|---|---|---|---|
| ex22 | [[ft_abs]] (`ft_abs.h`) | Ninguna | Macro `ABS(Value)` | — |
| ex23 | [[ft_point]] (`ft_point.h`) | Ninguna | `typedef struct` `t_point` | Mismo patrón que `t_list` |
| ex24 | `Makefile` | Ninguna | Compila `libft.a` desde `srcs/` e `includes/` | [[Makefile y librerías estáticas]] |
| ex25 | [[ft_foreach]] | Ninguna | Aplica `f` a cada entero de un array | [[ft_striteri]], [[ft_lstiter]] |
| ex26 | [[ft_count_if]] | Ninguna | Cuenta las cadenas para las que `f` devuelve 1 | — |

## Las que se repiten en libft

No tienen nota propia: el algoritmo es el mismo que en libft. Cambia el prototipo:

| Piscine Reloaded | libft | Diferencia |
|---|---|---|
| `void ft_putchar(char c);` | [[ft_putchar_fd]] | libft recibe el `fd`. `ft_putchar(c)` = `ft_putchar_fd(c, 1)` |
| `void ft_putstr(char *str);` | [[ft_putstr_fd]] | Igual: `ft_putstr(s)` = `ft_putstr_fd(s, 1)` |
| `int ft_strlen(char *str);` | [[ft_strlen]] | libft devuelve `size_t` y recibe `const char *` |
| `char *ft_strdup(char *src);` | [[ft_strdup]] | libft recibe `const char *s` |
| ex24 `Makefile` | [[Makefile y librerías estáticas]] | Aquí las fuentes están en `srcs/` y los headers en `includes/` (`-I includes`); en libft todo va en la raíz |

> [!warning] ex24: «¡Cuidado con los wildcards!»
> El subject lo avisa y la [[La Norma|Norma]] lo exige: lista las fuentes una a una (`srcs/ft_putchar.c srcs/ft_putstr.c …`), nada de `$(wildcard srcs/*.c)` ni `*.c`. La Moulinette lo prueba con sus propias fuentes de `ft_putchar`, `ft_putstr`, `ft_strcmp`, `ft_strlen` y `ft_swap`.

## Progreso

Cada nota tiene la propiedad `estado` (`hecho` / `pendiente`), igual que en libft.

## Relacionadas

[[libft - Índice]] · [[Variaciones - Índice]] · [[Piscine Reloaded - Diagramas de flujo]] · [[La Norma]] · [[Punteros en C]] · [[Recursión]] · [[Descriptores de fichero y read]] · [[Makefile y librerías estáticas]]

## Referencias

- Subject de Piscine Reloaded v2.0 (`/home/felixrod/cursus/piscine-reloaded/es.subject_piscine-reloaded.pdf`) — lista de ejercicios, funciones autorizadas e instrucciones (citas literales).
- La Norma v4 (`Cursus/42/Normativa 42/es.norm-2.pdf`) — reglas de estilo y prohibición de wildcards en el Makefile.
