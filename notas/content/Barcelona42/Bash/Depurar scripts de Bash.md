---
tags: [bash, depuracion, shellcheck, debug, errores]
aliases: ["bash -x", "shellcheck", "Debug en Bash", "PS4"]
---
# Depurar scripts de Bash

[[Inicio|← Inicio]] · [[Bash - Índice|Bash]]

> Bash no tiene un depurador como [[Depurar con gdb|gdb]], pero no le hace falta: con `bash -n` compruebas la sintaxis, con `bash -x` ves cada línea que se ejecuta con las variables ya sustituidas, y **shellcheck** encuentra la mayoría de errores antes de ejecutar nada.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Traza** (*trace*) | Lista de los comandos según se ejecutan |
| **`xtrace`** | Nombre de la opción `-x`: activa la traza |
| **`PS4`** | Prefijo que Bash pone delante de cada línea de la traza (por defecto `+ `) |
| **Linter** | Programa que lee el código sin ejecutarlo y avisa de errores y malas prácticas |
| **CRLF** | Fin de línea de Windows (`\r\n`); Linux usa solo `\n` (LF) |

## 1. `bash -n`: solo sintaxis

Lee el script **sin ejecutarlo** y avisa de errores de sintaxis (un `fi` que falta, comillas sin cerrar):

```bash
bash -n tareas.sh && echo "sintaxis OK"
```

```text
roto.sh: line 3: syntax error near unexpected token `fi'
```

No detecta errores de lógica ni variables mal escritas: eso lo hacen `-x` y shellcheck.

## 2. `bash -x`: ver qué se ejecuta

```bash
bash -x tareas.sh lista          # traza todo el script
```

Para trazar **solo una parte**, actívalo y desactívalo dentro:

```bash
set -x          # empieza la traza
n=2
saludar "Ana López"
set +x          # termina la traza
```

Cada línea de la traza empieza por `+` (`++` si viene de un `$( )` anidado) y muestra el comando **después** de las expansiones: ves el valor real de las variables y cómo quedaron las comillas.

### `PS4` con archivo, línea y función

```bash
export PS4='+ ${BASH_SOURCE##*/}:${LINENO}:${FUNCNAME[0]:-main}: '
```

Con comillas **simples**, para que se evalúe en cada línea. `FUNCNAME[0]` es la función en la que estás; fuera de una función no existe, y entonces `:-main` pone `main`.

> [!example]- El script `dbg.sh` de la traza
> ```bash
> #!/usr/bin/env bash
> export PS4='+ ${BASH_SOURCE##*/}:${LINENO}:${FUNCNAME[0]:-main}: '
> saludar() {
>     local nombre="$1"
>     echo "Hola, $nombre"
> }
> set -x
> n=2
> saludar "Ana López"
> set +x
> ```

Salida de `bash dbg.sh`:

```text
+ dbg.sh:8:main: n=2
+ dbg.sh:9:main: saludar 'Ana López'
+ dbg.sh:4:saludar: local 'nombre=Ana López'
+ dbg.sh:5:saludar: echo 'Hola, Ana López'
Hola, Ana López
+ dbg.sh:10:main: set +x
```

> [!tip] Traza a un archivo
> La traza sale por stderr. Para guardarla aparte sin mezclarla con los errores: `exec 3> traza.log; BASH_XTRACEFD=3; set -x`.

## 3. shellcheck

**shellcheck** es un linter para scripts de shell: lee el código y señala errores con un código `SCxxxx` y una explicación. Detecta comillas que faltan, variables sin usar, comparaciones mal escritas…

```bash
sudo apt install shellcheck      # en tu ordenador (en 42, sin sudo: shellcheck.net)
shellcheck tareas.sh
```

También se puede pegar el script en [shellcheck.net](https://www.shellcheck.net/) o integrarlo en el editor (ver [[Vim]] con ALE o similares).

| Código | Aviso | Arreglo |
|---|---|---|
| SC2086 | *Double quote to prevent globbing and word splitting* | `rm $f` → `rm "$f"` |
| SC2164 | *Use `cd ... \|\| exit` in case cd fails* | `cd "$dir" \|\| exit 1` |
| SC2155 | *Declare and assign separately to avoid masking return values* | `local x; x="$(cmd)"` |
| SC2034 | *Variable appears unused* | Bórrala o comprueba el nombre |
| SC2154 | *Variable is referenced but not assigned* | Typo, o viene de `source` |
| SC2006 | *Use `$(...)` instead of legacy backticks* | `` `cmd` `` → `$(cmd)` |
| SC2162 | *read without -r will mangle backslashes* | `read -r linea` |
| SC1090 | *Can't follow non-constant source* | `# shellcheck source=/dev/null` |

