---
tags: [tmux, terminal, atajos, cheatsheet]
aliases: ["Atajos de tmux", "Chuleta de tmux", "Prefijo de tmux"]
---
# tmux - Uso básico

[[Inicio|← Inicio]] · [[tmux - Índice|tmux]]

> Todo en tmux se maneja con **el prefijo** (`Ctrl-b`) seguido de una tecla. Esta nota es la chuleta del día a día: comandos para crear y recuperar sesiones desde la shell, y los atajos para ventanas, paneles, scroll y copiar.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Prefijo** (*prefix*) | Combinación que avisa a tmux de que la siguiente tecla es para él, no para el programa. Por defecto `Ctrl-b` |
| **Modo copia** (*copy mode*) | Modo en el que el panel se congela y puedes moverte por lo que ya salió, buscar y copiar |
| **Buffer de pegado** | Portapapeles interno de tmux (distinto del del sistema) |
| **Target** (`-t`) | A qué sesión, ventana o panel va dirigido un comando: `libft`, `libft:2`, `libft:2.1` |

## El prefijo: cómo se pulsa

`Ctrl-b` y luego la tecla, **soltando entre medias**: pulsas `Ctrl` + `b`, sueltas las dos, y pulsas `%`. En esta nota se escribe `Ctrl-b %`.

> [!tip] Si te pierdes
> `Ctrl-b ?` muestra **todos** los atajos (sal con `q`). Y si una tecla «no hace nada», probablemente no soltaste el `Ctrl` o pulsaste el prefijo dos veces.

## Desde la shell (fuera de tmux)

```bash
tmux new -s libft          # nueva sesión llamada "libft" (y entras)
tmux ls                    # lista sesiones:  libft: 2 windows (created ...)
tmux attach -t libft       # volver a entrar (también: tmux a -t libft)
tmux attach                # entra en la última sesión usada
tmux kill-session -t libft # cierra la sesión y todo lo que tiene dentro
tmux kill-server           # cierra TODAS las sesiones
```

Ponle siempre nombre con `-s`: «libft» se recuerda mejor que «0».

## Sesiones

| Atajo | Qué hace |
|---|---|
| `Ctrl-b d` | **Detach**: salir dejando todo corriendo |
| `Ctrl-b s` | Lista de sesiones para elegir (flechas + Intro) |
| `Ctrl-b $` | Renombrar la sesión actual |
| `Ctrl-b (` / `Ctrl-b )` | Sesión anterior / siguiente |

## Ventanas (pestañas)

| Atajo | Qué hace |
|---|---|
| `Ctrl-b c` | Crear ventana nueva |
| `Ctrl-b ,` | Renombrar ventana (útil: «codigo», «tests», «man») |
| `Ctrl-b n` / `Ctrl-b p` | Ventana siguiente / anterior |
| `Ctrl-b 0` … `Ctrl-b 9` | Ir a la ventana número N |
| `Ctrl-b l` | Volver a la última ventana usada |
| `Ctrl-b w` | Árbol de sesiones y ventanas para elegir |
| `Ctrl-b &` | Cerrar la ventana (pide confirmación con `y`) |

## Paneles (divisiones)

| Atajo | Qué hace |
|---|---|
| `Ctrl-b %` | Dividir **izquierda \| derecha** |
| `Ctrl-b "` | Dividir **arriba / abajo** |
| `Ctrl-b` + flecha | Moverse al panel en esa dirección |
| `Ctrl-b o` | Ir al siguiente panel |
| `Ctrl-b ;` | Volver al panel anterior |
| `Ctrl-b q` | Mostrar el número de cada panel (pulsa el número para ir) |
| `Ctrl-b z` | **Zoom**: el panel ocupa toda la ventana; otra vez `Ctrl-b z` y vuelve |
| `Ctrl-b {` / `Ctrl-b }` | Intercambiar el panel con el anterior / siguiente |
| `Ctrl-b Espacio` | Cambiar entre distribuciones predefinidas |
| `Ctrl-b !` | Sacar el panel a una ventana propia |
| `Ctrl-b x` | Cerrar el panel (o escribe `exit` en él) |

### Redimensionar

| Atajo | Qué hace |
|---|---|
| `Ctrl-b Ctrl-flecha` | Mueve el borde 1 celda. Mantén `Ctrl` y pulsa la flecha varias veces |
| `Ctrl-b Alt-flecha` | Mueve el borde 5 celdas |
| Arrastrar el borde | Solo con `mouse on` ([[Configurar tmux]]) |

## Scroll y modo copia

La rueda del ratón **no** hace scroll por defecto: lo que hace es enviar flechas al programa. Para ver lo que ya salió por pantalla (por ejemplo, el primer error de un `make` largo) entra en **modo copia**:

| Atajo | Qué hace |
|---|---|
| `Ctrl-b [` | Entrar en modo copia |
| `Ctrl-b RePág` | Entrar en modo copia y subir una página |
| Flechas, `RePág`, `AvPág` | Moverse por el historial |
| `Ctrl-s` / `Ctrl-r` | Buscar hacia abajo / arriba (modo emacs). En modo vi: `/` y `?` |
| `q` | Salir del modo copia |

**Copiar y pegar** dentro de tmux:

| Paso | Teclas modo emacs (por defecto) | Teclas modo vi (`mode-keys vi`) |
|---|---|---|
| Empezar selección | `Ctrl-Espacio` | `Espacio` |
| Copiar y salir | `Alt-w` | `Intro` |
| Pegar | `Ctrl-b ]` | `Ctrl-b ]` |

> [!note] ¿emacs o vi?
> tmux usa teclas de vi en modo copia si tu variable `EDITOR` o `VISUAL` contiene «vi»; si no, las de emacs. Compruébalo con `tmux show -gw mode-keys` y fíjalo en [[Configurar tmux]].

## El prompt de comandos `:`

`Ctrl-b :` abre una línea abajo para escribir **cualquier comando de tmux** sin el `tmux` delante. Son los mismos que usarás en scripts ([[Automatizar tmux desde scripts]]):

```
:new-window -n man
:split-window -h -l 40%
:rename-session ft_printf
:kill-pane
:source-file ~/.tmux.conf
:setw synchronize-panes on     ← lo que escribes va a todos los paneles a la vez
```

## Errores típicos

- **Pulsar `Ctrl-b %` sin soltar**: escribe `%` en la shell o no hace nada.
- **Hacer scroll con la rueda y ver comandos antiguos**: estás moviéndote por el historial de la shell, no por la salida. Usa `Ctrl-b [`.
- **Cerrar la ventana de la terminal pensando que cierras tmux**: solo haces detach. Para cerrar de verdad: `exit` en cada panel o `tmux kill-session`.
- **`Ctrl-b` en Vim o en Bash**: si necesitas enviar un `Ctrl-b` real al programa, pulsa `Ctrl-b Ctrl-b`.

## Relacionadas

[[tmux - Índice]] · [[Qué es tmux]] · [[Configurar tmux]] · [[Automatizar tmux desde scripts]] · [[Vim]]

## Referencias

- [man tmux (OpenBSD)](https://man.openbsd.org/tmux) — secciones DEFAULT KEY BINDINGS, WINDOWS AND PANES y COPY MODE (`mode-keys`, teclas de copia en vi y emacs).
- [tmux wiki: Getting Started](https://github.com/tmux/tmux/wiki/Getting-Started) — prefijo, modo copia y prompt de comandos.
