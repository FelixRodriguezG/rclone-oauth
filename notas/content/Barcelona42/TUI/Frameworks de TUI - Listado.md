---
tags: [tui, frameworks, bash, python, go, rust, javascript, c]
aliases: ["Librerías de TUI", "TUI frameworks", "Bibliotecas para TUI"]
---
# Frameworks de TUI - Listado

[[Inicio|← Inicio]] · [[TUI - Índice|TUI]]

> Un **framework de TUI** es una biblioteca que se encarga de lo pesado de una interfaz en la terminal: leer teclas, redibujar sin parpadeo, colores, ventanas y *widgets*. Esta nota es un mapa por lenguaje con el estado en octubre de 2026, para que sepas qué existe. Para decidir cuál usar, ver [[Elegir un framework de TUI]].

## Vocabulario

| Término | Qué significa |
|---|---|
| **Modo inmediato** | En cada fotograma **vuelves a dibujar todo** desde tu estado (Ratatui). Tú controlas el bucle |
| **Modo retenido** | Creas objetos (botones, listas) una vez y el framework los recuerda y redibuja (urwid, tview, cursive) |
| **Widgets** | Piezas ya hechas: botón, lista, campo de texto, tabla, barra de progreso |
| **Arquitectura Elm** | Modelo (estado) + `Update(mensaje)` que devuelve un estado nuevo + `View(estado)` que devuelve texto. Viene del lenguaje Elm (Bubble Tea) |
| **Declarativo / tipo React** | Describes «cómo debe verse» con componentes y el framework calcula los cambios (Ink, Textual con CSS) |
| **Bajo nivel** | Te da primitivas (posición, color, tecla); el resto lo programas tú (curses, ncurses) |

## Tabla por lenguaje

