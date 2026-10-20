---
tags: [tui, animacion, ansi, terminal, bash, python]
aliases: ["Animar en la terminal", "Fotogramas en la terminal", "FPS en la terminal"]
---
# Animación en la terminal - fundamentos

[[Inicio|← Inicio]] · [[TUI - Índice|TUI]]

> Animar en la terminal es lo mismo que en el cine: mostrar **fotogramas** uno detrás de otro, lo bastante rápido para que parezca movimiento. Sin curses se hace con `\r` y **secuencias ANSI** (mover el cursor, borrar). Esta nota explica las piezas comunes a [[Spinners y barras de progreso]] y [[curses - Animaciones]].

## Vocabulario

| Término | Qué significa |
|---|---|
| **Fotograma** (*frame*) | Una «foto» de lo que se ve en un instante |
| **FPS** | Fotogramas por segundo. 10 FPS basta para un spinner; 30 se ve fluido |
| **Secuencia ANSI** | Bytes que empiezan por `ESC` (`\033`, `\e`) y que la terminal interpreta como orden, no como texto (ver [[Cómo funciona la terminal]]) |
| **`\r`** (retorno de carro) | Lleva el cursor al **principio de la línea** sin bajar. Lo siguiente sobrescribe |
| **Parpadeo** (*flicker*) | Ver la pantalla vacía un instante entre fotogramas |
| **Delta time** (`dt`) | Tiempo real que ha pasado desde el fotograma anterior |
| **Reloj monótono** | Reloj que solo avanza (no salta si cambias la hora del sistema): `time.monotonic()` |

## Secuencias ANSI para animar

