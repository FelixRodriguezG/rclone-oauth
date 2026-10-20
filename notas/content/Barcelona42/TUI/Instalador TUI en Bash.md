---
tags: [tui, instalador, bash, whiptail]
aliases: ["Instalador con whiptail", "Asistente de instalación en Bash"]
---
# Instalador TUI en Bash

[[Inicio|← Inicio]] · [[TUI - Índice|TUI]]

> Un **asistente** (*wizard*) guía la instalación paso a paso: bienvenida → carpeta → componentes → resumen → progreso → fin. Aquí lo construimos con `whiptail` y con **plan B en texto plano** si no hay whiptail o no hay terminal. Sigue las reglas de [[Qué hace un buen instalador]] y usa los widgets de [[TUIs en Bash - dialog, whiptail y gum]].

## Vocabulario

| Término | Qué significa |
|---|---|
| **Asistente** (*wizard*) | Interfaz de pantallas sucesivas, una pregunta por pantalla |
| **Fallback** (plan B) | Lo que se usa cuando lo preferido no está disponible |
| **`[[ -t 0 ]]`** | Verdadero si la entrada estándar es una terminal (hay alguien para responder) |
| **`mapfile -t`** | Lee líneas en un array, sin el `\n` final (ver [[Arrays en Bash]]) |

## Diseño

- Una función por **tipo de diálogo** (`aviso`, `preguntar`, `confirmar`): cada una decide si usa whiptail o texto. El resto del script no sabe qué interfaz hay.
- `set -Eeuo pipefail` + `trap` en `ERR` e `INT` (ver [[Scripts robustos con set y trap]]). Sin la `-E`, el `trap … ERR` **no** salta dentro de funciones, y aquí casi todo está en funciones.
- `--yes` acepta los valores por defecto sin preguntar; `--prefix` cambia la carpeta (para probar sin tocar tu `~/.local`).

## El script completo

```bash
#!/usr/bin/env bash
# instalar.sh — asistente para instalar «tareas». Uso: ./instalar.sh [--yes] [--prefix DIR]
set -Eeuo pipefail          # -E: el trap ERR también salta dentro de funciones

PREFIX="$HOME/.local"
CONFIG_DIR="${XDG_CONFIG_HOME:-$HOME/.config}/tareas"
SI=false
TITULO="Instalador de tareas"
trap 'echo "✘ Error en la línea $LINENO; instalación incompleta." >&2' ERR
trap 'echo; echo "Cancelado." >&2; exit 130' INT

while (( $# > 0 )); do
    case "$1" in
        -y|--yes) SI=true ;;
        --prefix) PREFIX="${2:?--prefix necesita una carpeta}"; shift ;;
        *) echo "Opción desconocida: $1" >&2; exit 2 ;;
    esac
    shift
done

# whiptail solo si existe, hay terminal y no es --yes; si no, texto plano
if [[ "$SI" == false ]] && command -v whiptail >/dev/null && [[ -t 0 ]]; then
    UI=whiptail
else
    UI=texto
fi
wt() { whiptail --title "$TITULO" "$@" 3>&1 1>&2 2>&3; }   # respuesta → stdout

aviso() {                                  # aviso TEXTO
    if [[ "$UI" == whiptail ]]; then wt --msgbox "$1" 12 60 || true
    else printf '%s\n' "$1"; fi
}

preguntar() {                              # preguntar TEXTO DEFECTO → respuesta
    local resp
    if [[ "$UI" == whiptail ]]; then wt --inputbox "$1" 10 60 "$2"
    elif [[ "$SI" == true ]]; then printf '%s\n' "$2"
    else read -rp "$1 [$2]: " resp; printf '%s\n' "${resp:-$2}"; fi
}

confirmar() {                              # confirmar TEXTO → 0 si sí
    local resp
    if [[ "$UI" == whiptail ]]; then wt --yesno "$1" 14 64
    elif [[ "$SI" == true ]]; then return 0
    else read -rp "$1 [s/N]: " resp; [[ "$resp" == [sSyY]* ]]; fi
}

elegir_componentes() {                     # una etiqueta por línea
    if [[ "$UI" == whiptail ]]; then
        wt --separate-output --checklist "Espacio marca, Enter acepta" 12 60 3 \
            programa "El comando tareas" ON \
            config   "Configuración de ejemplo" ON \
            alias    "Alias t=tareas en ~/.bashrc" OFF
    else
        printf '%s\n' programa config
    fi
}

instalar() {                               # imprime porcentajes para el gauge
    local total=$# n=0 c
    for c in "$@"; do
        case "$c" in
            programa) mkdir -p "$PREFIX/bin"          # tu tareas.sh, como comando «tareas»
                      install -m 755 tareas.sh "$PREFIX/bin/tareas" ;;
            config)   mkdir -p "$CONFIG_DIR"                  # no pisar la del usuario
                      [[ -f "$CONFIG_DIR/tareas.conf" ]] ||
                          printf 'COLOR=si\n' > "$CONFIG_DIR/tareas.conf" ;;
            alias)    grep -qxF "alias t=tareas" "$HOME/.bashrc" 2>/dev/null ||
                          echo "alias t=tareas" >> "$HOME/.bashrc" ;;
        esac
        n=$(( n + 1 ))
        echo $(( n * 100 / total ))
        sleep 0.3                          # solo para que se vea avanzar
    done
}

main() {
    local comps lista
    [[ -f tareas.sh ]] || { echo "Ejecútalo desde la carpeta de tareas.sh" >&2; exit 1; }
    aviso "Bienvenido. Se instalará «tareas» en tu usuario (sin sudo)."
    PREFIX=$(preguntar "Carpeta de instalación:" "$PREFIX") || exit 1
    comps=$(elegir_componentes) || exit 1
    [[ -n "$comps" ]] || { aviso "No has elegido nada."; exit 0; }
    mapfile -t lista <<< "$comps"

    confirmar "Se va a instalar: ${lista[*]}
  Programa en: $PREFIX/bin
  Config en:   $CONFIG_DIR
¿Continuar?" || { aviso "Cancelado. No se ha cambiado nada."; exit 0; }

    if [[ "$UI" == whiptail ]]; then
        instalar "${lista[@]}" | whiptail --title "$TITULO" --gauge "Instalando…" 7 60 0
    else
        instalar "${lista[@]}" | while read -r pct; do printf '\r  %3d%%' "$pct"; done
        echo
    fi

    "$PREFIX/bin/tareas" help >/dev/null 2>&1 || { aviso "✘ La verificación falló."; exit 1; }
    local extra=""
    [[ ":$PATH:" == *":$PREFIX/bin:"* ]] ||
        extra="Añade a tu ~/.bashrc:  export PATH=\"$PREFIX/bin:\$PATH\""
    aviso "✔ Instalación completada.
$extra"
}

main "$@"
```

