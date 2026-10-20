---
tags: [tui, diseno, ux, accesibilidad, checklist]
aliases: ["Diseño de TUI", "UX en la terminal", "Checklist TUI"]
---
# Diseño de una TUI moderna

[[Inicio|← Inicio]] · [[TUI - Índice|TUI]]

> Que una TUI funcione no basta: tiene que poder usarla alguien que **no** la ha escrito, sin leer el código. Las TUI modernas (`lazygit`, `htop`, `k9s`) comparten unos pocos principios: se descubren solas, responden al instante, usan poco color y no pierden datos. Esta nota los resume con ejemplos de `tareas.py` y acaba con un checklist.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Descubrible** | Que puedas averiguar qué hace sin manual: lo que se puede hacer está a la vista |
| **Feedback** | Respuesta visible a cada acción: «Tarea añadida», un spinner, un error |
| **Estado vacío** | Lo que ve el usuario cuando aún no hay datos (cero tareas) |
| **Jerarquía visual** | Que lo importante destaque más que lo secundario |
| **Accesibilidad** | Que la puedan usar personas con daltonismo, baja visión o lector de pantalla |

## 1. Descubrible

- **Barra de atajos** abajo, siempre visible, con lo que se puede hacer **ahora** (cambia según el modo). Generada desde la tabla de atajos (ver [[curses - Teclado y atajos]]).
- **`?` muestra la ayuda** completa. Es la convención: la gente la prueba.
- Para muchas acciones, un **menú** con Espacio o `:` (como la paleta de comandos de un editor) en lugar de 30 atajos de memoria.
- El título dice dónde estás: `tareas.py · 12 tareas · filtro: libft`.

## 2. Feedback inmediato

- Toda acción produce un cambio visible en menos de 100 ms: la tarea aparece, el mensaje sale en la barra.
- Si algo tarda más de ~1 s, enséñalo: spinner o barra de progreso (ver [[Spinners y barras de progreso]]) y la interfaz sigue respondiendo (ver [[curses - Ejecutar comandos]]).
- Los **errores** dicen qué pasó y qué hacer: `✘ tareas.sh no encontrado: ¿está en la misma carpeta?`, no `Error 2`.
- Los mensajes temporales se van solos a los pocos segundos o con la siguiente tecla.

## 3. Estados vacíos con primeros pasos

Una pantalla en blanco no dice nada. Con cero tareas, `tareas.py` muestra:

```text
 tareas.py

   No hay tareas todavía.

   Pulsa  a  para añadir la primera.
   Pulsa  ?  para ver todos los atajos.
```

Igual para «el filtro no encontró nada» (`Ninguna tarea contiene «xyz». Esc para quitar el filtro.`) o «tareas.sh no responde».

## 4. Jerarquía visual con pocos colores

- Un acento para la **selección** (`A_REVERSE`), uno para **errores**, uno para **éxito**. El resto, color por defecto (ver [[curses - Colores y estilos]]).
- Lo secundario (atajos, contadores, fechas) en `A_DIM`.
- Fondo `-1`: respeta el tema claro u oscuro del usuario.
- Bordes solo donde separan algo. Espacio en blanco también separa.

## 5. Teclas consistentes

| Tecla | Significado esperado |
|---|---|
| `j`/`k`, `↓`/`↑` | Mover selección |
| `g`/`G`, Inicio/Fin | Principio / final |
| Enter | Abrir, confirmar |
| Esc | Cancelar, cerrar, volver (nunca salir del programa entero) |
| `q` | Salir (o cerrar el panel actual) |
| `/` | Buscar o filtrar |
| `?` | Ayuda |
| Espacio | Marcar / menú |

Las mismas teclas hacen lo mismo en **todas** las pantallas. Si usas `d` para borrar, que no sea «detalle» en otra.

## 6. No perder datos

- **Confirma** lo destructivo: `¿Borrar «Comprar pan»? (s/N)`, con **No** por defecto.
- Mejor aún: **deshacer** (`u`) en vez de preguntar siempre.
- Guarda nada más cambiar (con `tareas.sh` ya es así) o avisa al salir si hay cambios sin guardar.
- Ctrl+C sale limpio y restaura la terminal (ver [[curses - El bucle principal]]).

## 7. Accesibilidad

