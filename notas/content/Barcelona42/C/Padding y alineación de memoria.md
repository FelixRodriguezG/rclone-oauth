---
tags: [c, memoria, structs, arquitectura-computadores, libft]
aliases: ["Padding", "Relleno de memoria", "Padding bits", "Padding bytes", "Alineación de memoria", "Alineacion de Memoria"]
---
# Padding y alineación de memoria

[[Inicio|← Inicio]] · C

> El **padding** (relleno) son bytes o bits extra, sin uso para tu programa, que el compilador inserta para que cada dato quede en una dirección que el procesador pueda leer bien (**alineación**). Por eso `sizeof` de un struct suele ser mayor que la suma de sus campos, y por eso las funciones de memoria de la libft trabajan siempre con `unsigned char *`.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Alineación** (*alignment*) | Número de bytes del que debe ser múltiplo la dirección de un dato. Se consulta con `_Alignof(tipo)` |
| **Alineación natural** | Caso habitual: la alineación de un tipo coincide con su tamaño (`int` → 4, `double` → 8) |
| **Padding bytes** | Bytes vacíos que el compilador mete **entre campos** de un struct o **al final** |
| **Padding bits** | Bits sin valor **dentro** de un tipo entero. El estándar los permite, pero en x86_64 y ARM no existen |
| **Palabra** (*word*) | Bloque que la CPU lee de una vez: 4 bytes en 32 bits, 8 bytes en 64 bits |
| **`offsetof`** | Macro de `<stddef.h>`: a cuántos bytes del inicio del struct empieza un campo |
| **Struct empaquetado** (*packed*) | Struct sin padding, forzado con una extensión del compilador |

## Por qué existe el padding

### Rendimiento

La CPU no lee la RAM byte a byte, sino en bloques (palabras y, por debajo, líneas de caché de 64 bytes). Si un dato empieza en una dirección múltiplo de su tamaño, nunca queda partido entre dos bloques:

```text
Memoria en palabras de 4 bytes:
        ┌────┬────┬────┬────┐┌────┬────┬────┬────┐
        │ 0  │ 1  │ 2  │ 3  ││ 4  │ 5  │ 6  │ 7  │
        └────┴────┴────┴────┘└────┴────┴────┴────┘
int en 4 (alineado):          └──── 1 lectura ───┘
int en 2 (desalineado): └─ trozo 1 ─┘└─ trozo 2 ─┘   → 2 lecturas + combinar
```

Un `int` desalineado obliga a hacer dos lecturas y juntar los trozos con desplazamientos de bits. En x86_64 el hardware lo hace solo y casi no se nota, salvo si el dato cruza dos líneas de caché. En otras arquitecturas el coste es mucho mayor.

### Arquitecturas que no lo permiten

> [!warning] Bus error / alignment fault
> En ARM antiguos (ARMv5 y anteriores), en muchos microcontroladores y con ciertas instrucciones (atómicas, SIMD), un acceso desalineado provoca una **excepción de hardware**: el programa muere con `Bus error` (señal `SIGBUS`) o el sistema embebido se cuelga. ARM moderno (ARMv7, AArch64) tolera la mayoría de accesos desalineados normales, pero no todos.

Y en cualquier arquitectura, para el estándar de C acceder a un dato a través de un puntero desalineado es **comportamiento indefinido**. UBSan lo detecta con `-fsanitize=alignment` (ver [[Valgrind y sanitizers]]).

### Alineación de cada tipo en tu máquina

```c
printf("char=%zu short=%zu int=%zu long=%zu double=%zu void*=%zu\n",
	_Alignof(char), _Alignof(short), _Alignof(int), _Alignof(long),
	_Alignof(double), _Alignof(void *));
```

```text
char=1 short=2 int=4 long=8 double=8 void*=8
```

| Tipo | Puede empezar en… |
|---|---|
| `char` | Cualquier dirección |
| `short` | Direcciones pares: 0, 2, 4, 6… |
| `int` | Múltiplos de 4: 0, 4, 8, 12… |
| `long`, `double`, punteros | Múltiplos de 8: 0, 8, 16… |

> [!info] Depende de la máquina
> La alineación la decide la implementación, no el estándar. Por ejemplo, en Linux de 32 bits (i386) un `double` ocupa 8 bytes pero solo exige alineación 4. Mide siempre con `_Alignof` y `sizeof`.

## Padding en structs

### Las reglas

1. Los campos se guardan **en el orden en que los declaras**. El compilador no los reordena.
2. **Nunca** hay padding antes del primer campo: la dirección del struct es la de su primer campo.
3. Antes de cada campo se añade el relleno necesario para que empiece en un múltiplo de su alineación.
4. Al final se añade relleno para que el tamaño total sea **múltiplo de la mayor alineación** del struct. Así, en un array de structs, todos los elementos quedan alineados.

### Ejemplo