## Probarlo sin tocar tu sistema

```bash
cd ruta/a/tareas                                      # donde están instalar.sh y tareas.sh
prueba=$(mktemp -d)                                   # carpeta temporal
HOME="$prueba" ./instalar.sh --prefix "$prueba/local" # con whiptail
HOME="$prueba" ./instalar.sh --yes --prefix "$prueba/local" < /dev/null   # texto, sin preguntas
find "$prueba" -type f                                # ver qué se ha creado
```

Salida del modo `--yes`:

```text
Bienvenido. Se instalará «tareas» en tu usuario (sin sudo).
  100%
✔ Instalación completada.
Añade a tu ~/.bashrc:  export PATH="/tmp/tmp.Xa12/local/bin:$PATH"
```

> [!note] Por qué `wt` intercambia los descriptores
> whiptail dibuja en stdout y responde por stderr. `wt` hace el `3>&1 1>&2 2>&3` una sola vez para que `x=$(wt --inputbox …)` capture la respuesta. Para `--gauge` no se usa `wt`: ahí la entrada es la tubería con los porcentajes.

> [!warning] Cancelar a mitad
> Si el usuario pulsa *Cancel* en la carpeta o en los componentes, el script sale con `exit 1` **antes** de tocar nada. Por eso el plan se confirma **antes** del gauge: una vez empieza la copia, no se pregunta más.

## Errores típicos

| Error | Consecuencia |
|---|---|
| Diálogo sin `\|\| true` con `set -e` | Esc cierra el script sin mensaje |
| Asumir que whiptail existe | En macOS o Alpine falla: de ahí el plan B |
| Preguntar en modo `curl … \| bash` | `read` lee del propio script; ver [[Instalar con una línea]] |
| Mostrar el gauge sin `echo` de porcentajes | La barra se queda en 0 % hasta el final |

## Relacionadas

[[Qué hace un buen instalador]] · [[TUIs en Bash - dialog, whiptail y gum]] · [[Instalador TUI en Python]] · [[Instalar con una línea]] · [[Scripts robustos con set y trap]] · [[Argumentos y subcomandos en Bash]] · [[Leer entrada del usuario en Bash]] · [[Redirecciones y tuberías]] · [[Spinners y barras de progreso]]

## Referencias

- [whiptail(1)](https://manpages.debian.org/whiptail) — `--inputbox`, `--checklist --separate-output`, `--yesno`, `--gauge`.
- [Bash manual: mapfile](https://www.gnu.org/software/bash/manual/bash.html#index-mapfile) — leer líneas a un array.
- [Bash manual: The Set Builtin](https://www.gnu.org/software/bash/manual/bash.html#The-Set-Builtin) — `-e`, `-u`, `-o pipefail`.
- Probado con Bash 5.1 y whiptail 0.52.21 en tmux (Ubuntu 22.04): recorrido completo con los tres componentes, modo `--yes` (dos veces seguidas, idempotente), modo texto respondiendo «n» y opción desconocida (código 2).
