---
tags: [tui, indice, terminal, curses, python]
aliases: ["TUI", "Interfaces de texto", "Text User Interface"]
---
# TUI - Índice

[[Inicio|← Inicio]] · TUI

> Una **TUI** (*Text User Interface*, interfaz de usuario de texto) es un programa que ocupa toda la terminal y se maneja con el teclado, como `htop`, `vim` o `lazygit`. Estas notas enseñan a convertir un script de Bash en una TUI moderna escrita en Python con `curses`, y a hacer animaciones e instaladores para la terminal.

## Requisitos previos

| Necesitas | Dónde aprenderlo |
|---|---|
| Escribir scripts de Bash con subcomandos | [[Bash - Índice]] (sobre todo [[Argumentos y subcomandos en Bash]]) |
| Python básico: funciones, clases, listas, diccionarios | [[Python - Índice]] |
| Moverte por la terminal y dividirla en paneles | [[tmux - Índice]] (muy útil para probar la TUI en un panel mientras editas en otro) |

> [!tip] El hilo conductor: `tareas.py`
> En [[Bash - Índice]] se construye `tareas.sh`, una lista de tareas por terminal (`añadir`, `lista`, `borrar`). Aquí la convertimos poco a poco en `tareas.py`, una TUI que **usa `tareas.sh` por debajo**. Cada nota de `curses` añade una pieza: la ventana, el bucle, los paneles, los colores, los atajos y la ejecución de comandos.

## Ruta de aprendizaje

Sigue el orden de las secciones. Dentro de cada una, de arriba abajo.

### 1. Fundamentos

| Nota | Qué aprendes |
|---|---|
| [[CLI, TUI y GUI]] | Las tres formas de interfaz y cuándo merece la pena una TUI |
| [[Cómo funciona la terminal]] | tty, modo raw, secuencias de escape, tamaño y `TERM`: lo que hay debajo de cualquier TUI |
| [[De script a TUI - arquitectura]] | Separar lógica y vista; cómo llamar a `tareas.sh` desde Python |
| [[Unicode y emojis en la terminal]] | Por qué un emoji ocupa dos columnas y cómo recortar texto bien |
| [[Diseño de una TUI moderna]] | Principios de diseño y checklist final |

### 2. curses en Python

| Nota | Qué aprendes |
|---|---|
| [[curses - Primeros pasos]] | `curses.wrapper`, `addstr`, coordenadas `(y, x)`, `getkey` |
| [[curses - El bucle principal]] | Estado → dibujar → tecla → actualizar; una clase `App` |
| [[curses - Layout y paneles]] | Cabecera, lista con scroll, detalle, barra de atajos; redimensionar |
| [[curses - Colores y estilos]] | Pares de color, atributos, 256 colores, `NO_COLOR` |
| [[curses - Teclado y atajos]] | Flechas, Enter, Esc, tabla de atajos, ayuda con `?`, filtrar |
| [[curses - Ejecutar comandos]] | `subprocess` sin congelar la interfaz; abrir `vim` y volver |

### 3. Frameworks

| Nota | Qué aprendes |
|---|---|
| [[Frameworks de TUI - Listado]] | Textual, Rich, Bubble Tea, Ratatui… qué hay en cada lenguaje |
| [[Elegir un framework de TUI]] | Cuándo basta con `curses` y cuándo compensa un framework |
| [[TUIs en Bash - dialog, whiptail y gum]] | Menús y diálogos sin salir de Bash |

### 4. Animaciones

| Nota | Qué aprendes |
|---|---|
| [[Animación en la terminal - fundamentos]] | Fotogramas, `\r`, redibujar sin parpadeo, FPS |
| [[Spinners y barras de progreso]] | Indicar que algo está pasando y cuánto falta |
| [[curses - Animaciones]] | Animar dentro del bucle de `curses` con `timeout` |

### 5. Instaladores

| Nota | Qué aprendes |
|---|---|
| [[Qué hace un buen instalador]] | Comprobar requisitos, preguntar poco, poder deshacer |
| [[Instalador TUI en Bash]] | Un instalador con menús y progreso en Bash |
| [[Instalador TUI en Python]] | El mismo instalador con `curses` |
| [[Instalar con una línea]] | `curl ... \| bash`: cómo hacerlo y sus riesgos |

## Cómo practicar

1. Abre [[tmux - Uso básico|tmux]] con dos paneles: en uno editas con [[Vim]], en el otro ejecutas `python3 tareas.py`.
2. Escribe los ejemplos a mano, ejecútalos y **rómpelos a propósito**: escribe fuera de la pantalla, quita el `refresh()`, olvida el `wrapper`. Ver el error es la mejor forma de entenderlo.
3. Guarda `tareas.sh` y `tareas.py` en el mismo repo (ver [[Git para 42]]) y haz un commit por cada nota: verás crecer la interfaz.
4. Cuando funcione, graba una demo para el README (ver [[Grabar una demo de terminal]] y [[README de un proyecto]]).

> [!warning] Si la terminal se queda «rota»
> Si un programa de `curses` termina mal, la terminal puede quedarse sin eco o con caracteres raros. Escribe `reset` (aunque no veas lo que tecleas) y pulsa Enter. Para evitarlo, usa siempre `curses.wrapper` (ver [[curses - Primeros pasos]]).

## Relacionadas

[[CLI, TUI y GUI]] · [[Bash - Índice]] · [[Python - Índice]] · [[tmux - Índice]] · [[Colores en la terminal]] · [[Automatizar tmux desde scripts]] · [[GitHub CLI (gh)]]

## Referencias

- [curses — Terminal handling for character-cell displays](https://docs.python.org/3/library/curses.html) — referencia del módulo `curses` de Python.
- [Curses Programming with Python (HOWTO)](https://docs.python.org/3/howto/curses.html) — tutorial oficial; base de la ruta de las notas de `curses`.
- [ncurses — man pages](https://invisible-island.net/ncurses/man/ncurses.3x.html) — la biblioteca en C que hay debajo.
- [Text-based user interface — Wikipedia](https://en.wikipedia.org/wiki/Text-based_user_interface) — definición e historia de las TUI.
