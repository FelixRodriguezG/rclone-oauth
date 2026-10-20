---
tags: [bash, argumentos, getopts, subcomandos, cli]
aliases: ["getopts", "Parámetros posicionales", "$@ en Bash", "Subcomandos"]
---
# Argumentos y subcomandos en Bash

[[Inicio|← Inicio]] · [[Bash - Índice|Bash]]

> Los **argumentos** son las palabras que escribes detrás del nombre del script: `./tareas.sh añadir "Comprar pan"`. Con `$1`, `$@`, `shift`, `case` y `getopts` el script entiende **subcomandos** (`añadir`, `lista`…) y **opciones** (`-v`, `-f archivo`), como `git` o `docker`.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Argumento** | Cada palabra después del nombre del script (separadas por espacios, salvo entre comillas) |
| **Parámetro posicional** | La variable que guarda cada argumento: `$1`, `$2`… |
| **Opción** (*flag*) | Argumento que empieza por `-` y cambia el comportamiento: `-v` |
| **Subcomando** | Primera palabra que elige **qué** hace el programa: `git commit`, `tareas.sh lista` |
| **Uso** (*usage*) | Texto corto que explica cómo se llama el programa |

## Las variables de los argumentos

```bash
./args.sh uno "dos tres" cuatro
```

| Variable | Valor en el ejemplo |
|---|---|
| `$0` | `./args.sh` (cómo se llamó al script) |
| `$1`, `$2`, `$3` | `uno`, `dos tres`, `cuatro` (`${10}` a partir del décimo) |
| `$#` | `3` (cuántos hay) |
| `"$@"` | Todos, **cada uno por separado**: `"uno" "dos tres" "cuatro"` |
| `"$*"` | Todos **juntos en una sola cadena**: `"uno dos tres cuatro"` |

> [!warning] Escribe siempre `"$@"` con comillas
> Sin comillas, `dos tres` se partiría en dos argumentos (ver [[Variables y comillas en Bash]]). `"$@"` es la forma correcta de pasar los argumentos a otro comando o recorrerlos con `for arg in "$@"` (ver [[Bucles en Bash]]).

## `shift`: consumir argumentos

`shift` tira `$1` y corre los demás una posición: `$2` pasa a ser `$1` y `$#` baja en uno. Sirve para quitar el subcomando y quedarte con «el resto».

```bash
comando="$1"     # añadir
shift            # ahora "$@" = "comprar pan"
echo "Texto: $*" # Texto: comprar pan
```

## Subcomandos con `case`

`case` compara una palabra con varios patrones (ver [[Condiciones en Bash]]). Es el esqueleto de cualquier programa con subcomandos:

```bash
case "$comando" in
    añadir)         anadir "$@" ;;
    lista)          listar ;;
    borrar)         borrar "$@" ;;
    help|-h|--help) usage ;;
    *)              error_uso "comando desconocido: $comando" ;;
esac
```

`a|b` acepta varios nombres para lo mismo (aquí `help`, `-h` y `--help`) y `*)` recoge todo lo demás. `anadir`, `listar` y `borrar` son funciones (ver [[Funciones en Bash]]).

## Opciones con `getopts`

`getopts` lee las opciones cortas que van **antes** del subcomando, una por vuelta de `while`:

```bash
VERBOSE=0
ARCHIVO="${TAREAS_ARCHIVO:-$HOME/.tareas.txt}"   # -f lo cambia

while getopts ":vf:" opt; do
    case "$opt" in
        v)  VERBOSE=1 ;;
        f)  ARCHIVO="$OPTARG" ;;       # el valor que va detrás de -f
        :)  error_uso "la opción -$OPTARG necesita un argumento" ;;
        \?) error_uso "opción desconocida: -$OPTARG" ;;
    esac
done
shift $((OPTIND - 1))                  # quita las opciones ya leídas
```

Cómo se lee `":vf:"`:

- `v` → opción sin valor (`-v`).
- `f:` → los `:` detrás significan «lleva un valor» (`-f archivo`), que llega en `$OPTARG`.
- El `:` **inicial** pide modo silencioso: tú escribes los mensajes con los casos `:` (falta el valor) y `\?` (opción desconocida).

Acepta `-v -f x`, `-vf x` y `-fx`. **No** entiende opciones largas (`--verbose`); para eso tendrías que recorrerlas a mano con `while`/`case`/`shift`.

## `usage` y errores de uso

Un error de **uso** (comando mal escrito, falta un argumento) se avisa por `stderr` y se sale con código **2**, que es la convención (ver [[Códigos de salida y errores en Bash]]):

