---
tags: [tmux, configuracion, dotfiles, terminal]
aliases: ["tmux.conf", ".tmux.conf", "Configuración de tmux"]
---
# Configurar tmux

[[Inicio|← Inicio]] · [[tmux - Índice|tmux]]

> tmux lee al arrancar el fichero **`~/.tmux.conf`**: cada línea es un comando de tmux (los mismos del prompt `Ctrl-b :`). Con unas 25 líneas consigues ratón, numeración desde 1, colores correctos, una barra de estado a tu gusto y atajos propios.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Opción** | Un ajuste con nombre (`mouse`, `base-index`…). Se cambia con `set` (o `setw` para las de ventana) |
| **`-g`** (*global*) | Aplica el ajuste a todas las sesiones/ventanas, no solo a la actual |
| **`bind`** | Asociar una tecla (tras el prefijo) a un comando. `unbind` la libera |
| **`TERM`** | Variable que dice a los programas qué tipo de terminal tienen delante y qué sabe hacer (colores, cursor…) |
| **256 colores / *true color*** | Paleta de 256 colores o de 16 millones (24 bits, «RGB») |
| **Formato** `#{…}`, `#S` | Marcadores que tmux sustituye por datos: `#S` = nombre de sesión, `#{pane_current_path}` = carpeta del panel |

## Dónde está y cómo recargarlo

- Fichero: `~/.tmux.conf` (en tmux 3.2+ también vale `~/.config/tmux/tmux.conf`). Si no existe, créalo.
- Se lee **solo al arrancar el servidor**. Si ya tienes sesiones abiertas, cambiar el fichero no hace nada hasta que lo recargues:

```bash
tmux source-file ~/.tmux.conf     # desde la shell
# o dentro de tmux:  Ctrl-b :  source-file ~/.tmux.conf
```

> [!warning] Recargar no deshace
> `source-file` aplica las líneas encima de lo que ya hay. Si **borras** una línea (p. ej. un `bind`), el atajo sigue activo hasta que reinicies el servidor (`tmux kill-server`) o lo quites a mano con `unbind`.

## Una configuración mínima comentada

Copia esto en `~/.tmux.conf`. Cada bloque es independiente: quita lo que no quieras.

```tmux
# ~/.tmux.conf — configuración mínima comentada

# --- Prefijo: Ctrl-a en vez de Ctrl-b (opcional) ---
unbind C-b
set -g prefix C-a
bind C-a send-prefix          # Ctrl-a Ctrl-a envía un Ctrl-a real al programa

# --- Comodidad ---
set -g mouse on               # clic para cambiar de panel, rueda para scroll
set -g base-index 1           # las ventanas empiezan en 1 (tecla más cercana)
setw -g pane-base-index 1     # y los paneles también
set -g renumber-windows on    # al cerrar una ventana, renumera las demás
set -g history-limit 10000    # líneas que se guardan para hacer scroll
set -sg escape-time 10        # Esc instantáneo (importante para Vim)

# --- Colores ---
set -g default-terminal "tmux-256color"
set -as terminal-features ",xterm-256color:RGB"   # color de 24 bits

# --- Barra de estado ---
set -g status-style "bg=colour236,fg=colour250"
set -g status-left "#[bold] #S #[default]"         # nombre de la sesión
set -g status-right " %H:%M %d-%m "                # hora y fecha
setw -g window-status-current-style "bg=colour31,fg=white,bold"

# --- Atajos propios ---
bind r source-file ~/.tmux.conf \; display "Configuración recargada"
bind | split-window -h -c "#{pane_current_path}"   # | divide en vertical
bind - split-window -v -c "#{pane_current_path}"   # - divide en horizontal
bind -r h select-pane -L      # -r: se puede repetir sin volver a pulsar el prefijo
bind -r j select-pane -D
bind -r k select-pane -U
bind -r l select-pane -R
setw -g mode-keys vi          # modo copia con teclas de Vim
```

> [!info] Verificado
> Esta configuración se ha cargado en tmux 3.2a con `tmux -L prueba -f tmux.conf new -d` y comprobado con `tmux -L prueba show -g prefix` (→ `prefix C-a`), `show -g mouse` (→ `mouse on`) y `list-keys`. Así puedes probar una configuración nueva **sin tocar** tus sesiones: `-L` usa otro servidor y `-f` otro fichero.

## Qué hace cada bloque

