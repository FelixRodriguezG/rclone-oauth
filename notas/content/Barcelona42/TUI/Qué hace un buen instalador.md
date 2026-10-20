---
tags: [tui, instalador, bash, buenas-practicas]
aliases: ["Buen instalador", "Instalador", "Installer"]
---
# Qué hace un buen instalador

[[Inicio|← Inicio]] · [[TUI - Índice|TUI]]

> Un **instalador** copia tu programa al sitio correcto del sistema del usuario y lo deja listo para usar. Uno bueno **no sorprende**: avisa de lo que va a hacer, se puede repetir sin romper nada, no pide permisos de más, funciona sin preguntar en automático y sabe deshacerse. Esta nota es la teoría; la práctica está en [[Instalador TUI en Bash]] e [[Instalador TUI en Python]].

## Vocabulario

| Término | Qué significa |
|---|---|
| **Idempotente** | Ejecutarlo 2 veces deja el sistema igual que ejecutarlo 1 vez (no duplica líneas, no falla porque «ya existe») |
| **`PATH`** | Lista de carpetas donde la shell busca los comandos. Si tu programa no está en una, hay que escribir la ruta completa |
| **`~/.local/bin`** | Carpeta estándar para ejecutables **de tu usuario** (no necesita `sudo`) |
| **`~/.config/<app>`** | Carpeta estándar para la configuración del usuario (`$XDG_CONFIG_HOME`) |
| **XDG** | Especificación de freedesktop.org que dice dónde va cada cosa en Linux |
| **Modo no interactivo** | Funciona sin preguntar nada (`--yes`), para scripts y CI |
| **CI** | Integración continua: un servidor (p. ej. GitHub Actions) que ejecuta tus scripts sin nadie delante |

## Los pasos

| # | Paso | Qué hace | Ejemplo |
|---|---|---|---|
| 1 | **Requisitos** | Comprueba SO, comandos y versiones; si falta algo, lo dice y para | `command -v python3`, `uname -s` |
| 2 | **Preguntar** | Solo lo necesario, **siempre con valor por defecto** | `Carpeta [~/.local]:` |
| 3 | **Mostrar el plan** | Resumen de qué se va a copiar y dónde, y confirmar | «Se instalará en … ¿Continuar?» |
| 4 | **Instalar** | Copiar a `~/.local/bin`, crear config en `~/.config/<app>` | `install -m 755 …` |
| 5 | **PATH** | Si la carpeta no está en el `PATH`, añadirla **una vez** o explicar cómo | ver abajo |
| 6 | **Verificar** | Ejecutar el programa instalado y comprobar que responde | `tareas help` |
| 7 | **Resumen** | Qué se hizo, dónde está, cómo usarlo y cómo desinstalar | |

### 1. Comprobar requisitos y SO

```bash
case "$(uname -s)" in
    Linux)  so=linux ;;
    Darwin) so=macos ;;
    *) echo "Sistema no soportado: $(uname -s)" >&2; exit 1 ;;
esac
command -v python3 >/dev/null || { echo "Falta python3" >&2; exit 1; }
[[ -r /etc/os-release ]] && . /etc/os-release && echo "Distro: $ID $VERSION_ID"
```

### 5. PATH de forma idempotente

```bash
linea='export PATH="$HOME/.local/bin:$PATH"'
grep -qxF "$linea" "$HOME/.bashrc" 2>/dev/null || printf '%s\n' "$linea" >> "$HOME/.bashrc"
```

`grep -qxF`: busca la línea **exacta** (`-x`), como texto literal (`-F`), sin mostrar nada (`-q`). Si ya está, no la vuelve a añadir. Ver [[grep, sed y awk lo justo]].

> [!tip] Mejor avisar que tocar
> Modificar el `~/.bashrc` (o `~/.zshrc`) del usuario es invasivo. Muchos instaladores solo **imprimen** la línea que hay que añadir, o preguntan antes.

## Idempotencia

Ejecuta tu instalador dos veces seguidas: no debe fallar ni duplicar nada.