La tabla general (colores, mover el cursor a una posición, pantalla alternativa) está en [[Cómo funciona la terminal#Secuencias de escape|Cómo funciona la terminal]]. Para animar sin curses, estas son las que más vas a usar:

| Secuencia | Efecto |
|---|---|
| `\r` | Ir al principio de la línea |
| `\033[K` | Borrar desde el cursor hasta el final de la línea |
| `\033[2K` | Borrar la línea entera |
| `\033[3A` / `\033[3B` | Subir / bajar 3 líneas (para redibujar un bloque de varias líneas) |
| `\033[?25l` / `\033[?25h` | **Ocultar** / mostrar el cursor |
| `\033[2J` | Borrar toda la pantalla (evítalo en cada fotograma: parpadeo) |

## Redibujar una línea

```bash
for i in 3 2 1; do
    printf '\rEmpieza en %d…' "$i"    # \r: vuelve al inicio y sobrescribe
    sleep 1
done
printf '\r\033[K¡Ya!\n'              # \033[K: borra restos del texto anterior
```

> [!warning] Restos del texto anterior
> Si escribes `"Cargando…"` y luego `"Listo"` con solo `\r`, verás `Listondo…`. Borra con `\033[K` o rellena con espacios hasta el mismo ancho.

## Ocultar el cursor… y restaurarlo **siempre**

El cursor parpadeando sobre la animación queda feo, así que se oculta. Pero si el script muere (error, `Ctrl+C`) sin volver a mostrarlo, la terminal del usuario **se queda sin cursor**. Solución: un `trap` (ver [[Scripts robustos con set y trap]]):

```bash
printf '\033[?25l'                       # ocultar
trap 'printf "\033[?25h"' EXIT           # se ejecuta al salir, sea como sea
trap 'exit 130' INT                      # Ctrl+C → salir (y el trap EXIT se ejecuta)
```

En Python, el equivalente es `try/finally`. Con curses, [[curses - Primeros pasos|`curses.wrapper`]] restaura la terminal por ti.

## Evitar el parpadeo

1. **No borres la pantalla entera** (`\033[2J`, `clear`) en cada fotograma: el usuario ve el hueco.
2. **Prepara el fotograma completo** en una variable y escríbelo de una vez.
3. **Sobrescribe** encima de lo anterior y borra solo los restos (`\033[K`).
4. En Python, haz `flush()` **una vez** por fotograma, no por cada trozo.

curses hace todo esto por ti: dibujas en un búfer y `refresh()` solo envía lo que ha cambiado.

## Tiempo real frente a fotogramas

Si mueves la bola «1 casilla por fotograma», en un ordenador lento irá más despacio. Lo correcto es **velocidad × tiempo transcurrido**:

```python
#!/usr/bin/env python3
"""Una bola que va y viene por una línea, a 30 FPS, sin curses."""
import sys
import time

ANCHO, FPS, VELOCIDAD, DURACION = 30, 30, 15.0, 3.0   # VELOCIDAD en casillas/s


def dibujar(x: int) -> None:
    linea = " " * x + "●" + " " * (ANCHO - x - 1)
    sys.stdout.write(f"\r[{linea}]")
    sys.stdout.flush()                       # mostrar ya, sin esperar al \n


def main() -> None:
    sys.stdout.write("\033[?25l")            # ocultar cursor
    try:
        inicio = anterior = time.monotonic()
        pos, direccion = 0.0, 1
        while time.monotonic() - inicio < DURACION:
            ahora = time.monotonic()
            dt, anterior = ahora - anterior, ahora     # tiempo real del fotograma
            pos += direccion * VELOCIDAD * dt
            if pos <= 0 or pos >= ANCHO - 1:           # rebote en los bordes
                direccion = -direccion
                pos = max(0.0, min(pos, ANCHO - 1))
            dibujar(int(pos))
            espera = 1 / FPS - (time.monotonic() - ahora)
            if espera > 0:
                time.sleep(espera)           # dormir el resto del fotograma
    finally:
        sys.stdout.write("\r\033[K\033[?25h")   # borrar línea y mostrar cursor
        sys.stdout.flush()


if __name__ == "__main__":
    main()
```

## No consumir CPU

Un `while True:` que redibuja sin pausa pone un núcleo al 100 % y no se ve mejor (la terminal no pinta más rápido). **Duerme** entre fotogramas: `sleep 0.1` en Bash, `time.sleep()` en Python, `timeout()` en curses. Compruébalo con `top` mientras corre: debería estar cerca del 0-2 %.

> [!tip] ¿Cuántos FPS?
> Spinner: 8-12. Barra de progreso: lo que tarde cada paso (no hace falta un bucle aparte). Juegos o rebotes: 20-30. Más de 60 no aporta nada en una terminal.

## Errores típicos

| Error | Consecuencia |
|---|---|
| Sin `trap`/`finally` tras ocultar el cursor | La terminal queda sin cursor; arréglalo con `tput cnorm` o `reset` |
| `clear` en cada fotograma | Parpadeo |
| Olvidar `flush()` en Python | No se ve nada hasta el final (la salida va en búfer) |
| Mover por fotograma en vez de por tiempo | Velocidad distinta en cada ordenador |
| Animar cuando la salida no es una terminal | El log se llena de `\r` y códigos raros: comprueba `[[ -t 1 ]]` / `sys.stdout.isatty()` |

## Relacionadas

[[Cómo funciona la terminal]] · [[Colores en la terminal]] · [[Spinners y barras de progreso]] · [[curses - Animaciones]] · [[curses - El bucle principal]] · [[Scripts robustos con set y trap]] · [[Unicode y emojis en la terminal]]

## Referencias

- [ANSI escape code — Wikipedia](https://en.wikipedia.org/wiki/ANSI_escape_code) — CSI, `K`, `A`/`B`, `H`, `?25l/h`, `?1049h/l`.
- [XTerm Control Sequences](https://invisible-island.net/xterm/ctlseqs/ctlseqs.html) — referencia completa de secuencias.
- [time.monotonic](https://docs.python.org/3/library/time.html#time.monotonic) — reloj que no retrocede.
- [Bash manual: trap](https://www.gnu.org/software/bash/manual/bash.html#index-trap) — `EXIT` e `INT`.
- Ejemplo de Python ejecutado con Python 3.13 (3 s de rebote, sale con cursor restaurado).
