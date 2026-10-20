---
tags: [tui, instalador, bash, curl, github, seguridad]
aliases: ["curl | bash", "curl pipe bash", "install.sh"]
---
# Instalar con una línea

[[Inicio|← Inicio]] · [[TUI - Índice|TUI]]

> Muchos proyectos se instalan pegando **una sola línea**: `curl -fsSL https://…/install.sh | bash`. Descarga un script y se lo pasa a Bash sin guardarlo. Es cómodo, pero ejecutas código de internet **sin leerlo**. Esta nota explica cada pieza, sus riesgos, cómo escribir un `install.sh` que aguante descargas cortadas y cómo publicarlo en GitHub.

## Vocabulario

| Término | Qué significa |
|---|---|
| **`curl`** | Programa que descarga una URL y la escribe en stdout |
| **Tubería** (`\|`) | La salida de `curl` entra como entrada de `bash` (ver [[Redirecciones y tuberías]]) |
| **Checksum** (suma de verificación) | Huella del fichero (`sha256sum`). Si cambia un solo byte, cambia la huella |
| **Release** | Versión publicada en GitHub (`v1.0.0`) con ficheros adjuntos que no cambian |
| **raw.githubusercontent.com** | Dominio que sirve los ficheros de un repo **tal cual**, sin la página web alrededor |

## Qué hace cada flag

```bash
curl -fsSL https://raw.githubusercontent.com/USUARIO/tareas/main/install.sh | bash
```

| Flag | Significado | Sin él… |
|---|---|---|
| `-f` (`--fail`) | Si el servidor responde con error (404, 500), **no** escribe nada y sale con código ≠ 0 | Bash ejecutaría la página de error HTML |
| `-s` (`--silent`) | Sin barra de progreso | curl pinta su barra de progreso en la terminal (por stderr) encima de lo que muestra el script |
| `-S` (`--show-error`) | Con `-s`, **sí** muestra los errores | Los fallos serían silenciosos |
| `-L` (`--location`) | Sigue las redirecciones (301/302) | Descargarías la página de «movido a…» |

Para pasar opciones al script: `curl -fsSL URL | bash -s -- --yes` (`-s`: leer el script de stdin; `--`: lo que sigue son argumentos del script).

## Riesgos

> [!danger] Ejecutas sin leer
> El script corre con **tus** permisos: puede borrar tu `$HOME` o robar tus claves SSH. Solo hazlo con fuentes de confianza, y nunca con `| sudo bash`.

1. **Descarga cortada.** Bash ejecuta **mientras** llega el script, línea a línea. Si la conexión se corta entre dos órdenes, se queda una instalación a medias. Y si se corta a mitad de una línea, Bash puede ejecutar media orden: `rm -rf ~/.cache/tareas` cortado justo después de `rm -rf ~/` borraría tu carpeta personal.
2. **Contenido distinto.** Lo que ves en el navegador puede no ser lo que llega al `curl` (otra rama, otro servidor). Solo una versión fija con checksum lo garantiza.
3. **`read` no funciona.** La entrada de Bash es la tubería, así que `read` lee **líneas del propio script**. Para preguntar, lee de la terminal: `read -r resp < /dev/tty`, o usa valores por defecto (ver [[Qué hace un buen instalador]]).

## Defensa: todo dentro de `main`, llamada al final

Si todo el código está en funciones y la **única** orden de nivel superior es `main "$@"` en la última línea, una descarga cortada solo define funciones a medias y **no ejecuta nada**:

```bash
#!/usr/bin/env bash
# install.sh — instala «tareas» en ~/.local/bin
# Uso: curl -fsSL https://raw.githubusercontent.com/USUARIO/tareas/main/install.sh | bash
set -euo pipefail

REPO="USUARIO/tareas"
VERSION="${TAREAS_VERSION:-main}"
DESTINO="${TAREAS_DESTINO:-$HOME/.local/bin}"
URL="${TAREAS_URL:-https://raw.githubusercontent.com/$REPO/$VERSION/tareas.sh}"
TMP=""          # global: el trap EXIT se ejecuta cuando main ya ha terminado

info()  { printf '\033[1;34m==>\033[0m %s\n' "$*"; }
fallo() { printf '\033[1;31merror:\033[0m %s\n' "$*" >&2; exit 1; }

main() {
    command -v curl >/dev/null || fallo "hace falta curl"
    TMP=$(mktemp)
    trap 'rm -f "$TMP"' EXIT

    info "Descargando $URL"
    curl -fsSL "$URL" -o "$TMP" || fallo "no se pudo descargar"
    bash -n "$TMP" || fallo "el archivo descargado no es un script válido"

    mkdir -p "$DESTINO"
    install -m 755 "$TMP" "$DESTINO/tareas"     # copia + permisos en un paso
    info "Instalado en $DESTINO/tareas"

    case ":$PATH:" in
        *":$DESTINO:"*) ;;
        *) info "Añade a tu ~/.bashrc: export PATH=\"$DESTINO:\$PATH\"" ;;
    esac
}

main "$@"      # nada se ejecuta hasta que el archivo ha llegado entero
```

