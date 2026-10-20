---
tags: [tui, curses, python, colores, accesibilidad]
aliases: ["Colores en curses", "init_pair", "color_pair", "Atributos curses"]
---
# curses - Colores y estilos

[[Inicio|← Inicio]] · [[TUI - Índice|TUI]]

> En `curses` el color va en **pares** (texto + fondo) numerados, y se combina con **atributos** como negrita o vídeo inverso. La forma limpia de usarlos es una **paleta con nombres** («error», «título») que se inicia una vez y que, si la terminal no tiene color o el usuario lo desactiva, cae a negrita y símbolos. Esta etapa da a `tareas.py` colores con significado.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Par de color** | Número (1, 2, 3…) asociado a una combinación «color de texto + color de fondo». El 0 es fijo: blanco sobre negro, o los colores de tu terminal si llamas a `use_default_colors()` |
| **Atributo** | Estilo de texto: `A_BOLD` (negrita), `A_DIM` (tenue), `A_REVERSE` (invierte texto y fondo), `A_UNDERLINE` (subrayado) |
| **`-1`** | «El color por defecto de la terminal». Solo funciona tras `use_default_colors()` |
| **`curses.COLORS`** | Cuántos colores admite la terminal: 8, 16 o 256 |
| **`NO_COLOR`** | Variable de entorno: si existe y no está vacía, el usuario pide **no** usar colores |

Para los códigos de color que hay debajo (los `\e[31m`), ver [[Colores en la terminal]].

## Los 8 colores básicos

`COLOR_BLACK`, `COLOR_RED`, `COLOR_GREEN`, `COLOR_YELLOW`, `COLOR_BLUE`, `COLOR_MAGENTA`, `COLOR_CYAN`, `COLOR_WHITE` (valores 0 a 7). Cada usuario los tiene configurados distinto en su terminal: tu «rojo» puede ser rosa en otro tema. Por eso los colores deben **ayudar**, no ser la única pista.

## Paleta con nombres

```python
import curses
import os

# Nombre lógico → (color de texto, atributo extra). -1 = color por defecto.
PALETA = {
    "normal": (-1, curses.A_NORMAL),
    "titulo": (curses.COLOR_CYAN, curses.A_BOLD),
    "exito": (curses.COLOR_GREEN, curses.A_NORMAL),
    "error": (curses.COLOR_RED, curses.A_BOLD),
    "atajo": (curses.COLOR_YELLOW, curses.A_BOLD),
}
ESTILO: dict[str, int] = {}               # nombre → atributo listo para usar


def iniciar_colores() -> None:
    usar_color = curses.has_colors() and not os.environ.get("NO_COLOR")
    if curses.has_colors():               # también con NO_COLOR (ver abajo)
        curses.start_color()              # wrapper ya lo hizo; repetirlo no daña
        curses.use_default_colors()       # -1 = colores de la terminal
    for n, (nombre, (color, extra)) in enumerate(PALETA.items(), start=1):
        if usar_color:
            curses.init_pair(n, color, -1)
            ESTILO[nombre] = curses.color_pair(n) | extra
        else:
            ESTILO[nombre] = extra        # sin color: solo negrita, dim...


def main(stdscr: curses.window) -> None:
    curses.curs_set(0)
    iniciar_colores()
    stdscr.addstr(0, 0, "Mis tareas", ESTILO["titulo"])
    stdscr.addstr(2, 2, " 1. Comprar pan", ESTILO["normal"])
    stdscr.addstr(3, 2, "✔ Tarea añadida", ESTILO["exito"])
    stdscr.addstr(4, 2, "✘ tareas.sh no encontrado", ESTILO["error"])
    stdscr.addstr(6, 0, "q", ESTILO["atajo"])
    stdscr.addstr(" salir   ")            # sin (y, x): sigue donde iba
    stdscr.refresh()
    stdscr.getkey()


if __name__ == "__main__":
    curses.wrapper(main)
```

- `init_pair(n, texto, fondo)` define el par `n`; `color_pair(n)` lo convierte en un atributo que se pasa a `addstr`.
- Los atributos se **combinan** con `|` (o binario): `color_pair(2) | A_BOLD`.
- El resto del programa solo usa `ESTILO["error"]`. Si mañana cambias el rojo por magenta, tocas una línea.
- Guárdalo como `colores.py`. Con `NO_COLOR=1 python3 colores.py`, el título sigue en negrita y el error en negrita con `✘`: se entiende igual. Con `NO_COLOR=` (vacía) sí hay color: la convención solo cuenta la variable si tiene algún valor.

> [!warning] `use_default_colors()` también sin color
> `wrapper` ya llamó a `start_color()`. Si con `NO_COLOR` te saltas `use_default_colors()`, `curses` pinta todo el texto **blanco sobre negro** (lo comprobamos: envía `\e[37m\e[40m`), y en un tema claro la pantalla se vuelve negra. Llamándola siempre, el texto sin par usa los colores de tu terminal.

