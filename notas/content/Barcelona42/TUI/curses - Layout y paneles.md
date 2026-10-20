---
tags: [tui, curses, python, layout, paneles, scroll]
aliases: ["Paneles en curses", "Subventanas curses", "KEY_RESIZE", "Scroll en curses"]
---
# curses - Layout y paneles

[[Inicio|← Inicio]] · [[TUI - Índice|TUI]]

> El **layout** es cómo repartes la pantalla: una cabecera arriba, la lista de tareas, un panel de detalle y una barra de atajos abajo. En `curses` lo calculas a partir de `getmaxyx()` en **cada** fotograma, así la interfaz se adapta sola cuando cambias el tamaño de la ventana. Esta etapa de `tareas.py` añade paneles con borde, una lista con scroll y un modo estrecho.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Zona** | Rectángulo de la pantalla con un propósito: `(y, x, alto, ancho)` |
| **Subventana** (`derwin`) | Ventana que vive **dentro** de otra y comparte su memoria: coordenadas relativas a su esquina |
| **`newwin`** | Ventana independiente, con su propio búfer y su propio `refresh()` |
| **Scroll** | Mostrar solo un trozo de una lista larga, desplazándolo para que la selección siempre se vea |
| **`KEY_RESIZE`** | «Tecla» falsa que `getch()` devuelve cuando la terminal cambia de tamaño |
| **ACS** | *Alternate Character Set*: caracteres de línea (`ACS_HLINE`, `ACS_ULCORNER`…) que funcionan en cualquier terminal |

## El plano

```text
fila 0      │ tareas.py                                         (cabecera)
            ┌─ Tareas ──────────────┐┌─ Detalle ─────────────┐
filas 1..   │ Tarea seleccionada ◄  ││ Texto completo de la  │
alto-2      │ Otra tarea            ││ tarea seleccionada    │
            └──────────────── 3/40 ─┘└───────────────────────┘
fila alto-1 │ j/k mover  q salir                          (barra de atajos)
```

`◄` solo marca aquí la fila seleccionada: en pantalla se pinta en **vídeo inverso** (`curses.A_REVERSE`, colores de texto y fondo intercambiados). Con menos de 80 columnas, lado a lado no cabe: la lista va **arriba** y el detalle **debajo** (modo estrecho).

## El código

Primero, dos ayudantes: uno que escribe sin reventar en los bordes y otro que calcula las zonas.

```python
import curses

TAREAS = [f"Tarea de ejemplo número {n}" for n in range(1, 41)]
Zona = tuple[int, int, int, int]          # (y, x, alto, ancho)


def escribir(win: curses.window, y: int, x: int, texto: str,
             attr: int = 0) -> None:
    """addstr que recorta al ancho y no revienta en los bordes."""
    alto, ancho = win.getmaxyx()
    if 0 <= y < alto and 0 <= x < ancho:
        try:
            win.addnstr(y, x, texto, ancho - x, attr)
        except curses.error:
            pass                          # la última celda da error al pintar


def calcular_zonas(alto: int, ancho: int) -> dict[str, Zona]:
    cuerpo = alto - 2                     # sin cabecera ni barra de atajos
    if ancho >= 80:                       # ancho: lista | detalle
        mitad = ancho // 2
        return {"lista": (1, 0, cuerpo, mitad),
                "detalle": (1, mitad, cuerpo, ancho - mitad)}
    mitad = cuerpo // 2                   # estrecho: apilados
    return {"lista": (1, 0, mitad, ancho),
            "detalle": (1 + mitad, 0, cuerpo - mitad, ancho)}
```

`addnstr(y, x, texto, n)` escribe **como mucho `n` caracteres**: así un texto largo se corta en vez de dar error. (Para emojis y caracteres anchos, ver [[Unicode y emojis en la terminal]].)

Después, la `App`: paneles con borde, lista con scroll y redimensionado.

```python
class App:
    def __init__(self, stdscr: curses.window) -> None:
        self.stdscr = stdscr
        self.sel = 0
        self.scroll = 0                   # primera fila visible de la lista

    def panel(self, zona: Zona, titulo: str) -> curses.window:
        y, x, alto, ancho = zona
        win = self.stdscr.derwin(alto, ancho, y, x)
        win.box()
        escribir(win, 0, 2, f" {titulo} ", curses.A_BOLD)
        return win

    def dibujar_lista(self, win: curses.window) -> None:
        alto, ancho = win.getmaxyx()
        visibles = alto - 2               # quitando los dos bordes
        if self.sel < self.scroll:        # la selección se salió por arriba
            self.scroll = self.sel
        elif self.sel >= self.scroll + visibles:   # ... o por abajo
            self.scroll = self.sel - visibles + 1
        trozo = TAREAS[self.scroll:self.scroll + visibles]
        for fila, texto in enumerate(trozo):
            i = self.scroll + fila
            attr = curses.A_REVERSE if i == self.sel else 0
            escribir(win, 1 + fila, 1, f" {texto}".ljust(ancho - 2), attr)
        escribir(win, alto - 1, ancho - 9, f" {self.sel + 1}/{len(TAREAS)} ")

    def dibujar(self) -> None:
        self.stdscr.erase()
        alto, ancho = self.stdscr.getmaxyx()
        if alto < 8 or ancho < 30:
            escribir(self.stdscr, 0, 0, "Terminal muy pequeña")
            self.stdscr.refresh()
            return
        cabecera = " tareas.py".ljust(ancho)
        escribir(self.stdscr, 0, 0, cabecera, curses.A_REVERSE)
        zonas = calcular_zonas(alto, ancho)
        self.dibujar_lista(self.panel(zonas["lista"], "Tareas"))
        detalle = self.panel(zonas["detalle"], "Detalle")
        escribir(detalle, 1, 2, TAREAS[self.sel])
        escribir(self.stdscr, alto - 1, 0, " j/k mover  q salir", curses.A_DIM)
        self.stdscr.refresh()

    def run(self) -> None:
        curses.curs_set(0)
        self.stdscr.keypad(True)
        while True:
            self.dibujar()
            tecla = self.stdscr.getch()
            if tecla == ord("q"):
                break
            if tecla in (curses.KEY_DOWN, ord("j")):
                self.sel = min(len(TAREAS) - 1, self.sel + 1)
            elif tecla in (curses.KEY_UP, ord("k")):
                self.sel = max(0, self.sel - 1)
            elif tecla == curses.KEY_RESIZE:
                pass              # el bucle redibuja con el tamaño nuevo


if __name__ == "__main__":
    curses.wrapper(lambda stdscr: App(stdscr).run())
```

