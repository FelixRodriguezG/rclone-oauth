---
tags:
  - c
  - libft
  - printf
  - 42
date: 2026-10-01
---

# `printf` — Formato e Impresión en C

## 1. Concepto General
`printf` (Print Formatted) es una función de la librería estándar `<stdio.h>` utilizada para imprimir texto con formato en la salida estándar (`stdout`).

### Prototipo de la función
```c
int printf(const char *format, ...);
```
[!info] Valor de Retorno `printf` devuelve el **número total de caracteres impresos** en pantalla. Si ocurre un error de escritura, devuelve un valor negativo (`-1`).

## 2. Especificadores de Formato (_Format Specifiers_)

Los especificadores indican cómo debe interpretarse el valor pasado como argumento.

|**Especificador**|**Tipo de Dato Esperado**|**Descripción**|**Ejemplo de Salida**|
|---|---|---|---|
|`%c`|`int` (promocionado desde `char`)|Un único carácter|`A`|
|`%s`|`char *`|Cadena de caracteres (_string_)|`Hola 42`|
|`%p`|`void *`|Dirección de memoria (Puntero en Hexadecimal)|`0x7fff5fbff7c0`|
|`%d`|`int`|Entero con signo en base 10|`-42` / `42`|
|`%i`|`int`|Entero con signo (Base 10 / Base 8 u 16 en `scanf`)|`42`|
|`%u`|`unsigned int`|Entero sin signo en base 10|`4200000000`|
|`%x`|`unsigned int`|Entero en base 16 (Hexadecimal en minúsculas)|`2a`|
|`%X`|`unsigned int`|Entero en base 16 (Hexadecimal en mayúsculas)|`2A`|
|`%%`|N/A|Imprime el símbolo del porcentaje literalmente|`%`|

## 3. Ejemplos Prácticos de Uso

### A. Impresión básica de datos

```c
#include <stdio.h>

int main(void)
{
    char    letra = 'Z';
    char    *nombre = "42 Barcelona";
    int     num = 42;

    printf("Carácter: %c\n", letra);
    printf("String: %s\n", nombre);
    printf("Número: %d\n", num);
    return (0);
}
```

### B. Capturar el valor de retorno

```c
int len;

len = printf("Hola Mundo\n");
printf("Se han impreso %d caracteres.\n", len);
// Salida:
// Hola Mundo
// Se han impreso 11 caracteres. (incluye el '\n')
```

### C. Direcciones de memoria (`%p`)
```c
int var = 10; 
int *ptr = &var;

printf("Dirección de var: %p\n", (void *)&var); 
printf("Dirección en ptr: %p\n", (void *)ptr);
```

### 4. Modificadores de Formato Avanzados _(Ancho y Precisión)_
Puedes controlar el ancho mínimo de campo, alineación y número de decimales entre el `%` y la letra del especificador.

> [!note] Sintaxis General `%[banderas][ancho][.precisión]especificador`

#### Ejemplos de Banderas y Ancho:

- **Ancho mínimo (`%5d`):** Justifica a la derecha rellenando con espacios hasta completar 5 caracteres.
- **Relleno de ceros (`%05d`):** Rellena con ceros a la izquierda (`00042`).
- **Justificado a la izquierda (`%-5d`):** Añade espacios a la derecha.
- **Precisión en cadenas (`%.3s`):** Limita la impresión a los primeros 3 caracteres de la cadena.

```c
printf("[%5d]\n", 42);    // Output: [   42]
printf("[%05d]\n", 42);   // Output: [00042]
printf("[%-5d]\n", 42);   // Output: [42   ]
printf("[%.3s]\n", "Hola"); // Output: [Hol]
```

### 5. Puntos Clave para el Proyecto `ft_printf` en 42

[!warning] Errores Comunes y Casos Límite (_Edge Cases_)

1. **Punteros NULL en `%s`:** En muchas librerías (o según el SO), si pasas `NULL` a un `%s`, debe imprimir `(null)`.
2. **Punteros NULL en `%p`:** Suele imprimir `(nil)` o `0x0` según la implementación del sistema.
3. **Funciones Variádicas:** `ft_printf` utiliza las macros `<stdarg.h>` (`va_start`, `va_arg`, `va_copy`, `va_end`).
4. **Promoción de tipos:** Al usar `va_arg`, los tipos pequeños como `char` o `short` se promocionan automáticamente a `int`.

## Notas Relacionadas en Obsidian

- [[ft_printf_42]]
- [[stdarg.h_variadic_functions]]
- [[libft]]