---
tags: [tui, frameworks, decision]
aliases: ["Qué framework de TUI usar", "Elegir librería TUI"]
---
# Elegir un framework de TUI

[[Inicio|← Inicio]] · [[TUI - Índice|TUI]]

> No hay un framework «mejor»: hay uno que encaja con **tu lenguaje, tus restricciones y el tamaño de tu interfaz**. Esta nota te da cinco preguntas, un árbol de decisión y una regla para saber cuándo dejar curses. La lista completa de opciones está en [[Frameworks de TUI - Listado]].

## Vocabulario

| Término | Qué significa |
|---|---|
| **Dependencia** | Biblioteca externa que hay que instalar aparte (`pip install`, `go get`…) |
| **Biblioteca estándar** | Lo que ya trae el lenguaje: en Python, `curses` viene incluido (en Linux y macOS) |
| **Binario único** | Un solo ejecutable que copias y funciona, sin instalar Python ni paquetes (Go, Rust) |
| **Entorno virtual** (*venv*) | Carpeta aislada con los paquetes de Python de un proyecto |
| **Widget** | Pieza de interfaz ya hecha (lista, botón, campo de texto). Más términos (modo inmediato, arquitectura Elm…) en [[Frameworks de TUI - Listado#Vocabulario\|el listado]] |

## Las cinco preguntas

1. **¿Qué lenguaje ya sabes?** Aprender un framework **y** un lenguaje a la vez duplica la dificultad. Si sabes C y algo de Python, empieza por Python.
2. **¿Puedes instalar dependencias?** En un proyecto de 42 a veces solo vale la biblioteca estándar → `curses`. En un proyecto personal, puedes usar `pip install textual`.
3. **¿Cómo lo vas a distribuir?** Si quieres que otros lo instalen con un solo archivo sin tener Python, Go (Bubble Tea) o Rust (Ratatui) generan un **binario único**. Ver [[Instalar con una línea]].
4. **¿Qué complejidad tiene la interfaz?** Un menú y dos preguntas → Bash con whiptail o gum. Varias pantallas, listas largas, formularios → un framework con widgets.
5. **¿Necesitas widgets, ratón o estilos tipo CSS?** curses te deja hacerlo todo, pero **a mano**. Textual trae ratón, scroll, foco y CSS de serie.

## Árbol de decisión

```text
¿Es un script que hace 1-3 preguntas?
├── Sí → Bash + whiptail (ya instalado) o gum (más bonito)
│        └── ¿Elegir de una lista larga? → fzf
└── No, es una aplicación con pantallas
    ├── ¿Solo biblioteca estándar / quieres entender cómo funciona?
    │   └── Python + curses
    ├── ¿Python con dependencias permitidas?
    │   ├── Solo salida bonita (tablas, progreso) → Rich
    │   ├── Una línea de entrada con autocompletado → prompt_toolkit
    │   └── App completa con widgets/ratón → Textual
    ├── ¿Quieres un binario único?
    │   ├── Go → Bubble Tea (+ Lip Gloss, Bubbles) o tview si quieres widgets ya hechos
    │   └── Rust → Ratatui
    ├── ¿Ya trabajas con React/Node? → Ink
    └── ¿C puro? → ncurses (notcurses si quieres imágenes y true color)
```

## Comparativa rápida

| Si te importa… | Mejor opción | Por qué |
|---|---|---|
| Cero dependencias | curses / whiptail | Vienen con Python y con Debian/Ubuntu |
| Aprender los fundamentos | curses | Ves el bucle, el redibujado y las teclas tal cual (ver [[curses - El bucle principal]]) |
| Resultado bonito rápido | Textual / gum | Temas, bordes y colores sin pensar |
| Rendimiento y binario único | Ratatui / Bubble Tea | Compilado, arranca al instante |
| Comunidad y ejemplos | Textual, Bubble Tea, Ratatui | Son los más activos en 2026 |

## Cuándo quedarse con curses

- La interfaz cabe en **una pantalla** con unas pocas zonas (lista + detalle + barra de estado).
- No puedes instalar nada (máquinas de 42, servidores).
- Quieres **aprender**: todo lo que aprendas con curses (fotogramas, entrada no bloqueante, layout) sirve en cualquier otro framework.

## Cuándo pasar a Textual, Bubble Tea o Ratatui

> [!tip] Señales de que curses se te queda corto
> - Estás escribiendo tu propio sistema de **foco** (qué widget recibe las teclas) o de **scroll**.
> - Necesitas **ratón**, pestañas, diálogos modales encima de otras ventanas.
> - Tu función de dibujar pasa de 200 líneas y cada cambio rompe otra cosa.
> - Te piden que funcione en Windows sin trucos (curses no viene en Windows).

Entonces:

- **Textual** si sigues en Python. Lo que sabes de separar estado y dibujo (ver [[De script a TUI - arquitectura]]) se traslada directo.
- **Bubble Tea** si quieres aprender Go y distribuir un binario. Su arquitectura Elm obliga a separar estado, eventos y vista: muy buena escuela.
- **Ratatui** si quieres Rust y control total del bucle (modo inmediato, como curses pero con widgets).

> [!warning] No cambies de framework a mitad por moda
> Si tu TUI en curses funciona y es pequeña, mejorarla (colores, atajos, layout: ver [[Diseño de una TUI moderna]]) suele costar menos que reescribirla.

## Errores típicos

| Error | Consecuencia |
|---|---|
| Elegir Rust/Go solo por el binario sin saber el lenguaje | Pasas semanas peleando con el lenguaje, no con la TUI |
| Usar Rich esperando teclas y menús | Rich **solo** pinta salida; para interacción, Textual |
| Empezar con blessed (JS) o npyscreen por un tutorial viejo | Proyectos poco mantenidos; mira la fecha de la última versión |
| Hacer en Bash una app de 10 pantallas | El código se vuelve inmanejable; pasa a Python |

## Relacionadas

[[Frameworks de TUI - Listado]] · [[CLI, TUI y GUI]] · [[TUIs en Bash - dialog, whiptail y gum]] · [[curses - Primeros pasos]] · [[De script a TUI - arquitectura]] · [[Diseño de una TUI moderna]] · [[Instalar con una línea]]

## Referencias

- [Python curses HOWTO](https://docs.python.org/3/howto/curses.html) — qué incluye curses y que no está disponible en la versión de Windows de Python.
- [Textual — Guide](https://textual.textualize.io/guide/) — widgets, CSS, ratón y foco.
- [Bubble Tea](https://github.com/charmbracelet/bubbletea) — arquitectura Elm en Go.
- [Ratatui — Concepts](https://ratatui.rs/concepts/) — renderizado en modo inmediato.
- [Rich](https://rich.readthedocs.io/) — biblioteca de salida formateada (no interactiva).
