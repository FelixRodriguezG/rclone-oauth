---
tags: [tui, instalador, python, curses]
aliases: ["Instalador con curses", "Asistente de instalación en Python"]
---
# Instalador TUI en Python

[[Inicio|← Inicio]] · [[TUI - Índice|TUI]]

> El mismo asistente que en [[Instalador TUI en Bash]], pero hecho a mano con curses: las pantallas son una **lista de pasos**, un índice dice en cuál estás y cada tecla mueve ese índice. Así hay navegación **adelante y atrás**, un campo de texto, una checklist y una barra de progreso que avanza **sin congelar** la interfaz gracias a un hilo.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Máquina de estados** | Programa que está siempre en **un** estado (aquí, un paso) y pasa a otro según los eventos (teclas) |
| **Hilo** (*thread*) | Otra línea de ejecución dentro del mismo programa; aquí copia ficheros mientras el bucle dibuja |

## La idea: estado + teclas + dibujo

`PASOS = bienvenida → carpeta → componentes → resumen → instalando → fin`. **Enter** hace `paso += 1`, **←** hace `paso -= 1` y **Esc** cancela.

`tecla(k)` y `actualizar()` **solo cambian el estado** (paso, texto, marcas); `dibujar()` **solo lo lee** y pinta el paso actual. Es la arquitectura de [[De script a TUI - arquitectura]] y [[curses - El bucle principal]]. El hilo se arranca al pulsar Enter en el resumen, con `daemon=True` para que muera si el programa termina; `actualizar()` pasa a «fin» cuando el hilo acaba.

## El código

```python
#!/usr/bin/env python3
"""Asistente de instalación en curses. Enter: siguiente · ←: atrás · Esc: cancelar."""
import curses
import os
import threading
import time
from pathlib import Path

os.environ.setdefault("ESCDELAY", "25")      # Esc responde en 25 ms, no en 1 s
PASOS = ["bienvenida", "carpeta", "componentes", "resumen", "instalando", "fin"]


def linea(w: curses.window, y: int, texto: str, attr: int = 0) -> None:
    """Escribe en la fila y, recortando para no salirse del borde."""
    ancho = w.getmaxyx()[1] - 4
    w.addstr(y, 2, texto[-ancho:] if len(texto) > ancho else texto, attr)


class Asistente:
    def __init__(self) -> None:
        self.paso = 0
        self.carpeta = str(Path.home() / ".local")
        self.comps = {"programa": True, "config": True, "ejemplos": False}
        self.cursor = 0                       # fila marcada en la checklist
        self.progreso = 0.0
        self.error: str | None = None
        self.hilo: threading.Thread | None = None

    def instalar(self) -> None:               # se ejecuta en otro hilo
        elegidos = [c for c, si in self.comps.items() if si]
        try:
            for i, comp in enumerate(elegidos, start=1):
                destino = Path(self.carpeta, "bin" if comp == "programa" else comp)
                destino.mkdir(parents=True, exist_ok=True)
                (destino / f"{comp}.txt").write_text("instalado\n")
                time.sleep(0.5)               # simula trabajo lento
                self.progreso = i / len(elegidos)
        except OSError as e:
            self.error = str(e)

    def tecla(self, k: int) -> bool:          # devuelve False para salir
        nombre = PASOS[self.paso]
        if k == 27:                           # Esc
            return nombre == "instalando"     # no se cancela a medias
        if k == curses.KEY_LEFT and nombre not in ("instalando", "fin"):
            self.paso = max(0, self.paso - 1)
        elif nombre == "carpeta" and k in (curses.KEY_BACKSPACE, 127, 8):
            self.carpeta = self.carpeta[:-1]
        elif nombre == "carpeta" and 32 <= k < 127:
            self.carpeta += chr(k)
        elif nombre == "componentes" and k in (curses.KEY_UP, curses.KEY_DOWN):
            self.cursor = (self.cursor + (1 if k == curses.KEY_DOWN else -1)) % 3
        elif nombre == "componentes" and k == ord(" "):
            clave = list(self.comps)[self.cursor]
            self.comps[clave] = not self.comps[clave]
        elif k in (10, 13, curses.KEY_ENTER):
            if nombre == "fin":
                return False
            if nombre == "carpeta" and not self.carpeta.strip():
                self.error = "La carpeta no puede estar vacía"
            elif nombre == "componentes" and not any(self.comps.values()):
                self.error = "Marca al menos un componente"
            elif nombre != "instalando":
                self.error = None
                self.paso += 1
                if nombre == "resumen":       # arrancar la copia (una sola vez)
                    self.hilo = threading.Thread(target=self.instalar, daemon=True)
                    self.hilo.start()
        return True

    def actualizar(self) -> None:             # sin tecla: ¿ha acabado el hilo?
        if self.hilo and PASOS[self.paso] == "instalando" and not self.hilo.is_alive():
            self.paso += 1

    def dibujar(self, w: curses.window) -> None:
        w.erase()
        w.box()
        linea(w, 0, f" Instalador · paso {self.paso + 1}/{len(PASOS)} ")
        nombre = PASOS[self.paso]
        if nombre == "bienvenida":
            linea(w, 2, "Bienvenido. Se instalará «tareas» sin sudo.")
        elif nombre == "carpeta":
            linea(w, 2, "Carpeta de instalación:")
            linea(w, 3, self.carpeta + "▏", curses.A_REVERSE)
        elif nombre == "componentes":
            for i, (c, si) in enumerate(self.comps.items()):
                attr = curses.A_REVERSE if i == self.cursor else curses.A_NORMAL
                linea(w, 2 + i, f"[{'x' if si else ' '}] {c}", attr)
        elif nombre == "resumen":
            elegidos = ", ".join(c for c, si in self.comps.items() if si)
            linea(w, 2, f"Carpeta:     {self.carpeta}")
            linea(w, 3, f"Componentes: {elegidos or '(ninguno)'}")
            linea(w, 5, "Enter para instalar.")
        elif nombre == "instalando":
            llenos = int(self.progreso * 30)
            linea(w, 2, f"[{'█' * llenos}{'░' * (30 - llenos)}] {self.progreso:4.0%}")
        else:                                 # fin
            linea(w, 2, f"✘ {self.error}" if self.error else "✔ Instalación completada")
        if self.error and nombre != "fin":
            linea(w, 6, f"✘ {self.error}", curses.A_BOLD)
        w.addstr(w.getmaxyx()[0] - 1, 2, " Enter siguiente · ← atrás · Esc cancelar ")
        w.refresh()


def main(stdscr: curses.window) -> str:
    curses.curs_set(0)
    stdscr.keypad(True)                       # flechas → KEY_LEFT, etc.
    stdscr.timeout(100)                       # redibujar cada 100 ms (progreso)
    app = Asistente()
    while True:
        app.dibujar(stdscr)
        k = stdscr.getch()
        if k != -1 and not app.tecla(k):
            break
        app.actualizar()
    return "ok" if PASOS[app.paso] == "fin" and not app.error else "cancelado"


if __name__ == "__main__":
    print(curses.wrapper(main))
```

