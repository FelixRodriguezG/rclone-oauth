---
tags: [tui, curses, python, teclado, atajos]
aliases: ["Atajos de teclado curses", "get_wch", "keypad", "KEY_UP"]
---
# curses - Teclado y atajos

[[Inicio|← Inicio]] · [[TUI - Índice|TUI]]

> Una TUI se maneja con el teclado, así que leer bien las teclas es la mitad del trabajo: flechas, Enter, Esc y Backspace llegan distinto según la terminal, y los acentos necesitan `get_wch()`. La idea clave de esta nota es guardar los atajos en **un diccionario tecla → acción**: de ahí salen a la vez el comportamiento, la barra de atajos y la pantalla de ayuda.

## Vocabulario

| Término | Qué significa |
|---|---|
| **`keypad(True)`** | Pide a `curses` que traduzca secuencias como `ESC [ A` a una constante (`KEY_UP`). `wrapper` ya lo activa |
| **`getch()`** | Devuelve un `int`: el código del byte o una constante `KEY_*` |
| **`get_wch()`** | Devuelve un `str` (un carácter Unicode: `"ñ"`, `"é"`) o un `int` para teclas especiales |
| **Modo** | Estado que cambia lo que hace cada tecla: en «lista», `j` baja; en «filtro», `j` es una letra |

## Teclas especiales

| Tecla | Lo que suele llegar |
|---|---|
| Flechas | `KEY_UP`, `KEY_DOWN`, `KEY_LEFT`, `KEY_RIGHT` |
| Inicio, Fin, RePág, AvPág, F1… | `KEY_HOME`, `KEY_END`, `KEY_PPAGE`, `KEY_NPAGE`, `KEY_F1`… |
| **Enter** | `"\n"` (10), a veces `"\r"` (13) o `KEY_ENTER` |
| **Backspace** | `"\x7f"` (127), `"\b"` (8) o `KEY_BACKSPACE`: **depende de la terminal** |
| **Esc** | `"\x1b"` (27), con retraso (ver más abajo) |
| Tab / Shift+Tab | `"\t"` (9) / `KEY_BTAB` |
| Ctrl+letra | Código 1-26: Ctrl+A = 1… (`ord(c) & 0x1f`) |

Como Enter y Backspace varían, compara siempre contra **todas** las variantes (las tuplas `ENTER` y `BORRAR` del ejemplo).

## Ejemplo: atajos, ayuda y filtro

```python
import curses

TAREAS = ["comprar pan", "entregar libft", "repasar punteros", "ir al gym"]
ENTER = ("\n", "\r", curses.KEY_ENTER)
BORRAR = ("\b", "\x7f", curses.KEY_BACKSPACE)   # cambia según la terminal
ESC = "\x1b"


class App:
    def __init__(self, stdscr: curses.window) -> None:
        self.stdscr = stdscr
        self.sel = 0
        self.filtro = ""
        self.modo = "lista"               # "lista" | "filtro" | "ayuda"
        self.seguir = True
        # tecla → (descripción, función). Una sola fuente de verdad.
        self.acciones = {
            "j": ("bajar", self.bajar), curses.KEY_DOWN: ("", self.bajar),
            "k": ("subir", self.subir), curses.KEY_UP: ("", self.subir),
            "/": ("filtrar", self.empezar_filtro),
            "?": ("ayuda", self.ver_ayuda),
            "q": ("salir", self.salir),
        }

    def visibles(self) -> list[str]:
        return [t for t in TAREAS if self.filtro.lower() in t.lower()]

    def bajar(self) -> None:
        self.sel = min(self.sel + 1, max(0, len(self.visibles()) - 1))

    def subir(self) -> None:
        self.sel = max(0, self.sel - 1)

    def empezar_filtro(self) -> None:
        self.modo = "filtro"
        curses.curs_set(1)                # cursor visible mientras escribes

    def ver_ayuda(self) -> None:
        self.modo = "ayuda"

    def salir(self) -> None:
        self.seguir = False

    def tecla_filtro(self, tecla: str | int) -> None:
        if tecla in ENTER or tecla == ESC:
            if tecla == ESC:
                self.filtro = ""          # Esc cancela el filtro
            self.modo = "lista"
            curses.curs_set(0)
        elif tecla in BORRAR:
            self.filtro = self.filtro[:-1]
        elif isinstance(tecla, str) and tecla.isprintable():
            self.filtro += tecla
        self.sel = 0

    def atajos(self) -> list[tuple[str | int, str]]:
        return [(t, d) for t, (d, _) in self.acciones.items() if d]

    def barra(self) -> str:
        if self.modo == "filtro":
            return "Enter aceptar · Esc cancelar"
        if self.modo == "ayuda":
            return "cualquier tecla: volver"
        return " · ".join(f"{t} {d}" for t, d in self.atajos())

    def dibujar(self) -> None:
        self.stdscr.erase()
        alto, _ = self.stdscr.getmaxyx()
        if self.modo == "ayuda":
            for i, (t, d) in enumerate(self.atajos()):
                self.stdscr.addstr(1 + i, 2, f"{t:>3}  {d}")
        else:
            for i, texto in enumerate(self.visibles()):
                attr = curses.A_REVERSE if i == self.sel else 0
                self.stdscr.addstr(1 + i, 2, texto, attr)
        self.stdscr.addstr(alto - 1, 0, self.barra(), curses.A_DIM)
        if self.modo == "filtro":
            self.stdscr.addstr(alto - 2, 0, f"/{self.filtro}")
        self.stdscr.refresh()

    def run(self) -> None:
        curses.curs_set(0)
        self.stdscr.keypad(True)
        while self.seguir:
            self.dibujar()
            tecla = self.stdscr.get_wch()     # str (Unicode) o int (KEY_*)
            if self.modo == "filtro":
                self.tecla_filtro(tecla)
            elif self.modo == "ayuda":
                self.modo = "lista"
            elif tecla in self.acciones:
                self.acciones[tecla][1]()


if __name__ == "__main__":
    curses.set_escdelay(25)               # Esc responde en 25 ms
    curses.wrapper(lambda stdscr: App(stdscr).run())
```