```c
typedef struct s_datos
{
	char	a;      // 1 byte
	                // ← 3 bytes de padding (b debe empezar en múltiplo de 4)
	int		b;      // 4 bytes
	char	c;      // 1 byte
	                // ← 3 bytes de padding final (12 es múltiplo de 4)
}	t_datos;
```

```text
sizeof=12  offsetof a=0 b=4 c=8
```

```text
t_datos (12 bytes):
 0    1    2    3    4    5    6    7    8    9    10   11
┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐
│ a  │ pad│ pad│ pad│         b         │ c  │ pad│ pad│ pad│
└────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘
```

- Tamaño «teórico»: 1 + 4 + 1 = 6 bytes.
- Tamaño real: **12 bytes**, la mitad es relleno.

¿Por qué el padding final si `c` ya está colocado? Por la regla 4: en `t_datos tab[3]` (36 bytes), `tab[1]` empieza en el byte 12 y su `b` en el 16, múltiplo de 4. Con un tamaño de 9, `tab[1].b` caería en el byte 13.

### Optimizar: ordenar de mayor a menor

Con los mismos campos, ordenados de mayor a menor alineación:

```c
typedef struct s_datos_opt
{
	int		b;      // 4 bytes
	char	a;      // 1 byte
	char	c;      // 1 byte
	                // ← 2 bytes de padding final
}	t_datos_opt;
```

```text
sizeof=8  offsetof b=0 a=4 c=5
```

```text
t_datos_opt (8 bytes):
┌────┬────┬────┬────┬────┬────┬────┬────┐
│         b         │ a  │ c  │ pad│ pad│
└────┴────┴────┴────┴────┴────┴────┴────┘
```

4 bytes menos por struct. Con un millón de nodos en una lista son 4 MB. Con tipos de 8 bytes la diferencia crece:

```c
typedef struct s_mix
{
	char	c;      // 0      + 7 de padding
	double	d;      // 8
	short	s;      // 16     + 6 de padding final
}	t_mix;          // sizeof = 24 (ordenado: double, short, char → 16)
```

> [!tip] Que el compilador te avise
> `-Wpadded` (gcc y clang) avisa de cada hueco. No lo uses con `-Werror`: es solo para inspeccionar.
> ```text
> pad.c:8:7: warning: padding struct 'struct s_datos' with 3 bytes to align 'b' [-Wpadded]
> pad.c:5:16: warning: padding size of 'struct s_datos' with 3 bytes to alignment boundary [-Wpadded]
> ```
> En gdb, `ptype /o t_datos` muestra los offsets y los huecos de cada campo (ver [[Depurar con gdb]]).

## Quitar el padding: structs empaquetados

En drivers, cabeceras de protocolos de red o formatos de fichero binario, el struct tiene que coincidir byte a byte con un formato externo. Ahí se elimina el padding con una extensión de gcc/clang:

```c
typedef struct __attribute__((packed)) s_packed
{
	char	a;
	int		b;
	char	c;
}	t_packed;
```

```text
sizeof=6  align=1  offsetof a=0 b=1 c=5
```

| Ventaja | Inconveniente |
|---|---|
| Cero bytes desperdiciados | `b` queda en el byte 1, desalineado: acceso más lento o fallo en algunas CPU |
| Coincide con formatos externos | No es C estándar: depende del compilador |

Y un peligro menos evidente: un puntero a un campo empaquetado puede estar desalineado. El compilador avisa:

```text
addr.c:9:11: warning: taking address of packed member 'b' of class or structure 's_packed' may result in an unaligned pointer value [-Waddress-of-packed-member]
```

> [!note] En 42
> No lo necesitarás en el Common Core. Si lo usas, justifícalo en la evaluación: la [[La Norma|Norma]] no lo menciona, pero un evaluador puede preguntar.

## Relación con `unsigned char`, `ft_bzero` y `ft_memcmp`

El estándar de C garantiza que **`unsigned char` no tiene bits de relleno**: sus 8 bits representan el valor, y cualquier combinación es válida. Ningún otro tipo lo garantiza (aunque en la práctica `int`, `long`… tampoco los tienen en tu máquina).

Por eso [[ft_memset]], [[ft_bzero]], [[ft_memcpy]], [[ft_memmove]], [[ft_memchr]] y [[ft_memcmp]] convierten el `void *` a `unsigned char *`: es la única forma garantizada de leer y escribir **cada byte real** de un objeto, incluidos los bytes de padding de un struct (ver [[Punteros en C]] y [[Tipos enteros y conversiones]]).

### Por qué no puedes comparar structs con `memcmp`

El contenido de los bytes de padding **no está definido**: suele ser basura de lo que había antes en esa memoria. Dos structs con los mismos campos pueden ser distintos byte a byte:

