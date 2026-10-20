---
tags: [tui, bash, whiptail, dialog, gum, fzf]
aliases: ["whiptail", "dialog", "gum", "Menús en Bash"]
---
# TUIs en Bash - dialog, whiptail y gum

[[Inicio|← Inicio]] · [[TUI - Índice|TUI]]

> Para un script de Bash no hace falta programar una TUI: hay **programas que muestran un diálogo y devuelven la respuesta**. `whiptail` viene en Debian/Ubuntu, `dialog` tiene más widgets, `gum` es moderno y bonito, y `fzf` sirve para elegir de una lista. Tu script los llama como cualquier otro comando.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Cuadro de diálogo** | Ventana pequeña con un mensaje y botones (`<Ok>`, `<Cancel>`) |
| **Código de salida** | Número que devuelve el programa: `0` Aceptar/Sí, `1` Cancelar/No, `255` Esc (ver [[Códigos de salida y errores en Bash]]) |
| **Etiqueta** (*tag*) | En menús y checklists, el identificador que se devuelve (`1`, `programa`…) |
| **Búsqueda difusa** (*fuzzy*) | Encontrar `"tarea"` escribiendo `"tra"`: no hace falta escribirlo exacto |

## whiptail: los widgets

Todos tienen la forma `whiptail --TIPO "texto" ALTO ANCHO [extra]`:

| Widget | Para qué | Qué devuelve |
|---|---|---|
| `--msgbox` | Aviso con `<Ok>` | Solo código de salida |
| `--yesno` | Pregunta sí/no (`--defaultno` marca «No») | `0` sí, `1` no |
| `--inputbox` | Pedir un texto (con valor por defecto) | El texto **por stderr** |
| `--passwordbox` | Igual, sin mostrar lo que escribes | El texto por stderr |
| `--menu` | Elegir una opción | La etiqueta elegida |
| `--checklist` / `--radiolist` | Marcar varias / una | Las etiquetas marcadas |
| `--gauge` | Barra de progreso; lee porcentajes por stdin | — |
| `--textbox` | Mostrar un fichero (`--scrolltext`) | — |

### El truco `3>&1 1>&2 2>&3`

whiptail **dibuja en stdout** y **escribe la respuesta en stderr**. Con `$( )` solo capturas stdout, así que hay que intercambiarlos (ver [[Redirecciones y tuberías]]):

```bash
nombre=$(whiptail --inputbox "¿Cómo te llamas?" 8 40 "Ana" 3>&1 1>&2 2>&3)
#                                                            │     │     └─ stderr → donde apunta 3 (la captura)
#                                                            │     └─ stdout → la terminal (para dibujar)
#                                                            └─ 3 = copia de stdout (la captura)
echo "Hola, $nombre"
```

### Comprobar Cancelar

```bash
if ! nombre=$(whiptail --inputbox "Nombre" 8 40 3>&1 1>&2 2>&3); then
    echo "Cancelado"; exit 1
fi
whiptail --yesno "¿Seguro?" 8 30 && echo "Sí" || echo "No o Esc"
```

### Checklist y gauge

```bash
comps=$(whiptail --separate-output --checklist "Componentes" 12 50 3 \
    programa "El comando" ON \
    docs     "Documentación" OFF \
    alias    "Alias en ~/.bashrc" OFF 3>&1 1>&2 2>&3)
# --separate-output: una etiqueta por línea, sin comillas

for p in 25 50 75 100; do echo "$p"; sleep 0.3; done |
    whiptail --gauge "Instalando…" 7 50 0
```

## Ejemplo: menú de `tareas.sh`

Una versión con menús del mini-proyecto: mismo archivo (`$HOME/.tareas.txt`, o el de `TAREAS_ARCHIVO`), mismas funciones `anadir` y `listar`, y la lista sale con el mismo formato que `./tareas.sh lista` (` 1. Texto`).

