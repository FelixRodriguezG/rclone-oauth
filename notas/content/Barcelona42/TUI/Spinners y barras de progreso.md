---
tags: [tui, animacion, bash, python, curses, progreso]
aliases: ["Spinner", "Barra de progreso", "Progress bar"]
---
# Spinners y barras de progreso

[[Inicio|← Inicio]] · [[TUI - Índice|TUI]]

> Cuando algo tarda más de un segundo, el usuario necesita saber que el programa **sigue vivo**. Un **spinner** dice «estoy trabajando» (no sé cuánto falta); una **barra de progreso** dice «llevo el 60 %». Aquí están los dos en Bash, en Python sin curses y en curses. Las bases (`\r`, ocultar cursor) están en [[Animación en la terminal - fundamentos]].

## Vocabulario

| Término | Qué significa |
|---|---|
| **Progreso indeterminado** | No sabes cuánto falta (descarga sin tamaño, `make`): usa un **spinner** |
| **Progreso real** (determinado) | Sabes el total (5 ficheros, 200 MB): usa una **barra** con porcentaje |
| **ETA** | *Estimated Time of Arrival*: tiempo estimado que falta |
| **`$!`** | PID del último proceso lanzado en segundo plano con `&` (ver [[Subshells y procesos en Bash]]) |
| **`kill -0 PID`** | No mata nada: solo comprueba si el proceso **sigue existiendo** |

## Spinner en Bash

La idea: lanzar el trabajo con `&`, y mientras siga vivo, dibujar un fotograma cada 0,1 s.

```bash
#!/usr/bin/env bash
set -euo pipefail

spinner() {
    local pid="$1" msg="$2"
    local frames=(⠋ ⠙ ⠹ ⠸ ⠼ ⠴ ⠦ ⠧ ⠇ ⠏)       # o: (\| / - \\)
    local i=0
    printf '\033[?25l'                          # ocultar cursor
    while kill -0 "$pid" 2>/dev/null; do        # ¿sigue vivo?
        printf '\r%s %s' "${frames[i]}" "$msg"
        i=$(( (i + 1) % ${#frames[@]} ))
        sleep 0.1
    done
    printf '\r\033[K\033[?25h'                  # borrar línea y mostrar cursor
}

trap 'printf "\033[?25h"' EXIT                  # por si algo falla

sleep 2 &                                       # el trabajo largo
spinner "$!" "Descargando…"
if wait "$!"; then                              # wait recoge su código de salida
    echo "✔ Descarga terminada"
else
    echo "✘ Falló la descarga" >&2
fi
```

> [!tip] Usa un array para los fotogramas
> `${cadena:i:1}` cuenta **bytes** si el *locale* no es UTF-8, y `⠋` ocupa 3 bytes: saldría basura. Con un array cada elemento es un fotograma completo. Más en [[Unicode y emojis en la terminal]].

## Barra de progreso en Bash

```bash
barra() {                                       # barra ACTUAL TOTAL [ANCHO]
    local actual="$1" total="$2" ancho="${3:-30}"
    local pct=$(( actual * 100 / total ))
    local llenos=$(( actual * ancho / total ))
    local hechos resto
    printf -v hechos '%*s' "$llenos" ''         # N espacios en la variable…
    printf -v resto '%*s' "$(( ancho - llenos ))" ''
    printf '\r[%s%s] %3d%%' "${hechos// /#}" "${resto// /-}" "$pct"   # …cambiados por # y -
}

archivos=(a.txt b.txt c.txt d.txt e.txt)
for i in "${!archivos[@]}"; do
    sleep 0.2                                   # aquí, el trabajo real
    barra "$(( i + 1 ))" "${#archivos[@]}"
done
printf '\n'
```

Salida final: `[##############################] 100%`. El truco `printf -v` + `${var// /#}` está explicado en [[Expansiones en Bash]].

## En Python, sin curses

