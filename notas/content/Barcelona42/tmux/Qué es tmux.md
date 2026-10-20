---
tags: [tmux, terminal, multiplexor, conceptos]
aliases: ["Multiplexor de terminal", "Sesiones de tmux"]
---
# Qué es tmux

[[Inicio|← Inicio]] · [[tmux - Índice|tmux]]

> **tmux** (*terminal multiplexer*) es un **multiplexor de terminal**: un programa que reparte una sola terminal en varias pantallas independientes y las mantiene vivas en segundo plano. Puedes cerrar la ventana, volver horas después y encontrar todo como lo dejaste.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Multiplexor** | Algo que hace pasar varias cosas por un único canal. Aquí: varias terminales por una sola ventana |
| **Servidor** | Proceso de tmux que corre en segundo plano y es el **dueño** de todas las sesiones |
| **Cliente** | Lo que ves en tu terminal cuando escribes `tmux`: solo muestra una sesión y le pasa tus teclas al servidor |
| **Sesión** | Un conjunto de ventanas con nombre (p. ej. `libft`). Es lo que sobrevive |
| **Ventana** (*window*) | Una «pestaña» que ocupa toda la pantalla |
| **Panel** (*pane*) | Un trozo de una ventana con su propia shell. Una ventana se divide en paneles |
| **Detach** (desacoplar) | Desconectar el cliente; la sesión sigue funcionando |
| **Attach** (acoplar) | Volver a conectar un cliente a una sesión existente |

## Servidor y cliente

Cuando ejecutas `tmux` por primera vez pasan **dos** cosas:

1. Arranca un **servidor** tmux en segundo plano (un proceso sin ventana).
2. Tu terminal se convierte en un **cliente** que se conecta a él a través de un *socket* (un fichero especial para comunicar procesos, normalmente `/tmp/tmux-<tu UID>/default`).

```
  Tu terminal (cliente)            Servidor tmux (en segundo plano)
 ┌──────────────────────┐   socket ┌──────────────────────────────┐
 │ muestra la pantalla  │◄────────►│ sesión "libft"               │
 │ envía tus teclas     │          │   ├─ ventana 1: vim + make   │
 └──────────────────────┘          │   └─ ventana 2: tests        │
                                   │ sesión "notas"               │
  Otra terminal (cliente) ◄───────►│   └─ ventana 1: shell        │
                                   └──────────────────────────────┘
```

Los programas (Vim, `make`, tu shell) son **hijos del servidor**, no de la terminal. Por eso, si cierras la terminal, solo muere el cliente: el servidor y todo lo que cuelga de él siguen vivos.

## Sesiones, ventanas y paneles

Es una jerarquía: el servidor tiene sesiones, cada sesión tiene ventanas y cada ventana tiene paneles.

```
servidor
└── sesión "libft"
    ├── ventana 1 "codigo"
    │   ┌───────────────┬──────────────┐
    │   │ panel 1: vim  │ panel 2:     │
    │   │               │ make && test │
    │   │               ├──────────────┤
    │   │               │ panel 3: git │
    │   └───────────────┴──────────────┘
    └── ventana 2 "man"
        ┌──────────────────────────────┐
        │ panel 1: man 3 malloc        │
        └──────────────────────────────┘
```

- **Paneles**: para ver cosas **a la vez** (editor y salida del compilador).
- **Ventanas**: para cosas que no necesitas ver juntas (el manual, un `git log`).
- **Sesiones**: una por proyecto. `libft`, `ft_printf`… y cambias entre ellas sin cerrar nada.

## Detach y attach: por qué la sesión sigue viva

```bash
tmux new -s libft      # crea la sesión "libft" y te conecta
# ... trabajas, abres vim ...
# pulsas Ctrl-b y luego d  → detach: vuelves a tu shell normal
tmux ls                # lista las sesiones que siguen vivas
# libft: 1 windows (created Tue Oct 20 10:02:11 2026)
tmux attach -t libft   # vuelves a entrar: Vim sigue abierto donde estaba
```

Hacer *detach* no para nada: el `make` que estaba compilando sigue compilando. Lo mismo ocurre si se cierra la terminal o se corta el SSH: para el servidor es como un *detach* involuntario.

> [!warning] Lo que NO sobrevive
> Las sesiones viven en la memoria del servidor. Si **apagas o reinicias el ordenador**, o cierras tu sesión de usuario en los Mac de 42, el servidor muere y con él todas las sesiones. Guarda tus ficheros igual que siempre.

## ¿Y no basta con abrir varias terminales?

| | Varias ventanas de terminal | tmux |
|---|---|---|
| Ver editor y compilación a la vez | Sí, colocando ventanas con el ratón | Sí, con paneles y el teclado |
| Cerrar la ventana por error | Pierdes lo que corría | La sesión sigue; `tmux attach` |
| Corte de SSH | Se mata el proceso remoto | Sigue corriendo en el servidor remoto |
| Recuperar el mismo entorno mañana | Abrir todo a mano otra vez | Un script lo monta ([[Automatizar tmux desde scripts]]) |
| Funciona sin interfaz gráfica | No | Sí (por SSH, en una VM) |
| Curva de aprendizaje | Ninguna | Hay que aprender atajos |

> [!tip] Una analogía
> Tu terminal es una tele; tmux es el decodificador. Puedes apagar la tele y la grabación sigue; la enciendes otra vez (attach) y continúas viendo lo mismo.

## Errores típicos

- **Meter tmux dentro de tmux** sin querer (ejecutar `tmux` en un panel). Verás dos barras de estado y los atajos irán al tmux de fuera. Sal con `exit` y usa `tmux attach` o cambia de sesión con `Ctrl-b s`. El aviso `sessions should be nested with care, unset $TMUX to force` te lo indica.
- **Acumular sesiones olvidadas**: revisa de vez en cuando `tmux ls` y cierra las que sobren (`tmux kill-session -t nombre`).
- **Creer que tmux guarda tus ficheros**: guarda el *estado de los programas*, no sustituye a `:w` ni a [[Git para 42]].

## Relacionadas

[[tmux - Índice]] · [[tmux - Uso básico]] · [[Configurar tmux]] · [[Automatizar tmux desde scripts]] · [[Subshells y procesos en Bash]] · [[Cómo funciona la terminal]]

## Referencias

- [man tmux (OpenBSD)](https://man.openbsd.org/tmux) — secciones DESCRIPTION y CLIENTS AND SESSIONS: servidor, cliente, socket, sesiones, ventanas y paneles.
- [tmux wiki: Getting Started](https://github.com/tmux/tmux/wiki/Getting-Started) — jerarquía sesión/ventana/panel, attach y detach.
