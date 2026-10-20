---
tags: [tui, arquitectura, python, bash, subprocess, diseno]
aliases: ["Arquitectura de una TUI", "Separar lógica y vista", "Modelo vista"]
---
# De script a TUI - arquitectura

[[Inicio|← Inicio]] · [[TUI - Índice|TUI]]

> Para convertir un script en una TUI sin crear un monstruo, separa **qué hace** el programa (la lógica: añadir, listar, borrar tareas) de **cómo se ve** (la vista: paneles, colores, teclas). La TUI `tareas.py` no reescribe `tareas.sh`: lo **llama** con `subprocess` y muestra su salida bonita.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Lógica** (o dominio) | Las reglas del programa: qué es una tarea, cómo se guarda. No sabe nada de pantallas |
| **Vista** | El código que dibuja: paneles, colores, listas. No sabe dónde se guardan las tareas |
| **Modelo / estado** | Los datos que la TUI tiene en memoria **ahora**: las tareas, cuál está seleccionada, el filtro |
| **`subprocess`** | Módulo de Python para ejecutar otros programas (como `tareas.sh`) y leer su salida |
| **Contrato** | Lo que la TUI espera del script: nombres de subcomandos, formato de salida, códigos de salida |

## Dos formas de reutilizar el script

| Opción | Cómo | Cuándo |
|---|---|---|
| **A. Llamar al script** | `subprocess.run(["./tareas.sh", "lista"])` y parsear la salida | El script ya funciona, lo usa más gente o quieres mantener una sola lógica |
| **B. Reimplementar en Python** | Leer y escribir `~/.tareas.txt` desde un módulo Python | El script se queda corto, la salida es difícil de parsear o necesitas rendimiento |

Empieza siempre por **A**: tienes algo funcionando en una tarde y la CLI sigue sirviendo para automatizar (ver [[CLI, TUI y GUI]]). Si más adelante pasas a **B**, solo cambias el módulo de lógica: la vista ni se entera.

## El contrato con `tareas.sh`

Tomamos el `tareas.sh` de referencia de [[Argumentos y subcomandos en Bash]] (sección «Todo junto»). Guarda una tarea por línea en `$HOME/.tareas.txt` (o en el archivo que diga la variable `TAREAS_ARCHIVO`):

```console
$ ./tareas.sh añadir Comprar pan
$ ./tareas.sh lista
 1. Comprar pan
 2. Estudiar bash
$ ./tareas.sh borrar 9x; echo $?
tareas.sh: borrar necesita un número
2
```

Lo que la TUI necesita saber: los subcomandos (`añadir TEXTO`, `lista`, `borrar N`), que `lista` imprime ` 1. texto` por línea (o `(sin tareas)` si no hay ninguna), que los errores van a `stderr` y que el código de salida es distinto de 0 si algo falla (ver [[Códigos de salida y errores en Bash]]).

> [!warning] Si cambias la salida del script, rompes la TUI
> Añadir colores a `lista` o cambiar `1.` por `1)` hace que el parseo falle. Por eso la nota [[Colores en la terminal]] solo pinta cuando la salida es una terminal: cuando la lee `tareas.py`, sale texto limpio.

## La capa de lógica: `modelo.py`

Todo el trato con `tareas.sh` vive en **un solo archivo**. La vista solo ve objetos `Tarea`:

```python
"""modelo.py: habla con tareas.sh. No sabe nada de curses."""
from dataclasses import dataclass
from pathlib import Path
import re
import subprocess

SCRIPT = Path(__file__).with_name("tareas.sh")   # junto a este archivo
LINEA = re.compile(r"^\s*(\d+)\. (.*)$")          # " 1. Comprar pan"


@dataclass
class Tarea:
    num: int
    texto: str


class ErrorTareas(Exception):
    """tareas.sh terminó con un código distinto de 0."""


def ejecutar(*args: str) -> str:
    """Llama a tareas.sh con args y devuelve su stdout."""
    res = subprocess.run([str(SCRIPT), *args], capture_output=True,
                         text=True)
    if res.returncode != 0:
        raise ErrorTareas(res.stderr.strip() or f"código {res.returncode}")
    return res.stdout


def listar() -> list[Tarea]:
    tareas = []
    for linea in ejecutar("lista").splitlines():
        if m := LINEA.match(linea):      # ignora «(sin tareas)»
            tareas.append(Tarea(int(m[1]), m[2]))
    return tareas


def anadir(texto: str) -> None:
    ejecutar("añadir", texto)


def borrar(num: int) -> None:
    ejecutar("borrar", str(num))
```