> [!warning] `local` + `trap EXIT` + `set -u`
> Si `TMP` fuera `local` dentro de `main`, el `trap` se ejecutaría al salir del script, cuando la variable ya no existe, y `set -u` daría `TMP: unbound variable`. Por eso es global. Más en [[Scripts robustos con set y trap]].

Al cortar el script a la mitad (`head -c 300 install.sh | bash`) Bash llega a asignar alguna variable, dice `unexpected EOF` y **no instala nada**.

## Forma más segura: descargar, leer, ejecutar

```bash
curl -fsSLO https://github.com/USUARIO/tareas/releases/download/v1.0.0/install.sh
less install.sh                    # léelo (o ábrelo en Vim)
bash install.sh
```

Con **checksum** publicado junto al release:

```bash
curl -fsSLO https://github.com/USUARIO/tareas/releases/download/v1.0.0/SHA256SUMS
sha256sum -c --ignore-missing SHA256SUMS   # install.sh: OK
```

## Publicar `install.sh` en GitHub

| Dónde | URL | Ventaja / pega |
|---|---|---|
| En el repo | `https://raw.githubusercontent.com/USUARIO/REPO/main/install.sh` | Fácil; cambia con cada *commit* |
| En una etiqueta | `…/REPO/v1.0.0/install.sh` (en vez de `main`) | Fijo para siempre |
| Adjunto a un release | `https://github.com/USUARIO/REPO/releases/latest/download/install.sh` | `latest` apunta siempre a la última versión |

Crear el release con [[GitHub CLI (gh)]]:

```bash
sha256sum install.sh tareas.sh > SHA256SUMS
gh release create v1.0.0 install.sh tareas.sh SHA256SUMS --title "v1.0.0" --notes "Primera versión"
```

Pon la línea de instalación y la forma segura en el [[README de un proyecto|README]].

## Alternativas

- **pipx** (apps de Python): `pipx install git+https://github.com/USUARIO/tareas` crea un entorno aislado y deja el comando en `~/.local/bin`.
- **Paquetes del sistema**: `.deb`, Homebrew, AUR… Más trabajo para ti, más confianza y actualizaciones para el usuario.
- **Binario único** (Go, Rust): se descarga del release y se copia, sin dependencias (ver [[Elegir un framework de TUI]]).

## Errores típicos

| Error | Consecuencia |
|---|---|
| `curl -s` sin `-f` | Un 404 se ejecuta como script: en GitHub sale `404:: command not found`; con una página HTML, `syntax error near unexpected token` |
| Órdenes sueltas fuera de funciones | Una descarga cortada ejecuta media instalación |
| `read` sin `< /dev/tty` | El script se «come» sus propias líneas |
| Apuntar a `main` en la documentación | Un *commit* roto rompe la instalación de todos |

## Relacionadas

[[Qué hace un buen instalador]] · [[Instalador TUI en Bash]] · [[GitHub CLI (gh)]] · [[README de un proyecto]] · [[Scripts robustos con set y trap]] · [[Redirecciones y tuberías]] · [[Funciones en Bash]] · [[Git para 42]]

## Referencias

- [curl(1)](https://curl.se/docs/manpage.html) — `-f`, `-s`, `-S`, `-L`, `-O`.
- [Bash manual: Invoking Bash](https://www.gnu.org/software/bash/manual/bash.html#Invoking-Bash) — `bash -s` y `--`.
- [GitHub Docs: Linking to releases](https://docs.github.com/en/repositories/releasing-projects-on-github/linking-to-releases) — `releases/latest/download/FICHERO`.
- [gh release create](https://cli.github.com/manual/gh_release_create) — crear un release con ficheros adjuntos.
- [sha256sum(1)](https://man7.org/linux/man-pages/man1/sha256sum.1.html) — `-c` e `--ignore-missing`.
- [pipx](https://pipx.pypa.io/) — instalar apps de Python aisladas.
- `install.sh` probado con `curl -fsSL file://… | bash` (instala), con URL inexistente (error y código 1) y truncado a 300 bytes (no instala nada); un 404 real de raw.githubusercontent.com pasado a `bash` sin `-f`.
