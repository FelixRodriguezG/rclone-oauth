---
tags: [bash, cheatsheet, sintaxis, referencia]
aliases: ["Chuleta de Bash", "Bash cheatsheet", "Resumen de Bash"]
---
# Bash - Chuleta de sintaxis

[[Inicio|← Inicio]] · [[Bash - Índice|Bash]]

> Todo Bash en una página para **consultar**, no para aprender: cada bloque es un resumen mínimo y enlaza a la nota que lo explica con calma. Todos los ejemplos asumen Bash 4+ (en Ubuntu, Bash 5).

## Plantilla de script

```bash
#!/usr/bin/env bash
set -euo pipefail                         # para en errores, variables sin definir y tuberías
SCRIPT_DIR="$(dirname "$(readlink -f "${BASH_SOURCE[0]}")")"
die() { echo "${0##*/}: error: $*" >&2; exit 1; }
main() {
    [[ $# -ge 1 ]] || die "uso: ${0##*/} ARCHIVO"
    echo "Procesando $1"
}
main "$@"
```

→ [[Primer script en Bash]] · [[Scripts robustos con set y trap]] · [[Configuración y estado de un script]]

## Variables y comillas

```bash
nombre="Ana López"          # sin espacios alrededor del =
echo "$nombre"              # usar: SIEMPRE entre comillas dobles
echo "${nombre}_2"          # llaves para separar del texto que sigue
hoy="$(date +%F)"           # guardar la salida de un comando
export EDITOR=vim           # visible para los programas hijos
readonly VERSION="1.0"      # constante
```