| Lenguaje | Framework | Qué es | Estilo | Nivel | Enlace |
|---|---|---|---|---|---|
| Bash | **whiptail** | Cuadros de diálogo (menú, sí/no, checklist). Viene en Debian/Ubuntu | Diálogos sueltos | Muy fácil | [man whiptail](https://manpages.debian.org/whiptail) |
| Bash | **dialog** | Como whiptail, con más widgets (calendario, formularios, selector de ficheros) | Diálogos sueltos | Muy fácil | [invisible-island.net/dialog](https://invisible-island.net/dialog/) |
| Bash | **gum** | Componentes bonitos para scripts (`choose`, `input`, `spin`…), de Charm | Comandos sueltos | Muy fácil | [github.com/charmbracelet/gum](https://github.com/charmbracelet/gum) |
| Bash | **fzf** | Buscador difuso: eliges una línea de una lista escribiendo | Filtro interactivo | Muy fácil | [github.com/junegunn/fzf](https://github.com/junegunn/fzf) |
| Python | **curses** | Módulo estándar, envoltorio de ncurses | Bajo nivel | Medio | [docs.python.org](https://docs.python.org/3/library/curses.html) |
| Python | **urwid** | Widgets clásicos, bucle de eventos propio | Retenido / widgets | Medio | [urwid.org](https://urwid.org/) |
| Python | **prompt_toolkit** | Líneas de entrada potentes (autocompletado, historial); también pantallas completas | Widgets | Medio | [python-prompt-toolkit](https://python-prompt-toolkit.readthedocs.io/) |
| Python | **Rich** | Salida bonita: colores, tablas, barras de progreso, *markdown*. **No** es interactivo | Salida formateada | Fácil | [rich.readthedocs.io](https://rich.readthedocs.io/) |
| Python | **Textual** | Apps completas con widgets, ratón y estilos tipo **CSS**. Usa Rich por debajo | Retenido / widgets + CSS | Medio | [textual.textualize.io](https://textual.textualize.io/) |
| Python | **npyscreen** | Formularios con widgets sobre curses | Retenido / widgets | Medio | [github.com/npcole/npyscreen](https://github.com/npcole/npyscreen) |
| Go | **Bubble Tea** (+ **Lip Gloss** para estilos, **Bubbles** para widgets) | El ecosistema de Charm | Elm | Medio | [github.com/charmbracelet/bubbletea](https://github.com/charmbracelet/bubbletea) |
| Go | **tview** | Widgets listos (tablas, formularios, árboles) | Retenido / widgets | Fácil-medio | [github.com/rivo/tview](https://github.com/rivo/tview) |
| Rust | **Ratatui** | Dibuja widgets en cada fotograma; tú llevas el bucle | Inmediato | Medio-alto | [ratatui.rs](https://ratatui.rs/) |
| Rust | **cursive** | Vistas y diálogos con callbacks | Retenido / widgets | Medio | [github.com/gyscos/cursive](https://github.com/gyscos/cursive) |
| JS/TS | **Ink** | Componentes de **React** que se pintan en la terminal | Declarativo (React) | Medio | [github.com/vadimdemedes/ink](https://github.com/vadimdemedes/ink) |
| JS/TS | **blessed** | Widgets clásicos para Node | Retenido / widgets | Medio | [github.com/chjj/blessed](https://github.com/chjj/blessed) |
| C | **ncurses** | La biblioteca clásica; casi todas las demás vienen de ahí | Bajo nivel | Medio | [invisible-island.net/ncurses](https://invisible-island.net/ncurses/) |
| C | **notcurses** | Moderna: true color, imágenes y vídeo en la terminal | Bajo nivel (planos) | Alto | [notcurses.com](https://notcurses.com/) |

> [!warning] Mantenimiento (comprobado en PyPI, npm y GitHub, octubre de 2026)
> - **blessed** (JS): la última versión en npm (0.1.81) es de **2015**. Está abandonado; hay *forks* (`neo-blessed`…) también poco activos. Para Node, usa **Ink**.
> - **npyscreen**: estuvo parado de 2015 a 2026; en 2026 volvió a publicar (5.0.x), pero la comunidad es pequeña. Para algo nuevo en Python, mejor **Textual** o **urwid**.
> - **tview** y **cursive** siguen vivos pero con publicaciones espaciadas (última de 2025).
> - Activos: Textual 8.x, Rich 15, urwid 4.x, prompt_toolkit 3.0.x, Bubble Tea v2, Lip Gloss v2, Ratatui 0.30, Ink 8, gum 2, fzf 0.74, notcurses 3.0.

> [!note] curses no es un framework «de widgets»
> En [[curses - Primeros pasos]] verás que solo te da celdas, colores y teclas. Por eso es perfecto para **aprender** cómo funciona una TUI (ver [[Cómo funciona la terminal]]). Cuándo compensa dejarlo por otro: [[Elegir un framework de TUI#Cuándo pasar a Textual, Bubble Tea o Ratatui|Elegir un framework]].

## Hola mundo en cuatro estilos

**Bash con whiptail** (diálogos sueltos; más en [[TUIs en Bash - dialog, whiptail y gum]]):

```bash
whiptail --title "Saludo" --msgbox "Hola, mundo" 8 30
```

**Python con curses** (bajo nivel: tú decides cada celda):

```python
import curses


def main(stdscr: curses.window) -> None:
    stdscr.addstr(0, 0, "Hola, mundo (pulsa una tecla)")
    stdscr.refresh()
    stdscr.getch()


curses.wrapper(main)
```

**Python con Textual** (widgets + atajos; `pip install textual`):

```python
from textual.app import App, ComposeResult
from textual.widgets import Footer, Header, Static


class Hola(App):
    BINDINGS = [("q", "quit", "Salir")]          # tecla, acción, texto del pie

    def compose(self) -> ComposeResult:        # qué widgets hay en pantalla
        yield Header()
        yield Static("Hola, mundo")
        yield Footer()


if __name__ == "__main__":
    Hola().run()
```

**Go con Bubble Tea v2** (arquitectura Elm: `Init`, `Update`, `View`):

> [!warning] Sin probar aquí
> Este ejemplo **no se ha compilado**: el Go de la máquina (1.18) es demasiado viejo para Bubble Tea v2. Está escrito siguiendo el tutorial oficial; si no compila, compáralo con el README del proyecto.

```go
package main

import (
	"fmt"
	"os"

	tea "charm.land/bubbletea/v2"
)

type model struct{}

func (m model) Init() tea.Cmd { return nil }

func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
	if _, ok := msg.(tea.KeyPressMsg); ok { // cualquier tecla → salir
		return m, tea.Quit
	}
	return m, nil
}

func (m model) View() tea.View { return tea.NewView("Hola, mundo\n") }

func main() {
	if _, err := tea.NewProgram(model{}).Run(); err != nil {
		fmt.Println(err)
		os.Exit(1)
	}
}
```

> [!info] Ratatui (Rust)
> El «hola mundo» oficial (`cargo generate ratatui/templates`) es un bucle: `terminal.draw(...)` dibuja el fotograma y `event::read()` espera una tecla. Es modo inmediato puro: si no llamas a `draw`, no se ve nada. (Resumido de la documentación; **no se ha compilado ni probado aquí**.)

## Relacionadas

[[Elegir un framework de TUI]] · [[CLI, TUI y GUI]] · [[curses - Primeros pasos]] · [[TUIs en Bash - dialog, whiptail y gum]] · [[Diseño de una TUI moderna]] · [[De script a TUI - arquitectura]] · [[Python - Índice]]

## Referencias

- [Python curses](https://docs.python.org/3/library/curses.html) — módulo estándar, `curses.wrapper`.
- [Textual — Getting started](https://textual.textualize.io/getting_started/) — `App`, `compose`, `BINDINGS`.
- [Bubble Tea README](https://github.com/charmbracelet/bubbletea) — tutorial v2: `charm.land/bubbletea/v2`, `tea.KeyPressMsg`, `tea.NewView`.
- [Ratatui README](https://github.com/ratatui/ratatui) — plantilla *Hello World* con `ratatui::init`, `draw` y `restore`.
- [PyPI JSON API](https://pypi.org/) y [npm registry](https://registry.npmjs.org/blessed) — versiones y fechas (npyscreen, urwid, Rich, Textual, blessed).
- Páginas de *releases* en GitHub de cada proyecto — últimas versiones de tview, cursive, Ink, gum, fzf, notcurses.
- Ejemplos de curses y Textual (8.2.8) ejecutados con Python 3.13 en tmux. Bubble Tea y Ratatui **no** compilados aquí (Go 1.18 de la máquina es demasiado viejo para Bubble Tea v2); escritos siguiendo la documentación oficial.
