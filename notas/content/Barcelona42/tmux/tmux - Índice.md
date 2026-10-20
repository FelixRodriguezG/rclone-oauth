---
tags: [tmux, terminal, indice, herramientas]
aliases: ["tmux", "Índice de tmux"]
---
# tmux - Índice

[[Inicio|← Inicio]] · tmux

> **tmux** convierte una sola terminal en muchas: varias ventanas y paneles a la vez, y sesiones que siguen vivas aunque cierres la terminal o se corte el SSH. Esta carpeta va desde «qué es» hasta montar tu entorno de trabajo con un script.

## Por qué le sirve a un estudiante de 42

En un proyecto de 42 casi siempre tienes tres cosas abiertas a la vez:

```
┌──────────────────────────┬─────────────────────┐
│                          │ $ make && ./a.out   │
│   vim ft_split.c         │ (compilar/ejecutar) │
│                          ├─────────────────────┤
│                          │ $ ./tester.sh       │
│                          │ (tests, norminette) │
└──────────────────────────┴─────────────────────┘
```

Con tmux todo eso vive en **una sola ventana de terminal**, te mueves entre los trozos con el teclado y:

- Si cierras la terminal sin querer, **la sesión sigue ahí**: vuelves a conectarte y está todo igual (el Vim abierto, el test a medias).
- Si trabajas por SSH (por ejemplo contra una máquina virtual de [[Born2beroot - Índice|Born2beroot]]) y se corta la conexión, no pierdes nada.
- Puedes dejar algo largo corriendo (un test de 10 minutos), desconectarte y mirarlo luego.
- Puedes **automatizarlo**: un script que abre tu entorno con un solo comando.

> [!note] ¿Está instalado?
> Escribe `tmux -V`. Si responde algo como `tmux 3.2a`, ya lo tienes. Si no, en tu ordenador: `sudo apt install tmux` (Linux) o `brew install tmux` (macOS).

## Orden de lectura

| # | Nota | Qué aprendes |
|---|---|---|
| 1 | [[Qué es tmux]] | Servidor, sesiones, ventanas y paneles; por qué la sesión sobrevive |
| 2 | [[tmux - Uso básico]] | El prefijo `Ctrl-b` y los atajos del día a día |
| 3 | [[Configurar tmux]] | Tu `~/.tmux.conf`: ratón, colores, atajos propios |
| 4 | [[Automatizar tmux desde scripts]] | Montar el entorno con un script de Bash; leer la pantalla de un panel |

> [!tip] Cómo practicar
> Lee la nota 1, y abre ya una sesión (`tmux new -s prueba`) para hacer la 2 con los dedos, no solo con los ojos. La configuración (3) tiene más sentido cuando ya te has peleado un poco con los atajos por defecto.

## Lo mínimo para sobrevivir

Si solo vas a recordar unas pocas cosas, que sean estas (el detalle está en [[tmux - Uso básico]]):

| Qué quieres | Cómo |
|---|---|
| Empezar a trabajar en un proyecto | `tmux new -s libft` |
| Salir dejándolo todo vivo | `Ctrl-b d` |
| Volver a entrar | `tmux attach -t libft` (o `tmux ls` si no recuerdas el nombre) |
| Dividir la pantalla | `Ctrl-b %` (lado a lado) y `Ctrl-b "` (arriba/abajo) |
| Moverse entre paneles | `Ctrl-b` + flecha |
| Ver la salida que ya se fue hacia arriba | `Ctrl-b [` y flechas; `q` para salir |

> [!warning] Antes de cerrar el día
> Las sesiones sobreviven a cerrar la terminal, pero **no** a apagar el ordenador ni a cerrar tu sesión en un Mac de 42. Guarda (`:w`) y haz commit como siempre ([[Git para 42]]).

## Dónde encaja con lo demás

- Para escribir los scripts de la nota 4 necesitas lo básico de Bash: [[Bash - Índice]] (sobre todo [[Primer script en Bash]], [[Variables y comillas en Bash]] y [[Condiciones en Bash]]).
- tmux es también la herramienta para **probar TUIs** sin pelearte con tu propia terminal y para **grabar demos reproducibles**: ver [[TUI - Índice]] y [[Grabar una demo de terminal]].
- Dentro del panel del editor seguirás usando [[Vim]]; sus divisiones (`:vsp`) y las de tmux conviven sin problema.

## Relacionadas

[[Qué es tmux]] · [[tmux - Uso básico]] · [[Configurar tmux]] · [[Automatizar tmux desde scripts]] · [[Bash - Índice]] · [[TUI - Índice]] · [[Vim]]

## Referencias

- [man tmux (OpenBSD)](https://man.openbsd.org/tmux) — manual de referencia completo.
- [tmux wiki: Getting Started](https://github.com/tmux/tmux/wiki/Getting-Started) — guía oficial de introducción.
