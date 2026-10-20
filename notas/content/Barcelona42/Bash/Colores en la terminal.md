---
tags: [bash, colores, ansi, tput, terminal]
aliases: ["Códigos ANSI", "Secuencias de escape ANSI", "NO_COLOR", "tput"]
---
# Colores en la terminal

[[Inicio|← Inicio]] · [[Bash - Índice|Bash]]

> La terminal pinta colores cuando recibe unas **secuencias de escape ANSI**: caracteres especiales que no se muestran, sino que cambian el estilo del texto que viene detrás. Bien usados (errores en rojo, éxito en verde) hacen un script mucho más legible; mal usados, llenan los archivos de basura como `^[[31m`.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Secuencia de escape** | Texto que empieza por el carácter `ESC` (código 27, `\033` en octal, `\e`) y que la terminal interpreta como una orden |
| **SGR** | *Select Graphic Rendition*: las secuencias `ESC[…m` que cambian color y estilo |
| **Reset** | `ESC[0m`: vuelve al estilo normal. Sin él, el color «se escapa» al resto de la salida |
| **TTY** | Terminal real. Si la salida va a un archivo o a una tubería, **no** es una TTY |
| **terminfo** | Base de datos que describe qué sabe hacer cada tipo de terminal (`$TERM`); la usa `tput` |

Más detalles sobre cómo la terminal procesa esto en [[Cómo funciona la terminal]].

## La forma de una secuencia

```text
ESC [ <código> ; <código> ... m
\033[1;31m     → negrita + rojo
\033[0m        → reset
```

| Código | Efecto | Código | Efecto |
|---|---|---|---|
| `0` | Reset | `30`–`37` | Color de texto |
| `1` | Negrita | `40`–`47` | Color de fondo |
| `2` | Tenue | `90`–`97` | Texto en versión brillante |
| `3` | Cursiva | `38;5;N` | Texto, color N de la paleta de 256 |
| `4` | Subrayado | `48;5;N` | Fondo, color N de 256 |
| `7` | Invertido | `38;2;R;G;B` | Texto en color real (24 bits), si la terminal lo soporta |

Los 8 colores básicos, en orden del 0 al 7: **negro, rojo, verde, amarillo, azul, magenta, cian, blanco**. Así, `31` = texto rojo, `42` = fondo verde.

```bash
printf '\033[1;31mERROR\033[0m algo ha fallado\n'
printf '\033[38;5;208mnaranja (256 colores)\033[0m\n'
```

> [!tip] Ver los 256 colores
> `for i in {0..255}; do printf '\033[38;5;%dm%3d ' "$i" "$i"; done; printf '\033[0m\n'`

## `printf` mejor que `echo -e`

`echo` **no** interpreta `\033` por defecto; `echo -e` sí en Bash, pero no en otras shells (`sh` imprime `-e`). `printf` funciona igual en todas partes:

```bash
echo "\033[1mN"          # imprime literalmente \033[1mN
echo -e "\033[1mN"       # negrita en Bash
printf '\033[1mN\n'      # negrita en cualquier shell (recomendado)
```

Lo más limpio es guardar el carácter `ESC` real en una variable con `$'...'` (*ANSI-C quoting*, ver [[Variables y comillas en Bash]]). Así sirven hasta en `echo` normal:

```bash
ROJO=$'\033[31m'
RESET=$'\033[0m'
echo "${ROJO}rojo${RESET} normal"
```

## Colores solo cuando tiene sentido

Si alguien hace `./tareas.sh lista > copia.txt`, el archivo no debe llenarse de códigos. Dos reglas:

1. **Solo si stdout es una terminal**: `[[ -t 1 ]]` es verdadero si el fd 1 es una TTY (ver [[Redirecciones y tuberías]]).
2. **Respetar `NO_COLOR`**: si el usuario define esa variable (con cualquier valor), no uses colores. Es un estándar informal que siguen muchos programas.

## Patrón: variables de color al inicio