Pantalla del paso 3 (componentes):

```text
┌─ Instalador · paso 3/6 ──────────────────────────┐
│                                                  │
│ [x] programa                                     │
│ [x] config                                       │
│ [ ] ejemplos                                     │
│                                                  │
└─ Enter siguiente · ← atrás · Esc cancelar ───────┘
```

Los componentes son ficheros de prueba (`programa.txt`…); en el proyecto real copiarías `tareas.py`, `modelo.py` y `tareas.sh`. Probarlo sin tocar tu `~/.local`: `HOME=$(mktemp -d) python3 instalar.py` (la carpeta por defecto sale de `Path.home()`, que usa `$HOME`).

> [!warning] Hilos y curses
> **Solo el hilo principal dibuja.** El hilo de instalación únicamente cambia `self.progreso` y `self.error`; el bucle los lee cada 100 ms. Si dos hilos llaman a `addstr` a la vez, la pantalla se corrompe. Para lanzar comandos externos en vez de copiar, ver [[curses - Ejecutar comandos]].

## Errores típicos

| Error | Consecuencia |
|---|---|
| `getch()` bloqueante en el paso «instalando» | La barra solo avanza al pulsar teclas |
| Sin `ESCDELAY` (Esc espera por si es una flecha, 1000 ms por defecto) | Esc tarda un segundo en reaccionar |
| Olvidar `keypad(True)` | Las flechas llegan como 3 códigos sueltos (`27`, `91`, `68`) |
| `addstr` con una ruta más larga que la ventana | `_curses.error`: por eso existe `linea()` |
| Redirigir la salida (`python3 instalar.py > log`) | curses dibuja en stdout: pantalla en blanco |

## Relacionadas

[[Instalador TUI en Bash]] · [[Qué hace un buen instalador]] · [[curses - El bucle principal]] · [[curses - Teclado y atajos]] · [[curses - Ejecutar comandos]] · [[curses - Animaciones]] · [[De script a TUI - arquitectura]] · [[Spinners y barras de progreso]] · [[Instalar con una línea]]

## Referencias

- [Python curses](https://docs.python.org/3/library/curses.html) — `keypad`, `timeout`, `KEY_LEFT`, `KEY_BACKSPACE`, `set_escdelay`.
- [ncurses: ESCDELAY](https://invisible-island.net/ncurses/man/curs_variables.3x.html) — retraso por defecto de 1000 ms tras `Esc`.
- [Python threading](https://docs.python.org/3/library/threading.html) — `Thread(daemon=True)`, `is_alive()`.
- Probado con Python 3.13 en tmux 70×12: escribir carpeta, volver atrás con ←, marcar «ejemplos», instalar los tres componentes en un `HOME` temporal (`ok`) y cancelar con Esc (`cancelado`); tras la revisión, también el aviso al no marcar ningún componente.
