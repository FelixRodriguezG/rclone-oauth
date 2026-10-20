---
tags: [tui, curses, python, bucle, estado]
aliases: ["Bucle de eventos curses", "Main loop", "Clase App curses"]
---
# curses - El bucle principal

[[Inicio|← Inicio]] · [[TUI - Índice|TUI]]

> Una TUI es un bucle que repite siempre lo mismo: **dibujar el estado → leer una tecla → actualizar el estado**. Si mantienes esos tres pasos separados, el programa se puede leer y ampliar sin enredarse. En esta etapa `tareas.py` ya tiene una lista con selección que se mueve con `j`/`k` y un reloj que avanza solo.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Estado** | Todos los datos que deciden lo que se ve: tareas, cuál está seleccionada, si hay que salir |
| **Bucle principal** (*main loop*) | El `while` que repite dibujar → leer → actualizar hasta que el usuario sale |
| **Bloqueante** | Una función que se queda esperando: `getch()` espera una tecla para siempre si no le pones límite |
| **`timeout`** | Tiempo máximo que `getch()` espera. Si se cumple, devuelve `-1` y el bucle sigue |
| **Parpadeo** (*flicker*) | La pantalla se queda en blanco un instante entre fotograma y fotograma |

## El patrón

```text
            ┌──────────────────────────────┐
            ▼                              │
   1. dibujar(estado)   ← solo LEE el estado
            │
   2. tecla = getch()   ← espera como mucho N ms
            │
   3. manejar(tecla)    ← solo CAMBIA el estado ─┘
```

Regla: **dibujar no cambia nada** y **manejar no dibuja nada**. Así, si algo sale mal en pantalla, el fallo está en `dibujar`; si el programa hace algo raro, está en `manejar`.

## Una clase `App` sencilla

```python
import curses
import time


class App:
    def __init__(self, stdscr: curses.window) -> None:
        self.stdscr = stdscr
        self.tareas = ["comprar pan", "entregar libft", "repasar punteros"]
        self.sel = 0                      # índice de la tarea seleccionada
        self.inicio = time.monotonic()    # para el cronómetro
        self.seguir = True

    def dibujar(self) -> None:
        self.stdscr.erase()               # borra el búfer, no la pantalla
        segundos = int(time.monotonic() - self.inicio)
        self.stdscr.addstr(0, 0, f"tareas.py  ·  abierta hace {segundos} s")
        for i, texto in enumerate(self.tareas):
            attr = curses.A_REVERSE if i == self.sel else curses.A_NORMAL
            self.stdscr.addstr(2 + i, 2, texto, attr)
        self.stdscr.addstr(6, 0, "↑/↓ o j/k mover · q salir", curses.A_DIM)
        self.stdscr.refresh()

    def manejar(self, tecla: int) -> None:
        if tecla in (curses.KEY_UP, ord("k")):
            self.sel = max(0, self.sel - 1)
        elif tecla in (curses.KEY_DOWN, ord("j")):
            self.sel = min(len(self.tareas) - 1, self.sel + 1)
        elif tecla == ord("q"):
            self.seguir = False

    def run(self) -> None:
        curses.curs_set(0)
        self.stdscr.keypad(True)          # flechas → curses.KEY_UP, etc.
        self.stdscr.timeout(250)          # getch espera como mucho 250 ms
        while self.seguir:
            self.dibujar()                # 1. estado → pantalla
            tecla = self.stdscr.getch()   # 2. leer (-1 si no hubo tecla)
            if tecla != -1:
                self.manejar(tecla)       # 3. actualizar el estado


def main(stdscr: curses.window) -> None:
    App(stdscr).run()


if __name__ == "__main__":
    try:
        curses.wrapper(main)
    except KeyboardInterrupt:             # Ctrl+C: salir sin traza
        pass
```

Tras pulsar `↓` una vez (la fila invertida es la seleccionada):

```text
tareas.py  ·  abierta hace 3 s

  comprar pan
  entregar libft      ← en vídeo inverso
  repasar punteros

↑/↓ o j/k mover · q salir
```

- `ord("k")` convierte el carácter en su código (`107`), porque `getch()` devuelve enteros.
- `max`/`min` evitan que la selección se salga de la lista.
- Los datos vienen todavía de una lista fija. En [[curses - Ejecutar comandos]] los leeremos de `tareas.sh` con el `modelo.py` de [[De script a TUI - arquitectura]].

