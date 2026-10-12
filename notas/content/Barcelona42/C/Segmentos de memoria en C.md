---
date: 2026-10-04
tags:
  - c
  - sistemas
  - memoria
  - punteros
  - 42barcelona
  - cursus
  - low-level
---
# Segmentos de memoria en C

[[Inicio|← Inicio]] · C

> [!summary] Resumen
> El espacio de direcciones virtual de un proceso en C se divide en **5 segmentos principales**, más la zona de argumentos y entorno en la parte alta:
> 1. **Argumentos / Entorno (`argc`, `argv`, `envp`):** Cúspide del espacio de direcciones del usuario.
> 2. **STACK (Pila):** Memoria LIFO de alta velocidad para marcos de función, variables locales y parámetros. Crece hacia abajo.
> 3. **HEAP (Montículo):** Memoria dinámica administrada manualmente (`malloc`/`free`). Crece hacia arriba.
> 4. **BSS:** Variables globales/estáticas **NO inicializadas** (o igualadas a cero). El SO las inicializa en 0.
> 5. **DATA:** Variables globales/estáticas **inicializadas** con un valor distinto de cero.
> 6. **TEXT (Code) y RODATA:** Código máquina ejecutable y datos de solo lectura, como los literales de cadena (*string literals*).

---

## 1. Mapa de Memoria Virtual (Layout General)

Representación gráfica de la disposición de la memoria de un proceso de 32/64 bits en arquitectura x86/x64:

```text
  +-----------------------------------------+  Direcciones Altas (p. ej. 0x7FFF... / 0xFFFF...)
  | Argumentos de Entorno y del Sistema     |  envp, argv, argc
  +-----------------------------------------+
  |                  STACK                  |  <-- Crece hacia abajo (Direcciones Menores)
  |                    |                    |      Estructura LIFO (Frame Pointer / Stack Pointer)
  |                    v                    |
  |                                         |
  |       mmap / librerías compartidas      |  <-- libc.so, malloc grandes, etc.
  |                                         |
  |                    ^                    |
  |                    |                    |
  |                  HEAP                   |  <-- Crece hacia arriba (Direcciones Mayores)
  |                                         |      Asignado vía brk / sbrk / mmap
  +-----------------------------------------+
  |                  BSS                    |  <-- Block Started by Symbol
  |                                         |      Variables globales/estáticas NO inicializadas
  +-----------------------------------------+
  |                  DATA                   |  <-- Variables globales/estáticas inicializadas
  |                                         |      (Valores predeterminados guardados en el binario)
  +-----------------------------------------+
  |                  TEXT                   |  <-- Instrucciones de Código Máquina (Solo Lectura)
  |                                         |      Literales de cadena (String Literals)
  +-----------------------------------------+  Direcciones Bajas (p. ej. 0x0040... / 0x0804...)
```

---

## 2. Descripción Detallada Segmento por Segmento

### 2.1. Argumentos de Entorno y Sistema
- **Contenido:** Variables de entorno del sistema (`envp`), argumentos pasados al programa desde la línea de comandos (`argc`, `argv`).
- **Ubicación:** En la parte más alta de la memoria de espacio de usuario.
- **Acceso:** Lectura (y modificación de punteros `argv`/`envp` dentro del proceso).

### 2.2. Segmento Stack (Pila de Llamadas)
- **Propósito:** Almacena los llamados *Stack Frames* (marcos de pila). Cada frame contiene:
  - Variables locales (automáticas).
  - Parámetros pasados a las funciones.
  - Dirección de retorno (*Return Address*) para saber a dónde volver tras ejecutar `return`.
  - Registros guardados de la CPU (`EBP`/`RBP`, etc.).