**Prefijo alternativo.** `Ctrl-b` obliga a estirar la mano; mucha gente usa `Ctrl-a` (al lado del `Ctrl`). Pega con el «ir al principio de línea» de Bash: por eso está `bind C-a send-prefix`, para mandar un `Ctrl-a` real pulsándolo dos veces. Si no te molesta `Ctrl-b`, borra esas tres líneas.

**`mouse on`.** Clic en un panel para seleccionarlo, arrastrar bordes para redimensionar, rueda para hacer scroll (entra sola en modo copia). Para seleccionar texto con el ratón *de tu terminal* (y copiar al portapapeles del sistema) mantén `Shift` mientras arrastras.

**`base-index 1`.** Las ventanas empiezan en 1, que en el teclado está justo encima de `q`, en vez de en 0 al otro extremo.

**`escape-time`.** Las flechas y teclas de función llegan como secuencias que **empiezan por el byte `Esc`**. Por eso, tras un `Esc`, tmux espera un poco por si vienen más bytes: 500 ms por defecto en tmux 3.2a (desde tmux 3.5 el valor por defecto ya es 10 ms). En [[Vim]] eso se nota como un retraso al salir del modo Inserción, y en tus TUIs de `curses` se suma al `ESCDELAY` propio de curses (ver [[Cómo funciona la terminal#Por qué Esc tarda]]). El `-s` de `set -sg` indica que es una opción del **servidor**, no de una sesión. Compruébalo con `tmux show -s escape-time`.

**Colores y `TERM`.** Dentro de tmux, `TERM` debe valer `tmux-256color` o `screen-256color` (no `xterm-256color`). Comprueba:

```bash
echo "$TERM"            # dentro de tmux → tmux-256color
tput colors             # → 256
```

Si un tema de Vim se ve con colores «lavados», añade la línea `terminal-features … RGB` (tmux 3.2+) para activar color de 24 bits. Más detalle sobre `TERM` y colores en [[Cómo funciona la terminal]] y [[Colores en la terminal]].

> [!note] Si `tmux-256color` da problemas
> En sistemas antiguos puede faltar esa entrada de *terminfo* (la base de datos que describe terminales) y verás `missing or unsuitable terminal`. Usa `set -g default-terminal "screen-256color"`.

**Barra de estado.** `status-left` y `status-right` son texto con formatos: `#S` sesión, `#W` ventana, `#H` nombre del ordenador, `%H:%M` hora (como en `date`). `#[bold]`, `#[fg=red]` cambian el estilo; `#[default]` lo restablece. Los colores `colour0`…`colour255` son la paleta de 256.

**Atajos propios con `bind`.**

```tmux
bind r source-file ~/.tmux.conf \; display "Recargado"   # dos comandos: el \; los separa
bind | split-window -h -c "#{pane_current_path}"         # el panel nuevo abre en la misma carpeta
bind -r h select-pane -L                                 # -r: repetible
bind -n M-Left select-pane -L                            # -n: SIN prefijo (Alt + flecha)
```

`-c "#{pane_current_path}"` es muy cómodo: el panel nuevo empieza en la carpeta del proyecto en lugar de en `~`.

## Errores típicos

- **Editar el fichero y no recargar**: «no funciona» porque el servidor sigue con la configuración vieja.
- **`unknown command`** al cargar: una línea con una errata. tmux dice el número de línea: `~/.tmux.conf:12: unknown command: sett`.
- **Copiar configuraciones enormes de internet** con opciones que tu versión no tiene (`tmux -V`). Ve añadiendo líneas de una en una.
- **Cambiar el prefijo y olvidarlo**: todos los atajos de [[tmux - Uso básico]] pasan a ser `Ctrl-a …`.

## Relacionadas

[[tmux - Índice]] · [[tmux - Uso básico]] · [[Automatizar tmux desde scripts]] · [[Colores en la terminal]] · [[Cómo funciona la terminal]] · [[Vim]]

## Referencias

- [man tmux (OpenBSD)](https://man.openbsd.org/tmux) — secciones OPTIONS (`mouse`, `base-index`, `escape-time`, `default-terminal`, `terminal-features`, `status-left`), FORMATS y STYLES; comando `bind-key` (`-r`, `-n`).
- [tmux wiki: FAQ](https://github.com/tmux/tmux/wiki/FAQ) — `TERM` dentro de tmux, `tmux-256color` frente a `screen-256color`, color RGB.
