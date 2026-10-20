---
tags: [herramientas, terminal, demo, gif, asciinema, tmux]
aliases: ["asciinema", "agg", "vhs", "GIF de la terminal"]
---
# Grabar una demo de terminal

[[Inicio|← Inicio]] · Herramientas

> Para enseñar un script o una TUI en un README, lo mejor es un **GIF animado** de la terminal. El camino más fiable: **asciinema** graba la sesión como texto (`.cast`), **agg** la convierte en GIF y, si quieres que salga igual cada vez, **tmux** «teclea» por ti.

## Vocabulario

| Término | Qué significa |
|---|---|
| **asciinema** | Programa que graba lo que sale por la terminal (texto + tiempos), no píxeles |
| **`.cast`** (*asciicast*) | Fichero de la grabación: JSON, una línea por cada trozo de salida con su instante |
| **agg** | *asciinema gif generator*: convierte un `.cast` en `.gif` |
| **vhs** | Alternativa que graba a partir de un guion (`.tape`) |
| **Tiempo muerto** (*idle time*) | Ratos sin salida (pensando, leyendo). Se recortan para que la demo no aburra |
| **musl / glibc** | Dos versiones de la librería estándar de C en Linux. Un binario «musl» no depende de la glibc del sistema |

## 1. Grabar con asciinema

```bash
pip install --user asciinema      # sin sudo; queda en ~/.local/bin
asciinema --version               # → asciinema 2.4.0
# Si ~/.local/bin no está en el PATH: python3 -m asciinema --version
```

```bash
asciinema rec demo.cast                 # graba tu shell; sal con exit o Ctrl-d
asciinema rec -c './tareas.sh lista' demo.cast   # graba solo ese comando
asciinema rec -i 1 --cols 120 --rows 30 demo.cast   # recorta pausas a 1 s, tamaño fijo
asciinema play demo.cast                # reproducirla en la terminal
```

- `-c` (*command*): qué grabar; al terminar ese comando, termina la grabación.
- `-i` / `--idle-time-limit`: ninguna pausa durará más de N segundos.
- `--overwrite`: sobrescribe el fichero si ya existe.

El `.cast` es texto, así que puedes **editarlo**: la primera línea es la cabecera (tamaño, fecha) y cada línea siguiente es `[segundos, "o", "texto"]`.

> [!note] asciinema 3
> La versión 3 está reescrita en Rust y se instala como binario (no con `pip`); `rec`, `play`, `-c` e `-i` funcionan igual. Esta nota se probó con la 2.4.0.

## 2. Convertir a GIF con agg

agg es un único binario: descárgalo de *Releases* en `github.com/asciinema/agg`, dale permisos y ponlo en `~/.local/bin`:

```bash
chmod +x agg-x86_64-unknown-linux-gnu
mv agg-x86_64-unknown-linux-gnu ~/.local/bin/agg
```

> [!warning] `GLIBC_2.xx not found`
> Si tu Linux tiene una glibc antigua (mírala con `ldd --version`), el binario `…-linux-gnu` no arranca. Descarga el **`…-linux-musl`**: está enlazado de forma estática y funciona en cualquier distribución.

```bash
agg --font-family "DejaVu Sans Mono,Noto Color Emoji" \
    --font-size 16 --theme dracula \
    --idle-time-limit 2 --last-frame-duration 3 \
    demo.cast docs/demo.gif
```

| Opción | Para qué |
|---|---|
| `--font-family` | Lista de fuentes. La **primera debe ser monoespaciada** (todas las letras igual de anchas); añade `Noto Color Emoji` para que los emojis no salgan como cuadrados (ver [[Unicode y emojis en la terminal]]) |
| `--font-size` | Tamaño en píxeles; 14-16 se lee bien en GitHub |
| `--theme` | Colores: `asciinema`, `dracula`, `monokai`, `nord`, `github-dark`, `solarized-light`… |
| `--idle-time-limit` | Igual que en `rec`: recorta pausas (por defecto 5 s) |
| `--last-frame-duration` | Cuánto se queda quieto el último fotograma antes de repetir (por defecto 3 s) |
| `--speed` | `1.5` = un 50 % más rápido |

Las fuentes disponibles se ven con `fc-list | grep -i mono`. En versiones recientes también existen `--text-font-family` y `--emoji-font-family` por separado.

## 3. Demo reproducible con tmux

Grabar a mano sale con erratas y pausas. Mejor un script que **teclee por ti** con `tmux send-keys` (ver [[Automatizar tmux desde scripts]]) mientras asciinema graba la sesión de tmux:

