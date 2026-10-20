---
tags: [bash, procesos, subshell, jobs, segundo-plano]
aliases: ["Subshell", "Segundo plano en Bash", "wait en Bash", "Sustitución de procesos"]
---
# Subshells y procesos en Bash

[[Inicio|← Inicio]] · [[Bash - Índice|Bash]]

> Cada programa que lanzas desde Bash es un **proceso** hijo, y a veces Bash crea una copia de sí mismo, una **subshell**, para ejecutar parte del script. Lo que cambia en una subshell (variables, `cd`) **no vuelve** al script principal. Saber cuándo se crea una explica muchos «¿por qué mi variable está vacía?» y permite ejecutar tareas en paralelo.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Proceso** | Programa en ejecución, con su memoria y su número (**PID**) |
| **Proceso hijo** | Proceso creado por otro (el padre). Es una copia: lo que cambie no afecta al padre |
| **Subshell** | Proceso hijo que es otra copia de Bash |
| **Segundo plano** (*background*) | El comando sigue corriendo mientras el script continúa |
| **Job** | Comando en segundo plano que la shell controla |

## `( )` frente a `{ }`

Los dos agrupan comandos, pero de forma distinta:

| | `( comandos )` | `{ comandos; }` |
|---|---|---|
| Dónde corre | En una **subshell** | En la shell **actual** |
| Cambios de variables / `cd` | Se pierden al salir | Se quedan |
| Sintaxis | Libre | Espacio tras `{` y `;` (o salto de línea) antes de `}` |

```bash
x=1
( x=2; cd /tmp )          # cambia x y la carpeta... en la copia
echo "$x $PWD"            # 1 /home/ana   ← nada ha cambiado

{ x=3; cd /tmp; }
echo "$x $PWD"            # 3 /tmp
```

> [!tip] Usos típicos
> - `( cd carpeta && make )` → trabajar en otra carpeta sin tener que volver.
> - `{ echo "a"; echo "b"; } > salida.txt` → redirigir varias órdenes a la vez (ver [[Redirecciones y tuberías]]).

## Dónde se crean subshells sin que lo veas

- `$( comando )` → sustitución de comandos.
- Cada lado de una **tubería** `a | b` (en Bash, también el último).
- `( )` y los comandos con `&`.

El caso más traicionero, el contador que vuelve a cero:

```bash
n=0
printf 'a\nb\nc\n' | while IFS= read -r linea; do
    (( n++ ))                 # suma... dentro de la subshell de la tubería
done
echo "$n"                     # 0
```

Solución: que el `while` no esté en una tubería. Redirige un archivo con `<` o usa **sustitución de procesos** `< <( )`:

```bash
n=0
while IFS= read -r linea; do
    (( n++ ))
done < <(printf 'a\nb\nc\n')
echo "$n"                     # 3
```

## Sustitución de procesos: `<( )`

`<(comando)` ejecuta el comando y te da un **nombre de archivo** (algo como `/dev/fd/63`) desde el que leer su salida. Sirve para pasar la salida de un comando a algo que espera un archivo:

```bash
diff <(ls carpeta1) <(ls carpeta2)       # compara dos listados sin archivos temporales
echo <(true)                             # /dev/fd/63
```

`>(comando)` es lo contrario: un archivo donde lo que escribas llega al comando.

## Segundo plano: `&` y `wait`

Un `&` al final lanza el comando y **no espera** a que termine:

```bash
sleep 2 &                      # se queda corriendo
echo "Lanzado el PID $!"       # $! = PID del último proceso en segundo plano
echo "Yo soy el PID $$"        # $$ = PID del script
wait                           # espera a TODOS los hijos
echo "Terminado"
```

`wait PID` espera a uno concreto y **devuelve su código de salida**: así sabes si falló.

```bash
descargar() { sleep "$1"; echo "terminado $1"; }

descargar 2 & p1=$!
descargar 1 & p2=$!
wait "$p1" && echo "el primero fue bien"
wait "$p2" && echo "el segundo fue bien"
```

Las dos tareas corren a la vez: el total tarda unos 2 s, no 3.

> [!warning] La salida se mezcla
> Los procesos en segundo plano escriben en la misma terminal cuando quieren. Si su salida importa, mándala a un archivo por proceso.

## `jobs`, `fg`, `kill` (lo básico)

En una terminal interactiva:

| Comando | Qué hace |
|---|---|
| `jobs` | Lista los trabajos en segundo plano: `[1]+ Running sleep 100 &` |
| `fg %1` | Trae el trabajo 1 al primer plano |
| `Ctrl+Z` y luego `bg` | Pausa el comando actual y lo sigue en segundo plano |
| `kill %1` o `kill PID` | Le pide que termine (señal `TERM`) |
| `kill -9 PID` | Lo mata sin dejarle limpiar: último recurso |

En un script, guarda `$!` en una variable y usa `kill "$pid"`. Para limpiar los hijos si el script se interrumpe, usa `trap` (ver [[Scripts robustos con set y trap]]).

> [!note] Igual que en C
> Por debajo, Bash usa `fork` para crear la copia y `exec` para convertirla en el programa. Por eso el hijo hereda las variables exportadas (ver [[Variables y comillas en Bash]]) y los descriptores abiertos (ver [[Descriptores de fichero y read]]), pero nunca puede cambiar los del padre.

## Errores típicos

| Error | Causa |
|---|---|
| Variable a 0 o vacía tras un `cmd \| while read` | El `while` corre en una subshell: usa `done < <(cmd)` |
| `cd` en una función capturada con `$( )` no tiene efecto | `$( )` es una subshell |
| `{ echo a }` da error de sintaxis | Falta `;` antes de `}` |
| El script termina y deja procesos colgados | Falta `wait` o un `trap` que los mate |
| `wait` siempre devuelve 0 | Sin argumentos no informa de fallos: usa `wait "$pid"` |

Siguiente: [[grep, sed y awk lo justo]].

## Relacionadas

[[Funciones en Bash]] · [[Bucles en Bash]] · [[Redirecciones y tuberías]] · [[grep, sed y awk lo justo]] · [[Scripts robustos con set y trap]] · [[Códigos de salida y errores en Bash]] · [[Spinners y barras de progreso]] · [[Descriptores de fichero y read]]

## Referencias

- [GNU Bash Manual: Command Grouping](https://www.gnu.org/software/bash/manual/bash.html#Command-Grouping) — `( )` frente a `{ }`.
- [GNU Bash Manual: Process Substitution](https://www.gnu.org/software/bash/manual/bash.html#Process-Substitution) — `<( )` y `>( )`.
- [GNU Bash Manual: Pipelines](https://www.gnu.org/software/bash/manual/bash.html#Pipelines) — cada comando de una tubería en su subshell; opción `lastpipe`.
- [GNU Bash Manual: Job Control Basics](https://www.gnu.org/software/bash/manual/bash.html#Job-Control-Basics) — `jobs`, `fg`, `bg`, `%1`.
- [GNU Bash Manual: Special Parameters](https://www.gnu.org/software/bash/manual/bash.html#Special-Parameters) — `$$` y `$!`.
- [Greg's Wiki: BashFAQ/024](https://mywiki.wooledge.org/BashFAQ/024) — por qué se pierde la variable tras una tubería.
- `man 2 fork`, `man 2 wait` — lo que hay debajo.