Cada código tiene su página: `https://www.shellcheck.net/wiki/SC2086`. Para silenciar un aviso concreto **a propósito**, pon encima de la línea `# shellcheck disable=SC2086` y explica por qué.

## 4. Mensajes de depuración a stderr

Para mirar un valor concreto, imprímelo a **stderr** para no ensuciar la salida del script:

```bash
debug() {
    [[ -n "${DEBUG:-}" ]] && echo "DEBUG: $*" >&2
    return 0
}

debug "archivo=$ARCHIVO, n=$n"
```

`DEBUG=1 ./tareas.sh lista` los muestra; sin la variable, no. El `return 0` evita que la función «falle» con `set -e` cuando `DEBUG` no está definida (ver [[Scripts robustos con set y trap]]).

## Errores típicos

| Síntoma | Causa | Arreglo |
|---|---|---|
| `nombre: command not found` | `nombre = Ana`: espacios alrededor del `=` | `nombre="Ana"` (ver [[Variables y comillas en Bash]]) |
| `Ana: command not found` | `nombre= Ana`: asigna `""` y ejecuta `Ana` | Sin espacios |
| `/bin/bash^M: bad interpreter` | Archivo con finales CRLF (editado en Windows) | `sed -i 's/\r$//' script.sh` o `dos2unix` |
| `$'\r': command not found` | Lo mismo, en una línea vacía | Igual; `file script.sh` dice `with CRLF` |
| `ls: cannot access 'mi'` | `ls $f` con `f="mi archivo.txt"` | `ls "$f"` (SC2086) |
| `[: too many arguments` o `unary operator expected` | `[ $x = hola ]` con `x` con espacios o vacía | `[[ "$x" == hola ]]` |
| `unexpected end of file` | Falta un `fi`, `done`, `esac` o `}` | `bash -n` dice dónde empieza a perderse |
| `Permission denied` | Falta `chmod +x` (código 126) | Ver [[Primer script en Bash]] |
| La variable está vacía tras el bucle | `cmd \| while read`: el bucle va en una subshell | `while read ...; done < <(cmd)` (ver [[Subshells y procesos en Bash]]) |

> [!question] Método cuando algo no cuadra
> 1. `bash -n` → ¿sintaxis?
> 2. `shellcheck` → ¿comillas, typos?
> 3. `set -x` alrededor de la zona sospechosa → ¿qué valores llegan de verdad?
> 4. Reduce el problema a 5 líneas en un archivo aparte.

Con esto termina la ruta de Bash. Siguiente paso: [[tmux - Índice]] para trabajar con varias terminales, o [[TUI - Índice]] para convertir `tareas.sh` en una TUI.

## Relacionadas

[[Scripts robustos con set y trap]] · [[Códigos de salida y errores en Bash]] · [[Variables y comillas en Bash]] · [[Primer script en Bash]] · [[Subshells y procesos en Bash]] · [[Depurar con gdb]] · [[Vim]] · [[Bash - Chuleta de sintaxis]] · [[Bash - Índice]] · [[tmux - Índice]] · [[TUI - Índice]]

## Referencias

- [GNU Bash Manual: The Set Builtin](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html) — `-n`, `-x`.
- [GNU Bash Manual: Bash Variables](https://www.gnu.org/software/bash/manual/html_node/Bash-Variables.html) — `PS4`, `LINENO`, `FUNCNAME`, `BASH_XTRACEFD`.
- [ShellCheck](https://www.shellcheck.net/) y su [wiki](https://www.shellcheck.net/wiki/) — significado de cada código SC.
- [BashFAQ/052 (Greg's Wiki)](https://mywiki.wooledge.org/BashFAQ/052) — finales de línea CRLF.
- Ejemplos probados con Bash 5.1.16 en Ubuntu 22.04 (shellcheck no estaba instalado: los avisos están tomados de su wiki).
