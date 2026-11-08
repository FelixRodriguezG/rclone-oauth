---
tags: [c, comportamiento-indefinido, ub, estandar, sanitizers]
aliases: ["UB", "Undefined behavior", "Comportamiento no definido"]
---
# Comportamiento indefinido

[[Inicio|← Inicio]] · C

> El estándar de C dice qué hace cada instrucción **en los casos válidos**. Para algunos casos no válidos (desbordar un `int`, leer fuera de un array, usar una variable sin inicializar) dice: «el comportamiento es indefinido». Eso no significa «da un error»: significa que **puede pasar cualquier cosa**, y que lo que pase puede cambiar con el compilador, las opciones de compilación o el día. Un programa con comportamiento indefinido puede funcionar perfectamente en tu ordenador y fallar en la evaluación.

## Vocabulario

| Término | Qué significa | Ejemplo |
|---|---|---|
| **Comportamiento indefinido** (*undefined behavior*, UB) | El estándar no pone **ninguna** condición: crash, basura, que funcione, todo vale | Desbordar un `int` con signo |
| **Comportamiento no especificado** (*unspecified*) | Hay varias opciones válidas y el compilador elige, sin tener que decirte cuál | El orden en que se evalúan los argumentos de `f(a(), b())` |
| **Definido por la implementación** (*implementation-defined*) | El compilador elige **y lo documenta** | `sizeof(long)`: 8 en Linux de 64 bits |

Solo el primero es un fallo de tu código. Los otros dos son legales, pero tu programa no debe depender de una opción concreta.

## Por qué existe

C se diseñó para ser rápido en cualquier procesador. Si el estándar obligara a comprobar cada acceso a un array o cada suma, el código sería más lento. Así que el estándar dice: «no lo hagas; si lo haces, no te prometo nada». A cambio, el compilador puede **suponer que nunca lo haces** y optimizar con esa suposición.

## El ejemplo que lo explica todo

```c
static int	cabe_uno_mas(int x)
{
	return (x + 1 > x);
}
```

¿`x + 1 > x`? Para cualquier número normal, sí. Para `INT_MAX`, `x + 1` desborda. Salida real con el mismo código y dos opciones de compilación:

```text
$ cc -O0 overflow.c && ./a.out          $ cc -O2 overflow.c && ./a.out
cabe_uno_mas(5)       = 1               cabe_uno_mas(5)       = 1
cabe_uno_mas(INT_MAX) = 0               cabe_uno_mas(INT_MAX) = 1
```

- Con `-O0`, el procesador suma, el número da la vuelta a negativo y la comparación sale falsa.
- Con `-O2`, el compilador razona: «desbordar es indefinido, así que no pasa nunca; entonces `x + 1 > x` es siempre cierto» y cambia toda la función por `return (1)`.

**Ninguno de los dos está mal**: el programa no tenía un resultado correcto. Por eso «a mí me funciona» no demuestra nada.

## Los que te vas a encontrar en 42

| Comportamiento indefinido | Ejemplo | Dónde aparece |
|---|---|---|
| Desbordar un entero **con signo** | `INT_MAX + 1`, `-INT_MIN` | `ft_atoi`, `ft_itoa`, ft_printf con `%d` |
| Leer o escribir fuera de un array | `a[3]` en un `int a[3]` | Todos los proyectos |
| Usar una variable sin inicializar | `suma += i;` sin `suma = 0` | Todos |
| Desreferenciar `NULL` | `*p` con `p == NULL` | `ft_strlen(NULL)` en libft |
| Usar memoria ya liberada, o liberarla dos veces | `free(p); free(p);` | get_next_line, listas |
| Modificar una cadena literal | `char *s = "hola"; s[0] = 'H';` | libft |
| `va_arg` con un tipo que no es el que se pasó | `va_arg(args, char)` | ft_printf |
| Desplazar más bits de los que tiene el tipo | `1 << 32` en un `int` | Operadores de bits |
| Funciones con argumentos inválidos | `printf("%d", 3.5)`, `memcpy` con zonas solapadas | libft, ft_printf |

> [!note] Los enteros **sin** signo no desbordan
> `unsigned int` da la vuelta de forma **definida**: `UINT_MAX + 1` es `0`, siempre. Solo los enteros con signo tienen comportamiento indefinido al desbordar. Por eso en ft_printf `%u` con `-1` es seguro y da `4294967295` (ver [[Tipos enteros y conversiones]]).

## Cómo se ve: salidas reales

### `-INT_MIN`

```c
n = INT_MIN;
printf("-n        = %d\n", -n);
printf("-(long)n  = %ld\n", -(long)n);
```

```text
-n        = -2147483648
-(long)n  = 2147483648
```

`-n` «funciona» y devuelve el mismo `INT_MIN` negativo: de ahí los `ft_itoa` que imprimen `--2147483648`. Convertir a `long` **antes** de cambiar el signo lo arregla.

### Variable sin inicializar

```c
int	suma;
int	i;

i = 0;
while (i < 3)
	suma += i++;
printf("suma = %d\n", suma);
```

```text
-O0:  suma = 4198467
-O2:  suma = 1984740264
```

