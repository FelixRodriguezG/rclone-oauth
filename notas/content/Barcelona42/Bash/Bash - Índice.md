---
tags: [bash, indice, shell, scripting]
aliases: ["Bash", "Aprender Bash", "Shell scripting"]
---
# Bash - Índice

[[Inicio|← Inicio]] · Bash

> Ruta de aprendizaje para escribir **scripts en Bash**: desde el primer `echo` hasta un programa con subcomandos, colores, configuración y control de errores. Al terminar, el siguiente paso es organizar el trabajo con tmux y convertir el script en una **TUI** (interfaz de texto a pantalla completa).

## Cómo usar esta ruta

Lee las notas **en orden**: cada una usa lo que explicó la anterior. Todas son cortas (un tema por nota) y tienen ejemplos que puedes copiar y ejecutar.

A lo largo de las notas crece un mini-proyecto: **`tareas.sh`**, una lista de tareas por terminal. Guarda las tareas en `~/.tareas.txt`, una por línea (la variable de entorno `TAREAS_ARCHIVO` cambia la ruta), y se usa así:

```console
$ ./tareas.sh añadir Comprar pan
$ ./tareas.sh lista
 1. Comprar pan
$ ./tareas.sh borrar 1
$ ./tareas.sh help
```

| Nota | Qué aprende `tareas.sh` |
|---|---|
| [[Primer script en Bash]] | Existe: imprime un saludo |
| [[Variables y comillas en Bash]] | Sabe dónde está su archivo de datos |
| [[Condiciones en Bash]] | Comprueba si el archivo existe |
| [[Bucles en Bash]] | Lista las tareas numeradas |
| [[Funciones en Bash]] | Funciones `listar`, `anadir` y `borrar`, y un `case` con los subcomandos |
| [[Arrays en Bash]] | Carga todas las tareas en memoria |
| [[Leer entrada del usuario en Bash]] | Menú interactivo y confirmación s/n |
| [[Argumentos y subcomandos en Bash]] | Versión de referencia: opciones `-v` y `-f`, `help` y errores de uso |
| [[Scripts robustos con set y trap]] | Modo estricto: `set -euo pipefail` |
| [[Configuración y estado de un script]] | Configuración opcional en `config.sh` |
| [[Colores en la terminal]] | Mensajes en color, solo si la salida es una terminal |

Más adelante se convierte en una TUI en Python, `tareas.py` + `modelo.py`, que llama a `tareas.sh` (ver [[De script a TUI - arquitectura]]).

> [!tip] Practica escribiendo
> No copies y pegues todo: escribe los ejemplos a mano en [[Vim]], ejecútalos, rómpelos a propósito y mira qué error sale. Guarda tu `tareas.sh` en un repo (ver [[Git para 42]]) para ver cómo evoluciona.

## 1. Básicos

Lo mínimo para escribir un script que funcione.

1. [[Primer script en Bash]] — qué es un script, *shebang*, `chmod +x`, cómo ejecutarlo.
2. [[Variables y comillas en Bash]] — guardar datos, `"$var"`, comillas simples y dobles, `$( )`, `export`.
3. [[Condiciones en Bash]] — `if`, `[[ ]]`, comprobar archivos, cadenas y números, `case`.
4. [[Bucles en Bash]] — `for`, `while`, leer un archivo línea a línea.
5. [[Funciones en Bash]] — agrupar código, argumentos, `local`, devolver datos.
6. [[Redirecciones y tuberías]] — `>`, `>>`, `2>`, `|`, *here-docs*.
7. [[Códigos de salida y errores en Bash]] — `$?`, `exit`, mensajes por `stderr`.

## 2. Sintaxis a fondo

Lo que hace que un script sea corto y correcto.

8. [[Expansiones en Bash]] — recortar y sustituir texto en variables, aritmética, globs.
9. [[Arrays en Bash]] — listas y diccionarios (`declare -A`), `mapfile`.
10. [[Leer entrada del usuario en Bash]] — `read`, menús con `select`, confirmar s/n.
11. [[Subshells y procesos en Bash]] — `( )` frente a `{ }`, segundo plano, `wait`, `<( )`.
12. [[grep, sed y awk lo justo]] — filtrar y transformar texto sin volverse loco.
13. [[Bash - Chuleta de sintaxis]] — todo lo anterior en una página para consultar.

## 3. Organizar un script

Cuando el script deja de ser un juguete.

14. [[Argumentos y subcomandos en Bash]] — opciones, `getopts`, `--help`, subcomandos.
15. [[Scripts robustos con set y trap]] — `set -euo pipefail`, `trap`, limpiar al salir.
16. [[Configuración y estado de un script]] — archivos de configuración, dónde guardar datos.
17. [[Colores en la terminal]] — secuencias ANSI, `tput`, respetar `NO_COLOR`.
18. [[Depurar scripts de Bash]] — `bash -x`, `shellcheck`, `PS4`.

## Siguiente paso

Con un script organizado ya puedes:

- Trabajar con varias terminales a la vez y automatizar sesiones → [[tmux - Índice]].
- Convertir `tareas.sh` en una interfaz de texto moderna → [[TUI - Índice]].

## Relacionadas

[[Primer script en Bash]] · [[Bash - Chuleta de sintaxis]] · [[De script a TUI - arquitectura]] · [[Python - Índice]] · [[Git para 42]] · [[Vim]] · [[Descriptores de fichero y read]]

## Referencias

- [GNU Bash Reference Manual](https://www.gnu.org/software/bash/manual/bash.html) — manual oficial; estructura de los temas.
- `man bash` — referencia completa de la versión instalada (Bash 5.1 en Ubuntu 22.04).
- [BashGuide (Greg's Wiki)](https://mywiki.wooledge.org/BashGuide) — orden didáctico y buenas prácticas.
- [ShellCheck](https://www.shellcheck.net/) — analizador que detecta errores típicos en scripts.