- `mkdir -p` en vez de `mkdir` (no falla si existe).
- **No pisar la configuración** del usuario: `[[ -f "$conf" ]] || cp ejemplo.conf "$conf"`.
- Sobrescribir el **programa** sí está bien (es una actualización).
- Comprobar antes de añadir líneas a ficheros (`grep -qxF`).

## Desinstalar

Un buen instalador trae su contrario (`--uninstall` o `uninstall.sh`):

```bash
rm -f "$HOME/.local/bin/tareas"
read -rp "¿Borrar también la configuración en ~/.config/tareas? [s/N] " r
[[ "$r" == [sS]* ]] && rm -rf "$HOME/.config/tareas"
```

Por defecto, **conserva los datos del usuario**: borrarlos debe ser una decisión explícita.

## No pedir `sudo` si no hace falta

- Instalar en `~/.local/bin` no necesita permisos de administrador.
- Si **de verdad** hace falta (paquete del sistema), pide `sudo` solo para ese comando concreto y di por qué. Nunca ejecutes todo el instalador con `sudo`: los ficheros acabarían siendo de `root` dentro de tu `$HOME`.
- En 42 no tienes `sudo`: un instalador que lo exija no te sirve.

## Modo no interactivo

```bash
./instalar.sh --yes --prefix "$HOME/.local"     # acepta todos los valores por defecto
```

Si no hay terminal (`[[ -t 0 ]]` falso, p. ej. en CI o con `curl … | bash`), no se puede preguntar: usa los valores por defecto o falla con un mensaje claro. Las opciones se procesan como en [[Argumentos y subcomandos en Bash]].

## Logs

- Mensajes cortos en pantalla (`==> Copiando…`), detalles en un fichero. Con `mkdir -p "$HOME/.cache"` y `exec 3>>"$HOME/.cache/tareas-install.log"` abres el log como descriptor 3, y luego `echo "copiado $f" >&3` escribe en él (ver [[Redirecciones y tuberías]]).
- Errores por **stderr** y con código de salida distinto de 0 (ver [[Códigos de salida y errores en Bash]]).
- Si falla, di **qué paso** falló y qué ha quedado a medias: un `trap … ERR` con `$LINENO` ayuda (ver [[Scripts robustos con set y trap]]).

## Checklist

- [ ] Comprueba SO y dependencias antes de tocar nada
- [ ] Todas las preguntas tienen valor por defecto
- [ ] Muestra el plan y pide confirmación
- [ ] No necesita `sudo` (o lo pide solo donde hace falta)
- [ ] Instala en `~/.local/bin` y `~/.config/<app>`
- [ ] Ejecutarlo dos veces no rompe ni duplica nada
- [ ] No pisa la configuración existente
- [ ] Explica o arregla el `PATH`
- [ ] Verifica que el programa arranca
- [ ] Tiene `--yes` para modo no interactivo
- [ ] Tiene desinstalación
- [ ] Errores claros por stderr y código de salida ≠ 0

## Relacionadas

[[Instalador TUI en Bash]] · [[Instalador TUI en Python]] · [[Instalar con una línea]] · [[Scripts robustos con set y trap]] · [[Argumentos y subcomandos en Bash]] · [[Configuración y estado de un script]] · [[Códigos de salida y errores en Bash]] · [[README de un proyecto]]

## Referencias

- [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir-spec/latest/) — `$XDG_CONFIG_HOME` (`~/.config`) y `~/.local/bin` para ejecutables del usuario.
- [os-release(5)](https://man7.org/linux/man-pages/man5/os-release.5.html) — variables `ID` y `VERSION_ID`.
- [install(1)](https://man7.org/linux/man-pages/man1/install.1.html) — copiar y poner permisos en un paso.
- [grep(1)](https://man7.org/linux/man-pages/man1/grep.1.html) — opciones `-q`, `-x`, `-F`.
- Fragmentos probados con Bash 5.1 en Ubuntu 22.04 (el del PATH, ejecutado dos veces con un `HOME` temporal: una sola línea).