```bash
#!/usr/bin/env bash
set -euo pipefail

if [[ -t 1 && -z "${NO_COLOR:-}" ]]; then
    ROJO=$'\033[31m'
    VERDE=$'\033[32m'
    AMARILLO=$'\033[33m'
    NEGRITA=$'\033[1m'
    RESET=$'\033[0m'
else
    ROJO="" VERDE="" AMARILLO="" NEGRITA="" RESET=""
fi

ok()    { printf '%s✔%s %s\n' "$VERDE" "$RESET" "$*"; }
aviso() { printf '%s!%s %s\n' "$AMARILLO" "$RESET" "$*" >&2; }
fallo() { printf '%s✘ %s%s\n' "$ROJO$NEGRITA" "$*" "$RESET" >&2; }

ok "Tarea añadida"
aviso "El archivo está vacío"
fallo "No se pudo guardar"
```

Si los colores están desactivados, las variables valen `""` y el resto del código no cambia. Con `NO_COLOR=1 ./script.sh` o `./script.sh | cat` sale sin códigos.

> [!note] stdout y stderr pueden ir a sitios distintos
> `aviso` y `fallo` escriben en stderr. Para ser exactos, comprueba `[[ -t 2 ]]` para ellos; en scripts sencillos basta con `-t 1`.

En `tareas.sh`, usa `ok` tras `añadir` y `borrar` y `fallo` en `error_uso` (ver [[Argumentos y subcomandos en Bash]]). La salida de `lista` (` 1. Texto`) déjala **sin color**: la TUI `tareas.py` la lee y la parsea (ver [[De script a TUI - arquitectura]]). Los símbolos ✔ y ✘ se explican en [[Unicode y emojis en la terminal]].

## `tput`: la alternativa con terminfo

`tput` pregunta a terminfo la secuencia correcta para **tu** terminal, en vez de escribirla a mano:

| Orden | Efecto |
|---|---|
| `tput setaf N` / `tput setab N` | Color de texto / fondo N (0–7, o hasta 255) |
| `tput bold` | Negrita |
| `tput sgr0` | Reset |
| `tput colors` | Cuántos colores admite la terminal (8, 256…) |
| `tput cols` / `tput lines` | Tamaño de la terminal |
| `tput civis` / `tput cnorm` | Ocultar / mostrar el cursor |

```bash
if [[ -t 1 ]] && (( $(tput colors 2>/dev/null || echo 0) >= 8 )); then
    ROJO="$(tput setaf 1)"
    RESET="$(tput sgr0)"
else
    ROJO="" RESET=""
fi
echo "${ROJO}rojo${RESET}"
```

Ventaja: se adapta a terminales raras. Inconveniente: lanza un proceso por cada llamada (hazlo una vez al principio, no en un bucle). En la práctica, los códigos ANSI básicos funcionan en cualquier terminal moderna.

## Errores típicos

| Error | Qué pasa |
|---|---|
| Olvidar el reset | Toda la salida siguiente (y hasta tu prompt) queda de color |
| `echo "\033[31m"` sin `-e` | Se imprime el texto `\033[31m` |
| Colores siempre activos | Los archivos y `grep` reciben `^[[31m` |
| Solo color para dar información | Quien no distingue colores (o usa `NO_COLOR`) la pierde: añade texto o símbolos |
| Rojo/verde sobre fondo de color | Ilegible en algunos temas; usa los colores con moderación |

## Relacionadas

[[Depurar scripts de Bash]] · [[Cómo funciona la terminal]] · [[Redirecciones y tuberías]] · [[Variables y comillas en Bash]] · [[Unicode y emojis en la terminal]] · [[curses - Colores y estilos]] · [[Diseño de una TUI moderna]] · [[Bash - Chuleta de sintaxis]]

## Referencias

- [ANSI escape code — Wikipedia](https://en.wikipedia.org/wiki/ANSI_escape_code) — tabla de códigos SGR, 8/16/256 colores y 24 bits.
- [NO_COLOR](https://no-color.org/) — la convención de la variable `NO_COLOR`.
- `man tput`, `man 5 terminfo` — `setaf`, `sgr0`, `colors`, `civis`.
- [GNU Bash Manual: ANSI-C Quoting](https://www.gnu.org/software/bash/manual/html_node/ANSI_002dC-Quoting.html) — `$'\033'`.
- [GNU Bash Manual: Bash Conditional Expressions](https://www.gnu.org/software/bash/manual/html_node/Bash-Conditional-Expressions.html) — `-t fd`.
- Ejemplos probados con Bash 5.1.16 en Ubuntu 22.04 (salida comprobada con `cat -v`).
