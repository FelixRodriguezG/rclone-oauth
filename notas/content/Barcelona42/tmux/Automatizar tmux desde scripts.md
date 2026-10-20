---
tags: [tmux, bash, automatizacion, scripts]
aliases: ["Scripts de tmux", "tmux send-keys", "tmux capture-pane"]
---
# Automatizar tmux desde scripts

[[Inicio|← Inicio]] · [[tmux - Índice|tmux]]

> Cada atajo de tmux es en realidad un **comando** que puedes lanzar desde la shell: `tmux split-window`, `tmux send-keys`… Así, un script de Bash puede montar tu entorno de trabajo entero, escribir en los paneles por ti y **leer** lo que muestran. Es la base para probar TUIs y grabar demos reproducibles.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Desacoplado** (`-d`) | Crear la sesión sin conectarte a ella; queda en segundo plano |
| **Pane id** (`%3`) | Identificador único de un panel. No cambia aunque reordenes paneles: el más seguro para scripts |
| **Formato** (`-F`) | Plantilla `#{…}` que tmux rellena con datos (ver [[Configurar tmux]]) |
| **Socket** (`-L nombre`) | Canal de comunicación con un servidor tmux. Otro nombre = **otro servidor**, aislado del tuyo |
| **`$TMUX`** | Variable que tmux define dentro de sus paneles. Si existe, ya estás dentro de tmux |

## Los comandos clave

```bash
# Sesión nueva, desacoplada, con nombre, carpeta inicial y tamaño fijo (columnas x filas)
tmux new-session -d -s trabajo -c ~/libft -x 160 -y 40

# Dividir: -h = izquierda|derecha, -v = arriba/abajo; -l = tamaño del panel NUEVO
# -P imprime info del panel creado y -F elige qué: aquí, su id
der="$(tmux split-window -h -l 40% -t trabajo -P -F '#{pane_id}')"
echo "$der"                          # → %1

tmux send-keys -t "$der" 'make' Enter   # «teclea» make y pulsa Intro
tmux select-pane -t trabajo.0           # pone el foco en el panel 0
tmux has-session -t trabajo             # código de salida 0 si existe, 1 si no
tmux capture-pane -p -t "$der"          # imprime en stdout lo que se ve en el panel
```

> [!note] `-x`/`-y` solo importan mientras nadie mira
> Una sesión desacoplada no tiene terminal, así que tmux usa el tamaño que le digas (por defecto 80×24). En cuanto haces `attach`, la ventana se adapta al tamaño de tu terminal y los paneles se reparten **en proporción**.

### `send-keys`: escribir como si tecleases

- Cada argumento es texto o un nombre de tecla: `Enter`, `Escape`, `C-c` (Ctrl-c), `Up`…
- `-l` (literal) envía el texto tal cual, sin interpretar nombres de teclas: `send-keys -l 'Enter'` escribe la palabra.
- Para interrumpir lo que corre en un panel: `tmux send-keys -t "$der" C-c`.

> [!danger] El `;` final se lo come tmux
> Para tmux, un argumento que **termina** en `;` separa dos comandos (como en la shell). Probado en tmux 3.2a:
> ```bash
> tmux send-keys -t t 'echo uno; echo dos' Enter  # bien: el ; va en medio
> tmux send-keys -t t 'echo tres;' Enter          # → unknown command: Enter
> tmux send-keys -t t 'echo tres\;' Enter         # bien: escribe «echo tres;»
> ```
> Si un argumento acaba en `;`, escríbelo como `'\;'` (entre comillas simples, para que la barra llegue a tmux).

### `capture-pane`: leer la pantalla

`capture-pane -p` saca por la salida estándar el texto visible de un panel, así que puedes filtrarlo con [[grep, sed y awk lo justo|grep]]:

```bash
tmux send-keys -t "$der" 'make' Enter
sleep 2
if tmux capture-pane -p -t "$der" | grep -q 'Error'; then
	echo "La compilación ha fallado"
fi
```

Opciones útiles: `-S -100` incluye 100 líneas de historial; `-e` conserva los códigos de color; `-J` une líneas partidas. Es la forma de **probar una TUI** desde un script: lanzarla en un panel, enviarle teclas y comprobar lo que dibuja (ver [[curses - Primeros pasos]]).

### Sockets aislados con `-L`

```bash
tmux -L prueba new-session -d -s demo   # servidor nuevo, independiente del tuyo
tmux -L prueba ls                       # solo ve sus sesiones
tmux -L prueba kill-server              # lo cierra sin tocar tus sesiones
```

Úsalo para experimentar, para tests automáticos y para grabar demos ([[Grabar una demo de terminal]]). Añade `-f /dev/null` para arrancar **sin** tu `~/.tmux.conf` (entorno limpio y reproducible).