- **`NO_COLOR`**: si la variable existe y no está vacía, sin colores (ver [[curses - Colores y estilos]]).
- **Símbolos además de color**: `✔`/`✘`, `[x]`, `!`. Rojo y verde son los más confundidos por daltonismo.
- **Contraste**: nada de gris oscuro sobre negro para información importante.
- **Alternativa ASCII** si el locale no es UTF-8 (ver [[Unicode y emojis en la terminal]]).
- Todo se puede hacer **con teclado**; el ratón es un extra.
- Funciona en **80 × 24** (el tamaño mínimo clásico) y avisa si la terminal es más pequeña (ver [[curses - Layout y paneles]]).

## 8. Rendimiento

- Arranca en menos de medio segundo: carga los datos y dibuja; lo lento, después y en segundo plano.
- `erase()` en cada fotograma, no `clear()` (parpadeo).
- `timeout` razonable: no despiertes 100 veces por segundo si nada se mueve.
- Con miles de elementos, dibuja solo los visibles (scroll), no la lista entera.

## 9. Que la CLI siga existiendo

La TUI es una capa encima: `./tareas.sh lista` y `./tareas.sh añadir TEXTO` siguen funcionando para scripts (ver [[CLI, TUI y GUI]] y [[De script a TUI - arquitectura]]). El script usa el subcomando `help` (y acepta también `-h` y `--help`). `tareas.py` no tiene subcomandos, así que le basta con las opciones habituales: `python3 tareas.py --help` explica las teclas principales y sale sin abrir la interfaz (con `argparse` lo tienes gratis).

## Checklist final

- [ ] Hay barra de atajos y cambia según el modo.
- [ ] `?` muestra la ayuda, generada desde la tabla de atajos.
- [ ] `q` sale y Esc cancela, en todas las pantallas.
- [ ] Ctrl+C sale sin traceback y la terminal queda bien.
- [ ] Cada acción tiene feedback; lo lento muestra un spinner y no congela.
- [ ] Los errores explican qué hacer.
- [ ] Hay estado vacío con primeros pasos.
- [ ] Lo destructivo se confirma o se puede deshacer.
- [ ] Máximo 3-4 colores con significado; fondo por defecto.
- [ ] Funciona con `NO_COLOR=1` y se entiende por los símbolos.
- [ ] Funciona en 80 × 24, se adapta al redimensionar y avisa si es muy pequeña.
- [ ] Los textos largos y con acentos o emojis se recortan sin romper bordes.
- [ ] Esc responde rápido (`set_escdelay`).
- [ ] La CLI sigue funcionando sin la TUI.
- [ ] Hay un README con una demo grabada (ver [[README de un proyecto]] y [[Grabar una demo de terminal]]).

## Errores típicos

| Error | Por qué molesta |
|---|---|
| Atajos secretos | Nadie los encuentra: si no está en la barra o en `?`, no existe |
| Arcoíris de colores | Nada destaca y cansa la vista |
| Borrar sin confirmar ni deshacer | Una tecla mal pulsada = datos perdidos |
| Esc sale del programa | El usuario quería cancelar una cosa, no cerrar todo |
| Diseñar solo para tu terminal de 200 columnas | En el portátil o por SSH se rompe |

## Relacionadas

[[CLI, TUI y GUI]] · [[De script a TUI - arquitectura]] · [[curses - Teclado y atajos]] · [[curses - Colores y estilos]] · [[curses - Layout y paneles]] · [[Spinners y barras de progreso]] · [[Elegir un framework de TUI]] · [[Qué hace un buen instalador]] · [[curses - Primeros pasos]]

## Referencias

- [Command Line Interface Guidelines](https://clig.dev/) — feedback, errores útiles, confirmar acciones peligrosas, `NO_COLOR`, interactividad.
- [NO_COLOR](https://no-color.org/) — convención para desactivar colores.
- [Nielsen Norman Group: 10 Usability Heuristics](https://www.nngroup.com/articles/ten-usability-heuristics/) — visibilidad del estado, control y libertad (deshacer), consistencia, prevención de errores.
- [Response Times: The 3 Important Limits](https://www.nngroup.com/articles/response-times-3-important-limits/) — 0,1 s, 1 s y 10 s.
- [lazygit](https://github.com/jesseduffield/lazygit) y [htop](https://htop.dev/) — ejemplos de barra de atajos contextual y ayuda con `?`.
