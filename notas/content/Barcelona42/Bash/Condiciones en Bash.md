---
tags: [bash, condiciones, if, test, case]
aliases: ["if en Bash", "test en Bash", "[[ ]]", "case en Bash"]
---
# Condiciones en Bash

[[Inicio|← Inicio]] · [[Bash - Índice|Bash]]

> Las **condiciones** deciden qué parte del script se ejecuta. En Bash, `if` no mira un valor verdadero o falso como en C: mira el **código de salida** de un comando (`0` = éxito = verdadero). `[[ ]]` es el comando que se usa para comparar textos, números y comprobar archivos.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Código de salida** | Número que devuelve un comando al terminar: `0` bien, otro valor mal (ver [[Códigos de salida y errores en Bash]]) |
| **Test** | Comando que comprueba algo y sale con `0` o `1`: `[ ]`, `[[ ]]`, `(( ))` |
| **Operador** | Símbolo de la comprobación: `-f`, `==`, `-lt`… |
| **Expresión regular** | Patrón para buscar texto (`=~`) |

## `if`, `elif`, `else`

```bash
if comando; then
    echo "el comando salió con 0"
elif otro_comando; then
    echo "el segundo salió con 0"
else
    echo "ninguno"
fi                       # fi = if al revés: cierra el bloque
```

Cualquier comando vale como condición:

```bash
if grep -q "TODO" notas.txt; then     # -q: no imprime, solo devuelve 0 o 1
    echo "Quedan cosas por hacer"
fi
```

## `[ ]` frente a `[[ ]]`

| | `[ ]` (o `test`) | `[[ ]]` |
|---|---|---|
| Qué es | Un comando normal (POSIX) | Palabra clave de Bash |
| Variables sin comillas | Se rompen con espacios o vacías | Seguras (aun así, pon comillas por costumbre) |
| `&&`, `\|\|` dentro | No (usa `-a`, `-o`, desaconsejados) | Sí |
| Patrones `==` y `=~` | No | Sí |

> [!tip] Usa `[[ ]]` en Bash
> Solo usa `[ ]` si escribes para `sh`. Y respeta los **espacios**: `[[ "$a" == "b" ]]`. Sin ellos (`[["$a"=="b"]]`) es un error de sintaxis.

## Comprobar archivos

| Operador | Verdadero si… |
|---|---|
| `-e ruta` | Existe (lo que sea) |
| `-f ruta` | Existe y es un archivo normal |
| `-d ruta` | Existe y es una carpeta |
| `-r` / `-w` / `-x` | Se puede leer / escribir / ejecutar |
| `-s ruta` | Existe y no está vacío |

```bash
if [[ ! -d "$HOME/proyectos" ]]; then     # ! = negación
    mkdir -p "$HOME/proyectos"
fi
```

## Comparar cadenas

| Operador | Verdadero si… |
|---|---|
| `-z "$s"` | La cadena está vacía |
| `-n "$s"` | No está vacía |
| `"$a" == "$b"` / `!=` | Iguales / distintas |
| `"$s" == *.txt` | Encaja con el patrón (sin comillas en el patrón) |
| `"$s" =~ ^[0-9]+$` | Encaja con la expresión regular |

```bash
respuesta="42"
if [[ "$respuesta" =~ ^[0-9]+$ ]]; then
    echo "Es un número"
fi
```

> [!warning] El patrón va sin comillas
> `[[ "$f" == "*.txt" ]]` compara con el texto literal `*.txt`. Para que `*` funcione como comodín escribe `[[ "$f" == *.txt ]]`. Lo mismo con `=~`: guarda la regex en una variable y escribe `[[ "$s" =~ $re ]]`.

## Comparar números

Dentro de `[[ ]]`, `<` y `>` comparan **texto** (orden alfabético). Para números:

| `[[ ]]` | `(( ))` | Significado |
|---|---|---|
| `-eq` | `==` | Igual |
| `-ne` | `!=` | Distinto |
| `-lt` / `-le` | `<` / `<=` | Menor / menor o igual |
| `-gt` / `-ge` | `>` / `>=` | Mayor / mayor o igual |

```bash
n=7
if [[ "$n" -gt 5 ]]; then echo "mayor que 5"; fi
if (( n > 5 )); then echo "también"; fi     # (( )): aritmética, como en C
```

`(( ))` sale con `0` si el resultado no es cero (igual que un `if` en C). Más aritmética en [[Expansiones en Bash]].

## `&&` y `||` como atajos

- `a && b` → ejecuta `b` solo si `a` salió bien.
- `a || b` → ejecuta `b` solo si `a` falló.

```bash
[[ -f config.txt ]] && echo "hay configuración"
cd "$carpeta" || exit 1           # si no puede entrar, termina
```

> [!warning] `a && b || c` no es un if/else
> Si `b` falla, también se ejecuta `c`. Para dos caminos usa un `if` de verdad.

## `case`: muchas opciones sobre un valor

```bash
case "$1" in
    añadir)          echo "Añadir" ;;
    lista)           echo "Listar" ;;
    help|-h|--help)  echo "Ayuda" ;;               # a|b: varias opciones, misma rama
    *.txt)           echo "Un archivo de texto" ;;  # patrones como en [[ == ]]
    *)               echo "Opción desconocida: $1" ;;     # * = cualquier otra cosa
esac
```

Cada rama termina en `;;`. Se usa mucho para subcomandos (ver [[Argumentos y subcomandos en Bash]]).

## `tareas.sh` versión 2

```bash
#!/usr/bin/env bash
readonly ARCHIVO="${TAREAS_ARCHIVO:-$HOME/.tareas.txt}"

echo "=== Mis tareas ==="
if [[ ! -f "$ARCHIVO" ]]; then
    echo "Todavía no hay tareas."
elif [[ ! -s "$ARCHIVO" ]]; then
    echo "La lista está vacía."
else
    cat "$ARCHIVO"
fi
```

Siguiente: numerar las tareas con un bucle → [[Bucles en Bash]].

## Errores típicos

| Error | Causa |
|---|---|
| `[[: command not found` o `syntax error` | Faltan espacios dentro de `[[ ]]` |
| `[: too many arguments` | Usaste `[ $var ]` con una variable con espacios |
| `10 < 9` da verdadero | `<` en `[[ ]]` compara texto: usa `-lt` o `(( ))` |
| `if [[ $x = 1 ]]` funciona pero `if (( x = 1 ))` siempre es verdad | En `(( ))`, `=` **asigna**; compara con `==` |

## Relacionadas

[[Variables y comillas en Bash]] · [[Bucles en Bash]] · [[Códigos de salida y errores en Bash]] · [[Expansiones en Bash]] · [[Argumentos y subcomandos en Bash]] · [[Bash - Chuleta de sintaxis]]

## Referencias

- [GNU Bash Manual: Conditional Constructs](https://www.gnu.org/software/bash/manual/bash.html#Conditional-Constructs) — `if`, `case`, `[[ ]]`, `=~`.
- [GNU Bash Manual: Bash Conditional Expressions](https://www.gnu.org/software/bash/manual/bash.html#Bash-Conditional-Expressions) — tabla de `-f`, `-d`, `-z`, `-eq`…
- [Greg's Wiki: BashFAQ/031](https://mywiki.wooledge.org/BashFAQ/031) — diferencias entre `test`, `[` y `[[`.
