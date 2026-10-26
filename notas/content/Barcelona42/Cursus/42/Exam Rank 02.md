---
tags: [42, examen, exam-rank-02, rank-2, estrategia, c]
aliases: ["Examen Rank 02", "Exam 02", "examshell"]
---
# Exam Rank 02

[[Inicio|← Inicio]] · 42 · Rank 2

> El **Exam Rank 02** es el primer examen del Common Core. Se hace en **C**. Durante unas horas, en un ordenador del clúster **sin internet**, te van tocando ejercicios cortos de C de dificultad creciente que corrige una máquina. No hay ayudas: lo que no sepas escribir de memoria, no existe. Esta nota explica cómo funciona, qué tipo de ejercicios salen y cómo prepararlo **sin** darte las soluciones.

> [!warning] Comprueba el subject
> Resumen basado en versiones públicas del examen y en la experiencia común de estudiantes. El formato (número de niveles, ejercicios, tiempo, nota mínima, si se usa `examshell` u otra herramienta) **cambia entre campus y entre años**. Lo que manda es lo que diga tu intra y el personal del campus el día del examen.

> [!info] Dónde está en el nuevo Common Core (2025)
> En el holy graph actual el examen está en el **rank 2**, junto a [[Born2beroot - Índice|Born2beroot]], los [[Python - Índice|módulos de Python]] y [[A-Maze-ing - Índice|A-Maze-ing]]. Se desbloquea al terminar el rank 1 ([[libft - Índice|libft]] en el rank 0; [[ft_printf - Índice|ft_printf]], [[get_next_line - Índice|get_next_line]] y [[push_swap - Índice|push_swap]] en el rank 1), así que llega cuando ya tienes mucho C encima.
>
> Aunque en el rank 2 empiezas Python, **el Exam Rank 02 sigue siendo de C**: los simuladores públicos lo describen como un examen de ejercicios de C, y el examen en Python es el **Exam Rank 03**. Si tu campus usa una versión distinta, manda lo que diga la intra.

## Vocabulario

| Término | Qué significa |
|---|---|
| **examshell** | Programa del examen: te identifica, te da el ejercicio y manda a corregir |
| **subject** | Enunciado del ejercicio (en `~/subjects/…`) |
| **rendu** | Carpeta donde dejas tu solución (en francés, «entregado») |
| **grademe** | Orden de examshell que manda tu entrega a corregir |
| **Moulinette** | El corrector automático: compila y compara la salida de tu programa con la esperada |
| **Nivel** | Grupo de ejercicios de dificultad parecida. Te toca uno al azar de cada nivel |

## Cómo funciona un examen

Lo habitual es esto:

1. Te sientas en el sitio asignado y entras en la sesión especial de examen (el staff te dice el usuario).
2. Abres una terminal y ejecutas `examshell`. Te identificas con tu login del intra.
3. Examshell te asigna un ejercicio del **nivel 1** y crea el enunciado en `~/subjects/<ejercicio>/`.
4. Escribes la solución en `~/rendu/<ejercicio>/<fichero>.c`, que es un **repositorio Git**: haces `git add`, `git commit` y `git push` (ver [[Git para 42]]).
5. Vuelves a examshell y escribes `grademe`. Tras unos segundos o minutos te dice si está bien (**success**) o mal (**failure**).
6. Si aciertas, pasas al nivel siguiente con un ejercicio nuevo. Si fallas, te toca **otro** ejercicio del mismo nivel y suele haber una penalización de tiempo antes de poder volver a corregir.

```text
~/subjects/first_word/subject.en.txt   ← lees aquí
~/rendu/first_word/first_word.c        ← escribes aquí, y git push
```

> [!danger] Lo que no puedes llevar ni usar
> Ni internet, ni apuntes, ni móvil, ni tu libft. Solo las funciones listadas en «Allowed functions» de cada enunciado. Usar una no permitida es un **failure** directo aunque el resultado sea correcto.

> [!info] Se repite
> Si no apruebas, puedes volver a presentarte en la siguiente convocatoria. Los exámenes se repiten hasta aprobarlos, así que el primero sirve como entrenamiento.

## Cómo es un enunciado

Todos siguen el mismo formato. Ejemplo público del nivel 1 (ejemplos recortados):

```text
Assignment name  : first_word
Expected files   : first_word.c
Allowed functions: write
--------------------------------------------------------------------------------

Write a program that takes a string and displays its first word, followed by a
newline.

A word is a section of string delimited by spaces/tabs or by the start/end of
the string.

If the number of parameters is not 1, or if there are no words, simply display
a newline.

Examples:

$> ./first_word "FOR PONY" | cat -e
FOR$
$> ./first_word "   " | cat -e
$
$> ./first_word "a" "b" | cat -e
$
```

