---
tags: [bash, bucles, for, while, read]
aliases: ["for en Bash", "while en Bash", "Leer un archivo línea a línea"]
---
# Bucles en Bash

[[Inicio|← Inicio]] · [[Bash - Índice|Bash]]

> Un **bucle** repite un bloque de código. Bash tiene `for` (para recorrer una lista o contar), `while` y `until` (mientras se cumpla, o hasta que se cumpla, una condición). El patrón más útil es `while IFS= read -r línea`, que lee un archivo línea a línea sin estropearlo.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Iteración** | Cada vuelta del bucle |
| **Glob** | Patrón de nombres de archivo: `*.txt`, `foto?.png` |
| **`IFS`** | *Internal Field Separator*: caracteres que Bash usa para partir texto (espacio, tab, salto de línea) |
| **Redirigir** | Hacer que un comando lea de un archivo (`< archivo`) en vez del teclado |

## `for` sobre una lista

```bash
for fruta in manzana pera "uva blanca"; do
    echo "Me gusta la $fruta"
done
```

```text
Me gusta la manzana
Me gusta la pera
Me gusta la uva blanca
```

Con rangos de llaves: `{inicio..fin}` o `{inicio..fin..salto}`.

```bash
for i in {1..5}; do printf '%s ' "$i"; done; echo      # 1 2 3 4 5
for i in {0..10..5}; do printf '%s ' "$i"; done; echo  # 0 5 10
```

> [!warning] Las llaves no aceptan variables
> `{1..$n}` **no** funciona (las llaves se expanden antes que las variables, ver [[Expansiones en Bash]]). Para un límite variable usa el `for` estilo C.

## `for` estilo C

```bash
n=3
for (( i = 0; i < n; i++ )); do
    echo "vuelta $i"
done
```

Dentro de `(( ))` no hace falta `$` delante de las variables.

## `while` y `until`

`while` repite **mientras** la condición salga con `0`; `until`, **hasta que** salga con `0`.

```bash
cuenta=3
while (( cuenta > 0 )); do
    echo "$cuenta..."
    (( cuenta-- ))
done
echo "¡Despegue!"

until ping -c1 -W1 8.8.8.8 &>/dev/null; do    # espera a tener red
    sleep 1
done
```

Un bucle infinito es `while true; do ...; done` (se sale con `break`).

## Leer un archivo línea a línea

```bash
while IFS= read -r linea; do
    echo "Leo: [$linea]"
done < "lista.txt"          # el archivo entra por la entrada del bucle
```

Por qué cada pieza:

- `IFS=` → no quita los espacios del principio y final de la línea.
- `-r` → no interpreta las `\` (sin `-r`, `a\b` se convierte en `ab`).
- `< "lista.txt"` → el `while` entero lee del archivo. Es parecido a hacer `read` sobre un descriptor en C (ver [[Descriptores de fichero y read]]).

> [!tip] Última línea sin `\n`
> Si el archivo no termina en salto de línea, `read` devuelve error en la última línea y el bucle se la salta. Solución: `while IFS= read -r linea || [[ -n "$linea" ]]; do`.

> [!danger] No uses `for linea in $(cat archivo)`
> Eso parte por **palabras**, no por líneas, y además expande los `*`. Usa siempre `while read`.

## `break` y `continue`

- `break` → sale del bucle.
- `continue` → salta a la siguiente vuelta.

```bash
for i in {1..10}; do
    (( i % 2 == 0 )) && continue     # salta los pares
    (( i > 7 )) && break             # para al pasar de 7
    echo "$i"
done                                  # 1 3 5 7
```

## Recorrer archivos sin `ls`

Usa un **glob** directamente: Bash ya sabe listar archivos y respeta los espacios en los nombres.

```bash
for f in *.txt; do
    [[ -e "$f" ]] || continue        # si no hay ninguno, el patrón queda literal
    echo "Archivo: $f"
done
```

> [!warning] `for f in $(ls)` se rompe
> Un archivo llamado `mis notas.txt` se convierte en dos vueltas: `mis` y `notas.txt`. El glob `*` no tiene ese problema. La opción `shopt -s nullglob` hace que un patrón sin coincidencias desaparezca (ver [[Expansiones en Bash]]).

Para buscar también en subcarpetas, combina `find` con `while read`: `while IFS= read -r f; do ...; done < <(find . -name '*.c')`. La forma `< <( )` se explica en [[Subshells y procesos en Bash]].

## `tareas.sh` versión 3

Lista las tareas numeradas:

```bash
#!/usr/bin/env bash
readonly ARCHIVO="${TAREAS_ARCHIVO:-$HOME/.tareas.txt}"

echo "=== Mis tareas ==="
if [[ ! -s "$ARCHIVO" ]]; then
    echo "No hay tareas."
    exit 0
fi
n=1
while IFS= read -r tarea; do
    printf '%2d. %s\n' "$n" "$tarea"    # %2d: número en 2 columnas, como en C
    (( n++ ))
done < "$ARCHIVO"
```

```console
$ printf 'Comprar pan\nEstudiar Bash\n' > ~/.tareas.txt
$ ./tareas.sh
=== Mis tareas ===
 1. Comprar pan
 2. Estudiar Bash
```

El espacio delante del `1` es a propósito: con 10 tareas o más, los puntos siguen alineados. Este formato ` 1. Texto` se mantiene en todas las versiones.

Siguiente: separar `listar`, `anadir` y `borrar` en funciones → [[Funciones en Bash]].

## Errores típicos

| Error | Causa |
|---|---|
| `for i in {1..$n}` da `{1..5}` literal | Las llaves no expanden variables: usa `for (( ))` |
| Falta la última línea | El archivo no acaba en `\n`: añade `\|\| [[ -n "$linea" ]]` |
| Las `\` desaparecen | Falta `-r` en `read` |
| Un contador vale 0 tras el bucle | Usaste `cat archivo \| while ...`: la tubería crea una subshell (ver [[Subshells y procesos en Bash]]) |
| Sale `*.txt` como nombre | No hay archivos que encajen: comprueba con `-e` o usa `nullglob` |

## Relacionadas

[[Condiciones en Bash]] · [[Funciones en Bash]] · [[Arrays en Bash]] · [[Expansiones en Bash]] · [[Redirecciones y tuberías]] · [[Subshells y procesos en Bash]] · [[Descriptores de fichero y read]]

## Referencias

- [GNU Bash Manual: Looping Constructs](https://www.gnu.org/software/bash/manual/bash.html#Looping-Constructs) — `for`, `for (( ))`, `while`, `until`.
- [GNU Bash Manual: Bash Builtins (read)](https://www.gnu.org/software/bash/manual/bash.html#index-read) — opciones `-r` y uso de `IFS`.
- [Greg's Wiki: BashFAQ/001](https://mywiki.wooledge.org/BashFAQ/001) — leer un archivo línea a línea correctamente.
- [Greg's Wiki: ParsingLs](https://mywiki.wooledge.org/ParsingLs) — por qué no recorrer la salida de `ls`.
