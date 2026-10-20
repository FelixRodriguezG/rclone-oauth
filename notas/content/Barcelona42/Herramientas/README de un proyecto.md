---
tags: [herramientas, github, documentacion, markdown, readme]
aliases: ["README", "README.md", "Markdown de GitHub"]
---
# README de un proyecto

[[Inicio|← Inicio]] · Herramientas

> El **README** (`README.md`, en la raíz del repo) es la portada del proyecto: lo primero que GitHub muestra al entrar. En pocos segundos debe contestar *qué es*, *cómo lo instalo* y *cómo lo uso*. En los subjects recientes de 42 además es **obligatorio** y tiene un formato mínimo.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Markdown** | Formato de texto plano con marcas sencillas (`#`, `**`, `-`) que se convierte en página con estilo |
| **GFM** | *GitHub Flavored Markdown*: el Markdown de GitHub, con tablas, listas de tareas y avisos |
| **Ruta relativa** | Ruta desde la carpeta del fichero (`docs/demo.gif`), sin `https://…` ni `/home/…` |
| **Badge** (insignia) | Imagen pequeña tipo etiqueta: versión, licencia, estado de los tests |
| **Licencia** | Texto que dice qué pueden hacer otros con tu código. Sin licencia, legalmente nadie puede reutilizarlo |

## Secciones típicas

En este orden, de lo más importante a lo menos:

1. **Título y una frase**: qué hace, en una línea.
2. **Captura o GIF**: enseñar vale más que explicar, sobre todo en una TUI (ver [[Grabar una demo de terminal]]).
3. **Requisitos**: sistema, versión de Python/Bash, dependencias (`tmux ≥ 3.2`, `python3 ≥ 3.10`).
4. **Instalación**: comandos copiables, de uno en uno (ver [[Instalar con una línea]]).
5. **Uso**: ejemplos reales con la salida esperada; atajos de teclado si es interactivo.
6. **Estructura**: árbol corto de carpetas, si ayuda a orientarse.
7. **Créditos**: autores, recursos y librerías de terceros.
8. **Licencia**: una línea y el fichero `LICENSE` (MIT es la habitual en proyectos pequeños).

> [!info] README en 42
> Los subjects recientes (por ejemplo, el de libft) exigen un `README.md` en la raíz con, como mínimo:
> - Primera línea **en cursiva**: *Este proyecto ha sido creado como parte del currículo de 42 por tu-login* (si sois varios: `login1, login2`) (en el subject en inglés, la frase va en inglés).
> - Secciones **Descripción**, **Instrucciones** y **Recursos**; en Recursos hay que explicar **para qué se ha usado IA** y en qué partes.
>
> Comprueba siempre el subject de tu proyecto: puede pedir más secciones (el de libft, por ejemplo, añade una descripción detallada de la librería). Ver [[libft - Índice]].

## El Markdown que entiende GitHub

````markdown
# Título
## Sección
### Subsección

**negrita**, *cursiva*, `código`, ~~tachado~~ y [enlace](https://ejemplo.com)

- lista con guiones
1. lista numerada

- [ ] tarea pendiente
- [x] tarea hecha

| Tecla | Acción |
|-------|--------|
| `q`   | Salir  |

```bash
./tareas.sh añadir "Estudiar tmux"
```

> [!NOTE]
> Aviso destacado. También existen [!TIP], [!IMPORTANT], [!WARNING] y [!CAUTION].

<details>
<summary>Ver salida completa</summary>

Texto que se despliega al hacer clic.
</details>
````

- Pon el **lenguaje** tras las tres comillas (` ```bash `, ` ```python `) para tener colores.
- Cada elemento va en **su propia línea**: si juntas `# Título ## Sección` en una línea, GitHub lo ve como un único título.
- Los avisos `> [!NOTE]` de GitHub se parecen a los *callouts* de Obsidian, pero solo admiten esos cinco tipos y en mayúsculas.

## Imágenes y GIF con ruta relativa

Guarda las imágenes **dentro del repo** y enlázalas con ruta relativa:

```markdown
![Demo de tareas.py](docs/demo.gif)
```

```
mi-tui/
├── README.md
├── docs/
│   ├── demo.gif        ← la animación de la portada
│   └── captura.png
└── tareas.py
```

Para controlar el tamaño, usa HTML (GitHub lo permite):

```html
<img src="docs/demo.gif" alt="Demo de tareas.py" width="700">
```

> [!warning] Rutas que fallan
> `/home/tu-usuario/mi-tui/docs/demo.gif` solo existe en tu ordenador; `C:\…` tampoco funciona. Y si el GIF pesa decenas de MB, la página tardará en cargar: intenta quedarte por debajo de ~5 MB.

## Badges

Son imágenes que genera un servicio como [shields.io](https://shields.io) a partir de la URL:

```markdown
![Python](https://img.shields.io/badge/python-3.10%2B-blue)
![Licencia](https://img.shields.io/badge/licencia-MIT-green)
```

Úsalas con moderación: dos o tres arriba del todo. No sustituyen a una frase que explique el proyecto.

## Plantilla mínima

````markdown
# tareas

Gestor de tareas para la terminal, con interfaz de teclado.

![Demo](docs/demo.gif)

## Requisitos
- Linux o macOS
- Python 3.10 o superior

## Instalación
```bash
git clone https://github.com/tu-usuario/tareas.git
cd tareas
./install.sh        # instala el comando `tareas` en ~/.local/bin
```

## Uso
```bash
tareas añadir "Leer el subject"   # añade una tarea
tareas lista                      # →  1. Leer el subject
python3 tareas.py                 # abre la interfaz de teclado
```

| Tecla | Acción |
|---|---|
| `a` | Añadir tarea |
| `q` | Salir |

## Créditos
Hecho por tu-usuario. Basado en la documentación de `curses` de Python.

## Licencia
MIT — ver [LICENSE](LICENSE).
````

## Errores típicos

- **No decir qué hace** el proyecto en la primera línea.
- **Comandos que no funcionan** al copiarlos: pruébalos en una carpeta limpia o en otra máquina.
- **Imágenes rotas**: ruta absoluta, mayúsculas distintas (`Demo.gif` ≠ `demo.gif` en Linux) o el fichero no está en el commit.
- **Datos personales** en capturas: usuario, rutas de tu casa, tokens en el historial de la shell.
- **README desactualizado**: cuando cambies un atajo o una opción, cámbialo también aquí.

## Relacionadas

[[GitHub CLI (gh)]] · [[Grabar una demo de terminal]] · [[Git para 42]] · [[Instalar con una línea]] · [[Qué hace un buen instalador]]

## Referencias

- [GitHub Docs: Basic writing and formatting syntax](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax) — Markdown de GitHub, imágenes con ruta relativa, avisos `[!NOTE]`.
- [GitHub Docs: About READMEs](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-readmes) — dónde se coloca y qué incluir.
- [GitHub Flavored Markdown Spec](https://github.github.com/gfm/) — tablas, listas de tareas, tachado.
- [shields.io](https://shields.io) — generador de badges.
- Subject de libft (`~/cursus/es.subject_libft.pdf`), capítulo «Requisitos del Readme».