Lee las tres líneas de arriba con lupa:

- **Assignment name** y **Expected files**: el nombre exacto del fichero y de la carpeta. Una mayúscula distinta y no se corrige.
- **Allowed functions**: la lista completa de lo que puedes llamar.
- **«Write a program»** (con `main`, lees `argv`) o **«Write a function»** (sin `main`: si entregas un `main`, falla la compilación del corrector).

El `| cat -e` de los ejemplos marca el final de línea con `$`: así ves si sobra o falta un `\n` o un espacio.

## Tipos de ejercicio por nivel

En los repositorios públicos de práctica el examen tiene cuatro niveles. Estos son los ejercicios que aparecen en esas listas, agrupados por **lo que entrenan**:

| Nivel | Ejemplos | Qué entrenan |
|---|---|---|
| 1 | `first_word`, `fizzbuzz`, `ft_putstr`, `ft_strcpy`, `ft_strlen`, `ft_swap`, `repeat_alpha`, `rev_print`, `rot_13`, `rotone`, `search_and_replace`, `ulstr` | Recorrer una cadena, `write` carácter a carácter, `argc`/`argv`, aritmética con letras (`'a' + 13`) |
| 2 | `alpha_mirror`, `camel_to_snake`, `do_op`, `ft_atoi`, `ft_strcmp`, `ft_strcspn`, `ft_strdup`, `ft_strpbrk`, `ft_strrev`, `ft_strspn`, `inter`, `union`, `is_power_of_2`, `last_word`, `max`, `print_bits`, `reverse_bits`, `swap_bits`, `snake_to_camel`, `wdmatch` | Reimplementar funciones de `libc`, `malloc` sencillo, [[Operadores de bits]], tablas de «ya visto» (256 casillas) |
| 3 | `add_prime_sum`, `epur_str`, `expand_str`, `ft_atoi_base`, `ft_list_size`, `ft_range`, `ft_rrange`, `hidenp`, `lcm`, `pgcd`, `paramsum`, `print_hex`, `str_capitalizer`, `rstr_capitalizer`, `tab_mult` | Números (primos, MCD, MCM, bases), imprimir enteros con `write`, espacios y palabras, primeras [[Listas enlazadas]] |
| 4 | `flood_fill`, `fprime`, `ft_itoa`, `ft_list_foreach`, `ft_list_remove_if`, `ft_split`, `rev_wstr`, `rostring`, `sort_int_tab`, `sort_list` | `malloc` de cadenas y arrays de cadenas, punteros a función, [[Recursión]], [[Algoritmos de ordenación y complejidad\|ordenar]] arrays y listas |

> [!note] Por qué no hay soluciones aquí
> El examen mide si sabes escribirlo **tú**, sin mirar. Memorizar soluciones ajenas suele fallar en el primer caso borde. Usa las listas para practicar: lee el enunciado, resuélvelo en papel o en un fichero vacío, y solo después compara.

## Qué conviene saber escribir de memoria

Muchas piezas del examen son funciones de tu [[libft - Índice|libft]] o variantes. Si las has entendido de verdad, el nivel 2–4 se reduce a combinarlas:

| Función | Dónde reaparece |
|---|---|
| [[ft_strlen]] | En casi todo |
| `ft_putchar` / `ft_putstr` (versión simple de [[ft_putchar_fd]] y [[ft_putstr_fd]]) | Todos los «Write a program» |
| [[ft_putnbr_fd]] (escribir un `int` con `write`, incluido `INT_MIN`) | `do_op`, `paramsum`, `add_prime_sum`, `tab_mult`, `print_hex` (en base 16) |
| [[ft_atoi]] y su versión con base | `ft_atoi`, `ft_atoi_base`, `do_op`, `add_prime_sum` |
| [[ft_strdup]] | `ft_strdup` tal cual |
| [[ft_strncmp]] (versión sin `n`: `strcmp`) | `ft_strcmp` |
| [[ft_itoa]] | `ft_itoa` tal cual |
| [[ft_split]] | `ft_split`, `rev_wstr`, `rostring` (o al menos el patrón «saltar espacios / medir palabra») |
| [[ft_lstsize]], [[ft_lstiter]] | `ft_list_size`, `ft_list_foreach` |
| Un ordenamiento simple (burbuja) | `sort_int_tab`, `sort_list` |

