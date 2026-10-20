---
tags: [bash, expansiones, parametros, globbing, aritmetica]
aliases: ["Expansión de parámetros", "Globbing", "Aritmética en Bash", "${var}"]
---
# Expansiones en Bash

[[Inicio|← Inicio]] · [[Bash - Índice|Bash]]

> Antes de ejecutar una línea, Bash la **transforma**: sustituye variables, hace cuentas, convierte `*.txt` en nombres de archivo… Cada una de esas transformaciones es una **expansión**. Con la expansión de parámetros puedes recortar, sustituir y cambiar mayúsculas sin llamar a `sed` ni a `cut`.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Expansión de parámetros** | `${...}`: leer una variable transformándola por el camino |
| **Patrón** | Texto con comodines: `*` (cualquier cosa), `?` (un carácter), `[abc]` (uno de esos) |
| **Glob / globbing** | Usar patrones para nombrar archivos: `*.c` |
| **Prefijo / sufijo** | Trozo del principio / del final de un texto |

## Orden de las expansiones (resumido)

1. **Llaves** `{a,b}`, `{1..5}`
2. **Tilde** `~` → `$HOME`
3. **Parámetros** `$var`, `${...}`, **aritmética** `$(( ))` y **comandos** `$( )` (de izquierda a derecha)
4. **Separación de palabras** (solo en lo que no lleva comillas)
5. **Globbing** `*.txt` (solo sin comillas)
6. Se quitan las comillas

Por eso `{1..$n}` no funciona: las llaves van antes que las variables. Y por eso `"$var"` con comillas se libra de los pasos 4 y 5.

## Recortar: `#`, `##`, `%`, `%%`

`#` quita por la **izquierda** (prefijo), `%` por la **derecha** (sufijo). Uno solo quita lo **más corto** que encaje; doble, lo **más largo**.

```bash
ruta="/home/ana/fotos/playa.tar.gz"
echo "${ruta##*/}"     # playa.tar.gz       (quita hasta la última /: como basename)
echo "${ruta%/*}"      # /home/ana/fotos    (quita desde la última /: como dirname)
echo "${ruta#*.}"      # tar.gz             (quita hasta el primer punto)
echo "${ruta##*.}"     # gz                 (extensión)
echo "${ruta%.*}"      # /home/ana/fotos/playa.tar
echo "${ruta%%.*}"     # /home/ana/fotos/playa
```

> [!tip] Truco para acordarte
> En el teclado americano `#` está a la izquierda de `$` y `%` a la derecha.

## Longitud, sustituir y cortar trozos

```bash
s="hola mundo mundo"
echo "${#s}"              # 16      longitud
echo "${s/mundo/Bash}"    # hola Bash mundo     primera coincidencia
echo "${s//mundo/Bash}"   # hola Bash Bash      todas
echo "${s:5}"             # mundo mundo         desde la posición 5 (empieza en 0)
echo "${s:0:4}"           # hola                desde 0, 4 caracteres
echo "${s: -5}"           # mundo               los 5 últimos (¡espacio antes del -!)
```

## Mayúsculas y minúsculas

```bash
n="Ana Pérez"
echo "${n^^}"   # ANA PÉREZ
echo "${n,,}"   # ana pérez
echo "${n^}"    # Ana Pérez   (solo la primera letra)
```

Útil para comparar respuestas sin importar mayúsculas: `[[ "${resp,,}" == "s" ]]`.

## Valores por defecto y obligatorios

| Forma | Si `var` está vacía o no existe… |
|---|---|
| `${var:-texto}` | Usa `texto` (sin cambiar `var`) |
| `${var:=texto}` | Usa `texto` **y lo guarda** en `var` |
| `${var:?mensaje}` | Imprime `mensaje` por `stderr` y **termina** el script |
| `${var:+texto}` | Al revés: usa `texto` solo si `var` **tiene** valor |

```bash
destino="${1:-.}"                       # primer argumento o la carpeta actual
: "${EDITOR:=vim}"                      # ':' no hace nada; solo sirve para que se ejecute el :=
archivo="${1:?Falta el nombre del archivo}"
```

## Aritmética: `$(( ))`

Solo con **enteros** (para decimales, `bc` o `awk`). Dentro no hace falta `$`.

```bash
a=7 b=2
echo $(( a + b ))   # 9
echo $(( a / b ))   # 3   (división entera, como en C)
echo $(( a % b ))   # 1
echo $(( a ** b ))  # 49  (potencia)
(( a++ ))           # sin $ delante: solo hace la operación
```

## Llaves: `{a,b}` y `{1..3}`

Generan texto, existan o no los archivos:

```bash
echo foto{1..3}.png          # foto1.png foto2.png foto3.png
mkdir -p proyecto/{src,include,tests}
cp config.txt{,.bak}         # = cp config.txt config.txt.bak
```

## Globbing: `*`, `?`, `[ ]`

Bash sustituye el patrón por los **archivos que existen** y encajan:

| Patrón | Encaja con |
|---|---|
| `*.c` | Todo lo que acaba en `.c` |
| `ft_?.c` | `ft_a.c`, `ft_1.c` (un solo carácter) |
| `[abc]*` | Lo que empieza por `a`, `b` o `c` |
| `[!.]*` | Lo que **no** empieza por punto |

Por defecto `*` no incluye archivos ocultos (los que empiezan por `.`).

> [!warning] Patrón sin coincidencias
> Si nada encaja, Bash deja el patrón **tal cual**: `ls *.xyz` busca un archivo llamado literalmente `*.xyz`. Con `shopt -s nullglob` el patrón desaparece; con `shopt -s failglob`, da error.

```bash
shopt -s nullglob
archivos=( *.log )                 # array vacío si no hay ningún .log
echo "Hay ${#archivos[@]} logs"
```

## Errores típicos

| Error | Causa |
|---|---|
| `${s:-5}` no da los 5 últimos | Sin espacio es «valor por defecto»: escribe `${s: -5}` |
| `$(( 7 / 2 ))` da `3` | Aritmética entera: usa `awk` para decimales |
| `bad substitution` | Error de sintaxis dentro de `${ }`, o lo ejecutaste con `sh` |
| `rm *.tmp` borra lo que no querías | Pruébalo antes con `echo *.tmp` |

## Relacionadas

[[Variables y comillas en Bash]] · [[Arrays en Bash]] · [[Bucles en Bash]] · [[Condiciones en Bash]] · [[grep, sed y awk lo justo]] · [[Bash - Chuleta de sintaxis]]

## Referencias

- [GNU Bash Manual: Shell Expansions](https://www.gnu.org/software/bash/manual/bash.html#Shell-Expansions) — orden de las expansiones.
- [GNU Bash Manual: Shell Parameter Expansion](https://www.gnu.org/software/bash/manual/bash.html#Shell-Parameter-Expansion) — `#`, `%`, `/`, `:`, `^^`, `,,`, `:-`, `:=`, `:?`.
- [GNU Bash Manual: Filename Expansion](https://www.gnu.org/software/bash/manual/bash.html#Filename-Expansion) — globs, `nullglob`, `failglob`.
- [GNU Bash Manual: Shell Arithmetic](https://www.gnu.org/software/bash/manual/bash.html#Shell-Arithmetic) — operadores de `$(( ))`.
