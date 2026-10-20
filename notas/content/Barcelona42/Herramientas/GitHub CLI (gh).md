---
tags: [herramientas, git, github, gh, cli]
aliases: ["gh", "GitHub CLI", "gh auth login"]
---
# GitHub CLI (gh)

[[Inicio|← Inicio]] · Herramientas

> **`gh`** es el programa oficial de GitHub para la terminal. Sirve para iniciar sesión en GitHub sin pelearte con contraseñas, crear un repositorio a partir de tu carpeta y subirlo con un solo comando, cambiar si es público o privado, y gestionar *pull requests* e *issues* sin abrir el navegador.

## Vocabulario

| Término | Qué significa |
|---|---|
| **GitHub** | Web que aloja repositorios Git. No es lo mismo que Git (el programa) ni que Vogsphere (el servidor de entrega de 42) |
| **Token** | «Contraseña» larga que GitHub genera para un programa. `gh` la guarda por ti |
| **Código de dispositivo** | Código corto (`ABCD-1234`) que escribes en la web para autorizar a un programa que no tiene navegador |
| **Credential helper** | Programa al que Git pregunta usuario y contraseña cuando hace `push` por HTTPS |
| **Remoto** | Copia del repo en otro sitio (ver [[Git para 42]]). Un repo puede tener varios: `origin`, `github`… |
| **Visibilidad** | `public` (lo ve todo el mundo) o `private` (solo tú y quien invites) |

## Instalar

```bash
gh --version          # ¿ya está?  → gh version 2.x
```

- Linux (Debian/Ubuntu): `sudo apt install gh`, o el repositorio oficial de GitHub para tener una versión reciente (instrucciones en cli.github.com).
- macOS: `brew install gh`.
- **Sin `sudo`** (ordenadores de 42): descarga el `.tar.gz` de *Releases* en GitHub, descomprímelo y copia el binario `bin/gh` a `~/.local/bin/`. Esa carpeta debe estar en tu `PATH` (la lista de carpetas donde la shell busca los programas); compruébalo con `echo "$PATH"`.

## Iniciar sesión: `gh auth login`

```bash
gh auth login
```

Responde a las preguntas con las flechas:

1. *Where do you use GitHub?* → `GitHub.com`
2. *Preferred protocol for Git operations?* → `HTTPS` (lo más sencillo; con SSH necesitas tu clave subida)
3. *Authenticate Git with your GitHub credentials?* → `Yes`
4. *How would you like to authenticate?* → `Login with a web browser`

Te mostrará algo así:

```
! First copy your one-time code: 1A2B-3C4D
Press Enter to open https://github.com/login/device in your browser...
```

Abre **https://github.com/login/device** (en ese ordenador o en tu móvil, da igual), inicia sesión en GitHub, escribe el código y pulsa *Authorize*. La terminal continúa sola: `✓ Logged in as tu-usuario`.

> [!warning] El código caduca
> El código de dispositivo dura unos **15 minutos**. Si tardas más, `gh` falla y hay que volver a ejecutar `gh auth login` para obtener otro. Con `gh auth login --web --clipboard` se salta las preguntas y copia el código al portapapeles.

### Comprobar: `gh auth status`

```
github.com
  ✓ Logged in to github.com account tu-usuario (keyring)
  - Active account: true
  - Git operations protocol: https
  - Token: gho_************************************
  - Token scopes: 'gist', 'read:org', 'repo'
```

`keyring` significa que el token está guardado en el almacén seguro del sistema. Si no hay ninguno, `gh` lo guarda en texto plano en `~/.config/gh/hosts.yml`: **no** compartas esa carpeta ni la subas a un repo.

### Que `git push` no pida usuario: `gh auth setup-git`

```bash
gh auth setup-git
```

Configura `gh` como *credential helper* de Git: a partir de ahí, `git push` a GitHub por HTTPS usa el token de `gh` y no te pide usuario ni contraseña (GitHub ya no acepta tu contraseña normal en `git push`). Si en el paso 3 respondiste `Yes`, ya está hecho; ejecutarlo de nuevo no hace daño.

## Crear un repositorio desde tu carpeta

Tienes un proyecto con commits en local y quieres subirlo a GitHub:

```bash
cd ~/proyectos/mi-tui
git status                     # debe ser un repo con al menos un commit
gh repo create mi-tui --private --source=. --push
```

- `--source=.`: usa el repo de la carpeta actual en vez de crear uno vacío.
- `--push`: sube tus commits en el mismo paso.
- `--private` / `--public`: obligatorio elegir uno.
- Añade el remoto con el nombre `origin`, salvo que digas otro con `--remote`.

> [!danger] Repos de 42 y el remoto `origin`
> En un proyecto clonado de Vogsphere, `origin` **ya es** el repositorio de entrega. Usa `gh repo create mi-proyecto --private --source=. --remote github --push` para añadir GitHub como un segundo remoto llamado `github`, y sube con `git push github main`. Antes de hacer público un proyecto de 42, piensa en la normativa sobre compartir soluciones.

## Cambiar la visibilidad y abrir en la web

```bash
gh repo edit --visibility public --accept-visibility-change-consequences
gh repo view            # muestra el README en la terminal
gh repo view --web      # abre el repo en el navegador
```

`gh` exige `--accept-visibility-change-consequences` en **cualquier** cambio de visibilidad, porque tiene consecuencias: al pasar a **público**, todo el mundo ve el código y todo el historial de commits; al pasar a **privado**, se borran para siempre las estrellas y los *watchers* (quienes siguen el repo) y se desconectan los *forks* (copias de otros usuarios).

## Pull requests e issues (lo justo)

```bash
gh issue list                               # issues abiertas
gh issue create --title "Falla con líneas vacías" --body "Pasos: ..."
gh pr create --fill                         # PR con el título y texto de tus commits
gh pr list                                  # PR abiertas
gh pr checkout 12                           # traer la PR 12 a tu máquina para probarla
gh pr view 12 --web                         # verla en el navegador
```

Un ***pull request*** (PR) es una propuesta de cambios de una rama a otra para que alguien la revise; una ***issue*** es una incidencia o tarea apuntada en el repo.

## Errores típicos

- **`gh: To get started with GitHub CLI, please run: gh auth login`**: aún no has iniciado sesión.
- **`git push` pide usuario/contraseña**: falta `gh auth setup-git`, o el remoto usa SSH y no tienes clave.
- **`Unable to add remote "origin"`**: ya existe un `origin`; usa `--remote otro_nombre`.
- **`current directory is not a git repository`** con `--source=.`: haz antes `git init` y un primer commit.
- **Subir datos personales**: revisa `git log` y los ficheros antes de poner el repo `public`.

## Relacionadas

[[Git para 42]] · [[README de un proyecto]] · [[Grabar una demo de terminal]] · [[Instalar con una línea]]

## Referencias

- [GitHub CLI manual](https://cli.github.com/manual/) — `gh auth login` (`--web`, `--clipboard`, `-p`), `gh auth status`, `gh auth setup-git`, `gh repo create` (`--source`, `--push`, `--remote`, `--private/--public`), `gh repo edit --visibility`, `gh repo view --web`, `gh pr`, `gh issue`.
- [GitHub Docs: Authorizing OAuth apps — Device flow](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/authorizing-oauth-apps#device-flow) — `github.com/login/device` y caducidad del código a los 15 minutos.
- [GitHub Docs: Setting repository visibility](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/setting-repository-visibility) — consecuencias de cambiar la visibilidad.
