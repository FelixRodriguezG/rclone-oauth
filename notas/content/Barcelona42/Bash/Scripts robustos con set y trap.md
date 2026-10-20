---
tags: [bash, set, trap, errores, scripting]
aliases: ["set -euo pipefail", "Modo estricto de Bash", "trap en Bash"]
---
# Scripts robustos con set y trap

[[Inicio|← Inicio]] · [[Bash - Índice|Bash]]

> Por defecto, Bash **sigue ejecutando** aunque un comando falle, y una variable mal escrita vale `""` sin avisar. Con `set -euo pipefail` el script se para en cuanto algo va mal, y con `trap` puedes **limpiar** (borrar temporales, restaurar la terminal) pase lo que pase al salir.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Opción de shell** | Interruptor que cambia cómo se comporta Bash; se activa con `set -x` y se desactiva con `set +x` |
| **Señal** (*signal*) | Aviso que el sistema manda a un proceso: `INT` (Ctrl+C), `TERM` (`kill`), `HUP` (se cierra la terminal) |
| **`trap`** | Orden «cuando pase X, ejecuta este código» |
| **Pseudo-señal** | Evento propio de Bash que `trap` entiende como una señal: `EXIT`, `ERR` |
| **Archivo temporal** | Archivo de usar y tirar; se crea con `mktemp` con un nombre único |

## La cabecera recomendada

```bash
#!/usr/bin/env bash
set -euo pipefail
```

| Opción | Efecto | Sin ella… |
|---|---|---|
| `-e` (*errexit*) | Si un comando falla (sale con ≠ 0), el script termina | Sigue como si nada |
| `-u` (*nounset*) | Usar una variable no definida es un error | `$NOMBER` vale `""` en silencio |
| `-o pipefail` | Una tubería falla si falla **cualquiera** de sus comandos | Solo cuenta el **último** |

```bash
false | true; echo $?                  # 0: sin pipefail solo cuenta 'true'
set -o pipefail
false | true; echo $?                  # 1
```

Con `-u`, si una variable **puede** faltar, dale un valor por defecto (ver [[Expansiones en Bash]]):

```bash
set -u
echo "${EDITOR:-vim}"                  # vale 'vim' si EDITOR no existe
archivo="${1:-}"                       # $1 vacío en vez de error si no hay argumentos
```

## Las trampas de `set -e`

`-e` **no** salta en estos casos, porque Bash entiende que estás *comprobando* el resultado:

- La condición de un `if`, `while` o `until`.
- Cualquier comando de una lista `&&` / `||` salvo el último.
- Un comando precedido de `!`.
- **Todo lo que se ejecuta dentro** de una función llamada desde un `if` o un `&&`.

```bash
set -e
falla() {
    false                              # aquí -e NO salta…
    echo "sigo dentro de falla"        # …y esto se ejecuta
}
if falla; then echo "verdadero"; fi    # imprime las dos líneas
```

Otra sorpresa: `local x="$(cmd)"` **oculta** el fallo de `cmd`, porque el código de salida es el de `local` (siempre 0). Sepáralo:

```bash
local x
x="$(cmd)"                             # ahora -e sí ve el fallo
```

> [!warning] `set -e` no sustituye a comprobar errores
> Es una red de seguridad, no un control de errores. Para los fallos que esperas (archivo que no existe, comando que falta), compruébalos tú y da un mensaje claro: ver [[Códigos de salida y errores en Bash]].

> [!note] Comandos que «fallan» sin error
> `grep` sale con `1` si **no encuentra** nada y `((n++))` con `n=0` sale con `1`. Con `-e`, eso mata el script. Usa `grep ... || true` o `n=$((n + 1))`.

## `trap`: ejecutar algo al salir

```bash
trap 'código' SEÑAL [SEÑAL...]
```

| Evento | Cuándo se ejecuta |
|---|---|
| `EXIT` | Al terminar el script, **por cualquier motivo** (fin normal, `exit`, error con `-e`) |
| `INT` | Al pulsar Ctrl+C |
| `TERM` | Cuando otro proceso hace `kill PID` |
| `ERR` | Cada vez que un comando falla (como `-e`, mismas excepciones) |