```bash
usage() {
    cat <<FIN
Uso: ${0##*/} [-v] [-f archivo] <comando> [args]

Comandos:
  añadir TEXTO   Añade una tarea
  lista          Muestra las tareas numeradas
  borrar N       Borra la tarea número N
  help           Muestra esta ayuda
FIN
}

error_uso() {
    echo "${0##*/}: $1" >&2
    echo "Prueba '${0##*/} help'." >&2
    exit 2
}
```

`${0##*/}` es el nombre del script sin la ruta (ver [[Expansiones en Bash]]); `<<FIN` es un *here-doc* (ver [[Redirecciones y tuberías]]). La ayuda pedida con `help` va a `stdout` y sale con 0: no es un error.

## Todo junto: `tareas.sh`

Esta es la versión de referencia: la que usa la TUI `tareas.py` (ver [[De script a TUI - arquitectura]]). Va después de `usage`, `error_uso` y el bucle de `getopts`:

```bash
listar() {
    [[ -s "$ARCHIVO" ]] || { echo "(sin tareas)"; return 0; }
    nl -ba -w2 -s'. ' "$ARCHIVO"    # numera TODAS las líneas: " 1. texto"
}

anadir() {
    [[ -n "$*" ]] || error_uso "añadir necesita un texto"   # "" tampoco vale
    echo "$*" >> "$ARCHIVO"
    if (( VERBOSE )); then echo "Añadida a $ARCHIVO" >&2; fi
}

borrar() {
    local total=0

    [[ "${1:-}" =~ ^[0-9]+$ ]] || error_uso "borrar necesita un número"
    [[ -f "$ARCHIVO" ]] && total=$(wc -l < "$ARCHIVO")
    if (( 10#$1 < 1 || 10#$1 > total )); then   # 10#: «08» es decimal, no octal
        echo "${0##*/}: no existe la tarea $1" >&2
        exit 1                      # error de datos, no de uso (ver Códigos de salida)
    fi
    sed -i "${1}d" "$ARCHIVO"       # borra la línea N (ver sed)
}

[[ $# -ge 1 ]] || error_uso "falta el comando"
comando="$1"
shift

case "$comando" in
    añadir)         anadir "$@" ;;
    lista)          listar ;;
    borrar)         borrar "$@" ;;
    help|-h|--help) usage ;;
    *)              error_uso "comando desconocido: $comando" ;;
esac
```

`nl` numera líneas: `-w2` reserva 2 columnas para el número, `-s'. '` pone `. ` detrás y `-ba` numera también las líneas vacías (así el número coincide con el que borra `sed`).

```text
$ ./tareas.sh añadir Comprar pan
$ ./tareas.sh -v añadir "Estudiar bash"
Añadida a /home/ana/.tareas.txt
$ ./tareas.sh lista
 1. Comprar pan
 2. Estudiar bash
$ ./tareas.sh volar; echo $?
tareas.sh: comando desconocido: volar
Prueba 'tareas.sh help'.
2
```

> [!note] ¿Y el menú sin argumentos?
> Aquí, sin comando hay un error de uso. Para conservar el menú de [[Leer entrada del usuario en Bash]], añade **antes** de esa comprobación `if [[ $# -eq 0 && -t 0 ]]; then menu; exit 0; fi`: solo abre el menú si hay una persona en la terminal (`-t 0`).

## Errores típicos

| Error | Qué pasa |
|---|---|
| `$@` o `$1` sin comillas | Los argumentos con espacios se parten |
| Olvidar `shift $((OPTIND - 1))` | `$1` sigue siendo `-v` en vez del subcomando |
| Opciones **después** del subcomando (`tareas.sh lista -v`) | `getopts` para en el primer argumento que no es opción |
| Mensajes de error por `stdout` | Se mezclan con la salida útil y una tubería se los come |

## Relacionadas

[[Funciones en Bash]] · [[Condiciones en Bash]] · [[grep, sed y awk lo justo]] · [[Códigos de salida y errores en Bash]] · [[Leer entrada del usuario en Bash]] · [[Scripts robustos con set y trap]] · [[Configuración y estado de un script]] · [[De script a TUI - arquitectura]] · [[Bash - Chuleta de sintaxis]]

## Referencias

- [GNU Bash Manual: Special Parameters](https://www.gnu.org/software/bash/manual/html_node/Special-Parameters.html) — `$@`, `$*`, `$#`, `$0`.
- [GNU Bash Manual: Bourne Shell Builtins (`getopts`, `shift`)](https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html) — `OPTARG`, `OPTIND`, modo silencioso.
- [BashFAQ/035 (Greg's Wiki)](https://mywiki.wooledge.org/BashFAQ/035) — cómo leer opciones y argumentos.
- [GNU Coding Standards: Command-Line Interfaces](https://www.gnu.org/prep/standards/html_node/Command_002dLine-Interfaces.html) — `--help` y convenciones de opciones.
- Ejemplos probados con Bash 5.1.16 en Ubuntu 22.04.