- **Comportamiento:** Estructura **LIFO** (Last In, First Out). La memoria se reserva y libera de forma autogestionada al entrar/salir del ámbito (`scope`).
- **Dirección de Crecimiento:** Hacia direcciones de memoria **más bajas** (hacia abajo).
- **Ejemplo en C:**
  ```c
  void funcion(int param) {    // 'param' vive en el Stack Frame de funcion()
      int local = 42;          // 'local' vive en el Stack Frame de funcion()
  }                            // Al salir, el Stack Pointer se ajusta y se liberan
  ```
- **Riesgo:** *Stack overflow*, por recursión infinita o [[Arrays en C|arrays]] gigantes en el Stack como `int array[1000000]` (ver la sección 6.B).

### 2.3. Segmento Heap (Memoria Dinámica)
- **Propósito:** Almacenamiento de variables y estructuras cuyo tamaño o duración no se conocen en tiempo de compilación.
- **Gestión:** **100% Manual.** El programador solicita memoria explícitamente (`malloc`, `calloc`, `realloc`) y la devuelve (`free`).
- **Dirección de Crecimiento:** Hacia direcciones de memoria **más altas** (hacia arriba, hacia el Stack).
- **Ejemplo en C:**
  ```c
  int *ptr = (int *)malloc(sizeof(int) * 100); // 400 bytes en el Heap
  if (ptr != NULL) {
      ptr[0] = 10;
      free(ptr);                               // Liberación obligatoria
      ptr = NULL;                              // Buenísimo evitar dangling pointers
  }
  ```
- **Riesgo:** fugas de memoria (*memory leaks*), *use-after-free*, *double free* (ver la sección 6).

### 2.4. Segmento BSS (*Block Started by Symbol*)
- **Propósito:** Variables globales y variables estáticas (`static`) que **NO han sido inicializadas explícitamente** por el programador (o inicializadas a cero).
- **Características:** No ocupan espacio real dentro del archivo ejecutable binario en disco (solo una anotación de cuánto espacio se necesitará). Cuando el SO carga el ejecutable en RAM, el segmento BSS se rellena con ceros (`0` o `NULL`).
- **Ejemplo en C:**
  ```c
  int g_uninit_global;        // Reside en BSS (vale 0 por defecto)
  static char s_buffer[2048]; // Reside en BSS (lleno de '\0')
  ```

### 2.5. Segmento Data (Initialized Data Segment)
- **Propósito:** Variables globales y variables estáticas (`static`) que **SÍ han sido inicializadas explícitamente** con un valor distinto de cero.
- **Características:** Ocupan espacio directo dentro del archivo ejecutable compilado en disco, ya que sus valores iniciales deben conservarse para ser cargados en la memoria RAM al arrancar.
- **Ejemplo en C:**
  ```c
  int g_init_global = 100;         // Reside en DATA
  static float s_pi = 3.14159f;    // Reside en DATA
  ```

### 2.6. Segmento Text (Code Segment)
- **Propósito:** Contiene el código en instrucciones máquina compiladas que ejecuta el procesador. Junto a él está la sección **`.rodata`** (*read-only data*), donde van los literales de cadena (*string literals*) y las constantes globales. Técnicamente `.rodata` no es TEXT, pero también es de solo lectura y en la práctica se agrupan.
- **Permisos:** **Read-Only** (Solo lectura) y **Executable** (Ejecutable).
- **Protección:** Cualquier intento de escribir en este segmento provocará una interrupción por fallo de protección de memoria (**Segmentation Fault / SIGSEGV**).
- **Ejemplo en C:**
  ```c
  char *str = "Hola 42"; // "Hola 42" es una constante en .rodata (solo lectura).
                         // 'str' es un puntero en el Stack/Data apuntando a TEXT.
  ```

---

## 3. Matriz Comparativa y Modificadores de Almacenamiento

### 3.1. Tabla General de Segmentos

