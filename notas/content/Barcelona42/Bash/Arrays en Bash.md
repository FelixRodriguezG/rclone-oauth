---
tags: [bash, arrays, declare, mapfile, diccionarios]
aliases: ["Arrays Bash", "Listas en Bash", "Arrays asociativos", "declare -A", "mapfile", "readarray"]
---
# Arrays en Bash

[[Inicio|← Inicio]] · [[Bash - Índice|Bash]]

> Un **array** guarda varios valores en una sola variable. Bash tiene dos tipos: **indexados** (lista numerada desde 0, como en C) y **asociativos** (diccionario: cada valor tiene un nombre). Son la forma correcta de guardar listas de archivos o argumentos que pueden contener espacios.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Índice** | Posición de un elemento: `0`, `1`, `2`… |
| **Clave** | Nombre de un elemento en un array asociativo: `"ana"`, `"color"` |
| **Elemento** | Cada valor guardado |
| **Disperso** | Array con huecos: existen los índices 0 y 5 pero no del 1 al 4 |

## Arrays indexados

```bash
frutas=("manzana" "pera" "uva blanca")   # crear (sin comas: separados por espacios)
frutas+=("kiwi")                         # añadir al final
frutas[10]="mango"                       # asignar en una posición concreta

echo "${frutas[0]}"        # manzana          (las llaves son obligatorias)
echo "${frutas[-1]}"       # mango            (el último)
echo "${#frutas[@]}"       # 5                número de elementos
echo "${!frutas[@]}"       # 0 1 2 3 10       índices que existen
echo "${frutas[@]:1:2}"    # pera uva blanca  2 elementos desde el 1

unset 'frutas[1]'          # borra "pera" (deja un hueco: los índices no se recolocan)
unset frutas               # borra el array entero
```

> [!warning] `$frutas` no es el array
> `$frutas` (o `${frutas}`) da solo el **primer** elemento. Para todos, `"${frutas[@]}"`.

## Recorrer

```bash
for f in "${frutas[@]}"; do         # por valor
    echo "fruta: $f"
done

for i in "${!frutas[@]}"; do        # por índice (funciona aunque haya huecos)
    echo "$i → ${frutas[$i]}"
done
```

## `"${a[@]}"` frente a `"${a[*]}"`

| Forma | Resultado | Úsalo para |
|---|---|---|
| `"${a[@]}"` | Cada elemento como un argumento separado | Casi siempre: bucles, pasar a comandos |
| `"${a[*]}"` | **Un solo** texto con todos unidos por el primer carácter de `IFS` (un espacio) | Mostrar o unir: `(IFS=,; echo "${a[*]}")` → `a,b,c` |
| `${a[@]}` sin comillas | Se parte por espacios **y** expande globs | Nunca |

```bash
args=("-l" "mis notas.txt")
ls "${args[@]}"       # ls recibe 2 argumentos: -l  y  mis notas.txt
ls ${args[@]}         # ls recibe 3: -l  mis  notas.txt  → error
```

> [!tip] Construir comandos con arrays
> Si un comando lleva opciones que cambian según el caso, guárdalas en un array y pásalo con `"${opts[@]}"`. Meterlas en una cadena de texto se rompe con los espacios.

## Arrays asociativos: `declare -A`

Hay que **declararlos** antes de usarlos (Bash 4 o superior).

```bash
declare -A edad                  # sin esto, Bash lo trata como indexado
edad[ana]=20
edad["luis pérez"]=31
edad+=([marta]=25)

echo "${edad[ana]}"              # 20
echo "${#edad[@]}"               # 3
for nombre in "${!edad[@]}"; do  # claves (el orden NO está garantizado)
    echo "$nombre tiene ${edad[$nombre]} años"
done

if [[ -v edad[ana] ]]; then echo "ana existe"; fi   # -v: ¿existe la clave?
unset 'edad[ana]'
```

> [!note] Funciona como un diccionario de Python
> `edad[ana]=20` es como `edad["ana"] = 20`. Si necesitas un orden, guarda también las claves en un array indexado o pásalas por `sort`.

## Leer líneas a un array: `mapfile`

`mapfile` (también llamado `readarray`) mete cada línea de la entrada en un elemento:

```bash
mapfile -t lineas < "lista.txt"     # -t: quita el \n de cada línea
echo "El archivo tiene ${#lineas[@]} líneas"
echo "La primera es: ${lineas[0]}"

mapfile -t usuarios < <(cut -d: -f1 /etc/passwd)   # de la salida de un comando
```

`< <(comando)` es una sustitución de procesos (ver [[Subshells y procesos en Bash]]). No uses `cut ... | mapfile`: el array se quedaría en una subshell y se perdería.

## `tareas.sh`: tareas en memoria

Con las tareas en un array, contar, elegir una por número o reescribir el archivo es fácil:

```bash
#!/usr/bin/env bash
readonly ARCHIVO="${TAREAS_ARCHIVO:-$HOME/.tareas.txt}"
tareas=()

cargar() {
    [[ -f "$ARCHIVO" ]] && mapfile -t tareas < "$ARCHIVO"
}

guardar() {
    printf '%s\n' "${tareas[@]}" > "$ARCHIVO"   # printf repite el formato por elemento
}

borrar() {
    local i=$(( $1 - 1 ))                      # el usuario cuenta desde 1
    tareas=( "${tareas[@]:0:i}" "${tareas[@]:i+1}" )   # todo menos la i: sin huecos
    guardar
}

cargar
echo "Tienes ${#tareas[@]} tareas"
for i in "${!tareas[@]}"; do
    printf '%2d. %s\n' "$(( i + 1 ))" "${tareas[$i]}"   # " 1. Texto"
done
```

> [!warning] `guardar` con el array vacío
> `printf '%s\n'` sin elementos imprime una línea vacía. Si te importa, comprueba antes `(( ${#tareas[@]} ))`.

Siguiente: preguntarle al usuario qué hacer → [[Leer entrada del usuario en Bash]].

## Errores típicos

| Error | Causa |
|---|---|
| Solo sale el primer elemento | Escribiste `$a` en vez de `"${a[@]}"` |
| Elementos con espacios se parten | `${a[@]}` sin comillas |
| `edad[ana]` devuelve el valor de `edad[luis]` | Olvidaste `declare -A`: se trató como indexado (`ana` y `luis` valen 0) |
| `${#a[@]}` no coincide con el último índice | El array tiene huecos tras un `unset` |
| `mapfile` tras una tubería no guarda nada | La tubería crea una subshell: usa `< <(cmd)` |
| Todo junto en una línea | Usaste `"${a[*]}"` donde querías `"${a[@]}"` |

## Relacionadas

[[Expansiones en Bash]] · [[Bucles en Bash]] · [[Funciones en Bash]] · [[Subshells y procesos en Bash]] · [[Leer entrada del usuario en Bash]] · [[Arrays en C]] · [[Python - Índice]]

## Referencias

- [GNU Bash Manual: Arrays](https://www.gnu.org/software/bash/manual/bash.html#Arrays) — indexados, asociativos, `${!a[@]}`, `unset`, `[@]` frente a `[*]`.
- [GNU Bash Manual: Bash Builtins (mapfile)](https://www.gnu.org/software/bash/manual/bash.html#index-mapfile) — opción `-t` y `readarray`.
- [GNU Bash Manual: Bash Conditional Expressions](https://www.gnu.org/software/bash/manual/bash.html#Bash-Conditional-Expressions) — `-v` para comprobar si existe un elemento.
- [Greg's Wiki: BashFAQ/005](https://mywiki.wooledge.org/BashFAQ/005) — buenas prácticas con arrays.