```bash
#!/usr/bin/env bash
# grabar_demo.sh — graba una demo reproducible: tmux "teclea" y asciinema graba.
set -euo pipefail

SOCK="demo"                       # servidor tmux aislado: no toca tus sesiones
SALIDA="${1:-demo.cast}"

teclear() {                       # escribe un texto letra a letra, como una persona
	local texto="$1" i
	for (( i = 0; i < ${#texto}; i++ )); do
		tmux -L "$SOCK" send-keys -t demo -l "${texto:i:1}"
		sleep 0.05
	done
	sleep 0.3
	tmux -L "$SOCK" send-keys -t demo Enter
}

# Shell limpia (sin tu .bashrc) y prompt neutro: nada de usuario ni hostname en el vídeo.
tmux -L "$SOCK" -f /dev/null new-session -d -s demo -x 100 -y 25 \
	"env PS1='$ ' bash --norc --noprofile"
tmux -L "$SOCK" set -g status off  # sin barra de estado en el vídeo

# El "guion" de la demo, en segundo plano mientras se graba.
(
	sleep 1
	teclear 'echo "Hola, 42"'
	sleep 1
	teclear 'ls'
	sleep 2
	tmux -L "$SOCK" kill-server   # al cerrar tmux, termina la grabación
) &

asciinema rec --overwrite -q -i 1 --cols 100 --rows 25 \
	-c "tmux -L $SOCK attach -t demo" "$SALIDA"
wait

# Al cerrarse, tmux limpia la pantalla y escribe "[server exited]": quitamos esas 2 últimas líneas.
head -n -2 "$SALIDA" > "$SALIDA.tmp" && mv "$SALIDA.tmp" "$SALIDA"
```

```bash
./grabar_demo.sh docs/demo.cast
agg --theme dracula --font-size 16 docs/demo.cast docs/demo.gif
```

> [!info] Verificado
> Probado con tmux 3.2a, asciinema 2.4.0 y agg 1.9.0: genera un `.cast` de 100×25 y un GIF de ~16 KB cuyo último fotograma muestra el `echo` y el `ls`, sin barra de estado ni `[server exited]`.

Claves del script: servidor tmux **aislado** (`-L demo`) y sin tu configuración (`-f /dev/null`), shell sin `.bashrc` y con `PS1='$ '` (no sale tu usuario ni tu ordenador), tecleo letra a letra con `send-keys -l`, y recorte del final.

## Alternativa: vhs

[vhs](https://github.com/charmbracelet/vhs) graba a partir de un guion `.tape`, sin tmux:

```
Output docs/demo.gif
Set FontSize 16
Set Width 1200
Set Height 600
Type "./tareas.sh lista"
Enter
Sleep 2s
```

`vhs demo.tape` genera el GIF directamente. Necesita `ttyd` y `ffmpeg` instalados, que en un ordenador sin `sudo` es más complicado; asciinema + agg son dos ejecutables sueltos.

## Consejos

- **Tamaño**: 120×30 (o 100×25) es buen equilibrio: cabe la TUI y el texto se lee en el README.
- **Quita datos personales**: prompt neutro, nada de `~/` con tu login, ni tokens ni correos en pantalla.
- **Recorta el final**: que el último fotograma sea el resultado, no una pantalla vacía. Edita el `.cast` (borra las últimas líneas) o usa `agg --select 2..12` para quedarte solo con los segundos 2 a 12 (opción de las versiones recientes de agg).
- **Peso del GIF**: por debajo de ~5 MB. Reduce duración, tamaño de terminal o `--fps-cap`; evita animaciones que repintan toda la pantalla.
- **Corta**: 10-20 segundos enseñando lo esencial.

## Errores típicos

- **Emojis como cuadrados** o letras desalineadas: fuente no monoespaciada o sin fuente de emojis.
- **`open terminal failed: not a terminal`**: lanzas el script sin terminal; asciinema y `tmux attach` necesitan una.
- **La demo se graba con tu prompt y tus alias**: usa `bash --norc --noprofile`.

## Relacionadas

[[Automatizar tmux desde scripts]] · [[README de un proyecto]] · [[GitHub CLI (gh)]] · [[Unicode y emojis en la terminal]] · [[tmux - Índice]]

## Referencias

- [asciinema docs](https://docs.asciinema.org/) — `rec` (`-c`, `-i`, `--cols`, `--rows`), formato asciicast.
- [asciinema/agg](https://github.com/asciinema/agg) — binarios gnu/musl, `--font-family`, `--theme`, `--idle-time-limit`, `--last-frame-duration`, `--select` (también `agg --help`).
- [charmbracelet/vhs](https://github.com/charmbracelet/vhs) — formato `.tape` y dependencias.
- [man tmux](https://man.openbsd.org/tmux) — `send-keys -l`, `-L`, `-f`.