- Los argumentos van en una **lista**, sin `shell=True`: así un texto como `pan; rm -rf ~` llega como texto y no como comando.
- `capture_output=True, text=True` guarda `stdout` y `stderr` como `str` (ver [[curses - Ejecutar comandos]]).
- Se prueba **sin interfaz**, y con un archivo de prueba para no tocar tus tareas de verdad (`subprocess` pasa las variables de entorno al script):

```console
$ TAREAS_ARCHIVO=/tmp/prueba.txt python3 -c "import modelo; modelo.anadir('Comprar pan'); print(modelo.listar())"
[Tarea(num=1, texto='Comprar pan')]
```

  Si esto falla, la TUI fallará.

## El ciclo de la vista

Toda TUI repite lo mismo (ver [[curses - El bucle principal]]):

```text
   ┌──────────── estado (tareas, sel, filtro, mensaje) ◄───────────┐
   │                                                               │
   ▼                                                               │
 dibujar(estado) ──► leer tecla ──► acción: mover / añadir / borrar┘
                                        │
                                        └──► modelo.anadir() → tareas.sh
```

La vista **nunca** escribe en `~/.tareas.txt`: pide a `modelo` que lo haga y después recarga la lista.

## Estructura de archivos recomendada

```text
tareas/
├── tareas.sh        # la CLI de siempre: sigue funcionando sola
├── modelo.py        # lógica: llama a tareas.sh, devuelve objetos Tarea
├── tareas.py        # TUI: clase App, bucle, dibujo y teclas
├── README.md        # cómo instalar y usar
└── .gitignore
```

Para el `README.md`, ver [[README de un proyecto]]. Cuando `tareas.py` pase de unas 300 líneas, divídelo: `vista.py` (dibujo), `teclas.py` (tabla de atajos), `tareas.py` (solo arranca).

## Evolución por etapas

| Etapa | Qué añades | Nota |
|---|---|---|
| 0 | `tareas.sh` funciona por CLI | [[Bash - Índice]] |
| 1 | `modelo.py` y pruebas desde `python3 -c` | esta nota |
| 2 | Ventana que muestra «hola» y sale | [[curses - Primeros pasos]] |
| 3 | Bucle con selección `j`/`k` | [[curses - El bucle principal]] |
| 4 | Cabecera, lista con scroll, detalle, barra | [[curses - Layout y paneles]] |
| 5 | Colores con significado | [[curses - Colores y estilos]] |
| 6 | Tabla de atajos, ayuda `?`, filtro `/` | [[curses - Teclado y atajos]] |
| 7 | Añadir/borrar con `tareas.sh`, abrir `vim` | [[curses - Ejecutar comandos]] |
| 8 | Repaso de diseño | [[Diseño de una TUI moderna]] |

Haz un commit al final de cada etapa (ver [[Git para 42]]). Si algo se rompe, sabrás en qué paso fue.

## Errores típicos

| Error | Consecuencia |
|---|---|
| `subprocess.run("./tareas.sh añadir " + texto, shell=True)` | Comillas rotas e inyección de comandos |
| Ruta relativa `"./tareas.sh"` | Solo funciona si lanzas la TUI desde esa carpeta; usa `Path(__file__)` |
| No comprobar `returncode` | Un error del script se ve como «lista vacía» |
| Llamar a `curses` desde `modelo.py` | Ya no puedes probar la lógica sin abrir la interfaz |
| `tareas.sh` sin permiso de ejecución | `PermissionError` al llamarlo: `chmod +x tareas.sh` |

## Relacionadas

[[CLI, TUI y GUI]] · [[Argumentos y subcomandos en Bash]] · [[Configuración y estado de un script]] · [[Subshells y procesos en Bash]] · [[curses - El bucle principal]] · [[curses - Ejecutar comandos]] · [[Elegir un framework de TUI]] · [[Python - Índice]] · [[Unicode y emojis en la terminal]]

## Referencias

- [subprocess — Subprocess management](https://docs.python.org/3/library/subprocess.html) — `run`, `capture_output`, `text`, «Security Considerations» sobre `shell=True`.
- [dataclasses](https://docs.python.org/3/library/dataclasses.html) — `@dataclass`.
- [re — Regular expression operations](https://docs.python.org/3/library/re.html) — `match` y grupos `m[1]`.
- [Model–view–controller — Wikipedia](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) — idea de separar modelo y vista.
- `modelo.py` probado con Python 3.13 contra el `tareas.sh` de referencia de [[Argumentos y subcomandos en Bash]] (añadir, listar, borrar, textos con `;`, comillas y emojis, error de `borrar 9x` y archivo vacío).
