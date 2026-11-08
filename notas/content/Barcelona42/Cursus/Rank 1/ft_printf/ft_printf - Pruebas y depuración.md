---
tags: [proyecto, ft_printf, pruebas, valgrind, gdb, depuracion, rank-1]
aliases: ["Probar ft_printf", "Tests de ft_printf", "Depurar ft_printf"]
---
# ft_printf - Pruebas y depuración

[[Inicio|← Inicio]] · [[ft_printf - Índice|ft_printf]] · [[ft_printf - Guía paso a paso|Guía]] · Pruebas

> El subject dice: *«La función se comparará con el printf() original para verificar su comportamiento.»* Eso es exactamente lo que vas a hacer tú antes: comparar **byte a byte y el valor de retorno** contra `printf`, de forma automática, con la parte obligatoria y con el bonus. Esta nota da el tester, el script que revisa el Makefile, cómo ver las banderas por dentro y cómo se ve cada error típico.

> [!quote] Subject v12.0, «Instrucciones generales»
> «Te recomendamos crear programas de prueba para tu proyecto, aunque este trabajo no será entregado ni evaluado. […] Y sí, tienes permitido utilizar estas pruebas durante tu evaluación.»

## 1. El `main` para mirar a ojo

Pon cada conversión entre **corchetes** para ver los espacios, y compara siempre el **valor de retorno**:

```c
#include <stdio.h>
#include "ft_printf.h"

int	main(void)
{
	int	r1;
	int	r2;

	r1 = printf("[%-8.3d]\n", -42);
	fflush(stdout);
	r2 = ft_printf("[%-8.3d]\n", -42);
	printf("printf=%d ft_printf=%d\n", r1, r2);
	return (0);
}
```

```bash
make bonus && cc -Wall -Wextra -Iincludes -Ilibft main.c libftprintf.a -o test && ./test | cat -A
```

`cat -A` marca el final de línea con `$` y los bytes invisibles: `%c` con `'\0'` sale como `^@`.

## 2. No mezcles printf y ft_printf en la terminal

`printf` guarda lo que escribe en un buffer y lo vacía cuando le conviene; `ft_printf` llama a `write` al momento. Este programa:

```c
#include <stdio.h>
#include "ft_printf.h"

int	main(void)
{
	printf("1. printf\n");
	ft_printf("2. ft_printf\n");
	printf("3. printf\n");
	ft_printf("4. ft_printf\n");
	return (0);
}
```

Salida real en la terminal y redirigido a una tubería:

```text
$ ./mix                 $ ./mix | cat
1. printf               2. ft_printf
2. ft_printf            4. ft_printf
3. printf               1. printf
4. ft_printf            3. printf
```

En la terminal, `printf` vacía su buffer en cada `'\n'`. En una tubería o un fichero, lo vacía **al final del programa**. Tu ft_printf no está mal: es el buffer de `printf` (explicado en [[Buffer de stdout]]). Soluciones: `fflush(stdout)` después de cada `printf`, o comparar en ficheros separados como hace el tester.

## 3. El tester: comparar contra printf automáticamente

Por cada caso: redirige la salida estándar a un fichero, llama a `printf`, lee lo escrito; repite con `ft_printf`; compara bytes y valor de retorno. Solo imprime los que fallan. Los casos de bonus van dentro de `#ifdef BONUS`.