| Segmento | Asignación | Liberación | Ciclo de Vida | Permisos | Crece Hacia |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **TEXT** | Compilación | Fin del Proceso | Total del Programa | `R-X` (Read/Exec) | Fijo |
| **DATA** | Compilación | Fin del Proceso | Total del Programa | `RW-` (Read/Write) | Fijo |
| **BSS** | Carga del Proceso | Fin del Proceso | Total del Programa | `RW-` (Read/Write) | Fijo |
| **HEAP** | Ejecución (`malloc`) | Manual (`free`) | Hasta `free()` o Fin | `RW-` (Read/Write) | Direcciones Altas ($\uparrow$) |
| **STACK** | Entrada a Función | Salida de Función | Ámbito Local (`Scope`) | `RW-` (Read/Write) | Direcciones Bajas ($\downarrow$) |

### 3.2. Relación con Especificadores de Almacenamiento en C (*Storage Classes*)

- **`static` (Local):** Cambia la ubicación de la variable local del **Stack** al segmento **DATA** (si tiene valor inicial != 0) o **BSS** (si no). Preserva su valor entre llamadas a la función.
- **`static` (Global):** Mantiene la variable en **DATA/BSS**, pero restringe su visibilidad únicamente al archivo fuente actual (*Translation Unit*).
- **`const`:** 
  - `const` global inicializada: Suele almacenarse en una sección de solo lectura dentro de **TEXT** (`.rodata`).
  - `const` local: Vive en el **Stack**, impidiendo su modificación a nivel de compilador.
- **`register`:** Sugiere al compilador guardar la variable en un **registro de la CPU** (como `EAX`/`RAX`) en lugar del Stack. Los compiladores modernos lo ignoran y deciden solos; lo único que garantiza es que no puedes obtener su dirección (`&var`).
- **`extern`:** Indica que la variable reside en **DATA/BSS** pero está definida en otro archivo objeto.

---

## 4. Ejemplo Extenso de Código C Comentado Paso a Paso

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// -------------------------------------------------------------
// 1. SEGMENTO DATA (Globales inicializadas con valor != 0)
// -------------------------------------------------------------
int g_global_inicializada = 42; 
static int g_static_inicializada = 100;

// -------------------------------------------------------------
// 2. SEGMENTO BSS (Globales NO inicializadas o igualadas a 0)
// -------------------------------------------------------------
int g_global_no_inicializada; 
static double g_static_bss;

// -------------------------------------------------------------
// 3. SEGMENTO TEXT (Constante global en sección de solo lectura)
// -------------------------------------------------------------
const char g_constante_rodata[] = "CONSTANTE_GLOBAL";

void funcion_demo(int param_stack) {
    // ---------------------------------------------------------
    // 4. SEGMENTO STACK (Variables locales / automáticas)
    // ---------------------------------------------------------
    int local_stack = 10;
    char buffer_stack[64];

    // ---------------------------------------------------------
    // 5. SEGMENTO BSS/DATA (Estática local)
    // ---------------------------------------------------------
    // No vive en el Stack. Retiene su valor entre ejecuciones.
    static int contador_estatico_bss; 
    static int contador_estatico_data = 1;

    // ---------------------------------------------------------
    // 6. SEGMENTO TEXT & STACK (Puntero en Stack apuntando a TEXT)
    // ---------------------------------------------------------
    // 'ptr_literal' está en el STACK.
    // "Literal de Cadena" reside en el segmento TEXT (read-only).
    char *ptr_literal = "Mensaje en Text Segment";

    // ---------------------------------------------------------
    // 7. SEGMENTO HEAP (Asignación Dinámica)
    // ---------------------------------------------------------
    // 'ptr_heap' vive en el STACK.
    // La memoria reservada por malloc (20 bytes) reside en el HEAP.
    int *ptr_heap = (int *)malloc(sizeof(int) * 5);

    if (ptr_heap == NULL) {
        perror("Error asignando memoria en Heap");
        return;
    }

    for (int i = 0; i < 5; i++) {
        ptr_heap[i] = (i + 1) * 10;
    }

    // Liberación adecuada del HEAP
    free(ptr_heap);
    ptr_heap = NULL; // Evita Puntero Colgante (Dangling Pointer)
}

