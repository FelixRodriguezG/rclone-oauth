---
tags: [tui, unicode, utf-8, emojis, locale, python]
aliases: ["UTF-8 en la terminal", "Ancho de caracteres", "wcwidth", "Emojis en curses"]
---
# Unicode y emojis en la terminal

[[Inicio|← Inicio]] · [[TUI - Índice|TUI]]

> La terminal es una rejilla de celdas, pero no todos los caracteres ocupan una: `a` ocupa 1 columna, `日` ocupa 2, una tilde suelta ocupa 0 y un emoji depende de la terminal. Si cuentas con `len()`, tus columnas se descuadran y el texto se sale de los paneles. Esta nota explica UTF-8, el locale y cómo medir y recortar por **ancho visible**.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Unicode** | Catálogo que da un número (*code point*) a cada carácter: `ñ` = U+00F1, `🚀` = U+1F680 |
| **UTF-8** | Forma de guardar esos números en bytes: ASCII en 1 byte, `ñ` en 2, `🚀` en 4 |
| **Locale** | Configuración de idioma y codificación del sistema (`LANG=es_ES.UTF-8`). Le dice a los programas que los bytes son UTF-8 |
| **Ancho** | Columnas que ocupa un carácter en pantalla: 0, 1 o 2 |
| **Carácter combinable** | Se pinta **encima** del anterior y ocupa 0: `e` + `◌́` = `é` |
| **ZWJ** | *Zero Width Joiner* (U+200D): pega emojis para formar uno (👩 + ZWJ + 💻 = 👩‍💻) |

## Tres formas de medir «lo largo» de un texto

| Texto | Bytes UTF-8 | `len()` (caracteres) | Columnas en pantalla |
|---|---|---|---|
| `hola` | 4 | 4 | 4 |
| `año` | 4 | 3 | 3 |
| `日本語` | 9 | 3 | **6** |
| `🚀 lanzar` | 11 | 8 | **9** |
| `é` (e + tilde combinable) | 3 | 2 | **1** |

En C, `strlen` cuenta **bytes**; en Python, `len` cuenta **caracteres**. Ninguno cuenta **columnas**, que es lo que importa al dibujar.

## El locale en Python y `curses`

`ncurses` decide cómo interpretar los bytes según el locale. Python 3 lo configura al arrancar a partir de `LANG`/`LC_ALL`, pero la documentación de `curses` recomienda hacerlo explícito al principio del programa:

```python
import curses
import locale

locale.setlocale(locale.LC_ALL, "")      # usa el UTF-8 del sistema


def main(stdscr: curses.window) -> None:
    for fila, texto in enumerate(["[x] año", "[x] 日本語", "[x] 🚀 lanzar"]):
        stdscr.addstr(fila, 0, texto)
        stdscr.addstr(fila, 14, "|")      # columna 14 fija
    stdscr.getkey()


curses.wrapper(main)
```

Con un locale UTF-8 (`es_ES.UTF-8`, `C.UTF-8`):

```text
[x] año       |
[x] 日本語    |
[x] 🚀 lanzar |
```

Guárdalo como `uni.py`. Con `LC_ALL=C python3 uni.py` (locale sin UTF-8), los caracteres no ASCII **desaparecen**:

```text
[x] ao        |
[x]           |
[x]   lanzar  |
```

Si ves eso, o símbolos raros como `Ã±`, comprueba `locale` en la terminal: `LANG` debe acabar en `.UTF-8`.

## Calcular el ancho y recortar

Python no trae una función de ancho, pero `unicodedata` da una buena aproximación:

```python
import unicodedata


def ancho_car(c: str) -> int:
    """Columnas que ocupa un carácter: 0, 1 o 2 (aproximación)."""
    if unicodedata.combining(c) or c == "\u200d":  # tildes sueltas, ZWJ
        return 0
    if unicodedata.east_asian_width(c) in ("W", "F"):  # Wide / Fullwidth
        return 2
    return 1


def ancho(texto: str) -> int:
    return sum(ancho_car(c) for c in texto)


def recortar(texto: str, columnas: int, fin: str = "…") -> str:
    """Corta texto para que ocupe como mucho `columnas` en pantalla."""
    if ancho(texto) <= columnas:
        return texto
    salida, usado = "", ancho(fin)
    for c in texto:
        if usado + ancho_car(c) > columnas:
            break
        salida += c
        usado += ancho_car(c)
    return salida + fin


for t in ["hola", "año", "日本語", "🚀 lanzar", "é"]:
    print(f"{t!r:14} len={len(t)}  ancho={ancho(t)}  "
          f"recortado={recortar(t, 5)!r}")
```