Al arrancar, la barra se genera sola desde el diccionario: `j bajar · k subir · / filtrar · ? ayuda · q salir`. Escribiendo `/pun` + Enter queda solo `repasar punteros`; con `?` aparece la lista de atajos.

- **Un diccionario, tres usos**: ejecutar la acción, la barra de abajo y la ayuda. Si añades un atajo, no se te olvida documentarlo.
- **Modos**: en «filtro», las letras se escriben; en «lista», son atajos. La barra cambia según el modo (atajos contextuales).
- `get_wch()` permite filtrar por `ñ` o `á`, que con `getch()` llegarían como dos bytes sueltos (y necesita un locale UTF-8, ver [[Unicode y emojis en la terminal]]).

## Esc y `set_escdelay`

Esc es también el principio de las secuencias de las flechas, y `curses` espera **1000 ms** por defecto para distinguirlos (ver [[Cómo funciona la terminal#Por qué Esc tarda]]). `curses.set_escdelay(25)` (Python 3.9+) lo baja a 25 ms. Si trabajas en tmux, baja también `escape-time` (ver [[Configurar tmux]]).

## Convenciones que el usuario ya conoce

`j`/`k` y flechas para moverse, `g`/`G` principio y fin, `/` buscar, `?` ayuda, `q` salir, `Esc` cancelar, Enter abrir o confirmar, Espacio marcar. Las usan [[Vim]], `less`, `htop` y `lazygit`: si copias estas, nadie tiene que aprender nada nuevo (ver [[Diseño de una TUI moderna]]).

## Ratón (opcional)

```python
curses.mousemask(curses.ALL_MOUSE_EVENTS)   # una vez, al arrancar
# en el bucle (vale igual con getch() que con get_wch()):
if tecla == curses.KEY_MOUSE:
    _, x, y, _, botones = curses.getmouse()  # ¡aquí x va ANTES que y!
    if botones & curses.BUTTON1_CLICKED and 0 <= y - 1 < len(self.visibles()):
        self.sel = y - 1                     # clic izquierdo: la lista empieza en la fila 1
```

Un clic izquierdo en la fila 2, columna 4 da `x=4`, `y=2` y `BUTTON1_CLICKED`. `getmouse()` es de las pocas funciones que no siguen el orden `y, x`; `botones` lleva un bit por evento y `&` (y binario) comprueba si está el que buscas.

Útil para clicar en una fila, pero que **nunca** sea la única forma de hacer algo. Mientras está activo, el usuario no puede seleccionar texto con el ratón como siempre (suele funcionar con Shift + arrastrar).

## Errores típicos

| Error | Causa |
|---|---|
| Backspace no borra | Solo comparas con `KEY_BACKSPACE`; añade `"\x7f"` y `"\b"` |
| Enter no hace nada | Comparas con `KEY_ENTER`; casi siempre llega `"\n"` |
| Esc tarda un segundo | `ESCDELAY` por defecto: `set_escdelay(25)` |
| Las flechas escriben `[A` | `keypad(False)` en esa ventana (cada ventana tiene el suyo) |
| `tecla == "q"` con `getch()` | `getch()` da `int`: usa `ord("q")` o `get_wch()` |

## Relacionadas

[[curses - El bucle principal]] · [[curses - Layout y paneles]] · [[curses - Ejecutar comandos]] · [[Diseño de una TUI moderna]] · [[Cómo funciona la terminal]] · [[Leer entrada del usuario en Bash]] · [[Vim]]

## Referencias

- [curses — Python docs](https://docs.python.org/3/library/curses.html) — `getch`, `get_wch`, `keypad`, `set_escdelay`, `mousemask`, `getmouse`, constantes `KEY_*`.
- [Curses HOWTO: User Input](https://docs.python.org/3/howto/curses.html#user-input) — teclas especiales y `curses.ascii`.
- [ncurses: curs_getch(3x)](https://invisible-island.net/ncurses/man/curs_getch.3x.html) — `keypad`, `KEY_ENTER`, `KEY_BACKSPACE` y el retraso de Esc.
- Probado con Python 3.13 en tmux 3.2a: filtro `/pun`, Backspace, Esc que cancela, ayuda con `?` y salida con `q`.
