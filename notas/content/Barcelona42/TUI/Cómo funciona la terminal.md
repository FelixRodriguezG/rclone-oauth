---
tags: [tui, terminal, tty, stty, secuencias-escape, terminfo]
aliases: ["Terminal por dentro", "tty", "Modo raw", "Secuencias de escape ANSI"]
---
# Cómo funciona la terminal

[[Inicio|← Inicio]] · [[TUI - Índice|TUI]]

> Una TUI no dibuja píxeles: escribe **texto y códigos especiales** en su salida estándar, y la terminal los interpreta como «mueve el cursor», «pinta en verde» o «borra la pantalla». Entender esta capa explica casi todos los misterios de `curses`: por qué hay que restaurar la terminal, por qué Esc tarda o por qué una flecha son tres bytes.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Emulador de terminal** | El programa con ventana que ves: GNOME Terminal, iTerm2, Kitty, Alacritty. Dibuja los caracteres y te pasa las teclas |
| **tty / pty** | El «cable» virtual entre el emulador y tus programas. `tty` te dice cuál usas: `/dev/pts/5` (*pseudo-terminal*) |
| **Shell** | Programa que lee tus órdenes y las ejecuta (`bash`, `zsh`). **No** es la terminal: corre dentro de ella |
| **stdin / stdout** | Descriptores 0 y 1: por donde el programa lee teclas y escribe texto (ver [[Descriptores de fichero y read]]) |
| **Line discipline** | Capa del núcleo dentro del tty que decide si las teclas llegan una a una o por líneas, si hay eco, qué hace Ctrl+C… |
| **Secuencia de escape** | Bytes que empiezan por `ESC` (`\e`, `\033`, `0x1b`) y que la terminal interpreta como órdenes, no como texto |

```text
 teclado ─► emulador ─► pty (line discipline) ─► stdin  ┐
                                                         ├─ tu programa (bash, python3 tareas.py)
 pantalla ◄─ emulador ◄─ pty ◄──────────────── stdout ◄──┘
```

## Modo canónico y modo raw

Por defecto el tty está en **modo canónico** (o «cocinado»): guarda lo que escribes hasta que pulsas **Enter**, te deja borrar con Backspace y repite cada tecla en pantalla (**eco**). Es perfecto para `read` en Bash, pero una TUI necesita cada tecla **al momento** y sin eco: eso es el **modo raw** (crudo) o casi raw. El comando `stty` cambia estos ajustes:

```console
$ stty -a | head -2
speed 38400 baud; rows 30; columns 100; line = 0;
intr = ^C; quit = ^\; erase = ^?; kill = ^U; eof = ^D; eol = <undef>;
```

```bash
#!/usr/bin/env bash
# Lee teclas una a una, sin esperar a Enter y sin eco (modo «casi raw»)
viejo="$(stty -g)"                       # guarda la configuración actual
trap 'stty "$viejo"' EXIT                # y la restaura al salir, pase lo que pase
stty -icanon -echo                       # sin modo línea y sin eco
echo "Pulsa teclas (q para salir)"
while true; do
    tecla="$(dd bs=1 count=1 2>/dev/null)"
    [[ "$tecla" == "q" ]] && break
    printf 'byte: %q\n' "$tecla"
done
```

Pulsando `a`, `↑` y `q`:

```text
byte: a
byte: $'\E'
byte: \[
byte: A
```

¡La flecha arriba son **tres bytes**: `ESC [ A`! Por eso un programa necesita una biblioteca que traduzca esas secuencias a «tecla arriba» (en `curses`, `keypad(True)`; ver [[curses - Teclado y atajos]]). El `trap ... EXIT` es imprescindible: si el script muere sin restaurar, tu terminal se queda sin eco (ver [[Scripts robustos con set y trap]]).

> [!tip] ¿La terminal se ha quedado rara?
> Escribe `reset` a ciegas y pulsa Enter (o `stty sane`). Restaura modo canónico, eco y colores.

## Secuencias de escape

Las más usadas son las **ANSI/VT100** (`ESC [` + parámetros + letra). Los colores se ven en detalle en [[Colores en la terminal]].

| Secuencia | Efecto |
|---|---|
| `\e[2J` | Borra toda la pantalla |
| `\e[H` / `\e[5;10H` | Cursor a fila 1 columna 1 / a fila 5 columna 10 (empiezan en **1**) |
| `\e[K` | Borra desde el cursor hasta el final de la línea |
| `\e[1m`, `\e[32m`, `\e[0m` | Negrita, texto verde, quitar todos los estilos |
| `\e[?25l` / `\e[?25h` | Ocultar / mostrar el cursor |
| `\e[?1049h` / `\e[?1049l` | Entrar / salir de la **pantalla alternativa** |

