---
tags: [bash, funciones, local, return, subcomandos]
aliases: ["Funciones Bash", "local en Bash", "return en Bash"]
---
# Funciones en Bash

[[Inicio|← Inicio]] · [[Bash - Índice|Bash]]

> Una **función** es un trozo de script con nombre que puedes llamar como si fuera un comando. Sirve para no repetir código y para dividir un script largo en partes pequeñas. Ojo: en Bash una función **no devuelve datos** con `return` como en C; los **imprime** y quien la llama los captura con `$( )`.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Parámetros posicionales** | `$1`, `$2`…: los argumentos que recibe la función (o el script) |
| **`local`** | Hace que una variable solo exista dentro de la función |
| **Ámbito** | Parte del código donde una variable es visible (ver [[Variables static y ámbito]] en C) |
| **Subcomando** | Primera palabra que elige qué hace el programa: `git commit`, `git push` |

## Definir y llamar

```bash
saludar() {
    echo "Hola, $1"
}

saludar "Ana"          # Hola, Ana
saludar "Luis Pérez"   # Hola, Luis Pérez (las comillas lo mantienen como UN argumento)
```

La función tiene que estar **definida antes** de llamarla (Bash lee el script de arriba abajo). La forma `function saludar { ... }` también vale.

## Argumentos: `$1`, `$@`, `$#`

| Variable | Contiene |
|---|---|
| `$1`, `$2`… `${10}` | Cada argumento por posición |
| `$#` | Cuántos argumentos hay |
| `"$@"` | Todos, cada uno como argumento separado (úsalo **siempre con comillas**) |
| `$0` | El nombre del **script** (no de la función) |

```bash
mostrar() {
    echo "Recibo $# argumentos"
    for arg in "$@"; do
        echo "  - $arg"
    done
}
mostrar uno "dos tres"
```

```text
Recibo 2 argumentos
  - uno
  - dos tres
```

Dentro de una función, `$1` es el argumento **de la función**, no del script. Si necesitas los del script, pásaselos: `mi_funcion "$@"`.

## `local`: variables privadas

Sin `local`, toda variable es **global**: una función puede pisar una variable del resto del script sin que te des cuenta.

```bash
contador=10
mal()  { contador=0; }
bien() { local contador=0; }

bien; echo "$contador"     # 10
mal;  echo "$contador"     # 0  ← la función la ha pisado
```

> [!tip] Regla
> Declara con `local` **todas** las variables de una función.

## `return` (código) frente a `echo` (dato)

- `return N` → código de salida `0`-`255`. Sirve para decir **si salió bien**, no para devolver resultados. Se lee con `$?` o directamente en un `if`.
- `echo` → imprime el **dato**. Quien llama lo captura con `$( )`.

```bash
es_par() {
    (( $1 % 2 == 0 ))           # el código de salida de la última orden es el de la función
}

doble() {
    local n="$1"
    echo $(( n * 2 ))           # el "valor devuelto" es lo que imprime
}

if es_par 4; then echo "4 es par"; fi
resultado="$(doble 21)"
echo "El doble es $resultado"     # El doble es 42
```

> [!warning] No mezcles mensajes y datos
> Si una función que capturas con `$( )` imprime también avisos, los avisos acaban dentro de la variable. Manda los mensajes a la salida de error: `echo "aviso" >&2` (ver [[Redirecciones y tuberías]]).

> [!note] `$( )` es una subshell
> Una función llamada con `$( )` no puede cambiar variables del script (ver [[Subshells y procesos en Bash]]).

## Funciones como subcomandos: `tareas.sh` versión 4

Un patrón muy común: una función por acción y un `case` (ver [[Condiciones en Bash]]) que elige cuál llamar según el primer argumento.

```bash
#!/usr/bin/env bash
readonly ARCHIVO="${TAREAS_ARCHIVO:-$HOME/.tareas.txt}"

listar() {
    local n=1 tarea
    if [[ ! -s "$ARCHIVO" ]]; then
        echo "No hay tareas."
        return 0
    fi
    while IFS= read -r tarea; do
        printf '%2d. %s\n' "$n" "$tarea"
        (( n++ ))
    done < "$ARCHIVO"
}

anadir() {          # nombres de función en ASCII: sin ñ ni tildes
    if [[ -z "$1" ]]; then
        echo "Uso: $0 añadir \"texto de la tarea\"" >&2
        return 1
    fi
    echo "$1" >> "$ARCHIVO"
    echo "Añadida: $1"
}

borrar() {
    local num="$1"
    if [[ ! "$num" =~ ^[0-9]+$ ]]; then
        echo "Uso: $0 borrar NÚMERO" >&2
        return 1
    fi
    sed -i "${num}d" "$ARCHIVO"      # borra la línea número $num
    echo "Borrada la tarea $num"
}

case "$1" in
    ""|lista) listar ;;                # sin argumentos, también lista
    añadir)   anadir "$2" ;;
    borrar)   borrar "$2" ;;
    help)     echo "Uso: $0 {añadir TEXTO|lista|borrar N|help}" ;;
    *)        echo "Subcomando desconocido: $1" >&2; exit 1 ;;
esac
```

```console
$ ./tareas.sh añadir "Comprar pan"
Añadida: Comprar pan
$ ./tareas.sh añadir "Estudiar Bash"
Añadida: Estudiar Bash
$ ./tareas.sh borrar 1
Borrada la tarea 1
$ ./tareas.sh
 1. Estudiar Bash
```

El código de salida del script es el de la última orden: si `anadir` hace `return 1`, el script termina con `1`. Para opciones como `-v` y mejores mensajes de uso: [[Argumentos y subcomandos en Bash]]. `tareas.sh` sigue creciendo en [[Arrays en Bash]].

Siguiente en la ruta: [[Redirecciones y tuberías]] (ya has usado `>>` y `>&2`; ahí se explican a fondo).

## Errores típicos

| Error | Causa |
|---|---|
| `command not found` al llamar la función | La llamas antes de definirla |
| Una variable del script cambia sola | Falta `local` en la función |
| `return "hola"` da `numeric argument required` | `return` solo acepta números (0-255; `return 300` da `44`): usa `echo` |
| Los argumentos con espacios se parten | Usaste `$@` o `$*` sin comillas: usa `"$@"` |

## Relacionadas

[[Condiciones en Bash]] · [[Bucles en Bash]] · [[Argumentos y subcomandos en Bash]] · [[Códigos de salida y errores en Bash]] · [[Subshells y procesos en Bash]] · [[Redirecciones y tuberías]] · [[Variables static y ámbito]]

## Referencias

- [GNU Bash Manual: Shell Functions](https://www.gnu.org/software/bash/manual/bash.html#Shell-Functions) — sintaxis, `local`, `return`, parámetros posicionales.
- [GNU Bash Manual: Special Parameters](https://www.gnu.org/software/bash/manual/bash.html#Special-Parameters) — `$#`, `$@`, `$*`, `$0`, `$?`.
- [Greg's Wiki: BashFAQ/084](https://mywiki.wooledge.org/BashFAQ/084) — cómo «devolver» un valor de una función.
- `man sed` — `sed -i 'Nd'` para borrar la línea N.
