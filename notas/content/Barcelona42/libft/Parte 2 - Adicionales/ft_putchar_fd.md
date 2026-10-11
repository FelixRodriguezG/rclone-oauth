---
tags: [libft, libft/parte2]
funcion: ft_putchar_fd
original: ninguna
externas: "write"
estado: pendiente
---
# ft_putchar_fd

[[libft - Índice|← Índice]] · Parte 2 · Funciones adicionales

> Escribe un carácter en un fd

## Prototipo

```c
void ft_putchar_fd(char c, int fd);
```

> [!info] Sin función original
> No existe en la libc, así que no tiene página `man 3`. El comportamiento lo define el **subject** de libft.
> Las citas de `man` de abajo son de las funciones externas que usa.

## Comportamiento según el subject

> [!quote] Subject v19.3 — Descripción
> Envía el carácter ‘c’ al descriptor de archivo (file descriptor) especificado.

| | |
|---|---|
| **Parámetros** | `c`: El carácter a enviar.<br>`fd`: El descriptor de archivo sobre el que escribir. |
| **Valor devuelto** | Nada |
| **Funciones autorizadas** | write |

## Funciones externas (man)

> [!quote] man 2 write — DESCRIPTION / RETURN VALUE
> write() writes up to count bytes from the buffer starting at buf to the file referred to by the file descriptor fd.
>
> On success, the number of bytes written is returned.  On error, -1 is returned, and errno is set to indicate the cause of the error.

> [!note] Traducción
> write() escribe hasta count bytes del buffer que empieza en buf en el fichero al que hace referencia el descriptor de fichero fd.
>
> Si tiene éxito, devuelve el número de bytes escritos. En caso de error, devuelve -1 y pone en errno la causa del error.

## Puntos clave y casos límite

- `write(fd, &c, 1);`
- fd estándar: 0 = stdin, 1 = stdout, 2 = stderr.

## Relacionadas

[[ft_putstr_fd]]

---
*Descripción, parámetros y valor devuelto copiados literalmente del Subject libft v19.3 (es.subject_libft.pdf). Citas de `man`: Linux man-pages 5.10 (Ubuntu 22.04), con traducción propia.*
