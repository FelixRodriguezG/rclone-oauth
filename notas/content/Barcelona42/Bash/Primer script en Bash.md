---
tags: [bash, shell, scripting, shebang, permisos]
aliases: ["Hola mundo en Bash", "Shebang", "Ejecutar un script"]
---
# Primer script en Bash

[[Inicio|← Inicio]] · [[Bash - Índice|Bash]]

> **Bash** es la *shell* (el programa que interpreta lo que escribes en la terminal) más común en Linux. Un **script** es un archivo de texto con comandos de Bash que se ejecutan uno detrás de otro, como si los escribieras tú. Sirve para no repetir a mano tareas que haces a menudo.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Shell** | Programa que lee comandos y los ejecuta: `bash`, `zsh`, `sh`… |
| **Script** | Archivo de texto con comandos para una shell |
| **Shebang** | Primera línea `#!...`: dice qué programa debe ejecutar el archivo |
| **Permiso de ejecución** | Bit `x` del archivo; sin él no se puede lanzar con `./` |
| **`PATH`** | Variable con la lista de carpetas donde la shell busca los comandos |

## El primer script

Crea un archivo `hola.sh`:

```bash
#!/usr/bin/env bash
# Mi primer script: todo lo que va detrás de # es un comentario.

echo "Hola, $USER"          # $USER = tu nombre de usuario
echo "Hoy es $(date +%A)"   # $( ) ejecuta un comando y pone su salida aquí
```

Dale permiso de ejecución y lánzalo:

```console
$ chmod +x hola.sh
$ ./hola.sh
Hola, tu_login
Hoy es lunes
```

> [!note] La extensión `.sh` es opcional
> Linux no mira la extensión para saber qué es un archivo: mira el *shebang*. `.sh` solo te ayuda a ti (y a [[Vim]], que colorea la sintaxis).

## El shebang

`#!/usr/bin/env bash` significa: «ejecuta este archivo con el primer `bash` que encuentres en el `PATH`». Es más portable que `#!/bin/bash` (en macOS, por ejemplo, el `bash` moderno suele estar en otra ruta).

| Shebang | Cuándo |
|---|---|
| `#!/usr/bin/env bash` | Lo normal: usas cosas de Bash (`[[ ]]`, arrays…) |
| `#!/bin/sh` | Solo si escribes POSIX puro (sin nada propio de Bash) |

> [!warning] `sh` no es `bash`
> En Ubuntu, `/bin/sh` es `dash`, una shell más simple. Un script con `[[ ]]` o arrays lanzado con `sh script.sh` falla con errores raros como `[[: not found`.

## `./script.sh` frente a `bash script.sh`

| Forma | Qué pasa |
|---|---|
| `./hola.sh` | El sistema lee el *shebang* y lanza ese intérprete. Necesita `chmod +x` |
| `bash hola.sh` | Lanzas `bash` tú y le pasas el archivo. No necesita `x` y **ignora** el shebang |
| `source hola.sh` (o `. hola.sh`) | Ejecuta los comandos **en tu shell actual**: las variables que cree se quedan |

¿Por qué hace falta el `./`? Porque al escribir `hola.sh` sin ruta, Bash lo busca solo en las carpetas del `PATH`, y la carpeta actual no está ahí (por seguridad). `./` significa «el archivo `hola.sh` de esta carpeta».

```console
$ echo "$PATH"
/home/tu_login/.local/bin:/usr/local/bin:/usr/bin:/bin
$ hola.sh
bash: hola.sh: command not found
```

> [!tip] Tus scripts como comandos
> Si copias el script a `~/.local/bin/` (y esa carpeta está en tu `PATH`), podrás llamarlo desde cualquier sitio sin `./`.

## Comentarios y estructura

- `#` hasta el final de línea es un comentario (salvo el shebang de la línea 1).
- Un comando por línea. Puedes poner varios en una con `;`.
- Una línea larga se puede partir con `\` al final.

```bash
#!/usr/bin/env bash
echo "uno"; echo "dos"          # dos comandos en una línea
ls -l \
    /tmp                         # el mismo comando partido en dos líneas
```

## `tareas.sh` versión 0

El mini-proyecto de esta ruta (ver [[Bash - Índice]]) empieza aquí: un script que solo saluda.

```bash
#!/usr/bin/env bash
# tareas.sh — lista de tareas en la terminal (versión 0)

echo "=== Mis tareas ==="
echo "Todavía no hay tareas."
```

```console
$ chmod +x tareas.sh && ./tareas.sh
=== Mis tareas ===
Todavía no hay tareas.
```

En la siguiente nota guardará en una variable dónde están las tareas: [[Variables y comillas en Bash]].

## Errores típicos

| Error | Causa |
|---|---|
| `Permission denied` | Falta `chmod +x` |
| `command not found` al escribir `hola.sh` | Falta el `./` delante |
| `bad interpreter: No such file or directory` | Shebang mal escrito, o el archivo tiene finales de línea de Windows (`\r`) |
| `[[: not found` | Lo ejecutaste con `sh` en vez de `bash` |
| Nada se ejecuta como esperas | Espacios raros: en Bash los espacios **importan** (ver [[Variables y comillas en Bash]]) |

## Relacionadas

[[Bash - Índice]] · [[Variables y comillas en Bash]] · [[Códigos de salida y errores en Bash]] · [[Depurar scripts de Bash]] · [[Vim]] · [[Git para 42]]

## Referencias

- [GNU Bash Manual: Shell Scripts](https://www.gnu.org/software/bash/manual/bash.html#Shell-Scripts) — shebang, `bash archivo` frente a ejecutarlo.
- [GNU Bash Manual: Bourne Shell Builtins](https://www.gnu.org/software/bash/manual/bash.html#Bourne-Shell-Builtins) — `source` / `.`.
- `man chmod`, `man env` — permiso de ejecución y búsqueda en el `PATH`.
- [Ubuntu Wiki: DashAsBinSh](https://wiki.ubuntu.com/DashAsBinSh) — por qué `/bin/sh` es `dash` en Ubuntu.