> [!tip] Los patrones más repetidos
> - **Saltar espacios y tabuladores**: `while (s[i] == ' ' || s[i] == '\t') i++;`
> - **Comprobar `argc` primero**: casi todos los programas piden imprimir solo `\n` si el número de argumentos no es el esperado.
> - **Tabla de 256 posiciones** para «¿ya he visto este carácter?» (`inter`, `union`).
> - **Bits**: `(n >> i) & 1` para leer el bit `i` (`print_bits`, `reverse_bits`).

## Estrategia el día del examen

1. **Lee el enunciado entero dos veces**, ejemplos incluidos. Apunta los casos raros: sin argumentos, cadena vacía, negativos, `INT_MIN`.
2. **Escribe el `main` de prueba aparte** (en otro fichero, fuera de `rendu`) si el ejercicio es una función. No lo entregues.
3. **Compila como el corrector**: `cc -Wall -Wextra -Werror first_word.c`. Un aviso = failure.
4. **Prueba con `| cat -e`** los ejemplos del enunciado y tus casos raros. Compara carácter a carácter.
5. **Revisa qué entregas**: `git status` en la carpeta de `rendu`, nombre del fichero correcto, sin `a.out` ni `main` sobrante.
6. **`grademe` solo cuando estés seguro**. Cada failure te cambia el ejercicio y puede costarte tiempo de espera.
7. Si te atascas, **para y haz trazas en papel** con un ejemplo pequeño. No reescribas al azar.

> [!warning] Errores que suspenden
> - Olvidar el `\n` final o imprimir uno de más.
> - Usar `printf` cuando solo se permite `write`.
> - Fichero con nombre distinto (`firstword.c`) o en otra carpeta.
> - No hacer `git push` antes de `grademe`.
> - En funciones: entregar un `main`, o no reservar el `+ 1` del `'\0'` en un `malloc`.
> - Bucle infinito o segfault con la cadena vacía `""`.

> [!info] ¿Y la Norma?
> Habitualmente el corrector del examen no pasa la norminette, solo compila y compara salidas. Aun así, escribir con el estilo de [[La Norma]] te ayuda a no cometer errores y es la costumbre que ya tienes.

## Cómo prepararlo

- **Repasa tu libft sin mirar**: borra una función y reescríbela en 10 minutos. Si no puedes, todavía no la dominas.
- **Practica con enunciados públicos** por niveles y cronométrate.
- **Usa el entorno real**: terminal, [[Vim]] y `cc`. En el examen no hay VS Code.
- **Aprende a depurar sin depurador**: un `write` de control y `cat -e`. Puede no haber [[Depurar con gdb|gdb]] a mano, y nunca hay tiempo para usarlo a fondo.
- **Repasa los casos borde de [[Tipos enteros y conversiones]]**: `INT_MIN`, desbordamiento en `atoi`, `unsigned char` en los bits.

> [!summary] Regla de oro
> Enunciado → casos raros → código → compilar con banderas → probar con `cat -e` → `git push` → `grademe`. Siempre en ese orden.

## Relacionadas

[[libft - Índice]] · [[Git para 42]] · [[Vim]] · [[La Norma]] · [[Operadores de bits]] · [[Listas enlazadas]] · [[Recursión]] · [[Algoritmos de ordenación y complejidad]] · [[Cómo compila C]]

## Referencias

- [alexhiguera/Exam_Rank_02_42_School](https://github.com/alexhiguera/Exam_Rank_02_42_School) — lista de los 56 ejercicios por nivel (12/19/15/10) y formato de los enunciados (ejemplo `first_word`), con explicaciones en español.
- [48d31kh413k/1337-exam_rank_02-42](https://github.com/48d31kh413k/1337-exam_rank_02-42) — segunda recopilación pública de enunciados por nivel, usada para contrastar la lista.
- [Glagan/42-exam-rank-02](https://github.com/Glagan/42-exam-rank-02) — otra recopilación pública de enunciados.
- [Pro Git (es) — Guardando cambios en el Repositorio](https://git-scm.com/book/es/v2/Fundamentos-de-Git-Guardando-cambios-en-el-Repositorio) — `add`, `commit` y `status` para entregar en `rendu`.
- [42exam.net](https://42exam.net/) — simulador no oficial: describe el Rank 02 como ejercicios de C y el Rank 03 como ejercicios de Python.
- [martinnsanzz/42-Curriculum](https://github.com/martinnsanzz/42-Curriculum) — orden de proyectos del nuevo Common Core por ranks (milestones).
- Funcionamiento de `examshell`, `rendu` y `grademe`: experiencia común de estudiantes de 42; no hay documentación oficial pública. Confírmalo en tu campus.
