---
tags: [42, makefile, make, ar, libft]
---
# Makefile y librerías estáticas

[[Inicio|← Inicio]] · Herramientas

Esta guía cubre desde los conceptos esenciales hasta los errores más comunes que causan suspensos automáticos en las evaluaciones de 42 (como el temido relink o fallos de tabulación).

---

## Índice de Contenidos
1. [¿Qué es un Makefile y cómo funciona?](#1-qué-es-un-makefile-y-cómo-funciona)
2. [Estructura Básica de una Regla](#2-estructura-básica-de-una-regla)
3. [El Tabulador: La Regla de Oro que no perdona](#3-el-tabulador-la-regla-de-oro-que-no-perdona)
4. [Líneas Multilínea (\) y el Error "missing separator"](#4-líneas-multilínea--y-el-error-missing-separator)
5. [Variables: Definición, Expansión y Modos de Asignación](#5-variables-definición-expansión-y-modos-de-asignación)
6. [Variables Automáticas ($@, $<,$^, etc.)](#6-variables-automáticas--etc)
7. [Manipulación de Listas: addprefix y Sustitución de Sufijos](#7-manipulación-de-listas-addprefix-y-sustitución-de-sufijos)
8. [La Bandera -I y la Detección Real de Cambios en Headers](#8-la-bandera--i-y-la-detección-real-de-cambios-en-headers)
9. [Librerías Estáticas y el Comando ar](#9-librerías-estáticas-y-el-comando-ar)
10. [Reglas Falsas: ¿Por qué necesitamos .PHONY?](#10-reglas-falsas-por-qué-necesitamos-phony)
11. [El Relink: Qué es, por qué suspende y cómo evitarlo](#11-el-relink-qué-es-por-qué-suspende-y-cómo-evitarlo)
12. [Organización Profesional: Carpeta obj/ y mkdir -p](#12-organización-profesional-carpeta-obj-y-mkdir--p)
13. [Estética: Silenciar Comandos con @ y Mensajes Claros](#13-estética-silenciar-comandos-con--y-mensajes-claros)
14. [Plantilla Completa: libft (42 Standard)](#14-plantilla-completa-libft-42-standard)
15. [Plantilla Completa con Bonus (Sin Relink)](#15-plantilla-completa-con-bonus-sin-relink)

---

## 1. ¿Qué es un Makefile y cómo funciona?

Un Makefile no es un script de Bash secuencial. Es un **árbol de dependencias basado en marcas de tiempo (timestamps)**.

Cuando ejecutas `make`:
1. Comprueba la fecha de modificación del **archivo que quieres crear** (objetivo).
2. Comprueba la fecha de los **archivos necesarios para crearlo** (prerrequisitos).
3. Si los prerrequisitos son más recientes que el objetivo (o el objetivo aún no existe), ejecuta los comandos para crearlo o actualizarlo. Si nada ha cambiado, no hace nada.

---

## 2. Estructura Básica de una Regla

Toda regla sigue siempre este formato:

```makefile
objetivo: prerrequisito1 prerrequisito2
	comando_en_la_terminal
```

* **Objetivo (Target):** Casi siempre es un archivo que se va a generar (`libft.a`, `main.o`) o una orden abstracta (`clean`, `fclean`).
* **Prerrequisitos (Prerequisites):** Archivos que deben existir o actualizarse antes de que esta regla pueda ejecutarse.
* **Comandos (Recipe):** Órdenes de terminal que Make ejecuta para transformar los prerrequisitos en el objetivo.

---

## 3. El Tabulador: La Regla de Oro que no perdona

> [!warning] Make exige un tabulador real
> Make fue creado en 1976 y exige estrictamente un carácter de tabulación real (`\t`) al inicio de cada línea de comando.

* Si usas **4 espacios** en vez de una tabulación, Make fallará inmediatamente con:
  ```text
  Makefile:XX: *** missing separator. Stop.
  ```
* **Consejo en editores:** Configura tu editor de código para que en archivos con nombre `Makefile` respete siempre el carácter `Tab` real y no lo sustituya por espacios.

---

## 4. Líneas Multilínea (\) y el Error "missing separator"

En un Makefile, presionar `Enter` significa "aquí termina esta instrucción". Cuando listas 20 o 30 archivos fuente, no quieres una línea infinita horizontal, pero tampoco puedes dar `Enter` a secas.

### La solución: barra invertida (\)
La barra invertida al final de la línea le indica a Make: *"ignora este salto de línea y lee la siguiente línea como parte de la misma instrucción"*.

```makefile
# ✅ Correcto: la barra une las líneas en una sola lista continua
SRCS = srcs/ft_putchar.c \
       srcs/ft_putstr.c \
       srcs/ft_strlen.c

# ❌ Incorrecto: la segunda línea queda huérfana y Make dará "missing separator"
SRCS = srcs/ft_putchar.c
       srcs/ft_putstr.c
```

### ⚠️ El fallo invisible: El espacio traicionero
Nunca dejes un espacio después de la barra invertida (`\` + espacio + `Enter`). Si lo haces, Make interpretará que estás escapando el espacio en lugar del salto de línea, y la siguiente línea volverá a romperse con error de sintaxis.

---

## 5. Variables: Definición, Expansión y Modos de Asignación

Las variables evitan repetir texto y permiten modificar banderas de compilación en un solo lugar. Para usarlas se encierran entre paréntesis: `$(NOMBRE_VARIABLE)`.

Existen cuatro formas de asignar valores:

```makefile
# 1. Asignación diferida / recursiva (=)
# Se evalúa en el momento en que se USA la variable, no donde se declara.
CC = cc

# 2. Asignación inmediata (:=)
# Se evalúa exactamente en la línea donde se escribe. Es más rápida y predecible.
SRC_DIR := srcs

# 3. Asignación condicional (?=)
# Asigna el valor SOLO si la variable no existía previamente (por ejemplo, desde la terminal).
CFLAGS ?= -Wall -Wextra -Werror

# 4. Concatenación (+=)
# Añade texto al final de la variable existente, separándolo automáticamente con un espacio.
CFLAGS += -g
```

---

## 6. Variables Automáticas ($@, $<,$^, etc.)

Make provee atajos automáticos que representan los nombres de los archivos involucrados en la regla actual:

| Variable | ¿Qué significa? | Ejemplo práctico |
| :--- | :--- | :--- |
| `$@` | El **nombre del objetivo** actual | En `libft.a: $(OBJS)`, `$@` vale `libft.a` |
| `$<` | El **primer prerrequisito** de la lista | En `%.o: %.c`, `$<` es el archivo fuente `%.c` |
| `$^` | **Todos los prerrequisitos** juntos (sin duplicados) | En `$(NAME): $(OBJS)`, `$^` son todos los `.o` |
| `$?` | Solo los prerrequisitos **más nuevos que el objetivo** | Útil para saber qué ficheros han cambiado |
| `%` | **Comodín** de concordancia de patrones | `%.o: %.c` (aplica la regla a cada `.c` que deba ser `.o`) |

**Ejemplo de regla genérica usando variables automáticas:**
```makefile
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@
# Se traduce como: cc -Wall... -c archivo.c -o archivo.o
```

---

## 7. Manipulación de Listas: addprefix y Sustitución de Sufijos

Escribir `srcs/` delante de cada archivo es tedioso y propenso a errores. Make incluye funciones para automatizarlo:

### addprefix: Prefijar rutas automáticamente
```makefile
SRC_DIR = srcs
FILES   = ft_putchar.c ft_putstr.c ft_strlen.c

# addprefix agrega la carpeta a cada elemento de la lista
SRCS    = $(addprefix $(SRC_DIR)/, $(FILES))
# Resultado: srcs/ft_putchar.c srcs/ft_putstr.c srcs/ft_strlen.c
```

### Sustitución de sufijos: De .c a .o
Para obtener la lista de binarios objeto que necesitas compilar, sustituye la extensión en una sola línea:
```makefile
OBJS = $(SRCS:.c=.o)
# Transforma automáticamente cada 'srcs/archivo.c' en 'srcs/archivo.o'
```

> [!warning] Nada de comodines
> La Norma exige que **todos los archivos fuente estén nombrados explícitamente**. Están prohibidos `SRCS = *.c` y `$(wildcard src/*.c)`, aunque funcionen.

---

## 8. La Bandera -I y la Detección Real de Cambios en Headers

Muchos estudiantes cometen este error: compilan con `-Iincludes`, modifican una estructura o constante en su archivo `.h`, ejecutan `make` y Make responde: `"Nothing to be done"`.

### ¿Por qué pasa?
`-Iincludes` solo ayuda al compilador (`cc`) a encontrar el archivo de cabecera. Pero **Make no lee el interior de los `.c`**; Make solo mira la lista de prerrequisitos de la regla.

```makefile
# ❌ INCORRECTO: si editas libft.h, Make no se entera y no recompila
%.o: %.c
	$(CC) $(CFLAGS) -Iincludes -c $< -o $@

# ✅ CORRECTO: libft.h es prerrequisito explícito; si cambia, se recompilan los .o
HEADER = includes/libft.h

%.o: %.c $(HEADER)
	$(CC) $(CFLAGS) -Iincludes -c $< -o $@
```

---

## 9. Librerías Estáticas y el Comando ar

Una librería estática (`.a`) es simplemente un **archivo contenedor** (no comprimido) que empaqueta múltiples binarios objeto (`.o`) juntos.

Para generarla en 42 usamos:
```makefile
ar rcs libft.a $(OBJS)
```

* **`r` (replace):** Si el `.o` ya estaba dentro del `.a`, lo actualiza; si no estaba, lo añade.
* **`c` (create):** Crea el archivo `.a` silenciosamente si todavía no existe.
* **`s` (symbol index):** Crea y actualiza un índice interno de símbolos (funciones y variables). Hace exactamente lo mismo que el comando histórico `ranlib`.

---

## 10. Reglas Falsas: ¿Por qué necesitamos .PHONY?

Si creas una carpeta o un archivo llamado `clean` dentro de tu proyecto y luego ejecutas `make clean`:
1. Make buscará el objetivo `clean`.
2. Verá que ya existe un archivo o carpeta llamado `clean` en el disco.
3. Como `clean` no tiene prerrequisitos, Make pensará que ya está "al día" y responderá:
   ```text
   make: 'clean' is up to date.
   ```
4. **Tus comandos de borrado nunca se ejecutarán.**

Para indicarle a Make que un objetivo es una **acción** y no un archivo real del disco, usamos `.PHONY`:
```makefile
.PHONY: all clean fclean re bonus
```

---

## 11. El Relink: Qué es, por qué suspende y cómo evitarlo

> [!danger] Proyecto no funcional
> Según la [[La Norma|Norma]], si el Makefile hace relink el proyecto se considera **no funcional**: es un 0 directo en la evaluación.

### ¿Qué es el relink?
Hacer relink significa que Make ejecuta comandos de compilación o empaquetado **cuando nada en el código ha cambiado**.

```bash
$ make
cc -c src/ft_strlen.c -o src/ft_strlen.o
ar rcs libft.a src/ft_strlen.o
$ make
ar rcs libft.a src/ft_strlen.o  <-- ❌ FALLO: No has tocado nada y volvió a empaquetar
```

La respuesta correcta ante un segundo `make` inmediato debe ser:
```bash
$ make
make: Nothing to be done for 'all'.
```

### Las 2 causas más comunes de relink:
1. **Regla que depende de nombres .PHONY en vez de archivos reales:**
   Si pones `$(NAME): all`, como `all` es PHONY, Make siempre considerará que está desactualizado y recompilará.
2. **La trampa de la regla bonus:**
   Si la regla `bonus` llama a `$(NAME)` de nuevo o empaqueta sin control, cada vez que escribas `make bonus` volverá a ejecutar `ar rcs`. Se soluciona usando un **archivo bandera oculto** (ver sección 15).

---

## 12. Organización Profesional: Carpeta obj/ y mkdir -p

Tener los archivos `.o` mezclados con los `.c` ensucia el repositorio. Lo ideal es compilarlos en una carpeta separada llamada `obj/` o `.objs/`.

Para que `cc` no falle diciendo `No such file or directory`, debemos asegurar que la carpeta destino exista usando `@mkdir -p`:

```makefile
SRC_DIR = src
OBJ_DIR = obj

SRCS    = $(SRC_DIR)/ft_strlen.c $(SRC_DIR)/ft_putchar.c
# Reemplaza la ruta 'src/' por 'obj/' y la extensión '.c' por '.o'
OBJS    = $(SRCS:$(SRC_DIR)/%.c=$(OBJ_DIR)/%.o)

# Compilación de objetos creando la carpeta sobre la marcha
$(OBJ_DIR)/%.o: $(SRC_DIR)/%.c $(HEADER)
	@mkdir -p $(dir $@)
	$(CC) $(CFLAGS) -Iincludes -c $< -o $@
```
*(La función `$(dir $@)` extrae la ruta del directorio del archivo objeto que se va a compilar, por ejemplo `obj/`).*

---

## 13. Estética: Silenciar Comandos con @ y Mensajes Claros

Por defecto, Make imprime en la terminal cada comando exacto antes de ejecutarlo. Para mostrar mensajes limpios y profesionales:

* **Anteponer `@` al inicio del comando:** Le dice a Make que ejecute la orden en silencio sin imprimirla en pantalla.
* **Usar `echo` con colores ANSI:**

```makefile
clean:
	@$(RM) $(OBJS)
	@echo "\033[0;33m🧹 Archivos objeto eliminados.\033[0m"

fclean: clean
	@$(RM) $(NAME)
	@echo "\033[0;31m🗑️  Librería $(NAME) eliminada.\033[0m"
```

---

## 14. Plantilla Completa: libft (42 Standard)

Esta plantilla es totalmente compatible con la norma de 42, evita relinks y gestiona las dependencias de cabeceras correctamente.

### Estructura de carpetas recomendada:
```text
mi_proyecto/
├── Makefile
├── includes/
│   └── libft.h
└── src/
    ├── ft_putchar.c
    ├── ft_putstr.c
    └── ft_strlen.c
```

### Código del Makefile:
```makefile
# **************************************************************************** #
#                                                                              #
#                                                         :::      ::::::::    #
#    Makefile                                           :+:      :+:    :+:    #
#                                                     +:+ +:+         +:+      #
#    By: estudiante <estudiante@student.42barcel    +#+  +:+       +#+         #
#                                                 +#+#+#+#+#+   +#+            #
#    Created: 2026/09/27 12:00:00 by estudiante        #+#    #+#              #
#    Updated: 2026/09/27 12:00:00 by estudiante       ###   ########.fr        #
#                                                                              #
# **************************************************************************** #

# ========================================== #
# 1. NOMBRE DEL BINARIO O LIBRERÍA           #
# ========================================== #
NAME        = libft.a

# ========================================== #
# 2. HERRAMIENTAS Y BANDERAS                 #
# ========================================== #
CC          = cc
CFLAGS      = -Wall -Wextra -Werror
AR          = ar rcs
RM          = rm -f

INC_DIR     = includes
SRC_DIR     = src

# Flag de inclusión para el compilador y archivo de dependencia para Make
INCLUDES    = -I $(INC_DIR)
HEADER      = $(INC_DIR)/libft.h

# ========================================== #
# 3. ARCHIVOS FUENTE Y OBJETOS               #
# ========================================== #
SRC_FILES   = ft_putchar.c \
              ft_putstr.c \
              ft_strlen.c

SRCS        = $(addprefix $(SRC_DIR)/, $(SRC_FILES))
OBJS        = $(SRCS:.c=.o)

# ========================================== #
# 4. REGLAS PRINCIPALES                      #
# ========================================== #

# Primera regla: es la que se ejecuta al teclear solo 'make'
all: $(NAME)

# Creación de la librería estática (depende de que existan los .o)
$(NAME): $(OBJS)
	$(AR) $@ $^
	@echo "Librería $(NAME) creada con éxito."

# Regla implícita de compilación: cada .c genera su .o correspondiente
# Exige que el header esté listado para recompilar si este se modifica
%.o: %.c $(HEADER)
	$(CC) $(CFLAGS) $(INCLUDES) -c $< -o $@

# ========================================== #
# 5. REGLAS DE LIMPIEZA                      #
# ========================================== #

clean:
	$(RM) $(OBJS)
	@echo "🧹 Objetos temporales eliminados."

fclean: clean
	$(RM) $(NAME)
	@echo "🗑️  $(NAME) eliminado por completo."

re: fclean all

# ========================================== #
# 6. PHONY TARGETS                           #
# ========================================== #
.PHONY: all clean fclean re
```

---

## 15. Plantilla Completa con Bonus (Sin Relink)

### El truco del archivo centinela (.bonus)
Para la parte bonus de Libft, necesitas que `make bonus` compile tanto los obligatorios como los bonus, pero **sin recompilar ni volver a empaquetar si vuelves a teclear `make bonus`**.

Usamos un archivo oculto vacío llamado `.bonus` como objetivo real de Make:

```makefile
# ========================================== #
# VARIABLES BÁSICAS                          #
# ========================================== #
NAME        = libft.a
CC          = cc
CFLAGS      = -Wall -Wextra -Werror
AR          = ar rcs
RM          = rm -f

INC_DIR     = includes
SRC_DIR     = src
INCLUDES    = -I $(INC_DIR)
HEADER      = $(INC_DIR)/libft.h

# ========================================== #
# ARCHIVOS OBLIGATORIOS Y BONUS              #
# ========================================== #
FILES_MANDATORY = ft_putchar.c \
                  ft_strlen.c

FILES_BONUS     = ft_lstnew.c \
                  ft_lstadd_front.c

SRCS_MANDATORY  = $(addprefix $(SRC_DIR)/,$(FILES_MANDATORY))
SRCS_BONUS      = $(addprefix $(SRC_DIR)/,$(FILES_BONUS))

OBJS_MANDATORY  = $(SRCS_MANDATORY:.c=.o)
OBJS_BONUS      = $(SRCS_BONUS:.c=.o)

# ========================================== #
# REGLAS                                     #
# ========================================== #
.PHONY: all clean fclean re bonus

all: $(NAME)

# Regla estándar: solo compila la parte obligatoria
$(NAME): $(OBJS_MANDATORY)
	$(AR) $@ $^

# Regla bonus: depende del archivo centinela .bonus
bonus: .bonus

# .bonus depende de los objetos obligatorios Y de los bonus
# Si ninguno ha cambiado, touch no se ejecuta y no hay relink
.bonus: $(OBJS_MANDATORY) $(OBJS_BONUS)
	$(AR) $(NAME) $^
	@touch .bonus

%.o: %.c $(HEADER)
	$(CC) $(CFLAGS) $(INCLUDES) -c $< -o $@

clean:
	$(RM) $(OBJS_MANDATORY) $(OBJS_BONUS) .bonus

fclean: clean
	$(RM) $(NAME)

re: fclean all
```

## Relacionadas

[[La Norma]] · [[Vim]]
