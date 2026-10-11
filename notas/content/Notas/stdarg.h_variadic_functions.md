---
tags:
  - c
  - stdarg
  - variadic-functions
  - ft_printf
date: 2026-10-01
---

# `stdarg.h` y Funciones Variádicas

## 1. ¿Qué es una Función Variádica?
Una **función variádica** (*variadic function*) es una función en C que puede aceptar un **número indeterminado de argumentos** de entrada. 

El ejemplo más común en la librería estándar es `printf`, la cual puede recibir desde un solo argumento hasta decenas de ellos separados por comas.

### Prototipo en C
Se indica mediante tres puntos suspensivos (`...`) al final de la lista de parámetros.

```c
int ft_printf(const char *format, ...);
````

> [!danger] Reglas de Sintaxis

> 1. Debe existir **al menos un parámetro fijo** antes de los puntos suspensivos (`...`).
> 2. Los tres puntos deben colocarse **siempre al final** de la lista de parámetros.

## 2. Macros Principales de `<stdarg.h>`

Para poder acceder a los argumentos variables ocultos tras los tres puntos, la cabecera `<stdarg.h>` proporciona un tipo de dato especial y cuatro macros obligatorias:

|**Macro / Tipo**|**Descripción**|
|---|---|
|`va_list`|Tipo de variable que almacena el puntero/estado de la lista de argumentos.|
|`va_start(args, ultimo_param_fijo)`|Inicializa la lista `va_list` situándose justo después del último parámetro conocido.|
|`va_arg(args, TIPO)`|Extrae el **siguiente argumento** de la lista interpretándolo como el `TIPO` especificado y avanza el puntero.|
|`va_end(args)`|Limpia y destruye la variable `va_list` al finalizar el acceso.|
|`va_copy(dest, src)`|_(Opcional)_ Copia el estado actual de una lista `va_list` en otra.|

3. Ejemplo Mínimo Explicado

Este programa calcula la suma de N enteros recibidos como argumentos variádicos:
```c

#include <stdio.h>
#include <stdarg.h>

int sumar_todo(int count, ...)
{
    va_list args;       // 1. Declarar el puntero de argumentos
    int     total;
    int     i;

    total = 0;
    i = 0;
    
    va_start(args, count); // 2. Inicializar la lista usando 'count' como referencia
    
    while (i < count)
    {
        // 3. Extraer cada argumento indicando su tipo de dato (int)
        total += va_arg(args, int);
        i++;
    }
    
    va_end(args); // 4. Limpiar la memoria de va_list
    return (total);
}

int main(void)
{
    printf("Suma 1: %d\n", sumar_todo(3, 10, 20, 30));      // Resultado: 60
    printf("Suma 2: %d\n", sumar_todo(5, 1, 2, 3, 4, 5));   // Resultado: 15
    return (0);
}
```

4. Promoción por Defecto de Tipos (Default Argument Promotions)

Al pasar argumentos a través de ..., el compilador de C aplica una promoción de tipos automática antes de que los reciba la función:

    [!warning] Regla de Promoción

        - char y short son promocionados automáticamente a int.

        - float es promocionado automáticamente a double.

¿Qué significa esto para va_arg?

Nunca debes solicitar un char, short o float directamente en va_arg, ya que provocará un comportamiento indeterminado (undefined behavior).
```c

// INCORRECTO:
char c = va_arg(args, char);

// CORRECTO:
char c = (char)va_arg(args, int);
```

5. Relación Directa con ft_printf

En tu proyecto ft_printf de 42, evaluarás el string de formato caracter por caracter. Cuando encuentres un %, llamarás a va_arg con el tipo adecuado según la conversión:

- %c -> va_arg(args, int)
-   %s -> va_arg(args, char *)
-  %d / %i -> va_arg(args, int)
-    %u / %x / %X -> va_arg(args, unsigned int)
-   %p -> va_arg(args, void *)

Enlaces Relacionados

- [[printf_obsidian_note]]
-   [[ft_printf_42]]
-   [[libft]]