---
tags: [tui, cli, gui, terminal, diseno]
aliases: ["CLI vs TUI", "Tipos de interfaz", "Interfaz de línea de comandos"]
---
# CLI, TUI y GUI

[[Inicio|← Inicio]] · [[TUI - Índice|TUI]]

> Un programa puede hablar con la persona de tres formas: escribiendo órdenes y leyendo texto (**CLI**), con una pantalla de texto interactiva dentro de la terminal (**TUI**) o con ventanas, botones y ratón (**GUI**). Saber cuál elegir evita tanto el script imposible de usar como la interfaz que nadie necesitaba.

## Vocabulario

| Término | Qué significa |
|---|---|
| **CLI** (*Command Line Interface*) | Escribes un comando con argumentos, el programa responde con texto y termina: `ls -l`, `git status` |
| **TUI** (*Text User Interface*) | Programa que **ocupa toda la terminal**, se queda abierto y reacciona a cada tecla: `htop`, `vim` |
| **GUI** (*Graphical User Interface*) | Ventanas gráficas con botones, iconos y ratón: VS Code, el navegador |
| **REPL** | «Lee, evalúa, imprime, repite»: un intérprete interactivo línea a línea (`python3`, `bash`). Está entre CLI y TUI |
| **Script** | Programa corto, normalmente CLI, que automatiza una tarea |

## Ejemplos reales

| Tipo | Ejemplos | Cómo se usa |
|---|---|---|
| CLI | `ls`, `grep`, `git commit -m "..."`, `norminette`, tu `tareas.sh` | Una orden → una respuesta → vuelves al prompt |
| TUI | `htop` (procesos), [[Vim]], `lazygit` (git), `ranger` (archivos), `less`, `mc` | Se abre, navegas con flechas o `j`/`k`, sales con `q` |
| GUI | VS Code, Firefox, GitKraken, el explorador de archivos | Ratón, ventanas, menús |

Fíjate en que muchas herramientas tienen **las tres**: `git` (CLI), `lazygit` o `tig` (TUI) y GitKraken (GUI) hacen lo mismo de formas distintas. Una buena TUI suele ser una capa **encima** de una CLI, que es justo lo que haremos con `tareas.sh` (ver [[De script a TUI - arquitectura]]).

## Ventajas e inconvenientes

| | CLI | TUI | GUI |
|---|---|---|---|
| Automatizable (scripts, tuberías, cron) | ✔ La mejor | ✘ Difícil | ✘ Casi imposible |
| Fácil de descubrir para quien empieza | ✘ Hay que leer `--help` | ✔ Ves las opciones en pantalla | ✔ La más fácil |
| Funciona por SSH, en un servidor o en tmux | ✔ | ✔ | ✘ Necesita entorno gráfico |
| Ligera y rápida de arrancar | ✔ | ✔ | ✘ Más pesada |
| Ver estado que cambia (procesos, logs) | ✘ Imprime y se va | ✔ Se actualiza sola | ✔ |
| Coste de programarla | Bajo | Medio | Alto |
| Imágenes, gráficos, arrastrar y soltar | ✘ | ~ Limitado | ✔ |

> [!info] Por qué las TUI siguen vivas
> En 42 y en cualquier trabajo con servidores pasas el día en la terminal, muchas veces por SSH. Una TUI te da la comodidad de «ver y elegir» sin salir de ahí, y funciona igual en tu portátil que en una máquina remota sin pantalla.

## Señales de que tu script pide una TUI

- **Muchas opciones** que nadie recuerda: tu `--help` ocupa dos pantallas y siempre acabas consultándolo.
- **Estado que mirar**: quieres ver una lista que cambia (tareas, procesos, descargas, tests) y no volver a ejecutar el comando cada vez.
- **Flujo interactivo**: elegir un elemento de una lista, confirmar, editar, volver atrás. Encadenar eso con `read` en Bash se vuelve un laberinto (ver [[Leer entrada del usuario en Bash]]).
- **Navegar** por algo grande: árboles de archivos, logs largos, resultados de búsqueda.
- Lo usan personas **que no lo han escrito** y necesitan descubrirlo solas.

Con `tareas.sh`, en cuanto tienes 20 tareas y quieres borrar la 17, escribir `./tareas.sh lista`, buscar el número y luego `./tareas.sh borrar 17` cansa. Una lista con flechas y una tecla `d` es más cómoda.

## Cuándo **no** hacer una TUI

- Lo vas a usar **desde otros scripts**, `cron` o una tubería: necesitas una CLI con salida limpia y [[Códigos de salida y errores en Bash|códigos de salida]].
- Hace **una sola cosa** con uno o dos argumentos (`./compilar.sh`, `./tareas.sh añadir X`).
- Lo ejecutarás en CI o en un entorno sin terminal interactiva (no hay teclado al que esperar).
- Basta con un menú sencillo: usa `select` de Bash o `gum`/`whiptail` (ver [[TUIs en Bash - dialog, whiptail y gum]]) antes de escribir 300 líneas de Python.
- Necesitas imágenes, gráficas complejas o ratón como protagonista: eso es una GUI o una web.

> [!tip] La regla de oro
> **Primero la CLI, después la TUI.** Si la lógica funciona como comandos (`añadir`, `lista`, `borrar`), la TUI solo tiene que llamarlos y mostrarlos bonitos. Además, la CLI sigue sirviendo para automatizar.

## Errores típicos

| Error | Consecuencia |
|---|---|
| Meter la lógica dentro de la TUI | No puedes usarla desde scripts ni probarla sin abrir la interfaz |
| TUI sin forma de salir clara | La gente cierra la terminal y pierde datos (pon `q` y muéstralo) |
| Imitar una GUI (menús anidados, ratón obligatorio) | Más lenta de usar que la CLI que sustituía |
| Hacer una TUI para algo que se ejecuta una vez al mes | Mucho código que mantener para nada |

## Relacionadas

[[TUI - Índice]] · [[Cómo funciona la terminal]] · [[De script a TUI - arquitectura]] · [[Diseño de una TUI moderna]] · [[Elegir un framework de TUI]] · [[Argumentos y subcomandos en Bash]] · [[Primer script en Bash]]

## Referencias

- [Command-line interface — Wikipedia](https://en.wikipedia.org/wiki/Command-line_interface) — definición de CLI y comparación con GUI.
- [Text-based user interface — Wikipedia](https://en.wikipedia.org/wiki/Text-based_user_interface) — definición de TUI y ejemplos históricos.
- [Command Line Interface Guidelines](https://clig.dev/) — cuándo una CLI basta, salida para humanos frente a máquinas, interactividad.
- [htop](https://htop.dev/), [lazygit](https://github.com/jesseduffield/lazygit), [ranger](https://github.com/ranger/ranger) — ejemplos de TUI citados.