> [!faq]- Ver el tester completo (`tester.c`, no se entrega)
> ```c
> #include <stdio.h>
> #include <string.h>
> #include <unistd.h>
> #include <fcntl.h>
> #include <limits.h>
> #include "ft_printf.h"
>
> static char	a[4096], b[4096];
> static int	ok, ko;
>
>
> #define CASE(...) do { \
> 	int r1, r2, n1, n2, fd, sv; \
> 	fflush(stdout); sv = dup(1); \
> 	fd = open("/tmp/ft_o1", O_RDWR|O_CREAT|O_TRUNC, 0644); dup2(fd, 1); \
> 	r1 = printf(__VA_ARGS__); fflush(stdout); \
> 	lseek(fd, 0, SEEK_SET); n1 = read(fd, a, sizeof a); close(fd); \
> 	fd = open("/tmp/ft_o2", O_RDWR|O_CREAT|O_TRUNC, 0644); dup2(fd, 1); \
> 	r2 = ft_printf(__VA_ARGS__); \
> 	lseek(fd, 0, SEEK_SET); n2 = read(fd, b, sizeof b); close(fd); \
> 	dup2(sv, 1); close(sv); \
> 	if (r1 == r2 && n1 == n2 && !memcmp(a, b, n1)) ok++; \
> 	else { ko++; printf("KO line %d: %s\n  printf    [%.*s] %d\n  ft_printf [%.*s] %d\n", __LINE__, #__VA_ARGS__, n1, a, r1, n2, b, r2); } \
> } while (0)
>
> int	main(void)
> {
> 	char	*nul = NULL;
> 	int		x;
>
> 	CASE("hola mundo\n");
> 	CASE("%c%c%c", 'a', 0, 'z');
> 	CASE("%s|%s|%s", "abc", "", nul);
> 	CASE("%p %p %p", &x, (void *)0, (void *)ULONG_MAX);
> 	CASE("%d %i %d %d", 0, -42, INT_MIN, INT_MAX);
> 	CASE("%u %u %u", 0, -1, 4242);
> 	CASE("%x %X %x %X", 0, 255, -1, INT_MIN);
> 	CASE("%% %%%% 100%%");
> 	CASE("%c %s %p %d %i %u %x %X %%", 'x', "s", &x, 1, 2, 3u, 4, 5);
> #ifdef BONUS
> 	CASE("[%5c][%-5c][%-3c]", 'a', 'b', 0);
> 	CASE("[%10s][%-10s][%.2s][%7.3s][%-7.3s][%.0s]", "hola", "hola", "hola", "hola", "hola", "hola");
> 	CASE("[%10s][%.3s][%.6s][%-8s]", nul, nul, nul, nul);
> 	CASE("[%20p][%-20p][%10p][%-10p]", &x, &x, nul, nul);
> 	CASE("[%5d][%-5d][%05d][%.5d][%8.5d][%-8.5d][%08.5d]", 42, 42, 42, 42, 42, 42, 42);
> 	CASE("[%5d][%-5d][%05d][%.5d][%8.5d][%-8.5d][%08.5d]", -42, -42, -42, -42, -42, -42, -42);
> 	CASE("[%.0d][%5.0d][%-5.0d][%.0i][%.d]", 0, 0, 0, 0, 0);
> 	CASE("[%+d][% d][%+ d][% +d][%+d][% d]", 42, 42, 42, 42, -42, -42);
> 	CASE("[%+05d][% 05d][%+.3d][% 8.3d][%-+8d]", 7, 7, 7, 7, 7);
> 	CASE("[%+.0d][% .0d][%+5.0d]", 0, 0, 0);
> 	CASE("[%#x][%#X][%#x][%#08x][%#-8x][%#.5x][%#10.5X]", 255, 255, 0, 255, 255, 255, 255);
> 	CASE("[%#.0x][%.0x][%5.0X][%#5x]", 0, 0, 0, 0);
> 	CASE("[%10u][%-10u][%010u][%.10u][%.0u]", 42u, 42u, 42u, 42u, 0u);
> 	CASE("[%011d][%-11d][%.11d][%+d][% d]", INT_MIN, INT_MIN, INT_MIN, INT_MAX, INT_MAX);
> 	CASE("[%-05d][%0-5d][%--5d][%005d]", 1, 2, 3, 4);
> 	CASE("[%015p][%.5p][%-#5x]", &x, (void *)255, 1);
> 	CASE("[%1s][%1d][%0d][%.1x]", "hola", 12345, 9, 255);
> #endif
> 	printf("%s: %d OK, %d KO\n", BONUS_NAME, ok, ko);
> 	return (ko != 0);
> }
> ```

