---
tags: [tui, curses, python, animacion]
aliases: ["Animaciones en curses", "curses animation"]
---
# curses - Animaciones

[[Inicio|← Inicio]] · [[TUI - Índice|TUI]]

> En curses, animar es **cambiar el estado un poco en cada vuelta del bucle y volver a dibujar**. La clave es que `getch()` no se quede esperando una tecla para siempre: con `timeout()` el bucle da una vuelta cada pocos milisegundos aunque nadie pulse nada. Parte de [[curses - El bucle principal]] y de [[Animación en la terminal - fundamentos]].

## Vocabulario

| Término | Qué significa |
|---|---|
| **Bloqueante** | `getch()` normal: el programa se para hasta que pulsas una tecla. Nada se mueve |
| **`nodelay(True)`** | `getch()` vuelve **al instante**; si no hay tecla, devuelve `-1` |
| **`timeout(ms)`** | `getch()` espera **como mucho** `ms` milisegundos y devuelve `-1` si no hubo tecla. Sirve de reloj del bucle |
| **Delta time** (`dt`) | Segundos reales desde la vuelta anterior; se multiplica por la velocidad |
| **Interpolar** | Calcular un valor intermedio entre inicio y fin según el tiempo (`t` de 0 a 1) |
| **Easing** (suavizado) | Curva que hace que el movimiento acelere o frene en vez de ir a velocidad constante |

## `nodelay` frente a `timeout`

La tabla completa de modos de espera (`timeout`, `nodelay`, `halfdelay`) está en [[curses - El bucle principal]]. Para animar:

```python
stdscr.nodelay(True)   # getch() nunca espera → necesitas time.sleep() tú, o 100 % CPU
stdscr.timeout(33)     # getch() espera ≤ 33 ms → el propio getch limita a ~30 FPS
```

Usa **`timeout`**: una sola línea da el ritmo del bucle y, si el usuario pulsa una tecla, responde al instante (no espera a que acabe el fotograma).

## El bucle de animación

Cada vuelta hace siempre lo mismo, en este orden:

1. **Entrada**: `getch()` con `timeout` → tecla o `-1`.
2. **Actualizar**: mover cosas según `dt` (no según el número de vueltas).
3. **Dibujar** el fotograma completo: `erase()`, `addstr()`…, `refresh()`.

## Ejemplo: pelota que rebota y barra que se llena

```python
#!/usr/bin/env python3
"""Pelota que rebota + barra que se llena. q: salir, espacio: pausa."""
import curses
import time

FPS = 30
VEL_X, VEL_Y = 20.0, 8.0                   # celdas por segundo


def main(stdscr: curses.window) -> None:
    curses.curs_set(0)                     # ocultar cursor
    stdscr.timeout(1000 // FPS)            # getch espera como mucho 33 ms
    x, y, vx, vy = 1.0, 1.0, VEL_X, VEL_Y
    progreso, pausa = 0.0, False
    anterior = time.monotonic()

    while True:
        tecla = stdscr.getch()             # 1. entrada (no bloquea)
        if tecla == ord("q"):
            break
        if tecla == ord(" "):
            pausa = not pausa

        ahora = time.monotonic()           # 2. actualizar con tiempo real
        dt, anterior = ahora - anterior, ahora
        alto, ancho = stdscr.getmaxyx()
        if not pausa:
            x, y = x + vx * dt, y + vy * dt
            if x < 1 or x > ancho - 2:     # rebote lateral
                vx = -vx
                x = max(1.0, min(x, ancho - 2.0))
            if y < 1 or y > alto - 4:      # rebote vertical (sin pisar la barra)
                vy = -vy
                y = max(1.0, min(y, alto - 4.0))
            progreso = min(1.0, progreso + dt / 5)   # se llena en 5 s

        stdscr.erase()                     # 3. dibujar (borra el búfer, no la pantalla)
        stdscr.box()
        stdscr.addstr(int(y), int(x), "●")
        ancho_barra = ancho - 12
        llenos = int(progreso * ancho_barra)
        stdscr.addstr(alto - 2, 2, "█" * llenos + "░" * (ancho_barra - llenos)
                      + f" {progreso:4.0%}")
        estado = "PAUSA" if pausa else f"{1 / dt:4.0f} FPS" if dt else ""
        stdscr.addstr(0, 2, f" {estado} · q salir · espacio pausa ")
        stdscr.refresh()                   # envía solo lo que cambió: sin parpadeo


if __name__ == "__main__":
    curses.wrapper(main)
```