```bash
printf '\e[?1049h'          # entra en la pantalla alternativa
printf '\e[2J\e[H'          # borra la pantalla y lleva el cursor a (1,1)
printf '\e[5;10H'           # cursor a fila 5, columna 10
printf '\e[1;32mHola\e[0m'  # negrita + verde, y luego reset
read -r                     # espera Enter
printf '\e[?1049l'          # vuelve a la pantalla normal, intacta
```

La **pantalla alternativa** es un segundo búfer de pantalla: al entrar, tu historial desaparece; al salir, vuelve tal cual estaba. Es lo que hacen `vim`, `less` o `htop`, y lo que hace `curses` al arrancar.

## Tamaño: filas, columnas y SIGWINCH

La terminal mide **filas × columnas** de celdas (no píxeles). Puedes preguntarlo con `stty size` (`30 100`) o `tput lines` / `tput cols`. Cuando el usuario cambia el tamaño de la ventana, el núcleo envía la señal **SIGWINCH** (*window change*) al programa:

```bash
trap 'echo "nuevo tamaño: $(tput lines)x$(tput cols)"' WINCH
while true; do sleep 1; done
```

En `curses` esa señal llega como la tecla especial `KEY_RESIZE` (ver [[curses - Layout y paneles]]).

## TERM y terminfo

Cada emulador entiende secuencias un poco distintas. La variable **`TERM`** dice qué tipo de terminal eres (`xterm-256color`, `screen`, `tmux-256color`) y la base de datos **terminfo** guarda, para cada tipo, qué secuencia hace cada cosa y cuántos colores hay:

```console
$ echo "$TERM"
xterm-256color
$ infocmp -1 xterm-256color | grep -E 'colors|kcuu1'
	colors#0x100,
	kcuu1=\EOA,
```

`colors#0x100` = 256 colores; `kcuu1` = lo que envía la flecha arriba. `curses` y `tput` leen terminfo por ti: así un mismo programa funciona en cualquier terminal. Si `TERM` está mal (por ejemplo `screen` dentro de un tmux moderno), verás solo 8 colores o teclas que no responden (ver [[Configurar tmux]]).

## Por qué Esc tarda

Esc envía el byte `ESC`… que también es el **primer byte** de las flechas y teclas de función. Al recibirlo, `curses` no sabe si pulsaste Esc o si vienen más bytes, así que **espera**: por defecto `ESCDELAY` = **1000 ms**. Por eso Esc «tarda un segundo» en muchas TUI. Se acorta con la variable de entorno `ESCDELAY=25` o, en Python 3.9+, con `curses.set_escdelay(25)` (ver [[curses - Teclado y atajos]]). En tmux pasa lo mismo con `escape-time`.

## Errores típicos

| Error | Causa y solución |
|---|---|
| Tras un fallo no se ve lo que escribes | El programa no restauró el tty: `reset`. En `curses`, usa `curses.wrapper` |
| Salen `^[[A` al pulsar flechas | Estás en modo canónico y nadie traduce las secuencias (`cat -v` lo muestra a propósito) |
| Los colores o el tamaño salen mal en tmux | `TERM` incorrecto dentro de tmux |
| `curses` dice `setupterm: could not find terminal` | `TERM` vacío o desconocido (pasa en algunos IDE o en `cron`) |

## Relacionadas

[[Colores en la terminal]] · [[curses - Primeros pasos]] · [[curses - Teclado y atajos]] · [[Leer entrada del usuario en Bash]] · [[Scripts robustos con set y trap]] · [[Descriptores de fichero y read]] · [[Qué es tmux]] · [[Animación en la terminal - fundamentos]] · [[Unicode y emojis en la terminal]] · [[De script a TUI - arquitectura]] · [[Configurar tmux]]

## Referencias

- [man 1 stty](https://man7.org/linux/man-pages/man1/stty.1.html) — `-icanon`, `-echo`, `-g`, `sane`, `size`.
- [man 3 termios](https://man7.org/linux/man-pages/man3/termios.3.html) — modo canónico y no canónico, line discipline.
- [ANSI escape code — Wikipedia](https://en.wikipedia.org/wiki/ANSI_escape_code) — tabla de secuencias CSI y SGR.
- [XTerm Control Sequences](https://invisible-island.net/xterm/ctlseqs/ctlseqs.html) — pantalla alternativa `?1049`, cursor `?25`.
- [man 5 terminfo](https://man7.org/linux/man-pages/man5/terminfo.5.html) e `infocmp(1)` — capacidades `colors`, `kcuu1`, `smcup`/`rmcup`.
- [ncurses: ESCDELAY](https://invisible-island.net/ncurses/man/curs_variables.3x.html) — valor por defecto de 1000 ms.
- [man 7 signal](https://man7.org/linux/man-pages/man7/signal.7.html) — `SIGWINCH`.
- Ejemplos probados en tmux 3.2a (`TERM=screen`) sobre Linux; la salida de `cat -v`/`dd` y `stty -a` es real.
