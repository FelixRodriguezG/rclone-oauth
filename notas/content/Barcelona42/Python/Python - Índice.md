---
tags: [proyecto, python, indice, rank-2, modulos-python]
aliases: ["Python Modules", "Módulos de Python", "Python Module 00"]
---
# Python - Índice

[[Inicio|← Inicio]] · Proyectos · Rank 2

> Los **Python Modules** (00 a 10, más los «bis») son la introducción a Python del nuevo Common Core. Son series de ejercicios cortos que van de `print` y funciones hasta clases, excepciones, paquetes, entornos virtuales, validación con Pydantic y programación funcional. Preparan los proyectos en Python que vienen después: [[A-Maze-ing - Índice|A-Maze-ing]] en el rank 2 y Call Me Maybe y Fly-in en el rank 3.

> [!warning] Comprueba el subject
> Resumen basado en repositorios públicos de alumnos del nuevo cursus (2025–2026), no en los PDFs. Los detalles cambian entre versiones. Lo que manda es el PDF de tu intra.

## Dónde encajan en tu cursus

| Rank | Proyectos |
|---|---|
| 0 | [[libft - Índice\|libft]] |
| 1 | [[ft_printf - Índice\|ft_printf]] · [[get_next_line - Índice\|get_next_line]] · [[push_swap - Índice\|push_swap]] |
| 2 | [[Born2beroot - Índice\|Born2beroot]] · **Python Module 00–10** (+ 05 bis, 06 bis) · [[A-Maze-ing - Índice\|A-Maze-ing]] · [[Exam Rank 02]] |
| 3 | Call Me Maybe · Codexion · Fly-in · Exam Rank 03 |

> [!info] Cómo se aprueban
> En el repositorio público de referencia cada módulo se considera aprobado solo con un **100 %**; si falla un ejercicio, el módulo cuenta como suspendido. Compruébalo en tu intra.

## Los módulos de un vistazo

