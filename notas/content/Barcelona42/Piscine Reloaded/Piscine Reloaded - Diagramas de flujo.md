---
tags: [piscine-reloaded, diagramas]
---
# Piscine Reloaded - Diagramas de flujo

[[Piscine Reloaded - Índice|← Índice]] · Diagramas

> Todos los diagramas de flujo de las funciones de Piscine Reloaded en una sola página, en el orden del subject. Cada bloque está **incrustado** desde la sección «Diagrama de flujo» de su nota: si corriges un diagrama en la nota, aquí se actualiza solo. Las funciones que se repiten en libft se incrustan desde [[libft - Diagramas de flujo]].

> [!info] Cómo leer los diagramas
> | Forma | Significa |
> |---|---|
> | Óvalo `([ ])` | Inicio (nombre y parámetros) o fin / valor devuelto |
> | Rectángulo `[ ]` | Una instrucción o cálculo |
> | Rombo `{ }` | Una pregunta: se sigue la flecha `sí` o `no` |
> | Paralelogramo `[/ /]` | Escribir o leer (`write`, `read`, `ft_putchar`) |
> | `↻ recursión` | La función se llama a sí misma |
>
> Una flecha que vuelve hacia arriba es un **bucle**.

## Salida

### ft_print_alphabet

![[ft_print_alphabet#Diagrama de flujo]]

### ft_print_numbers

![[ft_print_numbers#Diagrama de flujo]]

### ft_is_negative

![[ft_is_negative#Diagrama de flujo]]

### ft_putstr

Igual que [[ft_putstr_fd]] con `fd = 1`.

![[libft - Diagramas de flujo#ft_putstr_fd]]

## Punteros

### ft_ft

![[ft_ft#Diagrama de flujo]]

### ft_swap

![[ft_swap#Diagrama de flujo]]

### ft_div_mod

![[ft_div_mod#Diagrama de flujo]]

## Matemáticas

### ft_iterative_factorial

![[ft_iterative_factorial#Diagrama de flujo]]

### ft_recursive_factorial

![[ft_recursive_factorial#Diagrama de flujo]]

### ft_sqrt

![[ft_sqrt#Diagrama de flujo]]

## Cadenas y memoria

### ft_strlen

Igual que [[ft_strlen]] de libft (allí devuelve `size_t`).

![[libft - Diagramas de flujo#ft_strlen]]

### ft_strcmp

![[ft_strcmp#Diagrama de flujo]]

### ft_strdup

Igual que [[ft_strdup]] de libft.

![[libft - Diagramas de flujo#ft_strdup]]

### ft_range

![[ft_range#Diagrama de flujo]]

## Programas

### ft_print_params

![[ft_print_params#Diagrama de flujo]]

### ft_sort_params

![[ft_sort_params#Diagrama de flujo]]

### ft_display_file

![[ft_display_file#Diagrama de flujo]]

## Headers y punteros a función

### ft_abs

![[ft_abs#Diagrama de flujo]]

### ft_point

![[ft_point#Diagrama de flujo]]

### ft_foreach

![[ft_foreach#Diagrama de flujo]]

### ft_count_if

![[ft_count_if#Diagrama de flujo]]

## Relacionadas

[[Piscine Reloaded - Índice]] · [[libft - Diagramas de flujo]] · [[Variaciones - Índice]]

## Referencias

- [Flowcharts — documentación de Mermaid](https://mermaid.js.org/syntax/flowchart.html) — sintaxis de los diagramas.
- [Obsidian: diagramas Mermaid](https://help.obsidian.md/Editing+and+formatting/Advanced+formatting+syntax#Diagram) e [incrustar secciones de otra nota](https://help.obsidian.md/Linking+notes+and+files/Embed+files#Embed+a+note+in+another+note) — por qué esta página se actualiza sola.