```bash
make && cc -Wall -Wextra -Wno-format -Iincludes -Ilibft -DBONUS_NAME='"obligatoria"' tester.c libftprintf.a -o t && ./t
make bonus && cc -Wall -Wextra -Wno-format -Iincludes -Ilibft -DBONUS -DBONUS_NAME='"bonus"' tester.c libftprintf.a -o t && ./t
```

Con la solución de la guía:

```text
obligatoria: 9 OK, 0 KO
bonus: 26 OK, 0 KO
```

- **Es una macro y no una función** porque `printf` y `ft_printf` necesitan recibir los argumentos variádicos tal cual. Una función con `...` no puede volver a pasarlos a otra función con `...` (para eso existen `vprintf` y `va_list`, que ft_printf no tiene).
- **`-Wno-format`**: clang avisa de los casos indefinidos (`%s` con `NULL`, `% s`). En el tester los queremos probar igualmente. Sin `-Werror`: el tester no se entrega.
- **Añade tus propios casos.** Cada fallo que encuentres, conviértelo en una línea `CASE(...)` para que no vuelva.

> [!warning] gcc y `printf("%s\n", NULL)`
> gcc puede cambiar `printf("%s\n", x)` por `puts(x)`, que **crashea** con `NULL`. No es tu fallo. En el tester los `%s` van entre corchetes (`"[%s]"`), y así gcc no hace el cambio.

## 4. Casos que tiene que pasar

### Obligatoria

| Caso | `printf` (glibc) | Por qué es difícil |
|---|---|---|
| `%c` con `'\0'` | 1 byte nulo, devuelve 1 | No vale `strlen` |
| `%s` con `NULL` | `(null)` | Crash si no lo compruebas |
| `%s` con `""` | nada, devuelve 0 | |
| `%p` con `NULL` | `(nil)` | En macOS es `0x0` |
| `%p` con `ULONG_MAX` | `0xffffffffffffffff` | Truncado si usas `unsigned int` |
| `%d` con `INT_MIN` | `-2147483648` | `-n` desborda |
| `%u` con `-1` | `4294967295` | Leído como `int` sale `-1` |
| `%x` con `-1` | `ffffffff` | |
| `%%` | `%` | No consume argumento |
| Formato vacío `""` | nada, devuelve 0 | |

### Bonus

