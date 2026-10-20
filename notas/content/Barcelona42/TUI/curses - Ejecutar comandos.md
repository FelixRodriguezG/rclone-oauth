---
tags: [tui, curses, python, subprocess, hilos]
aliases: ["subprocess en curses", "Hilo y cola", "Lanzar vim desde curses"]
---
# curses - Ejecutar comandos

[[Inicio|← Inicio]] · [[TUI - Índice|TUI]]

> Una TUI útil ejecuta cosas: `tareas.sh`, `make`, `git`. Si lo haces mal, la interfaz se **congela** mientras el comando trabaja. La solución es lanzar el comando en un **hilo** que mete su salida en una **cola**, y que el bucle principal la vaya vaciando. Al final, cómo cederle la terminal a un programa interactivo como `vim` y recuperarla.

## Vocabulario

| Término | Qué significa |
|---|---|
| **`subprocess.run`** | Ejecuta un comando, **espera** a que acabe y te da su salida y su código |
| **`subprocess.Popen`** | Arranca un comando y **vuelve enseguida**; lees su salida mientras corre |
| **Hilo** (`threading.Thread`) | Otra línea de ejecución dentro del mismo programa, que corre a la vez que el bucle |
| **Cola** (`queue.Queue`) | Buzón seguro entre hilos: uno mete (`put`), otro saca (`get_nowait`) |
| **`endwin()`** | Devuelve la terminal a su estado normal temporalmente |

## `run` o `Popen`

| | `subprocess.run` | `subprocess.Popen` |
|---|---|---|
| Espera a que termine | Sí | No |
| Leer salida mientras corre | No | Sí (`proceso.stdout`) |
| Cancelar | No (salvo `timeout=`) | Sí (`terminate()`) |
| Úsalo para | Comandos **rápidos**: `tareas.sh añadir`, `tareas.sh lista` | Comandos **lentos**: `make`, tests, descargas |

```python
res = subprocess.run(["./tareas.sh", "lista"],    # desde la carpeta del script
                     capture_output=True, text=True)
res.returncode   # 0 si fue bien
res.stdout       # " 1. Comprar pan\n 2. Estudiar bash\n"
res.stderr       # los mensajes de error, aparte
```

Lista de argumentos, nunca `shell=True` con texto del usuario. En `tareas.py` no llames a `subprocess` directamente: usa `modelo.py` (ver [[De script a TUI - arquitectura]]), que ya busca `tareas.sh` con una ruta absoluta. Añadir y borrar son rápidos, así que basta con esto en `manejar`:

```python
try:
    modelo.borrar(self.tareas[self.sel].num)   # o modelo.anadir(texto)
    self.tareas = modelo.listar()              # recarga la lista
except modelo.ErrorTareas as e:
    self.estado = "✘ " + str(e).splitlines()[0]   # 1.ª línea del error, en la barra
```

## No congelar la interfaz: hilo + cola

```python
import curses
import os
import queue
import subprocess
import threading

LENTO = ["bash", "-c", "for i in 1 2 3 4 5; do echo paso $i; sleep 1; done"]


class App:
    def __init__(self, stdscr: curses.window) -> None:
        self.stdscr = stdscr
        self.salida: list[str] = []
        self.cola: queue.Queue[str | None] = queue.Queue()
        self.proceso: subprocess.Popen[str] | None = None
        self.estado = "r ejecutar · c cancelar · e editar · q salir"

    def lanzar(self, cmd: list[str]) -> None:
        """Arranca cmd en un hilo: el bucle principal nunca espera."""
        if self.proceso and self.proceso.poll() is None:
            return                        # ya hay uno en marcha
        self.salida.clear()
        self.proceso = subprocess.Popen(
            cmd, stdout=subprocess.PIPE, stderr=subprocess.STDOUT,
            text=True, bufsize=1)         # bufsize=1: línea a línea
        threading.Thread(target=self.leer, args=(self.proceso,),
                         daemon=True).start()
        self.estado = "ejecutando… (c cancela)"

    def leer(self, proceso: subprocess.Popen[str]) -> None:
        """Hilo lector: NO toca curses, solo mete líneas en la cola."""
        assert proceso.stdout is not None
        for linea in proceso.stdout:
            self.cola.put(linea.rstrip("\n"))
        proceso.wait()
        self.cola.put(None)               # aviso: ha terminado

    def vaciar_cola(self) -> None:
        while True:
            try:
                linea = self.cola.get_nowait()
            except queue.Empty:
                return
            if linea is None:
                assert self.proceso is not None
                self.estado = f"terminado (código {self.proceso.returncode})"
            else:
                self.salida.append(linea)

    def editar(self, fichero: str) -> None:
        """Suelta la terminal a un programa interactivo y la recupera."""
        curses.def_prog_mode()            # guarda el modo de curses
        curses.endwin()                   # terminal normal otra vez
        subprocess.run([os.environ.get("EDITOR", "vim"), fichero])
        curses.reset_prog_mode()          # vuelve al modo guardado
        self.stdscr.clear()               # fuerza a repintarlo todo

    def dibujar(self) -> None:
        self.stdscr.erase()
        alto, ancho = self.stdscr.getmaxyx()
        ultimas = self.salida[-(alto - 2):]   # «scroll»: las últimas que caben
        for i, linea in enumerate(ultimas):
            self.stdscr.addnstr(i, 0, linea, ancho - 1)
        self.stdscr.addnstr(alto - 1, 0, self.estado, ancho - 1,
                            curses.A_REVERSE)
        self.stdscr.refresh()

    def run(self) -> None:
        curses.curs_set(0)
        self.stdscr.timeout(100)          # despierta 10 veces por segundo
        while True:
            self.vaciar_cola()
            self.dibujar()
            tecla = self.stdscr.getch()
            if tecla == ord("q"):
                break
            if tecla == ord("r"):
                self.lanzar(LENTO)
            elif tecla == ord("c") and self.proceso:
                self.proceso.terminate()  # manda SIGTERM
            elif tecla == ord("e"):
                self.editar("notas.txt")
        if self.proceso and self.proceso.poll() is None:
            self.proceso.terminate()      # no dejes procesos huérfanos


if __name__ == "__main__":
    curses.wrapper(lambda stdscr: App(stdscr).run())
```