## `erase()` frente a `clear()`

| Método | Qué hace | Úsalo |
|---|---|---|
| `erase()` | Vacía el **búfer**. En el `refresh()` solo se envían las celdas que cambiaron | En **cada** fotograma |
| `clear()` | Vacía el búfer **y** obliga a repintar toda la pantalla desde cero en el siguiente `refresh()` | Solo si la pantalla se ha «ensuciado» (p. ej. al volver de `vim`) |

Con `clear()` en cada vuelta verás **parpadeo**: la terminal borra todo y vuelve a escribirlo. Con `erase()` el resultado es el mismo pero sin parpadeo.

## Refrescar solo: `timeout` y `halfdelay`

Sin límite, `getch()` se queda esperando y el reloj de la cabecera no avanzaría hasta pulsar algo. Opciones:

| Llamada | Efecto |
|---|---|
| `stdscr.timeout(-1)` | Espera para siempre (por defecto) |
| `stdscr.timeout(250)` | Espera como mucho 250 ms; si no hay tecla, devuelve `-1` |
| `stdscr.timeout(0)` / `nodelay(True)` | No espera nada. ¡Cuidado: el bucle gira sin parar y gasta 100 % de CPU! |
| `curses.halfdelay(5)` | Igual que `timeout`, pero en décimas de segundo (5 = 0,5 s) y para todas las ventanas |

Elige el `timeout` según lo que necesites: 1000 ms para un reloj de segundos, 100 ms para un spinner (ver [[curses - Animaciones]] y [[Spinners y barras de progreso]]). Más corto = más fluido pero más CPU.

## Salir limpiamente

- **Una sola puerta**: `self.seguir = False` y el `while` termina. Evita `sys.exit()` desde dentro de `manejar`: funciona (el `wrapper` restaura la terminal), pero te salta el código de después del bucle (guardar, cerrar procesos).
- **Ctrl+C** lanza `KeyboardInterrupt`. `wrapper` restaura la terminal y relanza la excepción; el `try` de fuera la captura para que no salga un traceback. Probado: termina con código 0 y sin traza.
- Si hay datos sin guardar, pregunta antes de salir (ver [[Diseño de una TUI moderna]]).

> [!tip] Para depurar, no uses `print`
> `print` escribe encima de la interfaz y la descoloca. Escribe a un archivo (`logging.basicConfig(filename="debug.log")`) y míralo en otro panel con `tail -f debug.log` (ver [[tmux - Uso básico]]).

## Errores típicos

| Error | Síntoma |
|---|---|
| `clear()` en cada vuelta | Parpadeo |
| `timeout(0)` sin pausa | El ventilador se dispara: bucle al 100 % de CPU |
| Dibujar dentro de `manejar` | Lo dibujado desaparece en el siguiente `erase()` |
| Comparar `tecla == "q"` con `getch()` | Nunca es cierto: `getch()` da `int`; usa `ord("q")` |
| No tratar `-1` | Con `timeout`, cada vuelta sin tecla se procesa como una tecla rara |

## Relacionadas

[[curses - Primeros pasos]] · [[curses - Layout y paneles]] · [[curses - Teclado y atajos]] · [[curses - Ejecutar comandos]] · [[curses - Animaciones]] · [[Animación en la terminal - fundamentos]] · [[De script a TUI - arquitectura]]

## Referencias

- [curses — Python docs](https://docs.python.org/3/library/curses.html) — `window.timeout`, `nodelay`, `halfdelay`, `erase`, `clear`, `getch`.
- [Curses Programming with Python (HOWTO): User Input](https://docs.python.org/3/howto/curses.html#user-input) — `getch` con y sin espera.
- [ncurses: curs_clear(3x)](https://invisible-island.net/ncurses/man/curs_clear.3x.html) — diferencia entre `erase` y `clear` (repintado completo).
- [time.monotonic](https://docs.python.org/3/library/time.html#time.monotonic) — reloj que no salta si cambia la hora del sistema.
- Probado con Python 3.13 en tmux 3.2a: selección con `↓`, cabecera que avanza sola y salida con `q` y con Ctrl+C.
