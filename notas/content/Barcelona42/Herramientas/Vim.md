---
tags: [herramientas, vim, editor, cheatsheet]
---
# Vim

[[Inicio|← Inicio]] · Herramientas

> **Vim** es un editor de texto que funciona dentro de la terminal y se maneja solo con el teclado. Está instalado en los ordenadores de 42 y trae la cabecera oficial de 42. Esta nota es una chuleta: una configuración (`~/.vimrc`) que ayuda a cumplir la [[La Norma|Norma]] y los atajos que más se usan.

> [!tip] Para empezar desde cero
> Escribe `vimtutor` en la terminal: es un tutorial interactivo de unos 30 minutos que viene con Vim.

## Configuración del `.vimrc` para 42

El fichero `~/.vimrc` (en tu carpeta personal; el `.` del principio lo hace oculto) se lee cada vez que abres Vim. Copia este contenido en él:

```vim
syntax on
filetype plugin indent on

" --- CONFIGURACIÓN DE 42 (datos para la cabecera :Stdheader / F1) ---
let $USER = 'felixrod'
let $MAIL = 'tu_login@student.42barcelona.com'

" --- REGLAS DE LA NORMINETTE ---
set tabstop=4         " El tabulador mide 4 espacios
set shiftwidth=4      " Indentación automática en 4 espacios
set noexpandtab       " ¡Tabuladores reales obligatorios!
set softtabstop=4     " Borra un tab completo con Backspace
set autoindent        " Mantiene la indentación anterior
set smartindent       " Indentación inteligente

" --- LÍMITE DE 80 COLUMNAS ---
set colorcolumn=81    " Línea roja vertical de aviso
set nowrap            " Evita que el texto baje solo

" --- VISUALIZACIÓN ---
set number            " Números absolutos
set relativenumber    " Números relativos (ideal para saltar rápido)
set cursorline        " Resalta la línea actual
set showmatch         " Resalta llaves y paréntesis
set encoding=utf-8    " Evita caracteres raros

" --- AUTOMATIZACIONES ---
autocmd BufWritePre *.c,*.h %s/\s\+$//e  " Borra espacios al final al guardar
autocmd BufWritePost ~/.vimrc source %   " Recarga el vimrc al guardarlo
```

Qué aporta cada bloque respecto a la Norma:

| Bloque | Regla de la Norma que te ayuda a cumplir |
|---|---|
| `noexpandtab`, `tabstop=4` | Indentar con **tabulaciones reales** de 4 de ancho (y el `Makefile` también necesita tabs, ver [[Makefile y librerías estáticas]]) |
| `colorcolumn=81` | Una línea de color en la columna 81 marca el límite de **80 columnas** |
| `autocmd BufWritePre … %s/\s\+$//e` | Al guardar un `.c` o `.h`, borra los **espacios y tabs al final de línea** (prohibidos) |
| `$USER` / `$MAIL` | Los datos que pone la **cabecera de 42** |

> [!note] Vocabulario
> - **`autocmd`**: un comando que Vim ejecuta solo cuando ocurre un evento. `BufWritePre` = «justo antes de guardar»; `BufWritePost` = «justo después».
> - **`%s/\s\+$//e`**: sustitución en todo el fichero (`%`) de «uno o más espacios en blanco al final de línea» (`\s\+$`) por nada. La `e` evita un error si no encuentra ninguno.