| Comillas | Efecto |
|---|---|
| `"..."` | Expande `$var`, `$( )`, `\`; respeta espacios |
| `'...'` | Todo literal |
| `$'...'` | Literal con escapes: `$'\n'`, `$'\t'`, `$'\033'` |
| sin comillas | Se parte por espacios y expande `*` (casi nunca lo quieres) |

→ [[Variables y comillas en Bash]]

## Expansiones

| Sintaxis | Resultado (con `f="docs/notas.tar.gz"`) |
|---|---|
| `${f:-defecto}` | `f`, o `defecto` si está vacía o sin definir |
| `${f:=defecto}` | Igual, y además la asigna |
| `${f:?mensaje}` | Error con `mensaje` si está vacía |
| `${#f}` | Longitud: `17` |
| `${f#*/}` / `${f##*/}` | Quita prefijo corto / largo: `notas.tar.gz` |
| `${f%.*}` / `${f%%.*}` | Quita sufijo corto / largo: `docs/notas.tar` / `docs/notas` |
| `${f/tar/zip}` / `${f//a/A}` | Reemplaza la primera / todas |
| `${f:0:4}` | Subcadena desde 0, 4 caracteres: `docs` |
| `${f^^}` / `${f,,}` | Mayúsculas / minúsculas |
| `$(( 2 + 3 * 4 ))` | Aritmética entera: `14` |
| `{1..5}` / `{a,b}.txt` | `1 2 3 4 5` / `a.txt b.txt` |
| `*.c`, `?`, `[abc]` | Globs: nombres de archivo que coinciden |

→ [[Expansiones en Bash]]

## Condiciones

```bash
if [[ -f "$archivo" && -r "$archivo" ]]; then echo "legible"
elif (( n > 10 )); then echo "n grande"
else echo "otra cosa"; fi
[[ -d "$dir" ]] || mkdir -p "$dir"      # forma corta
case "$opcion" in
    s|S|si)  echo "sí" ;;
    *)       echo "no" ;;
esac
```

| Archivos | Cadenas | Números (`[[ ]]`) | Números (`(( ))`) |
|---|---|---|---|
| `-e` existe | `-z "$s"` vacía | `-eq` igual | `==` |
| `-f` archivo normal | `-n "$s"` no vacía | `-ne` distinto | `!=` |
| `-d` directorio | `"$a" == "$b"` | `-lt` menor | `<` |
| `-r -w -x` permisos | `"$s" == *.txt` (patrón) | `-le` menor o igual | `<=` |
| `-s` no vacío | `"$s" =~ ^[0-9]+$` (regex) | `-gt` / `-ge` mayor | `>` / `>=` |

→ [[Condiciones en Bash]]

## Bucles

```bash
for f in *.txt; do echo "$f"; done                 # archivos
for i in {1..5}; do echo "$i"; done                # rango fijo
for (( i = 0; i < n; i++ )); do echo "$i"; done    # estilo C
for arg in "$@"; do echo "$arg"; done              # argumentos
while IFS= read -r linea; do                       # archivo línea a línea
    echo "> $linea"
done < tareas.txt
while true; do ...; break; done                    # break / continue
```

→ [[Bucles en Bash]]

## Funciones

```bash
saludar() {
    local nombre="${1:-mundo}"     # local: solo existe dentro
    echo "Hola, $nombre"           # «devolver» texto: imprimirlo
    return 0                       # devolver estado: 0-255
}
saludar "Ana"
msg="$(saludar Ana)"               # capturar lo que imprime
# dentro, $1 $2 "$@" $# son los argumentos DE LA FUNCIÓN
```

→ [[Funciones en Bash]]

## Arrays

```bash
frutas=("pera" "manzana roja")        # indexado
frutas+=("kiwi")                      # añadir
echo "${frutas[0]}"                   # pera
echo "${#frutas[@]}"                  # 3 elementos
for f in "${frutas[@]}"; do echo "$f"; done
unset 'frutas[1]'                     # borrar uno
declare -A edad=([ana]=20 [luis]=31)  # asociativo (diccionario)
edad[eva]=25
echo "${!edad[@]}"                    # claves
mapfile -t lineas < tareas.txt        # archivo → array, una línea por elemento
```

→ [[Arrays en Bash]]

## Leer entrada

```bash
read -r -p "Nombre: " nombre                 # -r siempre; -p muestra un texto
read -r -s -p "Contraseña: " clave; echo     # -s no muestra lo escrito
read -r -n 1 -p "¿Seguro? [s/N] " r; echo    # una sola tecla
read -r -t 5 dato || echo "tiempo agotado"   # máximo 5 segundos
IFS=';' read -r estado prio texto <<< "$linea"   # partir por campos
select op in añadir lista salir; do ...; done   # menú numerado
```

→ [[Leer entrada del usuario en Bash]] · [[Descriptores de fichero y read]]

## Argumentos del script

```bash
echo "$0 $# $1 ${10}"             # nombre del script, nº de argumentos, 1.º, 10.º
for a in "$@"; do echo "$a"; done # "$@": todos, cada uno por separado
shift                             # quita $1 y corre los demás
while getopts ":vf:" opt; do      # -v, y -f VALOR (llega en $OPTARG)
    case "$opt" in v) V=1 ;; f) F="$OPTARG" ;; *) exit 2 ;; esac
done; shift $((OPTIND - 1))
```

→ [[Argumentos y subcomandos en Bash]]

## Redirecciones y tuberías

| Sintaxis | Efecto |
|---|---|
| `> f` / `>> f` / `2> f` | stdout a `f` (sobrescribe / añade) / stderr a `f` |
| `&> f` o `> f 2>&1` | Ambas a `f` (**el orden importa**) |
| `>&2` | Escribir en stderr |
| `< f` | stdin desde `f` |
| `a \| b` | stdout de `a` → stdin de `b` |
| `\| tee f` | Ver y guardar a la vez |
| `<<FIN` … `FIN` | Here-doc (`<<'FIN'` literal, `<<-FIN` quita tabs) |
| `<<< "$s"` | Here-string |
| `> /dev/null` | Descartar |
| `<(cmd)` | La salida de `cmd` como si fuera un archivo |

→ [[Redirecciones y tuberías]] · [[Subshells y procesos en Bash]]

## Códigos de salida y errores

| Código | Significado | | Código | Significado |
|---|---|---|---|---|
| `0` | Éxito | | `126` | No ejecutable |
| `1` | Error general | | `127` | Comando no encontrado |
| `2` | Mal uso | | `128+N` | Matado por la señal N (`130` = Ctrl+C) |

```bash
cmd; echo "$?"                              # código del último comando
cmd && echo "bien" || echo "mal"            # (cuidado: 'mal' también si falla el echo)
cd "$dir" || { echo "no existe" >&2; exit 1; }
command -v git > /dev/null || die "falta git"
```

→ [[Códigos de salida y errores en Bash]]

## `set` y `trap`

```bash
set -e                    # sale si un comando falla (salvo en if, &&, ||, !)
set -u                    # error al usar variables no definidas
set -o pipefail           # la tubería falla si falla cualquier parte
set -x; set +x            # activa / desactiva la traza
tmp="$(mktemp)"
trap 'rm -f "$tmp"' EXIT                    # siempre al salir
trap 'echo " cancelado" >&2; exit 130' INT  # Ctrl+C
```

→ [[Scripts robustos con set y trap]] · [[Depurar scripts de Bash]]

## Procesos, texto, colores y depuración

```bash
( cd /tmp && ls )                # subshell: el cd no afecta fuera
{ echo a; echo b; } > f          # grupo en la misma shell (; antes de })
cmd & wait                       # segundo plano ($! = su PID) y esperar
grep -inv 'patrón' f             # -i mayúsc., -n nº línea, -v invertir, -c -q -E
sed 's/viejo/nuevo/g' f          # sustituir; -i en el sitio; '/^#/d' borra; -n '2,5p'
awk -F';' '$1 == "x" { s += $4 } END { print s }' f    # columnas y sumas
[[ -t 1 && -z "${NO_COLOR:-}" ]] && ROJO=$'\033[31m' RESET=$'\033[0m'
bash -n s.sh; bash -x s.sh; shellcheck s.sh            # sintaxis, traza, linter
```

→ [[Subshells y procesos en Bash]] · [[grep, sed y awk lo justo]] · [[Colores en la terminal]] · [[Depurar scripts de Bash]]

## Relacionadas

[[Bash - Índice]] (todas las notas en orden) · [[Argumentos y subcomandos en Bash]] (siguiente en la ruta) · [[Vim]]

## Referencias

- [GNU Bash Reference Manual](https://www.gnu.org/software/bash/manual/bash.html) — sintaxis de todos los bloques.
- `man bash` — referencia de la versión instalada (Bash 5.1 en Ubuntu 22.04).
- [GNU Bash Manual: Shell Parameter Expansion](https://www.gnu.org/software/bash/manual/html_node/Shell-Parameter-Expansion.html) — tabla de expansiones `${...}`.
- [BashGuide (Greg's Wiki)](https://mywiki.wooledge.org/BashGuide) — buenas prácticas.
- Ejemplos comprobados con `bash -n` y ejecutados con Bash 5.1.16.