> [!tip] Los símbolos van además del color
> `✔`/`✘`, `[ ]`/`[x]` o `!` mantienen el significado aunque el usuario no distinga el rojo del verde (daltonismo), use `NO_COLOR` o tenga un tema raro. Ver [[Diseño de una TUI moderna]] y, para los emojis, [[Unicode y emojis en la terminal]].

## Atributos

| Atributo | Para qué usarlo |
|---|---|
| `A_BOLD` | Títulos, lo importante, errores |
| `A_DIM` | Información secundaria: barra de atajos, fechas, contadores |
| `A_REVERSE` | **Selección**: funciona incluso sin colores |
| `A_UNDERLINE` | Enlaces o la letra del atajo dentro de una palabra |
| `A_STANDOUT` | «El mejor resaltado que tenga esta terminal» (suele ser inverso) |

No todas las terminales muestran todos: `A_DIM` o `A_ITALIC` a veces no se ven. Nunca dependas solo de ellos.

## 256 colores

Si la terminal lo admite (`curses.COLORS >= 256` **después** de `start_color()`), puedes usar índices de 0 a 255: 16 básicos, un cubo de 6×6×6 colores (16-231) y 24 grises (232-255).

```python
import curses


def main(stdscr: curses.window) -> None:
    curses.start_color()
    curses.use_default_colors()
    naranja = 208 if curses.COLORS >= 256 else curses.COLOR_YELLOW
    curses.init_pair(1, naranja, -1)
    stdscr.addstr(0, 0, f"Esta terminal tiene {curses.COLORS} colores")
    stdscr.addstr(1, 0, "Texto naranja (o amarillo)", curses.color_pair(1))
    for c in range(min(curses.COLORS, 16)):  # muestrario de los 16 básicos
        curses.init_pair(10 + c, -1, c)      # fondo de color c
        stdscr.addstr(3, c * 3, f"{c:>2} ", curses.color_pair(10 + c))
    stdscr.getkey()


curses.wrapper(main)
```

Con `TERM=xterm-256color` dice `256 colores` y el texto sale naranja; con `TERM=screen` dice `8 colores` y sale amarillo. El número lo decide `TERM` (ver [[Cómo funciona la terminal#TERM y terminfo]]), no tu programa: ten siempre un **plan B** de 8 colores.

> [!info] ¿Y colores RGB exactos?
> `curses.init_color(n, r, g, b)` redefine un color (valores 0-1000) solo si `curses.can_change_color()` es `True`, y muchas terminales no lo permiten. Para colores *truecolor* fiables, mira los frameworks de [[Frameworks de TUI - Listado]].

## Pocos colores, con significado

- **Uno** para lo seleccionado o importante, **uno** para errores, **uno** para éxito. El resto, color por defecto.
- Fondo `-1` siempre: respeta el tema claro u oscuro del usuario.
- Si todo es de colores, nada destaca.

## Errores típicos

| Error | Causa |
|---|---|
| `_curses.error: init_pair() can't be called before start_color()` | Falta `start_color()` (`wrapper` lo llama por ti; sin `wrapper`, hazlo tú) |
| `-1` da error en `init_pair` | Falta `use_default_colors()` |
| `AttributeError: COLORS` | Lees `curses.COLORS` antes de `start_color()` (sin `wrapper`) |
| Fondo negro en un tema claro | Usaste `COLOR_BLACK` en vez de `-1` como fondo |
| Solo 8 colores dentro de tmux | `TERM` es `screen`; configura `tmux-256color` (ver [[Configurar tmux]]) |
| Rojo/verde como única diferencia | Ilegible con daltonismo o `NO_COLOR`: añade símbolos |

## Relacionadas

[[Colores en la terminal]] · [[curses - Layout y paneles]] · [[curses - Teclado y atajos]] · [[Diseño de una TUI moderna]] · [[Cómo funciona la terminal]] · [[Unicode y emojis en la terminal]] · [[Configurar tmux]]

## Referencias

- [curses — Python docs](https://docs.python.org/3/library/curses.html) — `start_color`, `use_default_colors`, `init_pair`, `color_pair`, `has_colors`, `COLORS`, `can_change_color`, atributos `A_*`.
- [Curses HOWTO: Attributes and Color](https://docs.python.org/3/howto/curses.html#attributes-and-color) — pares de color y atributos.
- [ncurses: default_colors(3x)](https://invisible-island.net/ncurses/man/default_colors.3x.html) — el color `-1`.
- [NO_COLOR](https://no-color.org/) — convención: si `NO_COLOR` existe y no está vacía, no emitir color.
- [256 colors — Wikipedia (ANSI escape code)](https://en.wikipedia.org/wiki/ANSI_escape_code#8-bit) — reparto de los 256 índices.
- Probado con Python 3.13 en tmux 3.2a con `TERM=screen` (8 colores), `TERM=xterm-256color` (256) y `NO_COLOR=1`.