Títulos sacados del repositorio público [martinnsanzz/Python-Modules](https://github.com/martinnsanzz/Python-Modules); los conceptos, de leer el código de sus ejercicios.

| Módulo | Tema (título del subject) | Conceptos que trabaja |
|---|---|---|
| 00 | *Python Fundamentals Through Garden Data* | `print`, `input`, variables, `if`, bucles, funciones, recursión vs. iteración, f-strings, primeras anotaciones de tipo |
| 01 | *Object-Oriented Garden Systems* | Clases, `__init__`, `self`, métodos, encapsulación (getters/setters), herencia (`class Flower(Plant)`), `super()` |
| 02 | *Data Engineering for Smart Agriculture* | Excepciones: `try`/`except`/`else`/`finally`, `raise`, varios tipos de error, excepciones propias (`class PlantError(Exception)`) |
| 03 | *Mastering Python Collections for Data Engineering* | `sys.argv`, listas, tuplas, conjuntos (`set`), diccionarios, comprensiones, generadores con `yield`, `sum`/`max`/`min` |
| 04 | *Archivist Training Protocol* | Ficheros: `open`, modos `r`/`w`, `with` (gestor de contexto), `sys.stdin`, `sys.stdout`, `sys.stderr` |
| 05 | *Polymorphic Data Streams in the Digital Matrix* | Clases abstractas (`abc.ABC`, `@abstractmethod`), polimorfismo, `typing.Any`, tipos unión `int \| float` |
| 05 bis | No verificado | — |
| 06 | *Mastering Python's Import Mysteries* | Módulos y paquetes, `__init__.py`, `import x` vs. `from x import y`, imports relativos (`from .mod import f`), imports circulares |
| 06 bis | No verificado | — |
| 07 | *Master the Art of Abstract Card Architecture* | Diseño con clases abstractas, herencia múltiple (capacidades), patrones *factory* y *strategy* |
| 08 | *Welcome to the Real World of Data Engineering* | Entornos virtuales (`venv`), `sys.prefix`, `pip`, gestor de paquetes (Poetry, `pyproject.toml`), variables de entorno y `.env` (`python-dotenv`) |
| 09 | *Discover Pydantic Models & Validation* | Pydantic: `BaseModel`, `Field`, `ValidationError`, `@model_validator`, `Enum`, `datetime`, modelos anidados |
| 10 | *Master the Ancient Arts of Functional Programming* | `lambda`, `map`/`filter`/`sorted`, funciones de orden superior, *closures* y ámbito (`nonlocal`), `functools` (`reduce`, `partial`, `lru_cache`, `singledispatch`), decoradores con `@wraps` |

> [!warning] Comprueba el subject
> No se ha encontrado ningún repositorio público con el contenido de **05 bis** ni de **06 bis**. Por la posición en el grafo, probablemente amplían los módulos 05 (polimorfismo) y 06 (imports), pero es una suposición: léelo en la intra.

## De C a Python: lo que cambia

| | C | Python |
|---|---|---|
| Ejecución | Compilas (`cc`) y ejecutas el binario | **Sin compilar**: `python3 fichero.py`. El intérprete lee y ejecuta |
| Errores | Muchos los ve el compilador | La mayoría aparecen **al ejecutar esa línea** (`NameError`, `TypeError`…) |
| Tipos | Estáticos: `int n;` | Dinámicos: la variable no tiene tipo, el **valor** sí. Las anotaciones (`n: int`) no se comprueban al ejecutar |
| Memoria | `malloc` / `free` a mano ([[Segmentos de memoria en C]]) | Automática: un recolector libera lo que nadie usa. No hay punteros ni leaks «clásicos» |
| Cadenas | `char *` terminado en `'\0'` ([[Carácter nulo y puntero NULL]]) | `str`, inmutable, con longitud propia: `len(s)` |
| Enteros | Tamaño fijo, desbordan ([[Tipos enteros y conversiones]]) | Tamaño **ilimitado**: `2 ** 100` funciona |
| Bloques | `{ }` | **Indentación** (4 espacios). Mezclar tabs y espacios da `TabError` |
| Arrays | Tamaño fijo, sin comprobar límites | `list` crece sola; `l[99]` fuera de rango lanza `IndexError` |
| Errores de ejecución | Códigos de retorno, `errno` ([[Control de errores en C]]) | **Excepciones**: `try` / `except` |
| `main` | `int main(int argc, char **argv)` | `if __name__ == "__main__":` y `sys.argv` |
| Estructuras | `struct` + funciones ([[Structs y typedef]]) | `class` con datos y métodos juntos |
| Estilo | [[La Norma]] + norminette | PEP 8 + flake8 |

El mismo programa en los dos lenguajes:

```c
#include <stdio.h>

int	main(int argc, char **argv)
{
	int	i;

	i = 1;
	while (i < argc)
	{
		printf("Argumento %d: %s\n", i, argv[i]);
		i++;
	}
	return (0);
}
```

```python
import sys


def main() -> None:
    """Imprime cada argumento con su número."""
    for i, arg in enumerate(sys.argv[1:], start=1):
        print(f"Argumento {i}: {arg}")


if __name__ == "__main__":
    main()
```

```text
$ python3 args.py hola 42
Argumento 1: hola
Argumento 2: 42
```

> [!note] Qué es `if __name__ == "__main__":`
> Cuando ejecutas un fichero, Python pone `__name__` a `"__main__"`. Cuando otro fichero lo **importa**, `__name__` es el nombre del módulo. Así el código de prueba no se ejecuta al importar. Es lo más parecido a un `main` de C.

> [!warning] Asignar no copia
> `b = a` con una lista **no** la copia: las dos variables apuntan a la misma lista (como dos punteros al mismo `malloc`). Para copiar: `b = a.copy()` o `b = list(a)`.

## Vocabulario

| Término | Significado |
|---|---|
| **Intérprete** | Programa (`python3`) que lee tu código y lo ejecuta directamente, sin generar un binario |
| **Módulo** | Un fichero `.py`. Se usa desde otro con `import` |
| **Paquete** | Carpeta de módulos, normalmente con un `__init__.py` |
| **Entorno virtual (venv)** | Carpeta con su propio Python y sus propias librerías, aislada del sistema |
| **pip** | Instalador de paquetes de Python (desde PyPI) |
| **Type hint** (anotación de tipo) | `def f(x: int) -> str:`. Documenta tipos; no se comprueban al ejecutar, sí con mypy |
| **Docstring** | Cadena entre `"""` justo debajo de una función o clase que explica qué hace |
| **Excepción** | Objeto de error que se «lanza» (`raise`) y se «captura» (`except`) |
| **Iterable / generador** | Algo que se puede recorrer con `for`. Un generador produce los valores de uno en uno con `yield` |
| **Decorador** | Función que envuelve a otra para añadirle comportamiento: `@mi_decorador` |

## Entorno de trabajo

```bash
python3 --version                 # versión instalada (en la VM de prueba: Python 3.13.1)
python3 -m venv .venv             # crea un entorno virtual en .venv/
source .venv/bin/activate         # actívalo: el prompt muestra (.venv)
python3 -m pip install flake8 mypy
deactivate                        # salir del entorno
```

- **Usa siempre `python3 -m pip`** en vez de `pip` a secas: así instalas en el Python que estás usando y no en otro.
- **No subas `.venv/` al repositorio**: añádelo a `.gitignore` (ver [[Git para 42]]).
- En el módulo 08 se trabaja justo esto: detectar si estás dentro de un venv (`sys.prefix != sys.base_prefix`), instalar dependencias y leer variables de entorno.

> [!tip] Ejecuta como el evaluador
> Prueba desde una terminal limpia, con el venv recién creado e instalando solo lo que diga tu `requirements.txt` o `pyproject.toml`. Si funciona solo en tu máquina, falla en evaluación.

## Herramientas que suelen exigir

| Herramienta | Qué hace | Cómo se usa |
|---|---|---|
| **flake8** | Comprueba estilo PEP 8 (líneas largas, espacios, imports sin usar…). Es la «norminette» de Python | `flake8 .` |
| **mypy** | Comprueba los type hints sin ejecutar el código | `mypy .` · más estricto: `mypy --strict .` |
| **Type hints** | Anotar parámetros y retorno de cada función | `def area(w: int, h: int) -> int:` |
| **Docstrings** | Explicar funciones y clases (PEP 257) | `"""Devuelve el área."""` |

> [!warning] Comprueba el subject
> El repositorio de referencia afirma que **todos los módulos pasan flake8 y mypy** (salvo los ejercicios que piden errores a propósito), y en A-Maze-ing las reglas `make lint` / `make lint-strict` lanzan flake8 y mypy. Que los docstrings sean **obligatorios** no está confirmado: escríbelos de todas formas.

Ejemplo que pasa flake8 y `mypy --strict`:

```python
def count_harvest(days: int) -> int:
    """Cuenta hacia atrás los días hasta la cosecha de forma recursiva."""
    if days <= 0:
        return 0
    return 1 + count_harvest(days - 1)
```

## Lo esencial de cada módulo

### Módulo 00: fundamentos

Funciones, `input()` (siempre devuelve `str`: conviértelo con `int()`), `if`/`elif`/`else`, `for`/`while`, f-strings. Un ejercicio pide lo mismo de forma **iterativa y recursiva** (ver [[Recursión]]).

> [!warning] `int(input())` puede explotar
> Si el usuario escribe `abc`, `int()` lanza `ValueError`. En C, `atoi` devolvía `0` en silencio.

### Módulo 01: programación orientada a objetos

```python
class Plant:
    """Planta con nombre y altura."""

    def __init__(self, name: str, height: float) -> None:
        self.name = name
        self._height = height          # '_' = «privado» por convención

    def grow(self, cm: float) -> None:
        if cm < 0:
            raise ValueError("una planta no encoge")
        self._height += cm


class Flower(Plant):
    def __init__(self, name: str, height: float, color: str) -> None:
        super().__init__(name, height)
        self.color = color
```

`self` es el objeto actual (como pasar un puntero a la `struct` en C). `super()` llama al constructor de la clase madre.

### Módulo 02: excepciones

```python
try:
    n = int("abc")
except ValueError as e:
    print(f"Error: {e}")
else:
    print("sin errores")
finally:
    print("esto se ejecuta siempre")
```

Excepciones propias: `class PlantError(Exception): ...` y `raise PlantError("mensaje")`.

### Módulo 03: colecciones

| Tipo | Ejemplo | Ordenado | Modificable | Duplicados |
|---|---|---|---|---|
| `list` | `[3, 1, 3]` | Sí | Sí | Sí |
| `tuple` | `(1, 2, 3)` | Sí | No | Sí |
| `set` | `{1, 2, 3}` | No | Sí | No |
| `dict` | `{"oro": 3}` | Por orden de inserción | Sí | Claves únicas |

Comprensiones (`[x * 2 for x in nums if x > 0]`) y generadores (`yield`), que producen valores de uno en uno sin guardar toda la lista en memoria.

### Módulo 04: ficheros y flujos

```python
with open("datos.txt", "r") as f:      # se cierra solo al salir del bloque
    for line in f:
        print(line.rstrip("\n"))
```

`with` sustituye al par `open`/`close` de C ([[Descriptores de fichero y read]]). Errores típicos: `FileNotFoundError`, `PermissionError`. `sys.stderr.write()` equivale a escribir en el fd 2.

### Módulo 05: polimorfismo y clases abstractas

Una clase abstracta (`class DataProcessor(ABC)`) define métodos con `@abstractmethod` que las hijas **deben** implementar. No se puede instanciar directamente. Polimorfismo: recorres una lista de procesadores distintos y llamas a `p.process(data)` sin saber de qué tipo es cada uno.

### Módulo 06: imports y paquetes

```text
alchemy/
├── __init__.py          ← convierte la carpeta en paquete
├── elements.py
└── grimoire/
    ├── __init__.py
    └── spellbook.py     ← from .validator import check  (import relativo)
```

Diferencia entre `import alchemy.elements` y `from alchemy.elements import fire`, qué exporta `__init__.py` y por qué un **import circular** (A importa B y B importa A) falla.

### Módulo 07: arquitectura con clases abstractas

Combinar varias clases base (`class Sproutling(Creature, HealCapability)`), *factories* que crean objetos y estrategias intercambiables. Prioriza diseño: qué responsabilidad tiene cada clase.

### Módulo 08: entornos y dependencias

`venv`, `pip`, un gestor de proyectos (Poetry con `pyproject.toml` en el repositorio de referencia), y configuración con variables de entorno: `os.getenv("API_KEY")` leyendo un `.env` con `python-dotenv`. Regla: el `.env` con secretos **no** se sube; se sube un `.env.example`.

### Módulo 09: Pydantic

Validar datos declarando los tipos en una clase:

```python
from pydantic import BaseModel, Field, ValidationError


class Station(BaseModel):
    name: str = Field(min_length=1)
    crew: int = Field(ge=1, le=20)


try:
    Station(name="ISS", crew=0)
except ValidationError as e:
    print(e)
```

Pydantic es una librería externa: se instala en el venv con `python3 -m pip install pydantic`.

### Módulo 10: programación funcional

```python
from functools import wraps
from typing import Any, Callable


def logged(func: Callable[..., Any]) -> Callable[..., Any]:
    @wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        print(f"llamando a {func.__name__}")
        return func(*args, **kwargs)
    return wrapper


@logged
def add(a: int, b: int) -> int:
    return a + b
```

`lambda`, `map`, `filter`, `sorted(key=...)`, *closures* (una función interna que recuerda variables de la externa, parecido a una [[Variables static y ámbito|`static`]] pero por instancia), `functools.reduce`, `partial` y `lru_cache` (memoización).

## Plan de estudio

1. Lee el [tutorial oficial de Python en español](https://docs.python.org/es/3/tutorial/index.html), capítulos 3 a 5, con un intérprete abierto (`python3` sin argumentos).
2. Monta el entorno: venv, flake8 y mypy. Pásalos desde el primer ejercicio, no al final.
3. Módulos 00–02 seguidos: sintaxis, clases y excepciones son la base de todo lo demás.
4. Módulos 03–04 con el tutorial de [estructuras de datos](https://docs.python.org/es/3/tutorial/datastructures.html) y de [entrada/salida](https://docs.python.org/es/3/tutorial/inputoutput.html).
5. Módulos 05–07 (y los «bis»): diseño orientado a objetos. Dibuja las clases en papel antes de escribirlas.
6. Módulo 08 antes de A-Maze-ing: lo necesitarás para el `Makefile`, el venv y el paquete instalable.
7. Módulos 09–10 al final.

## Errores frecuentes

> [!danger] Lo que suele suspender
> - flake8 o mypy con avisos (líneas de más de 79 caracteres, imports sin usar, funciones sin anotar).
> - Capturar todo con `except:` o `except Exception:` y ocultar el error real.
> - Código suelto a nivel de módulo que se ejecuta al importar (falta `if __name__ == "__main__":`).
> - Valores por defecto mutables: `def f(l: list = [])` comparte la misma lista entre llamadas. Usa `None`.
> - Mezclar tabs y espacios.
> - Subir `.venv/`, `__pycache__/` o el `.env` con secretos.
> - Usar una librería externa que el subject no permite.

## Relacionadas

[[A-Maze-ing - Índice]] · [[Exam Rank 02]] · [[Born2beroot - Índice]] · [[Recursión]] · [[Structs y typedef]] · [[Control de errores en C]] · [[Git para 42]] · [[La Norma]]

## Referencias

- [martinnsanzz/Python-Modules](https://github.com/martinnsanzz/Python-Modules) — títulos de los módulos 00–10, regla del 100 %, uso de flake8 y mypy; conceptos de cada módulo deducidos de su código.
- [martinnsanzz/42-Curriculum](https://github.com/martinnsanzz/42-Curriculum) — orden de proyectos por rank del nuevo Common Core.
- [Diogo-Serra/42_School](https://github.com/Diogo-Serra/42_School) — segundo alumno del nuevo cursus que confirma Python Modules y A-Maze-ing en el tronco común.
- [Tutorial de Python (es)](https://docs.python.org/es/3/tutorial/index.html) — sintaxis, [clases](https://docs.python.org/es/3/tutorial/classes.html), [errores y excepciones](https://docs.python.org/es/3/tutorial/errors.html), [módulos](https://docs.python.org/es/3/tutorial/modules.html), [estructuras de datos](https://docs.python.org/es/3/tutorial/datastructures.html), [entrada y salida](https://docs.python.org/es/3/tutorial/inputoutput.html).
- [Entornos virtuales y paquetes (es)](https://docs.python.org/es/3/tutorial/venv.html) y [venv](https://docs.python.org/es/3/library/venv.html) — crear y activar un venv.
- [abc](https://docs.python.org/es/3/library/abc.html), [typing](https://docs.python.org/es/3/library/typing.html), [functools](https://docs.python.org/es/3/library/functools.html) y [HOWTO de programación funcional](https://docs.python.org/es/3/howto/functional.html) — módulos 05, 07 y 10.
- [PEP 8](https://peps.python.org/pep-0008/) — guía de estilo (79 caracteres, 4 espacios). [PEP 257](https://peps.python.org/pep-0257/) — docstrings. [PEP 484](https://peps.python.org/pep-0484/) — type hints.
- [flake8](https://flake8.pycqa.org/en/latest/) y [mypy](https://mypy.readthedocs.io/en/stable/) — uso y opciones (`--strict`).
- [pip](https://pip.pypa.io/en/stable/), [Poetry](https://python-poetry.org/docs/), [python-dotenv](https://pypi.org/project/python-dotenv/) y [Pydantic](https://docs.pydantic.dev/latest/) — módulos 08 y 09.
- Los ejemplos de código se han ejecutado con Python 3.13.1 y pasan `flake8` y `mypy --strict` (Pydantic 2.14 en un venv).
