---
tags: [bash, configuracion, estado, xdg, source]
aliases: ["SCRIPT_DIR", "config.sh", "XDG Base Directory", "Escritura atómica"]
---
# Configuración y estado de un script

[[Inicio|← Inicio]] · [[Bash - Índice|Bash]]

> Un script que crece necesita guardar dos cosas: la **configuración** (lo que el usuario elige: rutas, colores, editor) y el **estado** (lo que el propio script recuerda: historial, contadores). Esta nota explica dónde ponerlas, cómo cargarlas con `source` y cómo escribir archivos sin corromperlos.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Configuración** | Preferencias que el usuario edita a mano; el script solo las lee |
| **Estado** | Datos que el script escribe y lee entre ejecuciones |
| **`source`** (o `.`) | Ejecuta otro archivo **dentro** del script actual: sus variables quedan definidas |
| **XDG** | Estándar de freedesktop.org que dice en qué carpetas del `$HOME` va cada tipo de archivo |
| **Escritura atómica** | Cambiar un archivo de forma que nunca quede a medias: o está el viejo o el nuevo |

## `SCRIPT_DIR`: dónde está el script

`$0` es **cómo** se llamó al script (`./tareas.sh`, `../bin/tareas.sh`…), no dónde está. Para encontrar archivos que viven junto a él:

```bash
SCRIPT_DIR="$(dirname "$(readlink -f "${BASH_SOURCE[0]}")")"
```

De dentro afuera:

- `${BASH_SOURCE[0]}` → la ruta de este archivo (funciona también si se hace `source`).
- `readlink -f` → la convierte en absoluta y **resuelve enlaces simbólicos** (si `~/bin/tareas` apunta a `~/proyectos/tareas/tareas.sh`, da el segundo).
- `dirname` → quita el nombre del archivo y deja la carpeta.

Así, `"$SCRIPT_DIR/config.sh"` funciona da igual desde dónde lo ejecutes. (`readlink -f` es de GNU; en macOS existe desde la versión 12.3.)

## Configuración por defecto + archivo opcional

Primero los valores por defecto; después, si existe un `config.sh`, se carga y sobrescribe lo que quiera:

```bash
# 1. Valores por defecto (TAREAS_ARCHIVO: variable de entorno, ver Variables)
ARCHIVO="${TAREAS_ARCHIVO:-$HOME/.tareas.txt}"
COLOR=1
EDITOR_TAREAS="${EDITOR:-vim}"

# 2. Si existe config.sh, lo carga
if [[ -f "$SCRIPT_DIR/config.sh" ]]; then
    # shellcheck source=/dev/null
    source "$SCRIPT_DIR/config.sh"
fi
```

`config.sh` es Bash normal, solo con asignaciones:

```bash
# config.sh
ARCHIVO="$HOME/Documentos/tareas.txt"
COLOR=0
```

> [!danger] `source` ejecuta código
> Todo lo que haya en `config.sh` se ejecuta con tus permisos. Úsalo solo con archivos que controla el propio usuario. Si la configuración viene de fuera, léela como texto: cada línea `clave=valor` con `while IFS='=' read -r clave valor` (ver [[Bucles en Bash]]).

La línea `# shellcheck source=/dev/null` le dice a [[Depurar scripts de Bash|shellcheck]] que no intente seguir ese archivo.

## `config.example.sh` en git, `config.sh` fuera

La configuración personal (rutas tuyas, tokens) no debe subirse al repositorio. El patrón habitual:

| Archivo | ¿En git? | Para qué |
|---|---|---|
| `config.example.sh` | Sí | Plantilla documentada con todas las opciones |
| `config.sh` | **No** | La copia de cada usuario |
| `.gitignore` | Sí | Contiene la línea `config.sh` |

```bash
cp config.example.sh config.sh    # el usuario la copia y la edita
echo "config.sh" >> .gitignore
```

Cuéntalo en el [[README de un proyecto|README]]. Más sobre `.gitignore` en [[Git para 42]].

## Archivos de estado

El estado lo escribe el script. Ejemplos: un **historial** (una línea por acción) y un **contador** (un número que se sobrescribe).

