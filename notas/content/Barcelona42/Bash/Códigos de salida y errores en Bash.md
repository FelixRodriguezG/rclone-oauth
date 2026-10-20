---
tags: [bash, errores, exit, codigos-de-salida, stderr]
aliases: ["Exit status", "$? en Bash", "die en Bash", "command -v"]
---
# Códigos de salida y errores en Bash

[[Inicio|← Inicio]] · [[Bash - Índice|Bash]]

> Cada comando, al terminar, devuelve un número: su **código de salida**. `0` significa «todo bien» y cualquier otro valor «algo falló». Un buen script comprueba esos códigos, avisa del problema **por `stderr`** con un mensaje claro y termina con el código adecuado.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Código de salida** (*exit status*) | Número de 0 a 255 que deja un comando al acabar; es lo que en C devuelve `main` o pasa `exit()` |
| **`$?`** | Variable con el código del **último** comando ejecutado |
| **`stderr`** | Salida de errores (descriptor 2); ver [[Redirecciones y tuberías]] |
| **`die`** | Nombre habitual de una función que imprime un error y sale |
| **Requisito** | Programa externo que tu script necesita (`git`, `curl`…) |

## `$?` y `exit`

```bash
ls /noexiste
echo "$?"            # 2   (ls usa 2 para «problema grave»)
grep -q hola /dev/null
echo "$?"            # 1   (grep: no encontró nada)
true; echo "$?"      # 0
```

> [!warning] `$?` cambia con cada comando
> Hasta un `echo` lo sobrescribe. Si lo necesitas más tarde, guárdalo justo después: `cmd; rc=$?`.

Para terminar el script con un código concreto:

```bash
exit 0      # éxito
exit 1      # error
exit        # sin número: sale con el código del último comando
```

Dentro de una función se usa `return N` (ver [[Funciones en Bash]]); `exit` cerraría **todo** el script.

## Convenciones de códigos

| Código | Significado habitual |
|---|---|
| `0` | Éxito |
| `1` | Error general |
| `2` | Mal uso: argumento o comando incorrecto (ver [[Argumentos y subcomandos en Bash]]) |
| `126` | El archivo existe pero no es ejecutable (falta `chmod +x`) |
| `127` | Comando no encontrado |
| `128 + N` | Muerto por la señal N: `130` = Ctrl+C (`INT`, 2), `143` = `kill` (`TERM`, 15) |

Solo hay 8 bits: `exit 300` sale con `300 % 256 = 44`. Quédate entre 0 y 125 para tus propios códigos.

## Reaccionar a un fallo

No hace falta mirar `$?` a mano: `if`, `&&` y `||` ya miran el código (ver [[Condiciones en Bash]]).

```bash
if cp datos.txt copia.txt; then
    echo "copiado"
fi

mkdir -p "$dir" && echo "listo"                     # solo si mkdir fue bien
cd "$dir" || { echo "no puedo entrar en $dir" >&2; exit 1; }
```

En el último, las llaves `{ ...; }` agrupan varios comandos sin crear un proceso nuevo (con `( )` el `exit` solo saldría de la subshell, ver [[Subshells y procesos en Bash]]). Fíjate en el `;` antes de `}`: es obligatorio.

## La función `die`

Repetir `{ echo ... >&2; exit 1; }` cansa. Se encapsula en una función:

```bash
die() {
    echo "${0##*/}: error: $*" >&2
    exit 1
}

[[ -r "$archivo" ]] || die "no puedo leer '$archivo'"
cp "$archivo" "$archivo.bak" || die "no se pudo hacer la copia"
```

```text
copia.sh: error: no puedo leer 'notas.txt'
```

Un buen mensaje de error dice **quién** habla (el nombre del script), **qué** falló y, si se puede, **qué hacer**. Compara con [[Control de errores en C]]: es la misma idea que `perror` + `exit(EXIT_FAILURE)`.

## Comprobar requisitos con `command -v`

`command -v nombre` imprime la ruta del programa y sale con 0 si existe, o sale con 1 si no. Es la forma portable de comprobarlo (mejor que `which`):

```bash
requiere() {
    local cmd
    for cmd in "$@"; do
        command -v "$cmd" > /dev/null \
            || die "falta '$cmd'; instálalo y vuelve a probar"
    done
}

requiere git curl
```

Ponlo **al principio** del script: es mejor fallar antes de empezar que a mitad de una tarea.

> [!tip] Orden recomendado al arrancar
> 1. `set -euo pipefail` (ver [[Scripts robustos con set y trap]]).
> 2. Definir `die` y demás funciones.
> 3. Comprobar requisitos y argumentos.
> 4. Hacer el trabajo.

## Mensajes claros

| Mal | Bien |
|---|---|
| `Error` | `tareas.sh: error: no existe '/home/ana/.tareas.txt'` |
| `echo "fallo"` (a `stdout`) | `echo "..." >&2` (a `stderr`) |
| Seguir tras el fallo | Salir con un código ≠ 0 |
| `exit 0` al final aunque algo fallara | Que el código refleje el resultado real |

Los mensajes a `stderr` no se mezclan con los datos: `./tareas.sh lista > copia.txt` guarda las tareas y los errores siguen saliendo en pantalla.

## Errores típicos

| Error | Qué pasa |
|---|---|
| `if [ $? = 0 ]` después de un `echo` | Mira el código del `echo`, no el del comando |
| `cmd \|\| echo "error"; exit 1` | El `exit 1` se ejecuta **siempre**; agrupa con `{ ...; }` |
| `( echo "error"; exit 1 )` | Solo sale de la subshell, el script sigue |
| Usar `exit` dentro de una función de librería | Cierra el script entero; usa `return` |
| Errores por `stdout` | Rompen tuberías y redirecciones |

Siguiente: [[Expansiones en Bash]].

## Relacionadas

[[Scripts robustos con set y trap]] · [[Redirecciones y tuberías]] · [[Expansiones en Bash]] · [[Condiciones en Bash]] · [[Funciones en Bash]] · [[Argumentos y subcomandos en Bash]] · [[Control de errores en C]] · [[Depurar scripts de Bash]] · [[Bash - Chuleta de sintaxis]]

## Referencias

- [GNU Bash Manual: Exit Status](https://www.gnu.org/software/bash/manual/html_node/Exit-Status.html) — 0 éxito, 126, 127, 128+N.
- [GNU Bash Manual: Bash Builtins (`command`)](https://www.gnu.org/software/bash/manual/html_node/Bash-Builtins.html) — `command -v`.
- [Exit codes with special meanings (TLDP)](https://tldp.org/LDP/abs/html/exitcodes.html) — tabla de códigos reservados.
- `man 7 signal` — números de señal (`INT` = 2, `TERM` = 15).
- Ejemplos probados con Bash 5.1.16 en Ubuntu 22.04.
