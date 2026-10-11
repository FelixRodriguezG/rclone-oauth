# Resumen de La Norma (Versión 4) - 42

---

## 1. Límites y Restricciones Estructurales
* **Por archivo `.c`**: Máximo 5 funciones. Nunca incluir archivos `.c` (solo `.h`).
* **Por función**:
  * Máximo **25 líneas** (sin contar las llaves de apertura/cierre de la función).
  * Máximo **4 parámetros** (si no recibe argumentos, usar explícitamente `void`).
  * Máximo **5 variables** declaradas por función.
  * Solo **1 línea vacía** permitida dentro del cuerpo, colocada exclusivamente para separar las declaraciones de variables del código ejecutable.
* **Líneas y columnas**:
  * Máximo **80 columnas** de ancho por línea (incluyendo comentarios).
  * **Una sola instrucción** por línea.
  * Una línea vacía debe estar completamente limpia (sin espacios ni tabulaciones).

---

## 2. Formato e Indentación
* **Tabulaciones**: Indentación obligatoria con caracteres de tabulación reales (equivalentes visualmente a 4 espacios).
* **Espacios**:
  * Prohibidos los espacios o tabulaciones al final de línea (*trailing spaces*).
  * No se permiten dos espacios consecutivos.
  * Espacio obligatorio después de comas `,` y puntos y coma `;` (salvo que sea fin de línea).
  * Espacio alrededor de operadores binarios (`+`, `-`, `=`, `==`, etc.).
  * Espacio obligatorio tras palabras clave de C (`if`, `while`, `return`), excepto en tipos primitivos (`int`, `char`, etc.) y `sizeof`.
* **Punteros**: El asterisco `*` va siempre pegado al nombre de la variable, no al tipo.
* **Retorno**: Los valores de retorno deben ir siempre entre paréntesis: `return (valor);`.
* **Declaración e inicialización**:
  * No declarar e inicializar en la misma línea (salvo variables globales permitidas, estáticas y constantes).
  * Declarar una sola variable por línea.
  * Prohibidas las asignaciones múltiples (`a = b = 0;`).

---

## 3. Nomenclatura y Convenciones
* **Idioma**: Todo en **inglés** y utilizando únicamente caracteres de la tabla **ASCII estándar**.
* **Estilo**: `snake_case` (minúsculas, números y guiones bajos) para nombres de variables, funciones, archivos y directorios.
* **Prefijos obligatorios**:
  * Estructuras: `s_`
  * Typedefs: `t_`
  * Uniones: `u_`
  * Enums: `e_`
  * Variables globales: `g_` (prohibidas salvo que el subject las permita expresamente).
* **Macros**:
  * Nombres completamente en **MAYÚSCULAS**.
  * Solo permitidas para valores literales y constantes. Prohibidas macros multilínea o que ejecuten lógica.

---

## 4. Prohibiciones Absolutas
* Estructuras de control y bucles: `for`, `do ... while`, `switch`, `case`, `goto`.
* Operador ternario (`? :`).
* Arrays de longitud variable (`VLAs`).
* Tipos implícitos en declaraciones.
* Comentarios dentro del cuerpo de las funciones (deben ir fuera o al final de la línea).
* Declarar `struct` dentro de un archivo `.c`.

---