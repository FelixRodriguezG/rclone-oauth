---
tags: [proyecto, get_next_line, pruebas, valgrind, gdb, depuracion, rank-1]
aliases: ["Probar get_next_line", "Tests de get_next_line", "Depurar get_next_line"]
---
# get_next_line - Pruebas y depuración

[[Inicio|← Inicio]] · [[get_next_line - Índice|get_next_line]] · [[get_next_line - Guía paso a paso|Guía]] · Pruebas

> Que tu get_next_line funcione con un fichero no significa que esté bien. El evaluador va a cambiar `BUFFER_SIZE`, pasarte ficheros vacíos, líneas enormes, la entrada estándar, fds cerrados… y buscar leaks. Esta nota te da los ficheros de prueba, un script que lo comprueba todo de una vez, cómo usar Valgrind y cómo **ver por dentro** qué hace tu función cuando algo falla.

> [!quote] Subject v14.0, «Entrega y evaluación»
> «Tanto el tamaño del buffer como el de la línea pueden tener valores muy diferentes.» «Un descriptor de archivo no solo apunta a archivos normales.» «Alguien inteligente […] prepararía una lotería de pruebas de cara a la evaluación.»

## 1. El `main` de prueba

Dos versiones, según lo que quieras ver.

**Para mirar a ojo**: numera cada línea y la pone entre comillas, para ver si trae `'\n'` o no (es el `main` de la [[get_next_line - Guía paso a paso#6. Construirlo por etapas|guía]]):

```text
[1] "ab
"
[2] "cdef
"
[3] "g"
[4] NULL
```

**Para comparar automáticamente** (`main_cat.c`): imprime las líneas **tal cual**, sin añadir nada. Si get_next_line funciona, la salida tiene que ser **idéntica byte a byte** al fichero, igual que hace `cat`.

```c
#include <fcntl.h>
#include <stdio.h>
#include "get_next_line.h"

int	main(int argc, char **argv)
{
	int		fd;
	char	*line;

	fd = 0;
	if (argc == 2)
		fd = open(argv[1], O_RDONLY);
	if (fd == -1)
		return (1);
	line = get_next_line(fd);
	while (line)
	{
		printf("%s", line);
		free(line);
		line = get_next_line(fd);
	}
	if (fd != 0)
		close(fd);
	return (0);
}
```

## 2. Los ficheros de prueba

Cada fichero prueba una situación distinta. Créalos con `printf` (no con `echo`, que añade un `'\n'` al final y no siempre lo quieres):

| Fichero | Contenido | Qué comprueba | Resultado esperado |
|---|---|---|---|
| `vacio.txt` | nada | Fichero vacío | `NULL` a la primera |
| `un_salto.txt` | `\n` | Una línea vacía | `"\n"` y luego `NULL` |
| `solo_saltos.txt` | `\n\n\n` | Varias líneas vacías | `"\n"` tres veces, luego `NULL` |
| `una_linea.txt` | `una linea\n` | El caso más simple | `"una linea\n"`, `NULL` |
| `sin_salto.txt` | `sin salto final` | Última línea sin `'\n'` | `"sin salto final"` (sin `\n`), `NULL` |
| `varias.txt` | `a\nbb\nccc\ndddd\n` | Líneas de distinto tamaño | Cada una con su `\n` |
| `linea_larga.txt` | 100 000 `x` sin salto | Línea mucho más grande que el buffer | Una sola línea de 100 000 caracteres |

## 3. Probarlo todo con un script

Este script crea los ficheros, compila con **seis** `BUFFER_SIZE` distintos y compara la salida de tu función con el fichero original usando `cmp` (que compara byte a byte). Guárdalo como `probar.sh` junto a tus ficheros:

```bash
#!/bin/bash
# Compara la salida de get_next_line con la de cat para varios BUFFER_SIZE.
mkdir -p casos
printf ''                    > casos/vacio.txt
printf '\n'                  > casos/un_salto.txt
printf '\n\n\n'              > casos/solo_saltos.txt
printf 'una linea\n'         > casos/una_linea.txt
printf 'sin salto final'     > casos/sin_salto.txt
printf 'a\nbb\nccc\ndddd\n'  > casos/varias.txt
head -c 100000 /dev/zero | tr '\0' 'x' > casos/linea_larga.txt

for bs in 1 2 5 42 9999 10000000; do
	cc -Wall -Wextra -Werror -D BUFFER_SIZE=$bs \
		get_next_line.c get_next_line_utils.c main_cat.c -o gnl_test || exit 1
	for f in casos/*.txt; do
		if ./gnl_test "$f" | cmp -s - "$f"; then
			echo "OK  BUFFER_SIZE=$bs  $f"
		else
			echo "KO  BUFFER_SIZE=$bs  $f"
		fi
	done
done
```

```bash
chmod +x probar.sh
./probar.sh
```

```text
OK  BUFFER_SIZE=1  casos/linea_larga.txt
OK  BUFFER_SIZE=1  casos/sin_salto.txt
OK  BUFFER_SIZE=1  casos/solo_saltos.txt
...
OK  BUFFER_SIZE=10000000  casos/varias.txt
```

Con una implementación correcta salen **42 OK** (7 ficheros × 6 tamaños). Si aparece un `KO`, ejecuta a mano ese caso con el `main` que numera las líneas para ver qué devuelve.

- `head -c 100000 /dev/zero | tr '\0' 'x'` genera 100 000 bytes a cero y los convierte en `x`. Sobre tuberías y redirecciones: [[Redirecciones y tuberías]].
- `cmp -s - "$f"`: el `-` es «lo que llega por la tubería»; `-s` hace que no imprima nada y solo devuelva si son iguales.

> [!tip] Compilar sin `-D`
> Añade también una compilación **sin** `-D BUFFER_SIZE` para comprobar que tu `#ifndef` del header funciona. El subject lo exige.

## 4. Entrada estándar y tuberías

El subject pide que funcione «cuando lea de un archivo y cuando lea de `stdin`». El `main` lee del fd `0` si no le pasas argumentos:

```bash
./gnl                          # escribe líneas a mano; Ctrl+D al principio de una línea = fin
./gnl < casos/varias.txt       # redirección: el fichero entra por stdin
cat casos/varias.txt | ./gnl   # tubería: el fd 0 es una tubería, no un fichero
```

```text
$ printf 'uno\ndos\n' | ./gnl
[1] "uno
"
[2] "dos
"
[3] NULL
```

Con el teclado, `read` vuelve en cuanto pulsas Intro, aunque no haya llenado el buffer (ver [[Descriptores de fichero y read#Leer por bloques]]). Por eso tu función nunca debe suponer que `read` llena el buffer entero.

## 5. fds inválidos

```c
printf("fd -1 -> %p\n", (void *)get_next_line(-1));
printf("fd 42 -> %p\n", (void *)get_next_line(42));
fd = open("t1.txt", O_RDONLY);
line = get_next_line(fd);
printf("1a linea: \"%s\"\n", line);
free(line);
close(fd);
printf("tras close -> %p\n", (void *)get_next_line(fd));
```

```text
fd -1 -> (nil)
fd 42 -> (nil)
1a linea: "ab
"
tras close -> (nil)
==778717== All heap blocks were freed -- no leaks are possible
```

El último caso es el más traicionero: después de la primera línea, el stash tenía `"c"` guardado. Al llamar con el fd cerrado, `read` devuelve `-1` y la función **tiene que liberar ese stash**. Si no, Valgrind lo verá.

## 6. Buscar leaks con Valgrind

Compila con `-g` (para que Valgrind diga números de línea) y ejecuta (ver [[Valgrind y sanitizers]]):

```bash
cc -g -Wall -Wextra -Werror -D BUFFER_SIZE=3 get_next_line.c get_next_line_utils.c main.c -o gnl
valgrind --leak-check=full --show-leak-kinds=all ./gnl casos/varias.txt
```

Lo que quieres ver al final:

```text
==778707== HEAP SUMMARY:
==778707==     in use at exit: 0 bytes in 0 blocks
==778707==   total heap usage: 14 allocs, 14 frees, 4,144 bytes allocated
==778707== All heap blocks were freed -- no leaks are possible
==778707== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
```

- **`allocs` = `frees`**: cada `malloc` tiene su `free`.
- **`ERROR SUMMARY: 0`**: ninguna lectura o escritura inválida.

Pasa Valgrind en **todos** los casos de la tabla, con `BUFFER_SIZE` pequeño (1, 3) y grande: los fallos de memoria salen en sitios distintos según el tamaño.

> [!note] `still reachable` si el `main` deja de leer antes del final
> Si el `main` solo pide una línea y termina, lo que sobraba sigue en la `static`:
> ```text
> ==778867== 2 bytes in 1 blocks are still reachable in loss record 1 of 1
> ==778867==    definitely lost: 0 bytes in 0 blocks
> ==778867==    still reachable: 2 bytes in 1 blocks
> ```
> No es `definitely lost`: la `static` todavía apunta a esa memoria. Si lees hasta recibir `NULL`, desaparece. Que lo consideren leak o no depende del evaluador; ten la explicación preparada.

## 7. Ver el stash por dentro

Cuando algo falla, lo más útil es **ver qué hay en el stash** en cada paso. Dos formas.

### Con `fprintf` temporales

Añade esta función al principio de `get_next_line.c` (necesita `#include <stdio.h>`). Imprime una cadena enseñando los `'\n'` como `\n`, y `NULL` si el puntero es `NULL`:

```c
static void	show(const char *label, const char *s)
{
	fprintf(stderr, "    %-8s= ", label);
	if (!s)
	{
		fprintf(stderr, "NULL\n");
		return ;
	}
	fprintf(stderr, "\"");
	while (*s)
	{
		if (*s == '\n')
			fprintf(stderr, "\\n");
		else
			fputc(*s, stderr);
		s++;
	}
	fprintf(stderr, "\"\n");
}
```

Y llámala en los puntos clave: al entrar en `get_next_line` (`show("stash", stash)`), después de cada `read` (`show("buf", buf)`), después de cada join y antes del `return (line)`. Se imprime en `stderr` (fd 2) para no mezclarse con la salida del `main`, y la puedes ver sola con `./gnl test.txt 2>&1 >/dev/null`. El resultado es la traza de la [[get_next_line - Guía paso a paso#7. Lo que pasa por dentro, llamada a llamada|sección 7 de la guía]].

> [!danger] Quítalo antes de entregar
> `fprintf` no está entre las funciones permitidas y la función `show` no cumple el objetivo del proyecto. Bórralo todo (o trabaja en una copia) antes de hacer `git push`.

### Con gdb

Sin tocar el código (ver [[Depurar con gdb]]):

```bash
cc -g -O0 -Wall -Wextra -Werror -D BUFFER_SIZE=4 get_next_line.c get_next_line_utils.c main.c -o gnl
gdb ./gnl
```

```text
(gdb) break ft_save_rest
(gdb) run t1.txt
Breakpoint 1, ft_save_rest (stash=0x4052c0 "ab\nc") at get_next_line.c:48
(gdb) continue
Breakpoint 1, ft_save_rest (stash=0x4052c0 "cdef\n") at get_next_line.c:48
(gdb) finish
0x0000000000401227 in get_next_line (fd=3) at get_next_line.c:69
69		stash = ft_save_rest(stash);
Value returned is $3 = 0x0
```

- `break ft_save_rest`: para cada vez que se entra en esa función. gdb te enseña los argumentos, con los `\n` escapados.
- `continue`: sigue hasta el siguiente breakpoint (la siguiente llamada).
- `finish`: termina la función actual y enseña lo que devuelve. Aquí `0x0` (`NULL`): no quedaba nada tras `"cdef\n"`.
- `print stash`, `next`, `step` y `display stash` también sirven.

## 8. Errores típicos y cómo se ven

Todas estas salidas son reales, provocadas metiendo el error a propósito.

| Error | Síntoma | Cómo se ve |
|---|---|---|
| Stash **sin `static`** | Faltan trozos de líneas | `"hola\n"`, `"do\n"` en vez de `"mundo\n"`; Valgrind: `definitely lost: 4 bytes in 1 blocks` |
| Olvidar **`buf[bytes] = '\0'`** | Basura, resultados que cambian | Valgrind: `Conditional jump or move depends on uninitialised value(s)` |
| `strjoin` que **no libera** el stash | Funciona, pero pierde memoria | Valgrind: `definitely lost: 6 bytes in 2 blocks` |
| **Buffer en la pila** con `BUFFER_SIZE=10000000` | Crash al llamar a la función | `Segmentation fault`, código de salida `139` |
| No liberar el stash si **`read` da `-1`** | Leak o datos viejos tras cerrar el fd | Valgrind tras el caso de la sección 5: `definitely lost: 2 bytes in 1 blocks` (el `"c"` que quedaba) |
| Devolver **`""`** en vez de `NULL` al final | El `main` no termina nunca | Bucle infinito imprimiendo `[n] ""` |

> [!tip] Lee el primer error de Valgrind, no el último
> Un error de memoria suele provocar otros detrás. Arregla el primero que aparezca (busca la línea `at 0x...: tu_funcion (fichero.c:LÍNEA)`) y vuelve a ejecutar.

## 9. Testers de la comunidad

Cuando tus pruebas pasen, prueba un tester público:

- [gnlTester (Tripouille)](https://github.com/Tripouille/gnlTester): el más usado; prueba muchos `BUFFER_SIZE`, stdin, leaks y el bonus.

> [!warning] Un tester no sustituye a entender tu código
> Pueden quedarse atrás respecto al subject o tener sus propias manías. Si un tester falla y no entiendes por qué, vuelve a tu `main` y a la traza: el evaluador te pedirá que lo expliques tú.

## Checklist antes de entregar

- [ ] Los 42 casos de `probar.sh` dan OK.
- [ ] Compila **sin** `-D BUFFER_SIZE`.
- [ ] Funciona con stdin (teclado, `<` y `|`).
- [ ] `-1`, `42` y un fd cerrado devuelven `NULL` sin leaks.
- [ ] Valgrind sin errores ni leaks en todos los casos, con `BUFFER_SIZE` 1 y 10000000.
- [ ] `norminette get_next_line.c get_next_line_utils.c get_next_line.h` sin errores.
- [ ] No queda ningún `printf`/`fprintf` de depuración.
- [ ] No se usa libft, `lseek` ni variables globales.
- [ ] Hay un `README.md` con lo que pide el subject (ver [[get_next_line - Guía paso a paso#11. El README (obligatorio en la v14)|la guía]]).
- [ ] En el repo solo están los ficheros que hay que entregar (ver [[Git para 42]]).

## Relacionadas

[[get_next_line - Guía paso a paso]] · [[get_next_line - Funciones auxiliares]] · [[get_next_line - Índice]] · [[Valgrind y sanitizers]] · [[Depurar con gdb]] · [[Descriptores de fichero y read]] · [[Redirecciones y tuberías]] · [[malloc y free]] · [[Git para 42]]

## Referencias

- Subject de get_next_line v14.0: stdin, `BUFFER_SIZE` 1 / 9999 / 10000000, consejo de preparar pruebas.
- [Valgrind: Memcheck manual](https://valgrind.org/docs/manual/mc-manual.html): *definitely lost*, *still reachable*, valores sin inicializar.
- [cmp(1), man7.org](https://man7.org/linux/man-pages/man1/cmp.1.html): comparación byte a byte y `-s`.
- [GDB documentation](https://sourceware.org/gdb/current/onlinedocs/gdb.html/): `break`, `continue`, `finish`.
- [gnlTester (Tripouille)](https://github.com/Tripouille/gnlTester).
- Salidas obtenidas con Ubuntu clang 12.0.1, Valgrind y GDB en Ubuntu 22.04, con una implementación de referencia que pasa los 42 casos del script.
