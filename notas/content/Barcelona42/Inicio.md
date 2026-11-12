---
tags: [inicio]
---
# Barcelona42

> Apuntes del Common Core de 42 Barcelona: los proyectos hasta el rank 2, los conceptos de C que necesitas para ellos y las herramientas del día a día. Cada nota termina con sus **Referencias** (de dónde sale la información).

> [!tip] Por dónde empezar si vienes de la Piscina
> 1. [[La Norma]] y [[Makefile y librerías estáticas]] para poder entregar.
> 2. [[Punteros en C]], [[Arrays en C]] y [[Carácter nulo y puntero NULL]] para entender las funciones de cadenas.
> 3. [[libft - Índice]] y, mientras programas, [[Depurar con gdb]] y [[Valgrind y sanitizers]].

## Proyectos

Todo lo de 42 está en la carpeta `Cursus/`: un subdirectorio por rank (`Rank 0`, `Rank 1`, `Rank 2`), más `42/` (Norma, examen y PDFs de normativa) y `Variaciones/`. Los conceptos generales (`C/`, `Herramientas/`, `Bash/`, `tmux/`, `TUI/`) siguen fuera porque sirven para cualquier proyecto.

| Rank | Proyecto | Lenguaje | Notas |
|---|---|---|---|
| 0 | **Piscine Reloaded** | C y shell | [[Piscine Reloaded - Índice\|Índice de ejercicios]] · [[Piscine Reloaded - Diagramas de flujo\|Diagramas de flujo]] |
| 0 | **libft** | C | [[libft - Índice\|Índice de funciones]] · [[libft - Anexo comparativo\|Comparativa por uso]] · [[libft - Edge cases y tests por función\|Tests por función]] · [[libft - Diagramas de flujo\|Diagramas de flujo]] |
| 1 | **ft_printf** | C | [[ft_printf - Índice]] |
| 1 | **get_next_line** | C | [[get_next_line - Índice\|Índice]] · [[get_next_line - Guía paso a paso\|Guía paso a paso]] · [[get_next_line - Funciones auxiliares\|Funciones auxiliares]] · [[get_next_line - Pruebas y depuración\|Pruebas]] |
| 1 | **push_swap** | C | [[push_swap - Índice\|Índice]] · [[push_swap - Índice de desorden\|Desorden]] · [[push_swap - Estrategias\|Estrategias]] · [[push_swap - Catálogo de algoritmos\|Algoritmos]] · [[push_swap - Guía paso a paso\|Guía]] · [[push_swap - Pruebas y depuración\|Pruebas]] · [[push_swap - checker (bonus)\|Checker]] · [[push_swap - Defensa y evaluación\|Defensa]] |
| 2 | **Born2beroot** | Sistemas (Debian/Rocky) | [[Born2beroot - Índice]] |
| 2 | **Python Modules 00–10** | Python | [[Python - Índice]] |
| 2 | **A-Maze-ing** | Python | [[A-Maze-ing - Índice]] |
| 2 | **Exam Rank 02** | C | [[Exam Rank 02]] |

> [!info] Cursus nuevo (2025)
> El rank 3 es Call Me Maybe, Codexion, Fly-in y Exam Rank 03.

## 42

- [[La Norma]]: resumen completo de la Norma v4 (límites, formato, nombres, headers, prohibiciones, Makefile).
- [[Exam Rank 02]]: cómo funciona el examen, estrategia y tipos de ejercicio.

## C

### Fundamentos

- [[Punteros en C]]: direcciones, `&` y `*`, aritmética, `char **`, `void *` y punteros a función.
- [[Arrays en C]]: declaración, índices, `sizeof`, arrays y punteros, cadenas.
- [[Carácter nulo y puntero NULL]]: `'\0'` vs `NULL`, en qué casilla va cada uno y qué dice el compilador si los mezclas.
- [[Tipos enteros y conversiones]]: `signed`/`unsigned`, `size_t`, límites, promociones y overflow.
- [[Structs y typedef]]: `struct`, `typedef`, `.` vs `->` y `t_list`.
- [[Listas enlazadas]]: nodos, recorrer, añadir y liberar sin leaks.
- [[Listas doblemente enlazadas]]: `prev`/`next`, listas circulares y rotar en O(1).
- [[Pilas y colas]]: LIFO y FIFO, y por qué las pilas de push_swap son anillos.
- [[Array circular]]: rotar moviendo un índice con `%`.
- [[Recursión]]: caso base, pila de llamadas, `ft_putnbr_fd` y flood fill.
- [[Operadores de bits]]: `&`, `|`, `^`, `~`, desplazamientos y máscaras.
- [[argc y argv]]: argumentos de `main`, flags y lo que hace la shell antes (bash frente a zsh).
- [[enum en C]]: constantes con nombre, `OP_COUNT` y arrays indexados por `enum`.
- [[División entera y módulo]]: truncar, signo del resto, dar la vuelta con `%` y redondear hacia arriba.
- [[Decimales con enteros]]: porcentajes con dos decimales sin `float` (punto fijo).
- [[Tablas de punteros a función]]: del texto `"rra"` a la función, sin cadenas de `if`.

### Memoria y errores

