---
tags: [c, errores, malloc, errno, memoria]
---
# Control de errores en C

[[Inicio|← Inicio]] · C

> C no tiene excepciones: **cada función avisa de un error con su valor de retorno**, y es responsabilidad de quien la llama comprobarlo. Si no lo compruebas, el programa sigue como si nada… hasta que crashea más adelante, lejos de la causa.

## Cómo avisa cada función

| Tipo de función | Valor si falla | Ejemplos |
|---|---|---|
| Devuelve un puntero | `NULL` | `malloc`, [[ft_calloc]], [[ft_strdup]], [[ft_split]], [[ft_lstnew]] |
| Llamada al sistema | `-1` (y pone `errno`) | `open`, `read`, `write`, `close` |
| Devuelve un estado | Lo que diga su `man` (`-1`, `0`, `EOF`…) | `printf` → negativo |
| No puede fallar o no avisa | — | [[ft_strlen]], [[ft_atoi]] (¡`"abc"` devuelve `0`, igual que `"0"`!) |

> [!tip] Regla de oro
> Antes de usar una función, mira en su `man` la sección **RETURN VALUE**: ahí dice qué devuelve cuando falla.

## 1. Comprobar siempre `malloc`

`malloc` puede devolver `NULL` (sin memoria, tamaño absurdo…). Usar ese `NULL` provoca un segfault.

```c
char	*dup_str(const char *s)
{
	char	*dup;
	size_t	len;

	len = ft_strlen(s) + 1;
	dup = malloc(len);
	if (!dup)
		return (NULL);
	ft_memcpy(dup, s, len);
	return (dup);
}
```

El error **se propaga hacia arriba**: la función no sabe qué hacer con el fallo, así que devuelve `NULL` y quien la llamó decide.

## 2. Liberar lo reservado antes de salir

Si una función hace varios `malloc` y falla el tercero, hay que **liberar los dos anteriores** antes de devolver `NULL`. Si no, hay fuga de memoria. Es el caso de [[ft_split]] o [[ft_lstmap]].

```c
static void	*free_partial(char **arr, size_t filled)
{
	while (filled > 0)
	{
		filled--;
		free(arr[filled]);
	}
	free(arr);
	return (NULL);
}

char	**dup_all(char **src, size_t n)
{
	char	**arr;
	size_t	i;

	arr = malloc(sizeof(char *) * (n + 1));
	if (!arr)
		return (NULL);
	i = 0;
	while (i < n)
	{
		arr[i] = dup_str(src[i]);
		if (!arr[i])
			return (free_partial(arr, i));
		i++;
	}
	arr[n] = NULL;
	return (arr);
}
```

> [!note] Sin `goto`
> En C es habitual limpiar con `goto cleanup;`, pero la [[La Norma|Norma]] de 42 prohíbe `goto`. La alternativa es una **función auxiliar de limpieza** que libere y devuelva `NULL`, para poder escribir `return (free_partial(arr, i));` en una sola línea.

## 3. Llamadas al sistema: `-1` y `errno`

Las llamadas al sistema devuelven `-1` y guardan el motivo en la variable global **`errno`** (`<errno.h>`). Para mostrarlo:

| Función | Qué hace | Ejemplo de salida |
|---|---|---|
| `perror("open")` | Escribe en stderr tu texto + el mensaje de `errno` | `open: No such file or directory` |
| `strerror(errno)` | Devuelve el mensaje como cadena, para formatearlo tú | `"No such file or directory"` |

```c
static int	print_file(const char *path)
{
	char	buf[64];
	int		fd;
	ssize_t	bytes;

	fd = open(path, O_RDONLY);
	if (fd == -1)
	{
		perror(path);
		return (-1);
	}
	bytes = read(fd, buf, sizeof(buf));
	while (bytes > 0)
	{
		if (write(1, buf, bytes) == -1)
			break ;
		bytes = read(fd, buf, sizeof(buf));
	}
	if (bytes == -1)
		perror("read");
	close(fd);
	return (bytes);
}
```

```text
$ ./prog /no/existe
/no/existe: No such file or directory
```

