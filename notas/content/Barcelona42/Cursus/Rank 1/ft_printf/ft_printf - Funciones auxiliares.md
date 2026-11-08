---
tags: [proyecto, ft_printf, utils, rank-1, principiante, bonus]
aliases: ["Utils de ft_printf", "Helpers de ft_printf", "ft_out_field"]
---
# ft_printf - Funciones auxiliares

[[Inicio|← Inicio]] · [[ft_printf - Índice|ft_printf]] · [[ft_printf - Guía paso a paso|Guía]] · Funciones auxiliares

> Las piezas pequeñas sobre las que se construye ft_printf con el diseño de la [[ft_printf - Guía paso a paso|guía]]: las que escriben (`ft_out_*`), la que coloca un campo con espacios y ceros (`ft_out_field`) y la que pasa un número a texto en cualquier base (`ft_utoa_base`). Esta nota explica cada una, por qué es así y no de otra forma, y cómo probarla sola.

> [!info] Antes de leer
> Esta nota acompaña a las etapas 1, 2 y 5 de [[ft_printf - Guía paso a paso#8. Construirlo por etapas|la guía]]. Conviene tener claro qué es una `struct` y la diferencia entre `.` y `->` ([[Structs y typedef]]), y cómo se divide y se saca el resto de un número sin signo ([[Tipos enteros y conversiones]]).

## Resumen

| Función | Fichero | Qué hace |
|---|---|---|
| `ft_out_char` | `ft_output.c` | Escribe 1 byte, cuenta y anota si `write` falla |
| `ft_out_nstr` | `ft_output.c` | Escribe los `n` primeros bytes de una cadena |
| `ft_out_repeat` | `ft_output.c` (`static`) | Escribe `n` veces el mismo carácter; nada si `n <= 0` |
| `ft_out_field` | `ft_output.c` | Escribe un campo completo: espacios, prefijo, ceros, cuerpo, espacios |
| `ft_utoa_base` | `ft_print_nums.c` (`static`) | Pasa un `unsigned long` a texto en la base que le des, sin `malloc` |
| `ft_base` | `ft_print_nums.c` (`static`) | Elige la base según la letra (`x`/`p` → hex minúscula, `X` → mayúscula, resto → decimal) |

Solo `ft_output.c` llama a `write`. Es una regla de diseño: si mañana quieres escribir en otro fd, lo cambias en **dos líneas**.

## Qué sirve de tu libft y qué no

En ft_printf **sí** puedes usar libft. Pero no todo vale:

| Función de libft | ¿Sirve? | Por qué |
|---|---|---|
| `ft_strlen` | ✅ | Longitud del prefijo y de `%s` |
| `ft_strchr` | ✅ | ¿Es este carácter una bandera? `ft_strchr("-0# +", c)` (con cuidado, ver abajo) |
| `ft_isdigit` | ✅ | Leer el ancho y la precisión |
| `ft_putchar_fd`, `ft_putstr_fd` | ❌ | Devuelven `void`: no sabes si `write` falló ni cuánto escribió |
| `ft_putnbr_fd` | ❌ | Escribe directamente: no puedes poner ceros entre el signo y los dígitos, ni saber la longitud antes |
| `ft_itoa` | ❌ | Usa `malloc` (tienes que liberar y gestionar el fallo) y solo base 10 |

> [!warning] `ft_strchr` encuentra el `'\0'`
> Como la `strchr` original, `ft_strchr("-0# +", '\0')` **no** devuelve `NULL`: devuelve la dirección del `'\0'` final de `"-0# +"`. Por eso en el bucle de banderas se comprueba antes `s[*i] &&`. Si no, un `%` al final del formato se tomaría por bandera.

## `ft_out_char` y `ft_out_nstr`: escribir y contar

> ```c
> void	ft_out_char(t_out *o, char c)
> {
> 	if (o->error)
> 		return ;
> 	if (write(1, &c, 1) == -1)
> 		o->error = 1;
> 	else
> 		o->len++;
> }
>
> void	ft_out_nstr(t_out *o, const char *s, int n)
> {
> 	if (o->error || n <= 0)
> 		return ;
> 	if (write(1, s, n) == -1)
> 		o->error = 1;
> 	else
> 		o->len += n;
> }
> ```

Las dos hacen lo mismo con 1 o con `n` bytes:

1. **Si ya hubo un error, no hacen nada.** Una vez que `write` ha fallado, el resultado de `ft_printf` es `-1` pase lo que pase: no tiene sentido seguir escribiendo.
2. **Si `write` devuelve `-1`**, se marca `o->error = 1`.
3. **Si no**, se suma a `o->len` lo escrito.

### Por qué una bandera de error en vez de devolver `-1`

Compara las dos formas de propagar un error a través de tres niveles de funciones:

```c
/* Devolviendo -1: cada llamada se comprueba */
ret = ft_print_prefix(...);
if (ret == -1)
	return (-1);
count += ret;
ret = ft_print_zeros(...);
if (ret == -1)
	return (-1);
count += ret;
/* ... y así en cada función ... */
```

```c
/* Con t_out: nadie comprueba nada */
ft_out_nstr(o, fl->prefix, prefix_len);
ft_out_repeat(o, '0', fl->zeros);
```

Con la segunda, la comprobación se hace **una sola vez**, al final de `ft_printf`. Las funciones quedan cortas y es imposible olvidar propagar un `-1`.

> [!note] `write` puede escribir menos de lo pedido
> En teoría `write(1, s, n)` puede devolver un número positivo menor que `n` (escritura parcial). Con la salida estándar en una terminal o un fichero no pasa en la práctica, y la `printf` de 42 no lo tiene en cuenta. Si un evaluador lo pregunta, saber que existe ya es una buena respuesta (ver [[Control de errores en C]]).

## `ft_out_repeat`: el relleno

> ```c
> static void	ft_out_repeat(t_out *o, char c, int n)
> {
> 	while (n-- > 0)
> 		ft_out_char(o, c);
> }
> ```

- `while (n-- > 0)`: si `n` es `0` o **negativo**, no entra nunca. Importa porque el número de espacios de un campo puede salir negativo (`%1s` con `"hola"`: `1 - 4 = -3`).
- Escribe byte a byte. Es lento si alguien pide `%1000000d`, pero es correcto y simple. Una mejora posible es escribir bloques desde un array de espacios, pero no hace falta.
- `static`: solo la usa `ft_out_field`.

## `ft_out_field`: el corazón del proyecto

> ```c
> void	ft_out_field(t_out *o, t_flags *f, t_field *fl)
> {
> 	int	prefix_len;
> 	int	pad;
>
> 	prefix_len = ft_strlen(fl->prefix);
> 	pad = f->width - prefix_len - fl->zeros - fl->body_len;
> 	if (f->zero && pad > 0)
> 	{
> 		fl->zeros += pad;
> 		pad = 0;
> 	}
> 	if (!f->minus)
> 		ft_out_repeat(o, ' ', pad);
> 	ft_out_nstr(o, fl->prefix, prefix_len);
> 	ft_out_repeat(o, '0', fl->zeros);
> 	ft_out_nstr(o, fl->body, fl->body_len);
> 	if (f->minus)
> 		ft_out_repeat(o, ' ', pad);
> }
> ```

Recibe **qué** imprimir (`t_field`) y **cómo** (`t_flags`), y lo coloca en este orden:

```text
 [espacios si NO '-'] [prefijo] [ceros] [cuerpo] [espacios si '-']
```

### Paso a paso con `%08d` y `-42`

```text
llega:   f->width = 8, f->zero = 1, f->minus = 0
         fl->prefix = "-", fl->body = "42", fl->body_len = 2, fl->zeros = 0

prefix_len = 1
pad = 8 - 1 - 0 - 2 = 5
f->zero y pad > 0  →  zeros = 0 + 5 = 5,  pad = 0
!f->minus          →  0 espacios
prefijo            →  "-"
ceros              →  "00000"
cuerpo             →  "42"
                       ─────────
resultado:             "-0000042"   (8 caracteres)
```

### Lo que esta función NO decide

`ft_out_field` no sabe si está imprimiendo un `%s` o un `%d`. Por eso no decide cuándo **no** aplicar la bandera `0`. Lo deciden las funciones que la llaman, poniendo `f->zero = 0` antes:

| Quién | Por qué anula el `0` |
|---|---|
| `ft_set_flags` (bonus) | `-` gana a `0` |
| `ft_print_number` | Con precisión, el `0` se ignora en números |
| `ft_print_char`, `ft_print_str` | glibc ignora el `0` en `%c` y `%s` |
| `ft_print_ptr` con `NULL` | `(nil)` no se rellena con ceros |

Así cada regla vive en un solo sitio y `ft_out_field` se queda en 17 líneas.

> [!tip] Pruébala sola
> No necesitas ft_printf entero para probar `ft_out_field`. Construye tú las piezas:
> ```c
> #include <stdio.h>
> #include "ft_printf.h"
>
> static void	campo(const char *prefix, const char *body, int zeros, t_flags f)
> {
> 	t_out	o;
> 	t_field	fl;
>
> 	o.len = 0;
> 	o.error = 0;
> 	fl.prefix = prefix;
> 	fl.body = body;
> 	fl.body_len = ft_strlen(body);
> 	fl.zeros = zeros;
> 	write(1, "[", 1);
> 	ft_out_field(&o, &f, &fl);
> 	printf("]  len=%d\n", o.len);
> 	fflush(stdout);
> }
>
> int	main(void)
> {
> 	t_flags	f;
>
> 	f = ft_init_flags();
> 	campo("-", "42", 0, f);     /* sin banderas        */
> 	f.width = 8;
> 	campo("-", "42", 0, f);     /* ancho 8             */
> 	f.minus = 1;
> 	campo("-", "42", 0, f);     /* ancho 8 con '-'     */
> 	f.minus = 0;
> 	f.zero = 1;
> 	campo("-", "42", 0, f);     /* ancho 8 con '0'     */
> 	f.zero = 0;
> 	campo("-", "42", 1, f);     /* ancho 8, precisión 3: 1 cero */
> 	f.width = 1;
> 	campo("", "hola", 0, f);    /* ancho menor que el cuerpo */
> 	return (0);
> }
> ```
> Salida real:
> ```text
> [-42]  len=3
> [     -42]  len=8
> [-42     ]  len=8
> [-0000042]  len=8
> [    -042]  len=8
> [hola]  len=4
> ```
> El `fflush(stdout)` es necesario porque `printf` usa buffer y `ft_out_field` escribe directamente: sin él, los corchetes y los `len=` saldrían desordenados (ver [[ft_printf - Pruebas y depuración#2. No mezcles printf y ft_printf en la terminal|Pruebas]]).

## `ft_utoa_base`: número a texto en cualquier base

> ```c
> static int	ft_utoa_base(unsigned long n, const char *base, char *buf)
> {
> 	unsigned long	b;
> 	unsigned long	tmp;
> 	int				len;
> 	int				i;
>
> 	b = ft_strlen(base);
> 	tmp = n;
> 	len = 1;
> 	while (tmp >= b)
> 	{
> 		tmp /= b;
> 		len++;
> 	}
> 	i = len;
> 	while (i-- > 0)
> 	{
> 		buf[i] = base[n % b];
> 		n /= b;
> 	}
> 	return (len);
> }
> ```

Es el método de las divisiones sucesivas de [[Bases numéricas]], escrito sin recursión. Dos pasadas:

1. **Contar cifras.** Divide una copia (`tmp`) entre la base hasta que sea menor que la base. Empieza en `len = 1` porque el `0` tiene una cifra.
2. **Rellenar de derecha a izquierda.** La última cifra es `n % b`; luego `n /= b` y a por la anterior. Como sabes `len`, empiezas en `buf[len - 1]` y no hace falta dar la vuelta a la cadena después.

```text
n = 3054, base "0123456789ABCDEF" (b = 16)

contar:   3054 → 190 → 11  (11 < 16, para)   len = 3

rellenar:  i = 2: 3054 % 16 = 14 → 'E'   n = 190
           i = 1:  190 % 16 = 14 → 'E'   n = 11
           i = 0:   11 % 16 = 11 → 'B'   n = 0

buf = [ B | E | E ]   → "BEE"   (printf("%X", 3054) da BEE)
```

### Por qué así y no de otra forma

| Decisión | Alternativa | Por qué no |
|---|---|---|
| `unsigned long` | `int` | `%p` necesita 64 bits; `%u` y `%x` necesitan el rango de `unsigned int`. Un solo tipo sirve para todos |
| Sin signo | Gestionar el `-` dentro | El signo es el **prefijo** del campo, no parte del cuerpo; si estuviera dentro, los ceros de `%05d` irían delante del `-` |
| Buffer del que llama (`char digits[20]`) | `malloc` | Sin `malloc` no hay leaks ni fallo de memoria que gestionar |
| La base como cadena | Un número `16` | La cadena da a la vez la base (`strlen`) y los caracteres; `%X` es solo otra cadena |
| Iterativa | Recursiva ([[Recursión]]) | La recursiva escribe al volver de cada llamada: no sirve para **preparar** el cuerpo antes de escribir |
| No pone `'\0'` | Ponerlo | No hace falta: se devuelve la longitud y `ft_out_nstr` escribe exactamente esos bytes |

> [!danger] El tamaño del buffer
> 20 es el máximo para un `unsigned long` de 64 bits **en base 10** (`18446744073709551615`). En base 16 son 16. Si añades `%b` (binario) en la evaluación, un `unsigned int` necesita **32** cifras: tendrás que agrandar `digits`. Es una pregunta trampa buena.

## `ft_base`: la base según la letra

> ```c
> static const char	*ft_base(char spec)
> {
> 	if (spec == 'x' || spec == 'p')
> 		return ("0123456789abcdef");
> 	if (spec == 'X')
> 		return ("0123456789ABCDEF");
> 	return ("0123456789");
> }
> ```

Existe para que `ft_print_number` no necesite un quinto parámetro (la [[La Norma|Norma]] permite 4). La letra ya va dentro de `t_flags`, así que la base se deduce de ella. Para añadir `%o` (octal) o `%b` (binario) basta otra línea aquí.

## Errores frecuentes

> [!danger] Los que más aparecen
> - **Usar `ft_putnbr_fd` / `ft_putstr_fd`** de libft: no devuelven cuánto han escrito ni si `write` falló.
> - **Pasar el número con signo** a la función de bases: `-42` en `unsigned long` es `18446744073709551574`.
> - **`-n` con `INT_MIN`** antes de convertirlo a `long`: sale `-18446744071562067968` (ver [[ft_printf - Pruebas y depuración#8. Errores típicos y cómo se ven|Pruebas]]).
> - **Contar cifras empezando en `0`**: el `0` sale como cadena vacía.
> - **Usar `strlen` para el cuerpo de `%c`**: con `'\0'` daría longitud 0 y no se escribiría nada.
> - **`while (n--)` en vez de `while (n-- > 0)`** en el relleno: con `n` negativo no para hasta desbordar el `int`: miles de millones de vueltas.

## Relacionadas

[[ft_printf - Guía paso a paso]] · [[ft_printf - Pruebas y depuración]] · [[ft_printf - Índice]] · [[Structs y typedef]] · [[Tipos enteros y conversiones]] · [[Control de errores en C]] · [[Recursión]] · [[La Norma]] · [[Bases numéricas]] · [[Comportamiento indefinido]]

## Referencias

- [write(2), man7.org](https://man7.org/linux/man-pages/man2/write.2.html): valor de retorno, `-1` y escrituras parciales.
- [strchr(3), man7.org](https://man7.org/linux/man-pages/man3/strchr.3.html): «The terminating null byte is considered part of the string».
- Código compilado con `cc -Wall -Wextra -Werror` (Ubuntu clang 12.0.1) junto a tu libft y probado contra el `printf` de glibc. Las salidas son reales.