```bash
#!/usr/bin/env bash
set -euo pipefail
readonly ARCHIVO="${TAREAS_ARCHIVO:-$HOME/.tareas.txt}"
touch "$ARCHIVO"

anadir() {
    local texto
    texto=$(whiptail --title "Nueva tarea" --inputbox "¿Qué tienes que hacer?" \
        10 50 3>&1 1>&2 2>&3) || return 0          # Cancelar → no hacer nada
    [[ -n "$texto" ]] && printf '%s\n' "$texto" >> "$ARCHIVO"
    return 0
}

listar() {
    local texto                                     # mismo formato que `tareas.sh lista`
    texto=$(nl -ba -w2 -s'. ' "$ARCHIVO")            # " 1. comprar pan"
    # Esc en un msgbox devuelve 255: con set -e mataría el script
    whiptail --title "Tus tareas" --scrolltext --msgbox "${texto:-(sin tareas)}" 20 60 || true
}

main() {
    local opcion
    while true; do
        opcion=$(whiptail --title "tareas.sh" --menu "Elige una opción" 15 50 3 \
            "1" "Añadir tarea" "2" "Ver tareas" "3" "Salir" \
            3>&1 1>&2 2>&3) || break                # Esc/Cancelar → salir
        case "$opcion" in
            1) anadir ;;
            2) listar ;;
            3) break ;;
        esac
    done
    clear
}

main "$@"
```

> [!warning] `set -e` y los diálogos
> Con [[Scripts robustos con set y trap|`set -e`]], cualquier diálogo que devuelva `1` o `255` **termina el script**. Pon siempre `|| true`, `|| return` o úsalo dentro de un `if`.

## dialog

Casi la misma sintaxis que whiptail (`dialog --menu …`), con widgets extra: `--calendar`, `--fselect` (elegir fichero), `--form` (varios campos), `--timebox`, `--tailbox`. No viene instalado por defecto: `sudo apt install dialog` en tu ordenador.

## gum

Herramienta de Charm con un comando por widget. Lo que eliges sale por **stdout**, sin trucos:

```bash
tipo=$(gum choose "feat" "fix" "docs")              # menú con flechas
titulo=$(gum input --placeholder "Título")          # campo de texto
gum confirm "¿Guardar?" && echo "guardado"          # sí/no por código de salida
gum spin --spinner dot --title "Compilando…" -- make # spinner mientras corre make
gum style --border double --padding "1 2" --foreground 212 "¡Hecho!"
```

## fzf: elegir de una lista

```bash
tarea=$(fzf --prompt="Tarea> " --height=40% < "$HOME/.tareas.txt") || exit 0
echo "Has elegido: $tarea"
```

Escribes unas letras, se filtra la lista y Enter devuelve la línea. Esc → código `130` (y el `|| exit 0` sale sin error).

## Errores típicos

| Error | Consecuencia |
|---|---|
| `x=$(whiptail --inputbox …)` sin `3>&1 1>&2 2>&3` | La pantalla se queda en blanco y `x` contiene el dibujo |
| Olvidar `\|\| true` con `set -e` | Pulsar Esc cierra el script de golpe |
| Locale sin UTF-8 | Las `ñ` y tildes salen cortadas. Comprueba `locale` |
| No hacer `clear` al salir | Queda el último diálogo dibujado |

## Relacionadas

[[Leer entrada del usuario en Bash]] · [[Redirecciones y tuberías]] · [[Scripts robustos con set y trap]] · [[Códigos de salida y errores en Bash]] · [[Instalador TUI en Bash]] · [[Spinners y barras de progreso]] · [[Frameworks de TUI - Listado]] · [[Elegir un framework de TUI]] · [[Animación en la terminal - fundamentos]]

## Referencias

- [whiptail(1)](https://manpages.debian.org/whiptail) — widgets, `--separate-output`, códigos de salida.
- [dialog](https://invisible-island.net/dialog/manpage/dialog.html) — lista completa de widgets.
- [gum README](https://github.com/charmbracelet/gum) — `choose`, `input`, `confirm`, `spin`, `style`.
- [fzf README](https://github.com/junegunn/fzf) — uso básico y código 130 al cancelar.
- Los ejemplos de **gum** siguen su README; no se han probado aquí (gum no estaba instalado).
- Menú probado con whiptail 0.52 en tmux (Ubuntu 22.04): añadir, ver la lista numerada, Esc en el visor y salir con código 0.
