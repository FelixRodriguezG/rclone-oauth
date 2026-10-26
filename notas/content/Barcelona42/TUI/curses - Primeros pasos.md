---
tags: [tui, curses, python, ncurses]
aliases: ["curses", "ncurses", "Python curses", "curses.wrapper"]
---
# curses - Primeros pasos

[[Inicio|← Inicio]] · [[TUI - Índice|TUI]]

> **curses** es la biblioteca clásica para hacer TUI: te deja escribir texto en cualquier posición de la terminal, leer teclas una a una y olvidarte de las secuencias de escape. Python la trae de serie en Linux y macOS (`import curses`). Aquí escribimos la primera versión de `tareas.py`: una pantalla que saluda y espera una tecla.

## Vocabulario

| Término | Qué significa |
|---|---|
| **ncurses** | «new curses»: la implementación libre en C que usa Linux. El módulo `curses` de Python es una capa sobre ella |
| **Ventana** (`curses.window`) | Rectángulo de celdas donde escribes. Puede ser toda la pantalla o un trozo |
| **`stdscr`** | *Standard screen*: la ventana que ocupa toda la terminal. Te la da `curses` al arrancar |
| **Celda** | Una posición de la rejilla: una fila y una columna, con un carácter y un estilo |
| **Búfer** | Copia en memoria de la pantalla. `addstr` escribe en el búfer; `refresh` lo envía a la terminal |

> [!info] En Windows
> El `curses` de Python no viene en Windows. Usa WSL (Linux dentro de Windows) o instala el paquete `windows-curses`. En los ordenadores de 42 (Linux) ya está.

## El primer programa

```python
import curses


def main(stdscr: curses.window) -> str:
    curses.curs_set(0)                      # oculta el cursor
    stdscr.addstr(0, 0, "Hola desde curses")
    stdscr.addstr(2, 4, "Pulsa una tecla para salir", curses.A_BOLD)
    alto, ancho = stdscr.getmaxyx()
    stdscr.addstr(4, 4, f"Tu terminal mide {alto} filas x {ancho} columnas")
    stdscr.refresh()                        # ahora sí se ve
    return stdscr.getkey()                  # espera una tecla


if __name__ == "__main__":
    tecla = curses.wrapper(main)    # aquí la terminal ya está restaurada
    print(f"Has pulsado: {tecla!r}")
```

Mientras corre (en una terminal de 30 × 100):

```text
Hola desde curses

    Pulsa una tecla para salir

    Tu terminal mide 30 filas x 100 columnas
```

Al pulsar `↑`, la pantalla de antes vuelve intacta y se imprime `Has pulsado: 'KEY_UP'`.

## Qué hace cada pieza

### `curses.wrapper(main)`: siempre

`wrapper` prepara la terminal, llama a `main(stdscr)` y, **pase lo que pase** (fin normal, excepción o Ctrl+C), la deja como estaba. Por dentro:

1. `initscr()`: entra en la pantalla alternativa y crea `stdscr`.
2. `noecho()` y `cbreak()`: modo casi raw, sin eco y tecla a tecla (ver [[Cómo funciona la terminal]]).
3. `stdscr.keypad(True)`: traduce flechas y F1… a constantes (`KEY_UP`).
4. `start_color()` si la terminal tiene colores.
5. Al terminar, en un `finally`: deshace todo y llama a `endwin()`.

Lo que devuelve `main` lo devuelve `wrapper`: así sacas datos (aquí, la tecla) para usarlos **después** de cerrar la interfaz.

> [!danger] Sin `wrapper`, un error te rompe la terminal
> Con `curses.initscr()` a mano, una excepción deja la terminal sin eco y sin saltos de línea correctos. El traceback sale en escalera:
> ```text
> itscr(); 1/0
>                                                  ~^~
>                                                     ZeroDivisionError: division by zero
> ```
> Arréglalo escribiendo `reset` a ciegas. Y usa `wrapper`.

### Coordenadas: `(y, x)`, fila primero