```bash
STATE_DIR="${XDG_STATE_HOME:-$HOME/.local/state}/tareas"
CONTADOR="$STATE_DIR/ejecuciones"
HISTORIAL="$STATE_DIR/historial.log"
mkdir -p "$STATE_DIR"                  # -p: no falla si ya existe

n=0
[[ -f "$CONTADOR" ]] && n="$(< "$CONTADOR")"   # $(< f) lee el archivo entero
n=$((n + 1))

printf '%s ejecución %d\n' "$(date '+%F %T')" "$n" >> "$HISTORIAL"
```

Para el historial basta con `>>` (ver [[Redirecciones y tuberías]]): solo añade al final. El contador hay que **reescribirlo**, y ahí está el peligro.

## Escribir de forma segura: temporal + `mv`

Si el script se corta (Ctrl+C, disco lleno) mientras hace `> archivo`, el archivo queda vacío o a medias. La solución es escribir en un temporal y renombrarlo:

```bash
tmp="$(mktemp "$CONTADOR.XXXXXX")"     # temporal en la MISMA carpeta
printf '%d\n' "$n" > "$tmp"
mv -f "$tmp" "$CONTADOR"               # el cambio es instantáneo
```

`mv` dentro del mismo sistema de archivos es **atómico**: quien lea el archivo ve el contenido viejo o el nuevo, nunca uno a medias. Por eso el temporal se crea junto al destino y no en `/tmp` (que puede ser otro disco). Si el script puede fallar antes del `mv`, borra el temporal con un `trap` (ver [[Scripts robustos con set y trap]]).

Lo mismo sirve para borrar una tarea: `grep -v` o `sed` sobre el original, salida al temporal, `mv` encima.

## XDG: dónde guarda cada cosa un programa

En vez de llenar `$HOME` de archivos ocultos (`~/.tareas.txt`), el estándar XDG propone:

| Variable | Por defecto | Qué va ahí |
|---|---|---|
| `XDG_CONFIG_HOME` | `~/.config` | Configuración: `~/.config/tareas/config.sh` |
| `XDG_DATA_HOME` | `~/.local/share` | Datos importantes del usuario: `tareas.txt` |
| `XDG_STATE_HOME` | `~/.local/state` | Estado que se puede perder: historial, contadores |
| `XDG_CACHE_HOME` | `~/.cache` | Cosas que se pueden regenerar |

Se usa siempre con valor por defecto: `"${XDG_CONFIG_HOME:-$HOME/.config}/tareas"`.

> [!tip] ¿Junto al script o en XDG?
> Para un script personal o un proyecto de 42, `config.sh` junto al script es más simple. Si lo vas a instalar para más gente (ver [[Qué hace un buen instalador]]), usa XDG: el script puede estar en `/usr/local/bin` sin permisos de escritura.

## Errores típicos

| Error | Qué pasa |
|---|---|
| Usar `$0` para buscar archivos | Falla si lo llamas desde otra carpeta o con un enlace |
| `source config.sh` sin ruta | Busca en la carpeta **actual**, no en la del script |
| Subir `config.sh` a git | Publicas tus rutas o contraseñas |
| `> archivo` directo para reescribir | Un corte deja el archivo vacío |
| `mktemp` en `/tmp` y `mv` a `$HOME` | Si son discos distintos, `mv` copia y deja de ser atómico |

## Relacionadas

[[Scripts robustos con set y trap]] · [[Colores en la terminal]] · [[Redirecciones y tuberías]] · [[Expansiones en Bash]] · [[Argumentos y subcomandos en Bash]] · [[Git para 42]] · [[README de un proyecto]] · [[Qué hace un buen instalador]] · [[Bash - Chuleta de sintaxis]]

## Referencias

- [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir-spec/latest/) — `XDG_CONFIG_HOME`, `XDG_STATE_HOME` y sus valores por defecto.
- [GNU Bash Manual: Bash Variables](https://www.gnu.org/software/bash/manual/html_node/Bash-Variables.html) — `BASH_SOURCE`.
- [GNU Bash Manual: Bourne Shell Builtins (`source`/`.`)](https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html).
- `man readlink`, `man mktemp`, `man 2 rename` — `rename` es atómico dentro del mismo sistema de archivos.
- [BashFAQ/028 (Greg's Wiki)](https://mywiki.wooledge.org/BashFAQ/028) — cómo localiza un script su propia carpeta.
- Ejemplos probados con Bash 5.1.16 en Ubuntu 22.04.