```c
t_datos	x;
t_datos	y;

memset(&x, 0xAA, sizeof(x));   // simula basura distinta en cada uno
memset(&y, 0x55, sizeof(y));
x.a = 'h'; x.b = 42; x.c = 'z';
y.a = 'h'; y.b = 42; y.c = 'z';
printf("%d\n", memcmp(&x, &y, sizeof(t_datos)) == 0);   // 0: «distintos»
```

```text
memcmp sin limpiar: 0
memcmp tras ft_bzero: 1
```

Si pones el struct entero a cero (con `ft_bzero`, `ft_memset` o reservándolo con [[ft_calloc]]) **antes** de rellenar los campos, el padding queda a 0 y `memcmp` funciona en la práctica. Aun así, lo robusto es comparar campo a campo. Por eso C no permite `x == y` con structs.

> [!warning] Fugas de información
> Copiar un struct a un fichero o enviarlo por red con `write(fd, &s, sizeof(s))` también envía su padding, que puede contener datos antiguos de la memoria (contraseñas, direcciones…). Es una vulnerabilidad real en kernels. Limpia el struct con `ft_bzero` antes de rellenarlo.

## Errores frecuentes

| Error | Consecuencia |
|---|---|
| Calcular el tamaño de un struct sumando campos | Reservas de menos → escrituras fuera del bloque. Usa siempre `sizeof(t_tipo)` |
| Comparar structs con `memcmp` sin limpiarlos antes | Resultados aleatorios por la basura del padding |
| Convertir un `char *` cualquiera a `int *` y leer | Puntero desalineado: UB, `SIGBUS` en algunas CPU |
| Usar `char *` en vez de `unsigned char *` en `ft_memcmp` | Bytes ≥ `0x80` salen negativos y la comparación falla |
| Escribir un struct con padding a un fichero binario | El formato depende del compilador y puede filtrar memoria |

> [!question] Pregunta típica de evaluación
> «¿Por qué `sizeof(t_list)` es 16 y no otra cosa? ¿Y por qué en `ft_memset` conviertes a `unsigned char *`?»
> `t_list` tiene dos punteros de 8 bytes, ya alineados, así que no hay padding: 8 + 8 = 16. En general el tamaño de un struct incluye el relleno de alineación, por eso uso `sizeof` y nunca lo calculo a mano. Convierto a `unsigned char *` porque `void *` no se puede desreferenciar y `unsigned char` es el único tipo que el estándar garantiza sin bits de relleno: accedo a cada byte tal cual, del 0 al 255.

## Relacionadas

[[Structs y typedef]] · [[Punteros en C]] · [[Tipos enteros y conversiones]] · [[Segmentos de memoria en C]] · [[Operadores de bits]] · [[ft_bzero]] · [[ft_memset]] · [[ft_memcmp]] · [[ft_calloc]] · [[Listas enlazadas]] · [[Valgrind y sanitizers]]

## Referencias

- [Objects and alignment — cppreference](https://en.cppreference.com/w/c/language/object) — alineación, representación de objetos y padding.
- [_Alignof operator — cppreference](https://en.cppreference.com/w/c/language/_Alignof) y [offsetof — cppreference](https://en.cppreference.com/w/c/types/offsetof).
- [Struct declaration — cppreference](https://en.cppreference.com/w/c/language/struct) — orden de los campos y relleno.
- [The Lost Art of Structure Packing — Eric S. Raymond](http://www.catb.org/esr/structure-packing/) — la guía clásica sobre padding y cómo reordenar campos.
- [Beej's Guide to C: Structs II](https://beej.us/guide/bgc/html/split/structs-ii-more-fun-with-structs.html) — padding, `offsetof` y por qué no comparar structs con `memcmp`.
- [GCC: Common Type Attributes (`packed`, `aligned`)](https://gcc.gnu.org/onlinedocs/gcc/Common-Type-Attributes.html) y [opciones de aviso (`-Wpadded`)](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html).
- [Data structure alignment — Wikipedia](https://en.wikipedia.org/wiki/Data_structure_alignment) — alineación en distintas arquitecturas.
- [EXP42-C. Do not compare padding data — SEI CERT C](https://wiki.sei.cmu.edu/confluence/display/c/EXP42-C.+Do+not+compare+padding+data) y [DCL39-C. Avoid information leakage when passing a structure across a trust boundary](https://wiki.sei.cmu.edu/confluence/display/c/DCL39-C.+Avoid+information+leakage+when+passing+a+structure+across+a+trust+boundary).
- [N1570 (borrador C11)](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf) — §6.2.6.1 (representación de tipos; los bytes de padding toman valores no especificados), §6.2.6.1p3 y §6.2.6.2p1 (`unsigned char` no tiene bits de relleno), §6.2.8 (alineación), §6.7.2.1p15 y p17 (sin padding al inicio; puede haberlo entre campos y al final).
- Tamaños y avisos medidos con `cc` (Ubuntu clang 12.0.1) en Linux x86_64.