```python
import itertools
import sys
import threading
import time


def con_spinner(mensaje: str, trabajo: threading.Thread) -> None:
    trabajo.start()
    for car in itertools.cycle("⠋⠙⠹⠸⠼⠴⠦⠧⠇⠏"):
        if not trabajo.is_alive():
            break
        sys.stdout.write(f"\r{car} {mensaje}")
        sys.stdout.flush()
        time.sleep(0.1)
    sys.stdout.write("\r\033[K")
    print(f"✔ {mensaje}")


def barra(actual: int, total: int, inicio: float, ancho: int = 30) -> None:
    fraccion = actual / total
    llenos = int(fraccion * ancho)
    pasado = time.monotonic() - inicio
    eta = pasado / fraccion - pasado            # regla de tres con lo que llevas
    sys.stdout.write(f"\r[{'█' * llenos}{'░' * (ancho - llenos)}] "
                     f"{fraccion:4.0%}  {actual}/{total}  ETA {eta:4.1f}s")
    sys.stdout.flush()


con_spinner("Preparando", threading.Thread(target=time.sleep, args=(1,)))
inicio = time.monotonic()
for i in range(1, 21):
    time.sleep(0.05)
    barra(i, 20, inicio)
print()
```

Salida final: `[██████████████████████████████] 100%  20/20  ETA  0.0s`.

## En curses

Dentro de una TUI no se usa `\r`: se escribe en una posición fija y se llama a `refresh()`. El par de color `1` hay que crearlo antes con `curses.init_pair` (ver [[curses - Colores y estilos]]).

```python
def barra(win: curses.window, y: int, x: int, ancho: int, fraccion: float) -> None:
    llenos = int(fraccion * ancho)
    win.addstr(y, x, "█" * llenos, curses.color_pair(1))      # parte llena en color
    win.addstr(y, x + llenos, "░" * (ancho - llenos))
    win.addstr(y, x + ancho + 1, f"{fraccion:4.0%}")
```

Para que la barra avance **sin congelar el teclado**, el trabajo va en un hilo y el bucle redibuja cada 100 ms: ver [[curses - Animaciones]] y [[curses - Ejecutar comandos]].

## Qué mostrar

| Situación | Muestra |
|---|---|
| < 1 s | Nada |
| Duración desconocida | Spinner + qué está haciendo («Descargando paquetes…») |
| Total conocido, pasos rápidos | Barra + porcentaje |
| Total conocido, > 10 s | Barra + porcentaje + `actual/total` + **ETA** |
| Al terminar | Borrar la línea y dejar un resumen (`✔ 5 ficheros en 1,2 s`) |

> [!warning] La ETA miente al principio
> Con el 2 % hecho, la estimación salta mucho. Muéstrala a partir de un 5-10 % o tras unos segundos.

## Errores típicos

| Error | Consecuencia |
|---|---|
| Spinner con `while true` sin `sleep` | 100 % de CPU |
| No hacer `wait "$pid"` | Pierdes el código de salida del trabajo |
| Barra cuando la salida va a un fichero | Fichero lleno de `\r`: comprueba `[[ -t 1 ]]` |
| Dividir con `total=0` | Error de división por cero; trata el caso vacío antes |

## Relacionadas

[[Animación en la terminal - fundamentos]] · [[curses - Colores y estilos]] · [[Subshells y procesos en Bash]] · [[Expansiones en Bash]] · [[Arrays en Bash]] · [[curses - Animaciones]] · [[curses - Ejecutar comandos]] · [[TUIs en Bash - dialog, whiptail y gum]] · [[Instalador TUI en Bash]]

## Referencias

- [Bash manual: Job Control](https://www.gnu.org/software/bash/manual/bash.html#Job-Control-Basics) — `&`, `$!`, `wait`.
- [kill(1)](https://man7.org/linux/man-pages/man1/kill.1.html) — señal 0 para comprobar si existe un proceso.
- [Bash manual: printf](https://www.gnu.org/software/bash/manual/bash.html#index-printf) — `printf -v`, `%*s`.
- [Python threading](https://docs.python.org/3/library/threading.html) — `Thread`, `is_alive()`.
- [Python curses](https://docs.python.org/3/library/curses.html) — `addstr`, `color_pair`, `use_default_colors`.
- Ejemplos ejecutados con Bash 5.1 y Python 3.13; la barra de curses, en tmux.
