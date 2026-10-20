---
tags: [bash, redirecciones, tuberias, stdin, stdout, stderr, here-doc]
aliases: ["Pipes en Bash", "2>&1", "Here-doc", "Here-string", "tee"]
---
# Redirecciones y tuberías

[[Inicio|← Inicio]] · [[Bash - Índice|Bash]]

> Todo programa lee de **stdin** y escribe en **stdout** y **stderr**. Las **redirecciones** (`>`, `<`, `2>`…) conectan esas salidas con archivos, y las **tuberías** (`|`) conectan la salida de un programa con la entrada de otro. Es la base para combinar comandos pequeños en Bash.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Descriptor de fichero** (*fd*) | Número que identifica algo abierto por el proceso; ver [[Descriptores de fichero y read]] |
| **stdin** (fd 0) | Entrada estándar: por defecto, el teclado |
| **stdout** (fd 1) | Salida estándar: los resultados, por defecto la pantalla |
| **stderr** (fd 2) | Salida de errores: avisos y errores, también a la pantalla |
| **Redirección** | Cambiar a dónde apunta un fd antes de ejecutar el comando |
| **Tubería** (*pipe*) | `a \| b`: el stdout de `a` entra como stdin de `b` |
| **Here-doc** | Bloque de texto escrito dentro del script que se pasa como stdin |

Son los mismos `0`, `1` y `2` que usas en C con `write(1, ...)` o `read(0, ...)`.

## Redirigir la salida

| Sintaxis | Efecto |
|---|---|
| `cmd > f` | stdout a `f` (lo **vacía** primero) |
| `cmd >> f` | stdout al **final** de `f` (añade) |
| `cmd 2> f` | stderr a `f` |
| `cmd 2>> f` | stderr al final de `f` |
| `cmd &> f` | stdout **y** stderr a `f` (solo Bash) |
| `cmd > f 2>&1` | Lo mismo, forma portable |
| `cmd < f` | stdin desde `f` |
| `cmd >&2` | stdout a donde vaya stderr (para mensajes de error) |

```bash
echo "Comprar pan" >> tareas.txt      # añade una tarea
wc -l < tareas.txt                    # 1 (sin el nombre del archivo)
ls /noexiste 2> errores.log           # el error va al archivo, no a la pantalla
echo "algo falló" >&2                 # mensaje de error de tu script
```

> [!danger] `>` borra el archivo antes de ejecutar
> `sort tareas.txt > tareas.txt` deja el archivo **vacío**: Bash lo trunca antes de que `sort` lo lea. Escribe en un temporal y luego `mv` (ver [[Configuración y estado de un script]]).

## `2>&1`: el orden importa

`2>&1` significa «haz que el fd 2 apunte **adonde apunte ahora** el fd 1». Las redirecciones se aplican de izquierda a derecha:

```bash
ls . /noexiste > todo.txt 2>&1   # 1→archivo; luego 2→donde 1 = archivo. Todo al archivo
ls . /noexiste 2>&1 > todo.txt   # 2→pantalla (donde estaba 1); luego 1→archivo
                                 # El error sigue saliendo en pantalla
```

Piensa en flechas que **se copian** en ese momento, no en enlaces que se actualizan.

## `/dev/null`: el agujero negro

`/dev/null` es un archivo especial que se traga todo lo que le escribes:

```bash
grep -q "pan" tareas.txt                 # -q ya no imprime nada
command -v git > /dev/null               # solo me interesa el código de salida
find / -name "*.conf" 2> /dev/null       # oculta los «Permission denied»
```

## Tuberías

```bash
sort tareas.txt | uniq | wc -l     # cuántas tareas distintas hay
```

Cada comando de la tubería se ejecuta **a la vez**, en su propio proceso (ver [[Subshells y procesos en Bash]]). Solo pasa **stdout**: los errores siguen yendo a la pantalla. Para pasar ambos: `cmd 2>&1 | less` o `cmd |& less`.

> [!note] Código de salida de una tubería
> Por defecto es el del **último** comando. Con `set -o pipefail` falla si falla cualquiera (ver [[Scripts robustos con set y trap]]).

### `tee`: guardar y ver a la vez

`tee` copia su entrada a un archivo **y** a stdout, como una T de fontanería:

```bash
make 2>&1 | tee compilacion.log          # lo ves en pantalla y queda guardado
echo "dos" | tee -a tareas.txt > /dev/null   # -a: añadir en vez de sobrescribir
echo "linea" | sudo tee /etc/archivo     # truco: sudo no afecta a '>' pero sí a tee
```

## Here-doc: `<<`

Pasa varias líneas escritas en el propio script como stdin. La palabra final (`FIN`, `EOF`…) la eliges tú y debe ir **sola en su línea**:

```bash
nombre="Ana"
cat <<FIN
Hola, $nombre
Hoy es $(date +%A)
FIN
```

```text
Hola, Ana
Hoy es martes
```

| Variante | Efecto |
|---|---|
| `<<FIN` | Expande variables y `$( )` |
| `<<'FIN'` | Texto **literal**: `$nombre` se queda tal cual |
| `<<-FIN` | Quita los **tabuladores** (no espacios) del principio de cada línea, para poder indentar |

```bash
if true; then
	cat <<-FIN
	Esta línea va indentada con un tab en el script
	FIN
fi
```

Uso típico: el texto de ayuda de `usage` (ver [[Argumentos y subcomandos en Bash]]) o generar un archivo: `cat > config.sh <<'FIN' ... FIN`.

## Here-string: `<<<`

Pasa **una cadena** como stdin, sin `echo` ni tubería:

```bash
read -r nombre apellido <<< "Ana López"
echo "$apellido"                   # López
grep -c "o" <<< "uno dos tres"     # 1 (una línea que contiene 'o')
```

Con `echo "Ana López" | read -r ...` las variables se perderían, porque `read` correría en una subshell (ver [[Subshells y procesos en Bash]]). Todo sobre `read`: [[Leer entrada del usuario en Bash]].

## Errores típicos

| Error | Qué pasa |
|---|---|
| `cmd 2>&1 > f` | El error no va al archivo: el orden importa |
| `sort f > f` | `f` queda vacío |
| `> f` cuando querías `>>` | Pierdes el contenido anterior |
| `2 > f` con espacio | Bash lo entiende como argumento `2` + `> f` |
| Indentar con espacios en `<<-` | Solo quita tabs; la palabra final no se reconoce |
| `sudo echo x > /etc/f` | La redirección la hace tu shell, sin permisos; usa `sudo tee` |

Siguiente: [[Códigos de salida y errores en Bash]].

## Relacionadas

[[Descriptores de fichero y read]] · [[Códigos de salida y errores en Bash]] · [[Subshells y procesos en Bash]] · [[Leer entrada del usuario en Bash]] · [[grep, sed y awk lo justo]] · [[Scripts robustos con set y trap]] · [[Bash - Chuleta de sintaxis]]

## Referencias

- [GNU Bash Manual: Redirections](https://www.gnu.org/software/bash/manual/html_node/Redirections.html) — orden de las redirecciones, `&>`, here-docs, here-strings.
- [GNU Bash Manual: Pipelines](https://www.gnu.org/software/bash/manual/html_node/Pipelines.html) — `|`, `|&` y código de salida.
- [BashGuide: Input and Output (Greg's Wiki)](https://mywiki.wooledge.org/BashGuide/InputAndOutput) — explicación de los fd y `2>&1`.
- `man tee`, `man null` — `tee -a` y `/dev/null`.
- Ejemplos probados con Bash 5.1.16 en Ubuntu 22.04.