### ¿Ya estoy dentro de tmux?

```bash
if [[ -n "${TMUX:-}" ]]; then
	tmux switch-client -t trabajo   # dentro: cambiar de sesión (attach anidaría tmux)
else
	tmux attach-session -t trabajo  # fuera: conectarse
fi
```

## Ejemplo completo: editor a la izquierda, herramienta a la derecha

`entorno.sh`: crea la sesión si no existe (si existe, la reutiliza) y te conecta.

```bash
#!/usr/bin/env bash
# entorno.sh — monta (o reutiliza) una sesión: editor a la izquierda, herramienta a la derecha.
# Uso: ./entorno.sh [carpeta]
set -euo pipefail

SESION="trabajo"
CARPETA="${1:-$PWD}"
SOCKET="${TMUX_SOCKET:-default}"     # cámbialo (TMUX_SOCKET=prueba) para probar aislado

t() { tmux -L "$SOCKET" "$@"; }      # atajo: todas las órdenes van al mismo servidor

# 1. ¿Ya existe la sesión? Entonces solo nos conectamos.
if ! t has-session -t "$SESION" 2>/dev/null; then
	# 2. Sesión nueva, desacoplada (-d), con tamaño fijo para que el reparto sea predecible.
	t new-session -d -s "$SESION" -c "$CARPETA" -x 160 -y 40
	izq="$(t display-message -p -t "$SESION" '#{pane_id}')"

	# 3. Partimos en vertical (-h): el panel nuevo, a la derecha, ocupa el 40 %.
	der="$(t split-window -h -l 40% -t "$izq" -c "$CARPETA" -P -F '#{pane_id}')"

	# 4. Escribimos órdenes en cada panel como si tecleáramos (Enter = pulsar Intro).
	t send-keys -t "$izq" 'vim .' Enter
	t send-keys -t "$der" 'ls -l; echo "listo"' Enter

	# 5. El foco empieza en el editor.
	t select-pane -t "$izq"
fi

# 6. Conectarse: si ya estamos dentro de tmux, cambiar de sesión; si no, attach.
if [[ -n "${TMUX:-}" ]]; then
	t switch-client -t "$SESION"
else
	t attach-session -t "$SESION"
fi
```

```bash
chmod +x entorno.sh
./entorno.sh ~/libft       # la primera vez crea; las siguientes, solo entra
```

Resultado (160×40): Vim en `%0` (ancho 95) y la shell con el `ls` en `%1` (ancho 64).

> [!info] Cómo se verificó
> Con `TMUX_SOCKET=verif ./entorno.sh demo` sobre un servidor aislado: `tmux -L verif list-panes -F '#{pane_id} #{pane_width} #{pane_current_command}'` mostró `%0 95 vim` y `%1 64 zsh`, `capture-pane` mostró la salida del `ls` y `listo`, y una segunda ejecución reutilizó la sesión en lugar de crear otra. Al final, `tmux -L verif kill-server`.

Por qué está escrito así:

- `set -euo pipefail` y `has-session … 2>/dev/null` dentro del `if`: el fallo esperado de `has-session` no detiene el script (ver [[Scripts robustos con set y trap]]).
- Ids de panel (`%0`, `%1`) en vez de números: no dependen de `pane-base-index` ni del orden.
- La función `t` centraliza el `-L`: cambiar de servidor es tocar una variable ([[Funciones en Bash]]).

## Errores típicos

- **`open terminal failed: not a terminal`**: haces `attach` desde algo sin terminal (cron, una tubería). Crea con `-d` y no hagas attach.
- **`duplicate session: trabajo`**: no comprobaste `has-session` antes de `new-session`.
- **`send-keys` antes de que el programa arranque**: las teclas llegan a la shell. Añade un `sleep` corto o comprueba con `capture-pane`.
- **Olvidar matar los servidores de prueba**: `tmux -L prueba kill-server`.

## Relacionadas

[[tmux - Índice]] · [[tmux - Uso básico]] · [[Configurar tmux]] · [[Primer script en Bash]] · [[Condiciones en Bash]] · [[Códigos de salida y errores en Bash]] · [[Grabar una demo de terminal]] · [[curses - Ejecutar comandos]]

## Referencias

- [man tmux (OpenBSD)](https://man.openbsd.org/tmux) — `new-session` (`-d -s -c -x -y`), `split-window` (`-h -l -P -F`), `send-keys` (`-l`), `capture-pane` (`-p -S -e -J`), `has-session`, `-L`, `-f`, COMMAND PARSING AND EXECUTION (el `;` como separador) y la variable `TMUX`.
- [tmux wiki: Advanced Use](https://github.com/tmux/tmux/wiki/Advanced-Use) — formatos, ids de panel y scripting.