Captura en una terminal de 60×15 (sin las filas vacías del medio):

```text
┌─   30 FPS · q salir · espacio pausa ─────────────────────┐
│                           ●                              │
│ █████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  28%    │
└──────────────────────────────────────────────────────────┘
```

> [!note] `erase()` y no `clear()`
> En cada fotograma, `erase()`: `clear()` repinta toda la pantalla y parpadea. Explicado en [[curses - El bucle principal]].

## Transiciones sencillas

Una transición es interpolar entre dos estados en un tiempo fijo. El **easing** hace que se vea natural:

```python
def suavizar(t: float) -> float:
    """Ease-out: rápido al principio, frena al final. t va de 0 a 1."""
    return 1 - (1 - t) ** 3
```

- **Aparecer** (máquina de escribir): dibuja `texto[:n]` con `n = int(len(texto) * t)`.
- **Deslizar**: `x = int(inicio_x + (fin_x - inicio_x) * suavizar(t))`.
- **Cambiar color**: a partir de cierto `t`, usa otro `color_pair` (ver [[curses - Colores y estilos]]).

Para que la transición **no bloquee la entrada**, no hagas un bucle aparte con `sleep`: guarda cuándo empezó y calcula `t` dentro del bucle principal.

```python
# en el estado: inicio_anim = time.monotonic() al abrir el panel
t = min(1.0, (time.monotonic() - inicio_anim) / 0.4)   # 0.4 s de duración
x_panel = int(ancho - ancho_panel * suavizar(t))        # entra desde la derecha
```

> [!warning] Animaciones que bloquean
> Una función `deslizar()` con su propio `while` y `time.sleep()` funciona, pero mientras dura **el teclado no responde**. Vale para una intro de medio segundo, no para algo que el usuario pueda querer saltarse.

## Limitar los FPS

- `timeout(1000 // FPS)` ya limita: si no hay teclas, cada vuelta dura ~33 ms.
- Si el usuario mantiene una tecla pulsada, `getch()` vuelve antes y el bucle va más rápido. Con `dt` la velocidad **sigue siendo correcta**; solo dibujas más fotogramas.
- Si no hay nada animándose, sube el `timeout` (p. ej. 250 ms) o vuelve al modo bloqueante: ahorras CPU.

## Errores típicos

| Error | Consecuencia |
|---|---|
| `getch()` sin `timeout`/`nodelay` | La animación solo avanza al pulsar teclas |
| `nodelay(True)` sin `sleep` | 100 % de CPU |
| Mover «1 celda por vuelta» | Va más rápido cuando pulsas teclas |
| `addstr` fuera de la ventana (al encoger la terminal) | `_curses.error`; recorta posiciones con `getmaxyx()` (ver [[curses - Layout y paneles]]) |
| Escribir en la última celda abajo a la derecha | `addstr` lanza error aunque se dibuje |

## Relacionadas

[[curses - El bucle principal]] · [[Animación en la terminal - fundamentos]] · [[Spinners y barras de progreso]] · [[curses - Primeros pasos]] · [[curses - Layout y paneles]] · [[curses - Colores y estilos]] · [[curses - Teclado y atajos]] · [[Diseño de una TUI moderna]] · [[Qué hace un buen instalador]]

## Referencias

- [Python curses: window.timeout / nodelay](https://docs.python.org/3/library/curses.html#curses.window.timeout) — comportamiento de `getch()` con y sin espera.
- [Curses Programming with Python (HOWTO)](https://docs.python.org/3/howto/curses.html) — `erase` frente a `clear`, `refresh`.
- [Easing functions](https://easings.net/) — curvas *ease-out cubic*.
- Ejemplo ejecutado con Python 3.13 en tmux 60×15: rebota a ~30 FPS, pausa con espacio, sale con `q`.