`addstr(y, x, texto)` escribe en la **fila `y`** y la **columna `x`**, contando desde **0** arriba a la izquierda. Es al revés que en matemáticas `(x, y)` y que las secuencias ANSI (que empiezan en 1). Casi todas las funciones de `curses` siguen el orden `y, x`: `getmaxyx()`, `move(y, x)`, `newwin(alto, ancho, y, x)`.

```text
 x→  0 1 2 3 4 5 …                    ancho-1
y 0  H o l a   d e s d e   c u r s e s
↓ 1
  2          P u l s a   u n a   t e c l a …
  …
  alto-1                              ← última fila
```

`getmaxyx()` devuelve `(alto, ancho)`: las filas válidas van de `0` a `alto - 1`.

### `refresh()`: hasta entonces no se ve nada

`addstr` solo escribe en el **búfer**. `refresh()` compara el búfer con lo que hay en pantalla y envía **solo los cambios**. Si olvidas `refresh()`, la pantalla se queda en blanco. (`getkey()` y `getch()` hacen un `refresh()` implícito de su ventana, pero no te fíes de eso.)

### Leer teclas: `getkey()` y `getch()`

| Función | Devuelve al pulsar `a` | Al pulsar `↑` |
|---|---|---|
| `getkey()` | `"a"` (str) | `"KEY_UP"` (str) |
| `getch()` | `97` (int, el código) | `259` (`curses.KEY_UP`) |

Ambas **esperan** hasta que pulses algo (salvo que configures un `timeout`, ver [[curses - El bucle principal]]). Para acentos y emojis hay `get_wch()` (ver [[curses - Teclado y atajos]]).

### `curs_set(0)`: oculta el cursor

`0` invisible, `1` normal, `2` muy visible. Algunas terminales no lo admiten y lanzan `curses.error`; si te pasa, envuélvelo en `try`.

## Errores típicos

| Error                                                                 | Causa                                                                                                                                                                          |
| --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `_curses.error: addwstr() returned ERR`                               | Escribiste **fuera** de la ventana (`addstr(50, 0, ...)` con 30 filas) o el texto no cabe hasta el final                                                                       |
| El mismo error al escribir en la **última celda** (`alto-1, ancho-1`) | `curses` intenta mover el cursor después del carácter y no puede. El carácter sí se pinta: captura `curses.error` (ver la función `escribir` en [[curses - Layout y paneles]]) |
| Pantalla en blanco                                                    | Falta `refresh()`                                                                                                                                                              |
| `addstr(x, y, …)`                                                     | Coordenadas al revés: el texto sale en otro sitio o da error                                                                                                                   |
| `_curses.error: setupterm: could not find terminal`                   | `TERM` vacío o raro (ver [[Cómo funciona la terminal#TERM y terminfo]])                                                                                                        |
| La terminal se queda rota                                             | No usaste `wrapper`: escribe `reset`                                                                                                                                           |
| Ejecutarlo dentro del «Run» de un IDE                                 | Esa consola no es una terminal real: ejecútalo en una terminal                                                                                                                 |

## Relacionadas

[[TUI - Índice]] · [[Cómo funciona la terminal]] · [[curses - El bucle principal]] · [[curses - Layout y paneles]] · [[curses - Teclado y atajos]] · [[De script a TUI - arquitectura]] · [[Python - Índice]]

## Referencias

- [curses — Python docs](https://docs.python.org/3/library/curses.html) — `wrapper`, `addstr`, `getmaxyx`, `getkey`, `getch`, `curs_set`, `window.refresh`.
- [Curses Programming with Python (HOWTO)](https://docs.python.org/3/howto/curses.html) — qué hace `wrapper` y orden `y, x`.
- [ncurses: curs_addstr(3x)](https://invisible-island.net/ncurses/man/curs_addstr.3x.html) — por qué falla escribir en la esquina inferior derecha.
- [windows-curses (PyPI)](https://pypi.org/project/windows-curses/) — `curses` para Windows.
- Ejemplos y errores probados con Python 3.13 en tmux 3.2a (30 × 100).
