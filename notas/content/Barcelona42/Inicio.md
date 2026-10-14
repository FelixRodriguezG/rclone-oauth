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

| Rank | Proyecto | Lenguaje | Notas |
|---|---|---|---|
| 0 | **libft** | C | [[libft - Índice\|Índice de funciones]] · [[libft - Anexo comparativo\|Comparativa por uso]] · [[libft - Suite de tests\|Suite de tests]] |
| 1 | **ft_printf** | C | [[ft_printf - Índice]] |
| 1 | **get_next_line** | C | [[get_next_line - Índice]] |
| 1 | **push_swap** | C | [[push_swap - Índice]] |
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
- [[Recursión]]: caso base, pila de llamadas, `ft_putnbr_fd` y flood fill.
- [[Operadores de bits]]: `&`, `|`, `^`, `~`, desplazamientos y máscaras.

### Memoria y errores

- [[Segmentos de memoria en C]]: stack, heap, BSS, DATA y TEXT, y los errores de memoria típicos.
- [[Variables static y ámbito]]: ámbito, duración, `static` en variables y funciones.
- [[Control de errores en C]]: comprobar `malloc` y llamadas al sistema, liberar al fallar, `errno` y `perror`.

### Entrada/salida y sistema

- [[Descriptores de fichero y read]]: qué es un fd, fd 0/1/2, ciclo `open` → `read`/`write` → `close`, permisos `0644` y errores.
- [[printf]]: especificadores, banderas y casos límite para ft_printf.
- [[stdarg y funciones variádicas]]: `va_list`, `va_start`, `va_arg`, `va_end` y la promoción de tipos.
- [[Preprocesador y headers]]: `#include`, guardas, `#define` y `-D BUFFER_SIZE`.

### Algoritmos

- [[Algoritmos de ordenación y complejidad]]: notación O grande, ordenaciones clásicas y radix (push_swap).

## Herramientas

- [[Cómo compila C]]: preprocesado, compilación, enlazado y librerías.
- [[Makefile y librerías estáticas]]: reglas, variables, `ar`, relink y plantillas.
- [[Depurar con gdb]]: breakpoints, paso a paso, `backtrace`, watchpoints y equivalencias con lldb.
- [[Valgrind y sanitizers]]: leaks, lecturas/escrituras inválidas, ASan y UBSan.
- [[Git para 42]]: repositorio de entrega, `.gitignore` y comprobar qué entregas.
- [[Vim]]: `.vimrc` para 42 y atajos esenciales.

## Normativa de 42

- [[Normativa 42/es.norm-2.pdf|La Norma (PDF oficial)]]
- [[Normativa 42/normativa-campus-42barcelona.pdf|Normativa del campus]]
- [[Normativa 42/normas-convivencia-42.pdf|Normas de convivencia]]