```text
'hola'         len=4  ancho=4  recortado='hola'
'año'          len=3  ancho=3  recortado='año'
'日本語'          len=3  ancho=6  recortado='日本…'
'🚀 lanzar'     len=8  ancho=9  recortado='🚀 l…'
'é'           len=2  ancho=1  recortado='é'
```

¡Mira el propio `print`! `{t!r:14}` rellena con `len()`, y por eso las columnas salen torcidas en las filas con `日本語`, `🚀` y `é`. Es exactamente el fallo que tendrías con `ljust()` en un panel. Para rellenar bien: `texto + " " * (columnas - ancho(texto))`.

> [!tip] La biblioteca `wcwidth`
> `pip install wcwidth` da `wcswidth(texto)`, que imita a la función `wcwidth` de C y conoce mejor los emojis modernos. Para una TUI pequeña, la aproximación de arriba basta.

## Emojis: cada terminal los pinta a su manera

- Los emojis «nuevos» o compuestos (👩‍💻, banderas 🇪🇸, tonos de piel) pueden verse como uno, como varios o como cuadrados, según la terminal y la fuente.
- Algunos símbolos (`⚠`, `✔`, `❤`) tienen ancho **ambiguo**: 1 columna en una terminal, 2 en otra. Tu cálculo y el de la terminal no coincidirán y la línea se descuadra.
- Dentro de tmux el ancho lo calcula tmux, que puede no coincidir con tu terminal (ver [[Qué es tmux]]).

Regla práctica: usa emojis **solo** en sitios donde un descuadre no rompa nada (al final de una línea o en mensajes), nunca en columnas alineadas o bordes.

## Alternativas ASCII

Ofrece un modo sin símbolos especiales (por ejemplo, si el locale no es UTF-8 o con una opción `--ascii`):

| Bonito | ASCII |
|---|---|
| `✔` / `✘` | `[x]` / `[!]` |
| `▸` (selección) | `>` |
| `…` | `...` |
| `●○○` (progreso) | `#--` |
| `🚀` | `*` |

```python
import locale

locale.setlocale(locale.LC_ALL, "")
codificacion = locale.nl_langinfo(locale.CODESET)   # "UTF-8" o "ANSI_X3.4-1968"
UNICODE = codificacion.lower().replace("-", "") == "utf8"
OK = "✔" if UNICODE else "[x]"
```

Con `LANG=C.UTF-8`, `UNICODE` vale `True`; con `LC_ALL=C`, `False`. No uses `locale.getpreferredencoding()`: con `LC_ALL=C` Python activa su «modo UTF-8» y te dice `utf-8` aunque la terminal no lo sea.

## Errores típicos

| Error | Causa |
|---|---|
| Columnas descuadradas con acentos o CJK | Rellenas con `len()`/`ljust()` en vez de ancho visible |
| `curses.error` con un texto «que cabía» | Tenía caracteres anchos y ocupó más columnas de las calculadas |
| `locale.Error: unsupported locale setting` al arrancar | `LANG` pide un locale que no está instalado (míralos con `locale -a`). Prueba con `LANG=C.UTF-8` |
| Caracteres que desaparecen o salen `Ã±` | Locale no UTF-8 (`LC_ALL=C`), o archivo leído sin `encoding="utf-8"` |
| Acentos rotos al teclear | Lees con `getch()` en vez de `get_wch()` (ver [[curses - Teclado y atajos]]) |
| Bordes rotos con emojis | Ancho ambiguo: quita el emoji de zonas alineadas |

## Relacionadas

[[Cómo funciona la terminal]] · [[curses - Layout y paneles]] · [[curses - Teclado y atajos]] · [[curses - Colores y estilos]] · [[Diseño de una TUI moderna]] · [[Spinners y barras de progreso]]

## Referencias

- [curses — Python docs](https://docs.python.org/3/library/curses.html) — nota sobre `locale.setlocale(locale.LC_ALL, '')` y la codificación.
- [unicodedata — Python docs](https://docs.python.org/3/library/unicodedata.html) — `east_asian_width`, `combining`.
- [Unicode Standard Annex #11: East Asian Width](https://www.unicode.org/reports/tr11/) — anchos `W`, `F` y `A` (ambiguo).
- [wcwidth (PyPI)](https://pypi.org/project/wcwidth/) — `wcwidth` y `wcswidth` en Python.
- [UTF-8 — Wikipedia](https://en.wikipedia.org/wiki/UTF-8) — bytes por carácter.
- Probado con Python 3.13 en tmux 3.2a, con `LANG=es_ES.UTF-8` y con `LC_ALL=C`.