- [[malloc y free]]: reservar y liberar, quién es dueño de cada bloque, el patrón `p = f(p)` y los errores típicos.
- [[Segmentos de memoria en C]]: stack, heap, BSS, DATA y TEXT, y los errores de memoria típicos.
- [[Padding y alineación de memoria]]: por qué `sizeof` de un struct no es la suma de sus campos, structs empaquetados y por qué se usa `unsigned char *`.
- [[Variables static y ámbito]]: ámbito, duración, `static` en variables y funciones.
- [[Control de errores en C]]: comprobar `malloc` y llamadas al sistema, liberar al fallar, `errno` y `perror`.

### Entrada/salida y sistema

- [[Descriptores de fichero y read]]: qué es un fd, fd 0/1/2, ciclo `open` → `read`/`write` → `close`, permisos `0644` y errores.
- [[printf]]: especificadores, banderas y casos límite para ft_printf.
- [[stdarg y funciones variádicas]]: `va_list`, `va_start`, `va_arg`, `va_end` y la promoción de tipos.
- [[Preprocesador y headers]]: `#include`, guardas, `#define` y `-D BUFFER_SIZE`.

### Variaciones

- [[Variaciones - Índice]]: funciones de la C Piscine y del Exam Rank 02 que no están en libft (`ft_putnbr_base`, `ft_atoi_base`, `ft_strcpy`, `ft_power`, `print_bits`…), cada una con su diagrama de flujo.

### Algoritmos

- [[Notación Big-O]]: O, Ω y Θ, O(n√n), contar operaciones en vez de tiempo y la cota mínima de n log n.
- [[Algoritmos de ordenación y complejidad]]: ordenaciones clásicas medidas y radix (push_swap).

## Herramientas

- [[Cómo compila C]]: preprocesado, compilación, enlazado y librerías.
- [[Makefile y librerías estáticas]]: reglas, variables, `ar`, relink y plantillas.
- [[Depurar con gdb]]: breakpoints, paso a paso, `backtrace`, watchpoints y equivalencias con lldb.
- [[Valgrind y sanitizers]]: leaks, lecturas/escrituras inválidas, ASan y UBSan.
- [[Git para 42]]: repositorio de entrega, `.gitignore` y comprobar qué entregas.
- [[Vim]]: `.vimrc` para 42 y atajos esenciales.
- [[GitHub CLI (gh)]]: iniciar sesión, crear y publicar repositorios desde la terminal.
- [[README de un proyecto]]: qué secciones poner y cómo añadir capturas o un GIF.
- [[Grabar una demo de terminal]]: asciinema + agg para convertir una sesión en GIF.

## Bash

> [!tip] Mapa visual
> [[Mapa - De script a TUI.canvas|Mapa: de script a TUI]] muestra todas las notas de Bash, tmux y TUI en un lienzo, con el orden de lectura.

De tu primer script a uno organizado, con subcomandos, configuración y control de errores. Empieza por [[Bash - Índice]].

- **Básicos:** [[Primer script en Bash]] · [[Variables y comillas en Bash]] · [[Condiciones en Bash]] · [[Bucles en Bash]] · [[Funciones en Bash]]
- **Sintaxis a fondo:** [[Expansiones en Bash]] · [[Arrays en Bash]] · [[Leer entrada del usuario en Bash]] · [[Subshells y procesos en Bash]] · [[Scripts robustos con set y trap]]
- **Organizar un script:** [[Argumentos y subcomandos en Bash]] · [[Códigos de salida y errores en Bash]] · [[Redirecciones y tuberías]] · [[grep, sed y awk lo justo]] · [[Colores en la terminal]] · [[Configuración y estado de un script]] · [[Depurar scripts de Bash]]
- **Consulta rápida:** [[Bash - Chuleta de sintaxis]]

## tmux

Varias terminales en una y sesiones que siguen vivas al cerrar la ventana. Empieza por [[tmux - Índice]].

- [[Qué es tmux]] · [[tmux - Uso básico]] · [[Configurar tmux]] · [[Automatizar tmux desde scripts]]

## TUI

Escalar un script a una interfaz de terminal moderna, con Python y curses. Empieza por [[TUI - Índice]].

- **Fundamentos:** [[CLI, TUI y GUI]] · [[Cómo funciona la terminal]] · [[De script a TUI - arquitectura]]
- **curses en Python:** [[curses - Primeros pasos]] · [[curses - El bucle principal]] · [[curses - Layout y paneles]] · [[curses - Colores y estilos]] · [[curses - Teclado y atajos]] · [[curses - Ejecutar comandos]] · [[Unicode y emojis en la terminal]] · [[Diseño de una TUI moderna]]
- **Frameworks:** [[Frameworks de TUI - Listado]] · [[Elegir un framework de TUI]] · [[TUIs en Bash - dialog, whiptail y gum]]
- **Animaciones:** [[Animación en la terminal - fundamentos]] · [[Spinners y barras de progreso]] · [[curses - Animaciones]]
- **Instaladores:** [[Qué hace un buen instalador]] · [[Instalador TUI en Bash]] · [[Instalador TUI en Python]] · [[Instalar con una línea]]

## Normativa de 42

- [[es.norm-2.pdf|La Norma (PDF oficial)]]
- [[normativa-campus-42barcelona.pdf|Normativa del campus]]
- [[normas-convivencia-42.pdf|Normas de convivencia]]