int main(int argc, char **argv) { // argc y argv en Argumentos/STACK
    funcion_demo(argc);
    return 0;
}
```

---

## 5. Herramientas de Inspección y Depuración

### 5.1. El Comando `size` (Binarios ELF)
Muestra la distribución exacta de bytes asignada en los segmentos fijos del compilado:

```bash
# Compilación del programa
gcc -Wall -Wextra -Werror main.c -o programa_demo

# Ejecución de size
size programa_demo
```

**Análisis del Output:**
```text
   text    data     bss     dec     hex filename
   1724     552      16    2292     8f4 programa_demo
```
- `text`: Tamaño del código ejecutable y constantes de lectura.
- `data`: Tamaño de variables globales/estáticas inicializadas.
- `bss`: Espacio reservado para globales/estáticas no inicializadas.
- `dec` / `hex`: Suma total de los segmentos fijos en decimal y hexadecimal.

### 5.2. Inspección de Símbolos con `nm` y `objdump`
Para ver exactamente a qué sección se envió cada variable o función:

```bash
nm programa_demo | grep -E "g_global|g_static"
```
**Tipos de Símbolos:**
- `T` / `t`: Se encuentra en el segmento **TEXT**.
- `D` / `d`: Se encuentra en el segmento **DATA**.
- `B` / `b`: Se encuentra en el segmento **BSS**.
- `R` / `r`: Se encuentra en **Read-Only Data** (`TEXT`).

### 5.3. Detección de Fugas e Incidencias con `valgrind`
`Valgrind` simula un entorno sintético para rastrear cada lectura, escritura y asignación en el Heap/Stack.

```bash
# Comando completo recomendado para desarrollo en 42
valgrind --leak-check=full --show-leak-kinds=all --track-origins=yes ./programa_demo
```

---

## 6. Errores de Memoria Frecuentes y sus Causas

### A. Segmentation Fault (SIGSEGV)
Ocurre cuando el proceso intenta acceder a una dirección de memoria no asignada o viola los permisos del segmento:
- **Escritura en TEXT:**
  ```c
  char *str = "Hola";
  str[0] = 'h'; // Error: TEXT es Read-Only
  ```
- **Desreferencia de Puntero NULL o Invalido:**
  ```c
  int *p = NULL;
  *p = 5; // Error: Puntero nulo
  ```

### B. Stack Overflow
Se agota el espacio límite reservado para la pila del sistema (usualmente 8 MB en Linux por defecto):
- **Recursión Infinita:**
  ```c
  void funcion_infinita(void) {
      funcion_infinita(); // Consume frames de Stack sin parar
  }
  ```
- **Reservas Locales Excesivas:**
  ```c
  void matriz_gigante(void) {
      char buffer[10 * 1024 * 1024]; // 10MB supera el límite típico del Stack
  }
  ```

### C. Fuga de Memoria (*Memory Leak*)
Asignar espacio en el Heap con `malloc` y perder la referencia al puntero sin llamar a `free()`:
```c
void leak_ejemplo(void) {
    int *p = malloc(100);
    // Falta free(p);
} // Al terminar la función, 'p' se destruye del Stack, perdiendo la memoria del Heap para siempre.
```

### D. Use-After-Free y Double Free
- **Use-After-Free:** Acceder a la memoria del Heap después de haber liberado la zona con `free()`.
- **Double Free:** Llamar dos veces a `free()` sobre la misma dirección de memoria asignada sin haber cambiado la dirección a `NULL`.

---

## 7. Relacionadas

- [[Carácter nulo y puntero NULL]]: `'\0'` termina cadenas, `NULL` indica «sin memoria» o fin de un array de punteros.
- [[Arrays en C]]: arrays locales en el Stack, literales de solo lectura.
- [[libft - Anexo comparativo]]: qué funciones de libft reservan memoria en el Heap.
- [[ft_calloc]] · [[ft_strdup]] · [[ft_split]]: ejemplos de reserva y liberación en el Heap.