> [!warning] Cuidado con `errno`
> - **Solo tiene sentido justo después de que una función falle.** Si la función tuvo éxito, `errno` puede tener un valor viejo de otro error.
> - Cualquier otra llamada (incluso `printf`) puede cambiarlo: léelo o muéstralo **inmediatamente**.
> - `perror` y `strerror` solo se pueden usar si el **subject** los autoriza (libft no; pipex, so_long, minishell… sí).

## 4. Desbordamientos de enteros

Algunos errores no avisan: el número simplemente da la vuelta.

- **Multiplicar tamaños**: `nmemb * size` puede desbordar `size_t` y reservar **menos** de lo pedido. Compruébalo antes, como hace el `calloc` original:
  ```c
  if (size != 0 && nmemb > SIZE_MAX / size)
  	return (NULL);
  ```
- **`size_t` por debajo de 0**: `ft_strlen(s) - 1` con `s == ""` no da `-1`, da un número enorme.
- **Convertir texto a número**: [[ft_atoi]] no detecta `"99999999999"` ni `"12abc"`. En proyectos como push_swap tendrás que escribir tu propia versión que valide.

## 5. Validar la entrada

- **Argumentos del programa**: comprueba `argc` antes de leer `argv[1]`. Si no hay argumento, `argv[1]` es `NULL` (ver el ejemplo de [[Depurar con gdb]]).
- **Parámetros `NULL`**: las funciones originales de la libc crashean con `NULL` (es comportamiento indefinido). En libft lo correcto es **imitar al original**; en tus propios proyectos, decide y sé coherente.
- **Formato de los datos**: en push_swap, so_long, etc., valida todo lo que viene de fuera (números, mapas, archivos) antes de usarlo.

## 6. Informar del error y salir

- Los mensajes de error van a la **salida de error (fd 2)**, no a la estándar: `ft_putstr_fd("Error\n", 2);` (ver [[ft_putstr_fd]]). Así no se mezclan con la salida normal del programa.
- El código de salida le dice al sistema si todo fue bien: `return (EXIT_SUCCESS);` (0) o `return (EXIT_FAILURE);` (1), definidos en `<stdlib.h>`.
- Muchos subjects piden un mensaje exacto, por ejemplo `"Error\n"`. Respétalo al pie de la letra.
- Antes de salir por un error, **libera lo que tengas reservado**: el sistema recupera la memoria al terminar, pero valgrind lo marcará y en la evaluación cuenta como leak.

```c
if (argc < 2)
{
	ft_putstr_fd("Error: falta el archivo\n", 2);
	return (EXIT_FAILURE);
}
```

```bash
$ ./prog; echo "código de salida: $?"
Error: falta el archivo
código de salida: 1
```

## 7. Herramientas para encontrar los errores que se escapan

| Herramienta | Encuentra | Uso |
|---|---|---|
| `-Wall -Wextra -Werror` | Variables sin inicializar, tipos incorrectos, retornos olvidados | Siempre al compilar |
| `-fsanitize=address -g` | Desbordamientos de buffer, *use-after-free*, leaks (al ejecutar) | `cc -fsanitize=address -g *.c` |
| `valgrind` | Leaks y lecturas de memoria sin inicializar | `valgrind --leak-check=full ./prog` |
| `gdb` | Dónde y por qué crashea, paso a paso | Ver [[Depurar con gdb]] |

## Checklist

- [ ] Cada `malloc` (y cada función que reserva) tiene su `if (!ptr)`.
- [ ] Si falla un `malloc` a mitad, libero todo lo anterior.
- [ ] Cada `open`/`read`/`write` comprueba el `-1`.
- [ ] Compruebo `argc` antes de usar `argv`.
- [ ] Los errores van por el fd 2 con el mensaje que pide el subject.
- [ ] Salgo con `EXIT_FAILURE` cuando algo falla.
- [ ] `valgrind` no muestra leaks, ni siquiera en los caminos de error.

## Relacionadas

[[Depurar con gdb]] · [[Segmentos de memoria en C]] · [[Arrays en C]] · [[La Norma]] · [[libft - Anexo comparativo]]
