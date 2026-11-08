---
tags: [bash, variables, comillas, entorno, export]
aliases: ["Variables en Bash", "Comillas en Bash", "export", "Sustitución de comandos"]
---
# Variables y comillas en Bash

[[Inicio|← Inicio]] · [[Bash - Índice|Bash]]

> Una **variable** guarda un texto con un nombre para usarlo después. En Bash todo es texto, y las **comillas** deciden si ese texto se parte en trozos o no. Entender las comillas evita la mayoría de los errores raros de un script.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Expandir** | Sustituir `$var` por su valor antes de ejecutar el comando |
| **Sustitución de comandos** | `$(comando)`: se reemplaza por lo que el comando imprime |
| **Separación de palabras** (*word splitting*) | Bash parte un valor sin comillas por los espacios |
| **Variable de entorno** | Variable que heredan los programas que lanzas (`PATH`, `HOME`…) |

## Asignar y leer

```bash
nombre="Ana"        # SIN espacios alrededor del =
edad=20
echo "$nombre tiene $edad años"     # Ana tiene 20 años
echo "${nombre}_backup"             # Ana_backup: las llaves marcan dónde acaba el nombre
```

> [!danger] Nada de espacios alrededor del `=`
> `nombre = "Ana"` no asigna: Bash ejecuta un **comando** llamado `nombre` con los argumentos `=` y `Ana` → `nombre: command not found`.

Convención: MAYÚSCULAS para variables de entorno y constantes (`PATH`, `ARCHIVO`); minúsculas para el resto, así no pisas una variable del sistema sin querer.

## `$var` frente a `"$var"`

Sin comillas, Bash **parte** el valor por los espacios y además expande `*`, `?`… como patrones de archivos. Con comillas dobles, el valor llega tal cual, como **un solo argumento**.

```bash
archivo="mis notas.txt"
touch "$archivo"     # crea UN archivo: «mis notas.txt»
rm $archivo          # intenta borrar «mis» y «notas.txt» → error
rm "$archivo"        # correcto
```

> [!tip] Regla de oro
> Pon **siempre** comillas dobles: `"$var"`, `"$(comando)"`. Solo se quitan cuando quieres que se parta a propósito, y eso casi nunca.

## Comillas simples y dobles

| Comillas | Qué hace | Ejemplo | Salida |
|---|---|---|---|
| `"..."` | Expande `$var`, `$( )` y `\` | `echo "Hola $USER"` | `Hola tu_login` |
| `'...'` | Texto literal, no expande nada | `echo 'Hola $USER'` | `Hola $USER` |
| `\` | Escapa un carácter | `echo "Cuesta \$5"` | `Cuesta $5` |
| `$'...'` | Literal, pero entiende escapes como `\n`, `\t`, `\033` (*ANSI-C quoting*) | `echo $'a\tb'` | `a` y `b` separadas por un tab |

Dentro de comillas simples no se puede poner otra comilla simple. Truco: cierra, añade la comilla escapada y vuelve a abrir: `echo 'it'\''s'` → `it's`. `$'...'` se usa sobre todo para guardar caracteres invisibles, como los colores de [[Colores en la terminal]].

## Sustitución de comandos: `$( )`

Guarda en una variable lo que imprime un comando:

```bash
hoy="$(date +%F)"            # 2026-10-20
n_archivos="$(ls | wc -l)"
echo "Hoy $hoy hay $n_archivos archivos"
```

Verás también la forma antigua con acentos graves `` `date` ``: funciona, pero `$( )` se lee mejor y se puede anidar. Usa `$( )`.

> [!note] Se quita el salto de línea final
> `$( )` elimina los `\n` del final de la salida. Por eso `x="$(echo hola)"` guarda `hola`, no `hola\n`.

## Variables de entorno y `export`

Una variable normal solo existe en **tu** script. Con `export` pasa a ser de **entorno**: la heredan los programas que lances desde ahí.

```bash
color="rojo"
bash -c 'echo "hijo ve: [$color]"'      # hijo ve: []
export color
bash -c 'echo "hijo ve: [$color]"'      # hijo ve: [rojo]
```

También puedes dar una variable a **un solo** comando: `LANG=C sort lista.txt`.

Variables que ya existen: `HOME` (tu carpeta), `USER`, `PATH`, `PWD` (carpeta actual), `EDITOR`. Las ves todas con `env`. La herencia es solo hacia abajo: un hijo nunca cambia las variables del padre (ver [[Subshells y procesos en Bash]]).

## Valor por defecto: `${var:-defecto}`

Si la variable está vacía o no existe, usa el valor de después del `:-`:

```bash
editor="${EDITOR:-vim}"     # usa $EDITOR si existe; si no, vim
echo "Abriré con $editor"
```

Hay más formas (`:=`, `:?`, recortar texto…): están en [[Expansiones en Bash]].

## Constantes: `readonly`

```bash
readonly VERSION="1.0"
VERSION="2.0"          # bash: VERSION: readonly variable
```

## `tareas.sh` versión 1

El script ya sabe dónde guarda las tareas. El usuario puede cambiar la ruta con una variable de entorno.

```bash
#!/usr/bin/env bash
# tareas.sh — versión 1
readonly ARCHIVO="${TAREAS_ARCHIVO:-$HOME/.tareas.txt}"

echo "=== Mis tareas ==="
echo "Guardo las tareas en: $ARCHIVO"
```

```console
$ ./tareas.sh
Guardo las tareas en: /home/tu_login/.tareas.txt
$ TAREAS_ARCHIVO=/tmp/prueba.txt ./tareas.sh
Guardo las tareas en: /tmp/prueba.txt
```

Siguiente: comprobar si ese archivo existe → [[Condiciones en Bash]].

## Errores típicos

| Error | Causa |
|---|---|
| `var: command not found` | Espacios alrededor del `=` |
| `rm` borra cosas de más o falla con nombres con espacios | `$var` sin comillas |
| Sale `$HOME` literal | Usaste comillas simples |
| La variable está vacía en otro programa | Falta `export` |
| `$nombre_backup` vacío | Bash busca la variable `nombre_backup`: usa `${nombre}_backup` |

## Relacionadas

[[Primer script en Bash]] · [[Expansiones en Bash]] · [[Condiciones en Bash]] · [[Subshells y procesos en Bash]] · [[Configuración y estado de un script]] · [[Bash - Chuleta de sintaxis]]

## Referencias

- [GNU Bash Manual: Quoting](https://www.gnu.org/software/bash/manual/bash.html#Quoting) — comillas simples, dobles y `\`.
- [GNU Bash Manual: Command Substitution](https://www.gnu.org/software/bash/manual/bash.html#Command-Substitution) — `$( )` y el recorte de saltos de línea finales.
- [GNU Bash Manual: Word Splitting](https://www.gnu.org/software/bash/manual/bash.html#Word-Splitting) — por qué `$var` sin comillas se parte.
- [GNU Bash Manual: Environment](https://www.gnu.org/software/bash/manual/bash.html#Environment) — `export` y `VAR=valor comando`.
- [Greg's Wiki: Quotes](https://mywiki.wooledge.org/Quotes) — la regla «comillas siempre».
