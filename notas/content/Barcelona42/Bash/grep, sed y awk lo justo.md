---
tags: [bash, grep, sed, awk, texto, filtros]
aliases: ["grep", "sed", "awk", "Filtrar texto en Bash"]
---
# grep, sed y awk lo justo

[[Inicio|← Inicio]] · [[Bash - Índice|Bash]]

> Tres herramientas clásicas para trabajar con texto línea a línea: **grep** busca líneas, **sed** las modifica y **awk** las trata como columnas para calcular o reformatear. Con lo básico de cada una se resuelve casi todo lo que un script necesita.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Filtro** | Programa que lee texto (de un archivo o de stdin), lo transforma y lo escribe en stdout |
| **Expresión regular** (*regex*) | Patrón de búsqueda: `^` inicio de línea, `$` final, `.` cualquier carácter, `*` repetir |
| **Campo** | Cada columna de una línea, separada por un **separador** (espacios, `;`, `:`…) |
| **En el sitio** (*in place*) | Modificar el propio archivo en vez de imprimir el resultado |

## El archivo de ejemplo

`plan.txt`, con campos separados por `;`. No es el `~/.tareas.txt` del mini-proyecto (una tarea por línea, sin columnas): tiene más campos para practicar con columnas.

```text
# estado;prioridad;tarea;horas
pendiente;alta;Entregar libft;3
hecha;baja;Comprar pan;1
pendiente;media;Leer man bash;2
hecha;alta;Configurar Vim;2
pendiente;baja;Ordenar escritorio;1
```

## grep: buscar líneas

```bash
grep pendiente plan.txt            # líneas que contienen 'pendiente'
grep -c '^pendiente' plan.txt      # 3: cuántas líneas empiezan así
grep -in 'VIM' plan.txt            # 5:hecha;alta;Configurar Vim;2
grep -v '^#' plan.txt              # todas menos el comentario
grep -E 'alta|media' plan.txt      # una cosa u otra
grep -q libft plan.txt && echo "hay libft"
```

| Opción | Qué hace |
|---|---|
| `-i` | Ignora mayúsculas/minúsculas |
| `-v` | Invierte: líneas que **no** coinciden |
| `-c` | Cuenta líneas en vez de mostrarlas |
| `-n` | Muestra el número de línea |
| `-E` | Regex extendida: `\|`, `+`, `?`, `()` sin barras |
| `-q` | Silencio: solo el código de salida (0 encontrado, 1 no) |
| `-r` | Busca en un directorio, recursivamente |
| `-F` | Texto literal, sin regex (útil si buscas `.` o `*`) |

> [!tip] `grep -q` en condiciones
> `if grep -q "^$tarea$" plan.txt; then ...` comprueba si una tarea ya existe sin imprimir nada (ver [[Condiciones en Bash]]).

## sed: editar líneas

```bash
sed 's/pendiente/TODO/' plan.txt        # sustituye la 1.ª aparición de cada línea
sed 's/;/ | /g' plan.txt                # g: todas las apariciones
sed '/^#/d' plan.txt                    # borra las líneas que empiezan por #
sed '3d' plan.txt                       # borra la línea 3
sed -n '2,3p' plan.txt                  # imprime solo las líneas 2 a 3
sed -n '/Comprar/p' plan.txt            # imprime solo las que contienen 'Comprar'
sed -i 's/;baja;/;BAJA;/' plan.txt      # -i: modifica el archivo
```

- `s/buscar/reemplazo/` sustituye; si el texto tiene `/`, cambia de separador: `s|/home|~|`.
- `-n` desactiva la impresión automática: solo sale lo que marques con `p`.
- Una dirección (`3`, `2,5`, `/regex/`) delante del comando dice a qué líneas se aplica.

> [!warning] `sed -i` no tiene deshacer
> Prueba primero **sin** `-i` para ver el resultado. Con `sed -i.bak` guarda una copia (`plan.txt.bak`). En macOS, `-i` necesita un argumento: `sed -i '' ...`.

Con variables, usa comillas **dobles** para que Bash las expanda: `sed -i "${n}d" plan.txt` borra la línea número `n`. Es lo que hace `borrar N` en `tareas.sh` (ver [[Argumentos y subcomandos en Bash]]).