Las reglas con su salida real están en [[ft_printf - Guía paso a paso#9. Dónde se aplica cada regla del bonus|la tabla de la guía]]. Las combinaciones más traicioneras:

| Formato | Valor | `printf` |
|---|---|---|
| `%.0d` | `0` | `[]` |
| `%5.0d` | `0` | `[     ]` |
| `%+.0d` | `0` | `[+]` |
| `%#x` | `0` | `[0]` |
| `%#.0x` | `0` | `[]` |
| `%05d` | `-42` | `[-0042]` |
| `%08.5d` | `42` | `[   00042]` |
| `%-05d` | `1` | `[1    ]` |
| `%.3s` | `NULL` | `[]` |
| `%-3c` | `'\0'` | `\0` + 2 espacios, devuelve 3 |
| `%5%` | — | `[%]` |

## 5. El `-1` de `write`

`ft_printf` tiene que devolver `-1` si `write` falla, igual que `printf`. Dos formas de provocarlo sin tocar el código:

```bash
./test >&-            # cierra la salida estándar: write da -1 (EBADF)
./test > /dev/full    # dispositivo siempre lleno: write da -1 (ENOSPC)
```

Para ver el valor de retorno, imprímelo por la **salida de error**, que sigue abierta:

```c
r = ft_printf("hola %d\n", 42);
fprintf(stderr, "ret=%d\n", r);
```

```text
$ ./m
hola 42
ret=8
$ ./m >&-
ret=-1
$ ./m > /dev/full
ret=-1
```

## 6. Valgrind y el Makefile

### Valgrind

```bash
valgrind -q --leak-check=full ./t
```

Con `-q` no imprime nada si todo está bien. El diseño de la guía no usa `malloc`, así que no puede haber leaks, pero Valgrind sigue siendo útil para **lecturas fuera de una cadena** (por ejemplo, avanzar el índice más allá del `'\0'` con un `%` final). Más en [[Valgrind y sanitizers]].

### Script para revisar el Makefile

Comprueba lo que mira un evaluador: la librería en la raíz, que no haya relink en `make` ni en `make bonus`, que el bonus **sustituya** el fichero de banderas, y que tocar el header recompile.

```bash
#!/bin/bash
# Uso: ./check_make.sh   (desde la raíz de ft_printf)
make fclean > /dev/null
make > /dev/null 2>&1 || { echo "KO: make falla"; exit 1; }
[ -f libftprintf.a ] && echo "OK  libftprintf.a en la raíz" || echo "KO  no hay libftprintf.a"
make -q && echo "OK  make no hace relink" || echo "KO  make hace relink"
ar t libftprintf.a | grep -q '^ft_flags.o$' && echo "OK  obligatoria: contiene ft_flags.o" || echo "KO  obligatoria sin ft_flags.o"
make bonus > /dev/null 2>&1 || { echo "KO: make bonus falla"; exit 1; }
[ "$(make -n bonus 2>/dev/null | grep -cE '(^|[[:space:]])(cc|ar) ')" -eq 0 ] && echo "OK  make bonus no hace relink" || echo "KO  make bonus hace relink"
ar t libftprintf.a | grep -q '^ft_flags_bonus.o$' && echo "OK  bonus: contiene ft_flags_bonus.o" || echo "KO  bonus sin ft_flags_bonus.o"
ar t libftprintf.a | grep -q '^ft_flags.o$' && echo "KO  bonus: ft_flags.o sigue dentro" || echo "OK  bonus: ft_flags.o no está"
touch includes/ft_printf.h
[ "$(make -n bonus 2>/dev/null | grep -c 'ft_printf.c')" -gt 0 ] && echo "OK  cambiar el .h recompila" || echo "KO  cambiar el .h NO recompila"
make fclean > /dev/null
```

Resultado con el Makefile de la guía:

```text
OK  libftprintf.a en la raíz
OK  make no hace relink
OK  obligatoria: contiene ft_flags.o
OK  make bonus no hace relink
OK  bonus: contiene ft_flags_bonus.o
OK  bonus: ft_flags.o no está
OK  cambiar el .h recompila
```

- `make -q` devuelve `0` si todo está al día: si no, haría relink.
- `make -n` **enseña** los comandos sin ejecutarlos (incluidos los que llevan `@`). Si en un `make bonus` que ya está hecho aparece `cc` o `ar`, hay relink.
- `ar t` lista los `.o` que hay dentro de la librería.

## 7. Ver por dentro qué ha leído `ft_set_flags`

Cuando un caso del bonus falla, lo primero es saber si el fallo está **leyendo** las banderas o **imprimiendo** el campo.

### Con un `fprintf` temporal

Justo después de `flags = ft_set_flags(format, &i);` en `ft_printf`:

```c
fprintf(stderr, "[flags] minus=%d zero=%d width=%d prec=%d sharp=%d space=%d plus=%d spec=%c\n",
	flags.minus, flags.zero, flags.width, flags.precision,
	flags.sharp, flags.space, flags.plus, flags.spec);
```

Con `ft_printf("[%-08.3d] [%#x] [%+ 5d]\n", -42, 255, 7)`, la salida de error real:

```text
[flags] minus=1 zero=0 width=8 prec=3 sharp=0 space=0 plus=0 spec=d
[flags] minus=0 zero=0 width=0 prec=-1 sharp=1 space=0 plus=0 spec=x
[flags] minus=0 zero=0 width=5 prec=-1 sharp=0 space=0 plus=1 spec=d
```

Fíjate: en el primero `zero=0` aunque el formato tenga `0` (el `-` lo anula); en el tercero `space=0` aunque tenga espacio (el `+` lo anula). Si las banderas están bien y la salida mal, el fallo está en la preparación del campo o en `ft_out_field`.

> [!danger] Quítalo antes de entregar
> `fprintf` no está permitido y `<stdio.h>` no debe estar en tu código. Bórralo en cuanto encuentres el fallo.

### Con gdb

Si no has usado gdb nunca, empieza por [[Depurar con gdb]]. Compila con `-g` (en `CFLAGS` del Makefile y en el `main`) y para en `ft_out_field`, donde ya están todas las piezas:

```bash
make fclean && make bonus CFLAGS="-Wall -Wextra -Werror -g -Iincludes -Ilibft"
cc -g -Iincludes -Ilibft main.c libftprintf.a -o g
gdb -q ./g
(gdb) break ft_out_field
(gdb) run
(gdb) print *f
(gdb) print *fl
```

Sesión real con `ft_printf("[%08.3d]\n", -42)`:

```text
Breakpoint 1, ft_out_field (o=0x7fffffffdc38, f=0x7fffffffdc18, fl=0x7fffffffd8f8) at srcs/ft_output.c:34
34		prefix_len = ft_strlen(fl->prefix);
$1 = {minus = 0, zero = 0, width = 8, precision = 3, sharp = 0, space = 0, plus = 0, spec = 100 'd'}
$2 = {prefix = 0x40301e "-", body = 0x7fffffffd910 "42", body_len = 2, zeros = 1}
```

Se ve todo el diseño: `zero` ya vale `0` (lo anuló la precisión en `ft_print_number`), el prefijo es `"-"`, el cuerpo `"42"` y falta 1 cero para llegar a la precisión 3. Con eso puedes calcular la salida a mano: `8 - 1 - 1 - 2 = 4` espacios → `[    -042]`.

> [!note] `print *fl` y el cuerpo
> gdb imprime `body` como cadena hasta encontrar un `'\0'`. Como `digits` no lleva `'\0'`, a veces verás basura detrás de los dígitos. No es un fallo: tu código solo usa `body_len` caracteres.

## 8. Errores típicos y cómo se ven

Salida real del tester con cada error metido a propósito en la solución de la guía. Si ves una de estas, ya sabes dónde mirar.

### `precision` inicializada a `0` en vez de `-1`

```text
KO line 34: "%s|%s|%s", "abc", "", nul
  printf    [abc||(null)] 11
  ft_printf [||] 2
KO line 36: "%d %i %d %d", 0, -42, INT_MIN, INT_MAX
  printf    [0 -42 -2147483648 2147483647] 28
  ft_printf [ -42 -2147483648 2147483647] 27
```

**Todas** las cadenas salen vacías (precisión 0 = 0 caracteres) y el `0` desaparece (precisión 0 con valor 0 = nada). Rompe incluso la parte obligatoria.

### Los ceros antes del prefijo

```text
KO line 47: "[%5d][%-5d][%05d][%.5d][%8.5d][%-8.5d][%08.5d]", -42, ...
  printf    [[  -42][-42  ][-0042][-00042][  -00042][-00042  ][  -00042]] 59
  ft_printf [[  -42][-42  ][00-42][000-42][  000-42][000-42  ][  000-42]] 59
```

El valor de retorno coincide (59) pero el orden no: en `ft_out_field` se escriben los ceros antes que el prefijo. Lo mismo pasa con `+`, espacio y `0x`.

### `-n` sin convertir a `long` (`INT_MIN`)

```text
KO line 36: "%d %i %d %d", 0, -42, INT_MIN, INT_MAX
  printf    [0 -42 -2147483648 2147483647] 28
  ft_printf [0 -42 -18446744071562067968 2147483647] 38
```

`-n` con `INT_MIN` desborda y sigue siendo `INT_MIN` (negativo); al pasarlo a `unsigned long` se convierte en un número enorme.

### La precisión no anula la bandera `0`

```text
KO line 46: "[%5d][%-5d][%05d][%.5d][%8.5d][%-8.5d][%08.5d]", 42, ...
  printf    [[   42][42   ][00042][00042][   00042][00042   ][   00042]] 58
  ft_printf [[   42][42   ][00042][00042][   00042][00042   ][00000042]] 58
```

Solo falla el último: `%08.5d`. Falta `if (f->precision >= 0) f->zero = 0;` en `ft_print_number`.

### `#` con el valor 0

```text
KO line 52: "[%#x][%#X][%#x][%#08x][%#-8x][%#.5x][%#10.5X]", 255, 255, 0, ...
  printf    [[0xff][0XFF][0][0x0000ff][0xff    ][0x000ff][   0X000FF]] 56
  ft_printf [[0xff][0XFF][0x0][0x0000ff][0xff    ][0x000ff][   0X000FF]] 58
KO line 53: "[%#.0x][%.0x][%5.0X][%#5x]", 0, 0, 0, 0
  printf    [[][][     ][    0]] 18
  ft_printf [[0x][][     ][  0x0]] 20
```

`printf` no pone `0x` delante del 0. Falta `n != 0` en la condición de `ft_print_unsigned`.

## 9. Testers de la comunidad

Úsalos **después** de los tuyos: si no entiendes por qué falla un caso, no te sirve de nada saber que falla.

- **printfTester** (Tripouille): `make m` (obligatoria) y `make b` (bonus). Muy usado en las evaluaciones.
- **francinette** (xicodomingues): pasa varios testers y norminette de una vez.

Enlaces en [[ft_printf - Índice#Referencias]].

> [!warning] Los testers prueban comportamiento de Linux
> `(nil)` para `%p` con `NULL` y `(null)` para `%s` son de glibc. Si pruebas en macOS, los resultados cambian. Evalúa en el mismo sistema que tus compañeros (los Mac/Linux del campus).

## Checklist antes de entregar

- [ ] El tester pasa con `make` (obligatoria) y con `make bonus`.
- [ ] `valgrind -q` no imprime nada.
- [ ] `./test >&-` y `./test > /dev/full` → `-1`.
- [ ] `check_make.sh` todo `OK`.
- [ ] `norminette includes srcs libft` sin errores (cabecera de 42 incluida).
- [ ] Ningún `printf`, `fprintf` ni `<stdio.h>` en los `.c` y `.h` que se entregan.
- [ ] No hay `main.c`, `tester.c` ni `.o` en el repositorio (o están en `.gitignore`).
- [ ] printfTester `make m` y `make b`.
- [ ] `README.md` con la primera línea en cursiva y la explicación del algoritmo.

## Relacionadas

[[ft_printf - Guía paso a paso]] · [[ft_printf - Funciones auxiliares]] · [[ft_printf - Índice]] · [[printf]] · [[Valgrind y sanitizers]] · [[Makefile y librerías estáticas]] · [[Control de errores en C]] · [[get_next_line - Pruebas y depuración]] · [[Buffer de stdout]] · [[Depurar con gdb]] · [[Comportamiento indefinido]]

## Referencias

- Subject de ft_printf v12.0 (`~/cursus/rank-1/es.subject_ft_printf.pdf`): comparación con `printf`, programas de prueba, Makefile sin relink.
- [write(2), man7.org](https://man7.org/linux/man-pages/man2/write.2.html): errores `EBADF` y `ENOSPC`.
- [setbuf(3), man7.org](https://man7.org/linux/man-pages/man3/setbuf.3.html): por qué `stdout` va con buffer de línea en una terminal y con buffer completo en un fichero o tubería.
- [GNU make: Instead of Executing Recipes](https://www.gnu.org/software/make/manual/html_node/Instead-of-Execution.html): `make -n` y `make -q`.
- Todas las salidas de esta nota son reales: Ubuntu 22.04, clang 12.0.1, glibc, GNU gdb y Valgrind del campus. Los errores de la sección 8 se han obtenido metiendo cada fallo a propósito en la solución de la guía.