Debería dar `3`. Da lo que hubiera antes en esa zona de memoria, distinto según cómo compiles. Con `-Wall` el compilador avisa: `warning: variable 'suma' is uninitialized when used here`.

### Leer fuera del array

```c
int	a[3];
/* ... a[0], a[1], a[2] ... */
printf("a[3] = %d\n", a[3]);
```

```text
-O0:  a[3] = 0
-O2:  a[3] = -1804195832
```

Con `-O0` sale un `0` que parece correcto. Es el peor caso: el fallo está ahí y no lo ves.

### Modificar una cadena literal

```text
$ ./literal
Segmentation fault (core dumped)
```

Las cadenas literales viven en una zona de solo lectura (ver [[Segmentos de memoria en C]]). En este caso el crash es lo mejor que puede pasar: al menos lo ves.

## Cómo detectarlo

No se puede ver probando a mano: el programa puede dar el resultado esperado. Hacen falta herramientas (detalle en [[Valgrind y sanitizers]]):

| Herramienta | Detecta | Cómo |
|---|---|---|
| `-Wall -Wextra` | Variables sin inicializar, formatos de `printf` mal, algunos desbordamientos de constantes | Al compilar |
| `-fsanitize=undefined` (UBSan) | Desbordamiento con signo, desplazamientos, índices fuera de rango de arrays conocidos | Al ejecutar |
| `-fsanitize=address` (ASan) | Accesos fuera de memoria, uso tras `free`, doble `free` | Al ejecutar |
| Valgrind | Lecturas de memoria sin inicializar, accesos inválidos al heap, leaks | Al ejecutar, sin recompilar |

Salida real de UBSan con los ejemplos de arriba (`cc -g -fsanitize=undefined`):

```text
overflow.c:6:12: runtime error: signed integer overflow: 2147483647 + 1 cannot be represented in type 'int'
neg.c:9:29: runtime error: negation of -2147483648 cannot be represented in type 'int'; cast to an unsigned type to negate this value to itself
fuera.c:14:24: runtime error: index 3 out of bounds for type 'int [3]'
```

Y ASan con la lectura fuera del array:

```text
==3278989==ERROR: AddressSanitizer: stack-buffer-overflow on address 0x7ffd6070e64c ...
READ of size 4 at 0x7ffd6070e64c thread T0
```

> [!tip] Prueba con `-O2` además de con `-O0`
> Muchos casos de comportamiento indefinido solo cambian el resultado al optimizar. Si tu proyecto da resultados distintos con `-O0` y con `-O2`, casi seguro tienes uno.

## Comportamiento indefinido y la evaluación

Los subjects de 42 dicen: *«Tus funciones no deben terminar de forma inesperada (segfault, bus error, double free, etc) **excepto en el caso de comportamientos indefinidos**.»*

- **Del lado de quien llama**: si el estándar dice que `strlen(NULL)` es indefinido, tu `ft_strlen(NULL)` puede crashear y no es un fallo. Igual con ft_printf y las combinaciones de banderas que el estándar no define (`%#d`, `%05s`).
- **Del lado de tu código**: si **tu** código tiene comportamiento indefinido con entradas válidas (`ft_itoa(INT_MIN)`), sí es un fallo, aunque te funcione.
- **En la evaluación**: si un evaluador dice que algo crashea y tú dices que es comportamiento indefinido, tienes que poder decir **qué regla** del estándar lo hace indefinido (por ejemplo, la página de `man` o cppreference).

## Errores frecuentes

> [!danger] Lo que más se oye
> - «Me funciona, así que está bien»: el comportamiento indefinido puede funcionar hasta el día de la evaluación.
> - Pensar que desbordar un `int` da siempre la vuelta a negativo: con `-O2` puede no pasar.
> - Confundir **sin signo** (desborda de forma definida) con **con signo** (indefinido).
> - Llamar «comportamiento indefinido» a cualquier crash propio para no arreglarlo.

## Relacionadas

[[Tipos enteros y conversiones]] · [[Valgrind y sanitizers]] · [[Arrays en C]] · [[Punteros en C]] · [[Carácter nulo y puntero NULL]] · [[malloc y free]] · [[Segmentos de memoria en C]] · [[stdarg y funciones variádicas]] · [[Operadores de bits]] · [[Cómo compila C]] · [[ft_printf - Guía paso a paso]]

## Referencias

- [cppreference: Undefined behavior](https://en.cppreference.com/w/c/language/behavior): las tres categorías y ejemplos, incluido el del desbordamiento optimizado.
- [What Every C Programmer Should Know About Undefined Behavior (LLVM blog)](https://blog.llvm.org/2011/05/what-every-c-programmer-should-know.html): por qué el compilador se aprovecha del comportamiento indefinido para optimizar.
- [Clang UndefinedBehaviorSanitizer](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html).
- Subjects de 42 (libft, get_next_line, ft_printf): «excepto en el caso de comportamientos indefinidos».
- Salidas reales con Ubuntu clang 12.0.1 en Ubuntu 22.04. Los valores basura cambian de una ejecución o máquina a otra.