### Limpiar archivos temporales

```bash
#!/usr/bin/env bash
set -euo pipefail

tmp="$(mktemp)"                        # p. ej. /tmp/tmp.xgoRO36sTE

limpiar() {
    rm -f "$tmp"
    echo "limpieza hecha" >&2
}
trap limpiar EXIT
trap 'echo "error en la línea $LINENO" >&2' ERR

echo "datos" > "$tmp"
grep -q "nada" "$tmp"                  # falla: no encuentra 'nada'
echo "no llego aquí"
```

Salida:

```text
error en la línea 14
limpieza hecha
```

`mktemp -d` crea un **directorio** temporal (bórralo con `rm -rf "$dir"`). Nunca inventes nombres como `/tmp/mio.txt`: dos ejecuciones a la vez se pisarían.

### Ctrl+C y restaurar la terminal

Si el script oculta el cursor o cambia la terminal (ver [[Cómo funciona la terminal]]), tiene que dejarla bien aunque el usuario lo corte:

```bash
restaurar() {
    tput cnorm                         # vuelve a mostrar el cursor
    stty echo                          # vuelve a mostrar lo que se escribe
}
trap restaurar EXIT
trap 'echo " cancelado" >&2; exit 130' INT    # 130 = 128 + 2 (SIGINT)

tput civis                             # oculta el cursor
stty -echo
```

Al hacer `exit 130` dentro del `trap INT`, también se ejecuta el `trap EXIT`: la limpieza queda **en un único sitio**.

> [!tip] Patrón
> Pon toda la limpieza en una función y engánchala a `EXIT`. Las demás señales solo deciden con qué código salir.

## `tareas.sh` en modo estricto

Añade `set -euo pipefail` justo debajo del *shebang* de la versión de [[Argumentos y subcomandos en Bash]]. Funciona sin más cambios porque ya está preparada:

- Comprueba `$#` antes de leer `$1`, y usa `${1:-}` en `borrar`: con `-u`, leer un `$1` que no existe sería un error.
- Usa `if (( VERBOSE ))` y no `(( VERBOSE )) && ...`: con `VERBOSE=0`, `(( ))` sale con 1; como es la última orden de `anadir`, la función devolvería 1 y `-e` cortaría el script.

Si un día `borrar` reescribe el archivo con un temporal (ver [[Configuración y estado de un script]]), engancha `rm -f "$tmp"` a `trap ... EXIT` como arriba.

## Errores típicos

| Error | Qué pasa |
|---|---|
| Confiar en `-e` dentro de funciones usadas en `if` | No se para; ver la lista de excepciones |
| `trap "rm $tmp" EXIT` con comillas dobles | `$tmp` se expande **al definir** el trap; usa simples o una función |
| Poner el `trap` después de crear el temporal… y fallar entre medias | Crea el temporal y pon el `trap` justo después |
| `set -u` y `$1` sin argumentos | `unbound variable`; usa `${1:-}` |
| Activar `set -e` en tu terminal interactiva | Cualquier error **cierra la terminal**: úsalo solo en scripts |

## Relacionadas

[[Códigos de salida y errores en Bash]] · [[Subshells y procesos en Bash]] · [[Depurar scripts de Bash]] · [[Configuración y estado de un script]] · [[Cómo funciona la terminal]] · [[Bash - Chuleta de sintaxis]]

## Referencias

- [GNU Bash Manual: The Set Builtin](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html) — `-e`, `-u`, `pipefail` y las excepciones de `-e`.
- [GNU Bash Manual: Bourne Shell Builtins (`trap`)](https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html) — `EXIT`, `ERR`, sintaxis de `trap`.
- [BashFAQ/105 (Greg's Wiki)](https://mywiki.wooledge.org/BashFAQ/105) — por qué `set -e` no hace lo que parece.
- `man mktemp`, `man 7 signal` — archivos temporales y números de señal.
- Ejemplos probados con Bash 5.1.16 en Ubuntu 22.04.