## awk: columnas y cálculos

Un programa de awk es una lista de `condición { acción }`. Para cada línea, si se cumple la condición, ejecuta la acción. Dentro, `$1`, `$2`… son los campos, `$0` la línea entera, `NR` el número de línea y `NF` cuántos campos tiene.

```bash
awk -F';' '{ print $3 }' plan.txt
# tarea / Entregar libft / Comprar pan / ...

awk -F';' 'NR > 1 && $1 == "pendiente" { print NR-1 ": " $3 }' plan.txt
# 1: Entregar libft
# 3: Leer man bash
# 5: Ordenar escritorio

awk -F';' 'NR > 1 { total += $4 } END { print "Horas totales:", total }' plan.txt
# Horas totales: 9

awk -F';' '$1 == "pendiente" { h += $4 } END { printf "Pendiente: %d h\n", h }' plan.txt
# Pendiente: 6 h

awk -F';' 'NR > 1 { n[$2]++ } END { for (p in n) print p, n[p] }' plan.txt | sort
# alta 2 / baja 2 / media 1
```

- `-F';'` fija el separador. Sin `-F`, separa por espacios y tabs (y junta los repetidos): `echo "uno   dos" | awk '{ print $2 }'` → `dos`.
- `END { }` se ejecuta una vez al final: ideal para totales.
- Las variables empiezan en 0 o `""`, no hace falta declararlas. `n[$2]++` es un diccionario.
- El programa va entre comillas **simples** para que Bash no toque los `$1` de awk. Para pasarle una variable de Bash: `awk -v p="$prio" -F';' '$2 == p' plan.txt`.

## ¿Cuál uso?

| Quiero… | Herramienta |
|---|---|
| Saber si algo está / filtrar líneas | `grep` |
| Contar líneas que cumplen algo | `grep -c` |
| Reemplazar texto o borrar líneas | `sed` |
| Ver un rango de líneas | `sed -n 'A,Bp'` |
| Sacar una columna | `awk` (o `cut -d';' -f3` si es muy simple) |
| Sumar, contar por grupos, reformatear | `awk` |
| Lógica compleja, muchos pasos | Un bucle de Bash (ver [[Bucles en Bash]]) o Python |

Se combinan con tuberías (ver [[Redirecciones y tuberías]]):

```bash
grep -v '^#' plan.txt | grep pendiente | awk -F';' '{ print "- " $3 }'
```

Aunque `awk` solo ya lo hace: `awk -F';' '$1 == "pendiente" { print "- " $3 }' plan.txt`.

## Errores típicos

| Error | Qué pasa |
|---|---|
| `grep 'alta\|media'` sin `-E` | El `\|` se busca como carácter literal y no encuentra nada; usa `grep -E` |
| `sed "s/$x/y/"` con `/` dentro de `$x` | `unterminated s command`; cambia de separador |
| `awk "{ print $1 }"` con comillas dobles | Bash cambia `$1` por **su** primer argumento |
| `grep` sin resultados con `set -e` | Sale con 1 y mata el script; añade `\|\| true` |
| Olvidar `-F` en awk con `;` | `$1` es la línea entera |

## Relacionadas

[[Redirecciones y tuberías]] · [[Expansiones en Bash]] · [[Bucles en Bash]] · [[Condiciones en Bash]] · [[Argumentos y subcomandos en Bash]] · [[Bash - Chuleta de sintaxis]]

## Referencias

- [GNU grep manual](https://www.gnu.org/software/grep/manual/grep.html) — opciones y regex básicas/extendidas.
- [GNU sed manual](https://www.gnu.org/software/sed/manual/sed.html) — `s`, `d`, `p`, `-n`, `-i`, direcciones.
- [The GNU Awk User's Guide](https://www.gnu.org/software/gawk/manual/gawk.html) — campos, `NR`, `NF`, `END`, arrays.
- `man grep`, `man sed`, `man awk`.
- Ejemplos probados en Ubuntu 22.04 (GNU grep, GNU sed, GNU awk).