Pulsando `r`, las líneas aparecen una por segundo (`paso 1`, `paso 2`…) mientras la interfaz sigue respondiendo; al final la barra dice `terminado (código 0)`. Si pulsas `c` a mitad, dice `terminado (código -15)`: código negativo = muerto por la señal 15, `SIGTERM` (ver [[Subshells y procesos en Bash]]).

## Por qué así

- **Solo el hilo principal toca `curses`**. `curses` no es seguro entre hilos: si el hilo lector dibujara, la pantalla se corrompería. El lector solo hace `cola.put`.
- El **`timeout(100)`** hace que el bucle se despierte aunque no pulses nada, para vaciar la cola (ver [[curses - El bucle principal]]).
- `stderr=subprocess.STDOUT` mezcla errores y salida en el orden en que llegan. Si quieres pintarlos en rojo, usa dos tuberías y dos hilos.
- `daemon=True`: si sales de la TUI, el hilo no impide que Python termine. Para mover la salida con ↑/↓, reutiliza el scroll de [[curses - Layout y paneles]].

> [!warning] Programas que escriben poco a poco
> Muchos programas guardan su salida en un búfer cuando no escriben en una terminal y la sueltan toda al final. Para Python, lanza con `python3 -u`; para otros, `stdbuf -oL comando`.

## Lanzar `vim` (u otro programa interactivo) y volver

`vim`, `less` o `git commit` necesitan la terminal **entera**. Los pasos de `editar()`:

1. `def_prog_mode()`: guarda el modo actual de `curses`.
2. `endwin()`: sale de la pantalla alternativa y restaura el tty.
3. `subprocess.run([...])`: el programa usa la terminal hasta que termina.
4. `reset_prog_mode()` + `clear()`: vuelve a `curses` y repinta todo (sin `clear()` quedarían restos de `vim`).

Usa `$EDITOR` si está definido (con `vim` de respaldo). Probado: `e` abre `vim`, `ihola` + `:wq` guarda el archivo y la TUI reaparece intacta. Ojo: si `EDITOR` apunta a un programa gráfico (`xdg-open`, `code`), no bloquea la terminal y el truco no tiene sentido.

## Errores típicos

| Error | Consecuencia |
|---|---|
| `subprocess.run` de algo lento en el bucle | La TUI se congela hasta que termina |
| Dibujar desde el hilo lector | Pantalla corrupta o cuelgues |
| `getch()` sin `timeout` | La salida solo aparece cuando pulsas una tecla |
| No esperar (`wait`) al proceso | Procesos zombi; `returncode` es `None` |
| Salir sin `terminate()` | El comando sigue corriendo en segundo plano |

## Relacionadas

[[De script a TUI - arquitectura]] · [[curses - El bucle principal]] · [[curses - Layout y paneles]] · [[Spinners y barras de progreso]] · [[Subshells y procesos en Bash]] · [[Códigos de salida y errores en Bash]] · [[Vim]] · [[Frameworks de TUI - Listado]]

## Referencias

- [subprocess — Python docs](https://docs.python.org/3/library/subprocess.html) — `run`, `Popen`, `PIPE`, `STDOUT`, `bufsize`, `terminate`, `returncode` negativo.
- [queue — Python docs](https://docs.python.org/3/library/queue.html) — `Queue`, `put`, `get_nowait`, `Empty`.
- [threading — Python docs](https://docs.python.org/3/library/threading.html) — `Thread`, `daemon`.
- [curses — Python docs](https://docs.python.org/3/library/curses.html) — `def_prog_mode`, `reset_prog_mode`, `endwin`.
- [ncurses: curs_kernel(3x)](https://invisible-island.net/ncurses/man/curs_kernel.3x.html) — `def_prog_mode` y salidas temporales al shell.
- [man 1 stdbuf](https://man7.org/linux/man-pages/man1/stdbuf.1.html) — salida por líneas.
- Probado con Python 3.13 en tmux 3.2a: salida en vivo, cancelación (`-15`) y `vim` con `:wq`.