Tras bajar 30 veces en una terminal de 100 columnas × 30 filas, la lista ha hecho scroll (sale `Tarea … 6` arriba y `31` abajo, seleccionada) y el contador marca `31/40`. Al encoger la ventana a 60 columnas, los paneles se apilan solos.

## Cómo funciona el scroll

`self.scroll` es el índice de la primera tarea visible. Solo se mueve cuando la selección **se sale** de la ventana: si sube por encima, `scroll = sel`; si baja por debajo, `scroll = sel - visibles + 1`. Así la lista no «salta» a cada tecla.

Es la única excepción a la regla «dibujar no cambia nada» de [[curses - El bucle principal]]: cuántas filas caben (`visibles`) solo se sabe al dibujar, así que el ajuste de `scroll` vive en `dibujar_lista`.

## Subventanas, `newwin` o coordenadas

| Opción | Ventajas | Inconvenientes |
|---|---|---|
| `derwin` (aquí) | Coordenadas relativas al panel; el `erase()` y `refresh()` de `stdscr` las incluyen | Si el tamaño no cabe en la pantalla, lanza `curses.error` (por eso el `if alto < 8`) |
| `newwin` | Independientes; útiles para ventanas emergentes (ayuda, confirmar) | Hay que hacer `refresh()` de cada una, en orden (o `noutrefresh` + `curses.doupdate()`) |
| Todo en `stdscr` con coordenadas | Lo más sencillo para 2-3 zonas | Tú sumas los desplazamientos a mano |

## Bordes y líneas

- `win.box()`: borde alrededor de la ventana con caracteres ACS (se ven como `┌─┐` en una terminal UTF-8).
- `win.hline(y, x, curses.ACS_HLINE, n)` y `win.vline(y, x, curses.ACS_VLINE, n)`: líneas sueltas para separar zonas sin ventanas.
- Usa ACS en vez de escribir `─` a mano: si la terminal no es UTF-8, `curses` los sustituye por `-`, `|` y `+`.

## Redimensionar: `KEY_RESIZE`

Cuando cambias el tamaño de la ventana, `ncurses` recibe `SIGWINCH` (ver [[Cómo funciona la terminal]]), ajusta `stdscr` y `getch()` devuelve `KEY_RESIZE`. Como `dibujar` vuelve a leer `getmaxyx()` y recalcula las zonas **cada vez**, basta con volver a dibujar. No guardes el tamaño en `__init__`.

## Errores típicos

| Error | Causa |
|---|---|
| `curses.error` al hacer la ventana pequeña | `derwin`/`addstr` fuera de los límites: comprueba un tamaño mínimo |
| Los paneles no se adaptan | Calculas las zonas una sola vez al arrancar |
| La selección desaparece por abajo | Falta ajustar `scroll` |
| Restos de texto viejo | Falta `erase()` al principio de `dibujar` |
| Texto que se sale del borde | `addstr` en vez de `addnstr` con el ancho disponible |

## Relacionadas

[[curses - El bucle principal]] · [[curses - Colores y estilos]] · [[curses - Teclado y atajos]] · [[Unicode y emojis en la terminal]] · [[Diseño de una TUI moderna]] · [[Cómo funciona la terminal]]

## Referencias

- [curses — Python docs](https://docs.python.org/3/library/curses.html) — `getmaxyx`, `derwin`, `newwin`, `box`, `hline`, `addnstr`, `KEY_RESIZE`, constantes `ACS_*`.
- [ncurses: resizeterm(3x)](https://invisible-island.net/ncurses/man/resizeterm.3x.html) — cómo ncurses trata `SIGWINCH` y genera `KEY_RESIZE`.
- [ncurses: curs_border(3x)](https://invisible-island.net/ncurses/man/curs_border.3x.html) — `box`, `hline`, `vline` y caracteres ACS.
- Probado con Python 3.13 en tmux 3.2a: scroll con 40 tareas, `resize-window` a 60 × 24 (apilado), 80 × 12 (lado a lado) y 25 × 6 (aviso de tamaño).