> [!tip] Cabecera de 42
> Con `$USER` y `$MAIL` configurados, `:Stdheader` (o `F1`) inserta la cabecera obligatoria de la [[La Norma|Norma]] y la actualiza al guardar. En los ordenadores de 42 ya viene instalada; en tu ordenador personal tienes que instalar el plugin [42header](https://github.com/42Paris/42header). Ese plugin también acepta `let g:user42 = '...'` y `let g:mail42 = '...'`, que no tocan las variables de entorno `$USER` y `$MAIL`.

## Modos de Vim

Vim tiene **modos**: la misma tecla hace cosas distintas según el modo en que estés. Es lo que más confunde al principio. Si no sabes en qué modo estás, pulsa `Esc` para volver al Normal.

* `Esc` → **modo Normal**: moverse, borrar, ejecutar comandos.
* `i` → **modo Inserción**: escribir código.
* `v` / `V` → **modo Visual**: seleccionar texto (por caracteres / por líneas).
* `:` → **modo Comando**: guardar, salir, ejecutar comandos de la terminal.

### Guardar y salir

* `:w` → guardar.
* `:q` → salir (falla si hay cambios sin guardar).
* `:wq` o `:x` → guardar y salir.
* `:q!` → salir **descartando** los cambios.

## Movimientos rápidos

* `h`, `j`, `k`, `l` → izquierda, abajo, arriba, derecha.
* `w` / `b` → siguiente palabra / palabra anterior.
* `0` / `$` → principio / final de línea.
* `gg` / `G` → primera / última línea del archivo.
* `[número]` + `j`/`k` → bajar o subir N líneas (ej. `12j`). Con `relativenumber` ves directamente ese número al lado de cada línea.
* `:42` → ir a la línea 42 (útil cuando el compilador o la norminette te dan un número de línea).

## Edición y portapapeles

* `i` / `a` → insertar antes / después del cursor.
* `o` / `O` → nueva línea abajo / arriba.
* `x` → borrar un carácter.
* `dd` → cortar la línea entera.
* `yy` → copiar la línea entera.
* `p` / `P` → pegar después / antes.
* `u` / `Ctrl + r` → deshacer / rehacer.
* `>>` / `<<` → añadir / quitar un nivel de indentación a la línea.

## Pantallas divididas y varios archivos

Para trabajar con tu `.c` y tu `.h` al mismo tiempo:

* `:sp [archivo]` → divide la pantalla horizontalmente.
* `:vsp [archivo]` → divide la pantalla verticalmente (cómodo para ver el `.c` y el `.h` lado a lado).
* `Ctrl + w` + flechas (o `h`/`j`/`k`/`l`) → moverse entre las divisiones.
* `:q` → cerrar la división actual.
* `:tabnew [archivo]` → abrir en una pestaña nueva.
* `gt` / `gT` → cambiar de pestaña (siguiente / anterior).
* `:e [archivo]` → abrir otro archivo en la misma ventana. El anterior sigue abierto en segundo plano (un **buffer** es un archivo cargado en memoria).
* `:ls` → ver la lista de buffers abiertos.

## Búsqueda y compilación

* `/palabra` → buscar texto (pulsa `n` para la siguiente coincidencia y `N` para la anterior).
* `:%s/antiguo/nuevo/g` → reemplazar texto en todo el archivo.
* `:!make` → ejecutar un comando de la terminal sin salir de Vim (aquí, compilar).
* `:!norminette %` → pasar la norminette al archivo actual (`%` = archivo abierto).
* `:set list` → mostrar tabs y finales de línea, para ver si hay espacios donde debería haber tabs. `:set nolist` lo quita.

## Relacionadas

[[La Norma]] · [[Makefile y librerías estáticas]] · [[Depurar con gdb]] · [[Git para 42]]

## Referencias

- [Vim help: options](https://vimhelp.org/options.txt.html) — `tabstop`, `shiftwidth`, `expandtab`, `softtabstop`, `colorcolumn`, `relativenumber`, `list`.
- [Vim help: autocmd](https://vimhelp.org/autocmd.txt.html) — eventos `BufWritePre` y `BufWritePost`.
- [Vim help: windows](https://vimhelp.org/windows.txt.html) — `:sp`, `:vsp`, `Ctrl + w`, buffers.
- [42Paris/42header](https://github.com/42Paris/42header) — plugin de la cabecera de 42 para Vim (`:Stdheader`, `F1`).
- La Norma v4, capítulos III.2 «Formato» y III.6 «La cabecera de 42» (`Normativa 42/es.norm-2.pdf`).
