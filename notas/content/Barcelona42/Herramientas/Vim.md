---
tags: [42, vim, herramientas, cheatsheet]
---
# Vim

[[Inicio|← Inicio]] · Herramientas

> Cheatsheet de Vim para el cursus de 42. Copia el bloque de configuración en tu `~/.vimrc` del Mac del cluster.

## ⚙️ 1. Configuración del `.vimrc` (Para 42)
Copia este contenido en tu archivo `~/.vimrc`:

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

> [!tip] Cabecera de 42
> Con `$USER` y `$MAIL` configurados, `:Stdheader` (o `F1`) inserta la cabecera obligatoria de la [[La Norma|Norma]] y la actualiza al guardar.

## ⌨️ 2. Modos de Vim
* `Esc` -> **Modo Normal**: Moverse, borrar, ejecutar comandos.
* `i` -> **Modo Inserción**: Escribir código.
* `v` / `V` -> **Modo Visual**: Seleccionar texto o líneas.
* `:` -> **Modo Comando**: Guardar, salir, ejecutar terminal.

### Guardar y salir
* `:w` -> Guardar.
* `:q` -> Salir (falla si hay cambios sin guardar).
* `:wq` o `:x` -> Guardar y salir.
* `:q!` -> Salir **descartando** los cambios.

## 🚀 3. Movimientos Rápidos
* `h`, `j`, `k`, `l` -> Izquierda, abajo, arriba, derecha.
* `w` / `b` -> Siguiente palabra / Palabra anterior.
* `0` / `$` -> Principio / Final de línea.
* `gg` / `G` -> Primera / Última línea del archivo.
* `[número] + j/k` -> Bajar o subir N líneas (ej. `12j`).

## ✂️ 4. Edición y Portapapeles
* `i` / `a` -> Insertar antes / después del cursor.
* `o` / `O` -> Nueva línea abajo / arriba.
* `x` -> Borrar letra.
* `dd` -> Cortar línea entera.
* `yy` -> Copiar línea entera.
* `p` / `P` -> Pegar después / antes.
* `u` / `Ctrl + r` -> Deshacer / Rehacer.

## 📐 5. Pantallas Divididas y Múltiples Archivos
Para trabajar con tu `.c` y tu `.h` al mismo tiempo:

* `:sp [archivo]` -> Divide la pantalla horizontalmente.
* `:vsp [archivo]` -> Divide la pantalla verticalmente (¡perfecto para 42!).
* `Ctrl + w` + flechas (o `h`/`j`/`k`/`l`) -> Moverse entre las divisiones.
* `:q` -> Cerrar división actual.
* `:tabnew [archivo]` -> Abrir en una pestaña nueva.
* `gt` / `gT` -> Cambiar de pestaña (siguiente / anterior).
* `:e [archivo]` -> Abrir archivo en segundo plano (Buffers).
* `:ls` -> Ver lista de archivos abiertos en buffers.

## 🔍 6. Búsqueda y Compilación
* `/palabra` -> Buscar texto (pulsa `n` para la siguiente coincidencia).
* `:%s/antiguo/nuevo/g` -> Reemplazar texto globalmente.
* `:!make` -> Ejecutar un comando de la shell sin salir de Vim (aquí, compilar).
* `:!norminette %` -> Pasar la norminette al archivo actual (`%` = archivo abierto).

## Relacionadas

[[La Norma]] · [[Makefile y librerías estáticas]] · [[Depurar con gdb]]
