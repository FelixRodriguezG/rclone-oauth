---
tags: [c, memoria, malloc, free, heap, leaks]
aliases: ["malloc", "free", "Memoria dinámica", "Quién libera la memoria"]
---
# malloc y free

[[Inicio|← Inicio]] · C

> `malloc` te presta un trozo de memoria del **heap** que dura hasta que tú lo devuelves con `free`. Lo difícil no es llamarlas, sino **no perder la pista** de cada trozo: saber en cada momento quién es su dueño y quién tiene que liberarlo. Casi todos los leaks y crashes de [[get_next_line - Índice|get_next_line]], [[ft_split]] o [[ft_strjoin]] vienen de ahí.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Heap** (montón) | Zona de memoria para reservas dinámicas. Dura hasta que haces `free` (ver [[Segmentos de memoria en C]]) |
| **Bloque** | El trozo de memoria que te da un `malloc` |
| **Dueño** (*owner*) | La parte del código responsable de hacer `free` de un bloque |
| **Leak** (fuga) | Bloque que ya nadie puede liberar porque se ha perdido su dirección |
| **Puntero colgante** (*dangling pointer*) | Puntero que sigue guardando la dirección de un bloque ya liberado |
| ***Use after free*** | Leer o escribir en un bloque después de liberarlo |
| ***Double free*** | Liberar dos veces el mismo bloque |

## Las dos funciones

```c
#include <stdlib.h>

void	*malloc(size_t size);
void	free(void *ptr);
```

| Función | Qué hace | Devuelve |
|---|---|---|
| `malloc(n)` | Reserva `n` bytes **sin inicializar** (contienen basura) | La dirección del bloque, o `NULL` si no hay memoria |
| `free(p)` | Devuelve el bloque al sistema | Nada |

```c
char	*s;

s = malloc(5);          /* 4 letras + '\0' */
if (!s)
	return (NULL);      /* SIEMPRE comprobarlo */
s[0] = 'h';
s[1] = 'o';
s[2] = 'l';
s[3] = 'a';
s[4] = '\0';
/* ... usar s ... */
free(s);
```

```text
pila (variables locales)            heap
┌──────────────┐               ┌───┬───┬───┬───┬────┐
│ s  ──────────┼─────────────► │ h │ o │ l │ a │ \0 │   5 bytes
└──────────────┘               └───┴───┴───┴───┴────┘
  muere al salir de la función   vive hasta free(s)
```

Fíjate: la variable `s` (el puntero) vive en la **pila** y desaparece al salir de la función. El bloque vive en el **heap** y **no** desaparece solo. Si la función termina sin hacer `free` y sin devolver `s` a nadie, el bloque queda huérfano: es un leak.

> [!tip] Por qué existe el heap
> Las variables locales mueren al salir de la función y su tamaño se decide al compilar. Con `malloc` decides el tamaño **mientras el programa corre** (por ejemplo, según lo que mida una línea) y la memoria sobrevive a la función que la creó. Es lo que permite que [[ft_strdup]] o `get_next_line` **devuelvan** una cadena nueva.

## Las reglas de free

1. **Solo** se puede pasar a `free` una dirección que haya devuelto `malloc` (o `calloc`/`realloc`), **exactamente** esa: no una que apunte a la mitad del bloque.
2. Cada bloque se libera **una sola vez**.
3. Después de `free(p)`, **no** se puede usar `p[...]`: el bloque ya no es tuyo.
4. `free(NULL)` es legal y no hace nada. Esto simplifica mucho el código de error.

```c
char	*p;

p = NULL;
free(p);        /* no pasa nada */
free(NULL);     /* tampoco */
```

```text
$ valgrind ./freenull
==779705== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
```

### Qué pasa si las rompes

```c
s = strdup("hola");
free(s);
printf("%c\n", s[0]);    /* regla 3: use after free */
free(s);                 /* regla 2: double free    */
```

Puede que el programa «funcione» al ejecutarlo normalmente: eso es lo peligroso. Valgrind lo ve (ver [[Valgrind y sanitizers]]):

```text
==779693== Invalid read of size 1
==779693==    at 0x40117F: main (uaf.c:11)
==779693==  Address 0x4a9e040 is 0 bytes inside a block of size 5 free'd
==779693==    at 0x484B27F: free (...)
==779693==    by 0x40117A: main (uaf.c:10)
==779693== Invalid free() / delete / delete[] / realloc()
==779693==    at 0x484B27F: free (...)
==779693==    by 0x40119B: main (uaf.c:12)
```

Valgrind te dice **dónde** lees (línea 11), **dónde** se liberó (línea 10) y dónde está el segundo `free` (línea 12).

## Quién libera cada bloque

> [!important] La regla de la propiedad
> En cada momento, cada bloque tiene **un único dueño**. El dueño es quien debe hacer `free`, o quien debe **pasarle la propiedad** a otro (devolviéndolo, o guardándolo en un sitio que otro liberará).

Las formas en las que la propiedad cambia de manos:

| Situación | Ejemplo | Quién libera |
|---|---|---|
| Una función devuelve memoria nueva | `s = ft_strdup("hola");` | **Quien llama** (el que recibe el puntero) |
| Una función recibe memoria y la «consume» | `stash = ft_strjoin_free(stash, buf);` | **La función**, que libera el viejo y te da uno nuevo |
| Una función recibe memoria solo para leerla | `ft_strlen(s)` | Quien ya era dueño: no cambia nada |
| Se guarda en una estructura | `ft_lstnew(contenido)` | Quien libere la estructura (`ft_lstclear`) |
| Se guarda en una `static` | `static char *stash;` | La propia función, en una llamada futura |

Por eso cada función que reserva memoria debe **documentar** quién la libera. En el subject de libft, por ejemplo, `ft_strjoin` devuelve una cadena «reservada con `malloc`»: eso significa «ahora es tuya, libérala tú».

## El patrón p = f(p)

Es el error más frecuente en get_next_line:

```c
s = strdup("hola");
s = duplicar_y_alargar(s);   /* devuelve un bloque NUEVO con "hola!" */
free(s);
```

```text
antes:    s ──► [h|o|l|a|\0]            (bloque A)
después:  s ──► [h|o|l|a|!|\0]          (bloque B)
                [h|o|l|a|\0]            (bloque A: nadie apunta a él → leak)
```

```text
$ valgrind --leak-check=full ./reasigna
==779689== 5 bytes in 1 blocks are definitely lost in loss record 1 of 1
==779689==    at 0x4848899: malloc (...)
==779689==    by 0x491A54E: strdup (strdup.c:42)
==779689==    by 0x40119D: main (reasigna.c:20)
```

Al escribir `s = ...` has **sobrescrito la única dirección** del bloque A. Hay dos soluciones:

**1. Guardar el viejo y liberarlo tú:**

```c
tmp = duplicar_y_alargar(s);
free(s);
s = tmp;
```

**2. Que la función libere lo que recibe** (lo que hace `ft_strjoin_free` en [[get_next_line - Funciones auxiliares|get_next_line]]):

```c
char	*duplicar_y_alargar(char *viejo)
{
	/* ... crea nuevo ... */
	free(viejo);
	return (nuevo);
}
```

La segunda es más cómoda, pero entonces **nunca** debes usar ni liberar `viejo` después de la llamada: ya no es tuyo.

## Liberar y poner a `NULL`

Después de `free(p)`, `p` sigue guardando la dirección vieja (puntero colgante). Si luego alguien pregunta `if (p)`, parecerá que todavía hay algo. Por eso es buena costumbre:

```c
free(p);
p = NULL;
```

Para hacerlo desde **otra** función necesitas la dirección del puntero (`char **`), porque C pasa los argumentos por copia (ver [[Punteros en C]]):

```c
static void	liberar_mal(char *p)
{
	free(p);
	p = NULL;          /* cambia la copia, no el original */
}

static void	liberar_bien(char **p)
{
	free(*p);
	*p = NULL;         /* cambia el puntero de quien llama */
}
```

```text
$ ./ftfree
a = 0x... (sigue apuntando a memoria liberada)
b = (nil)
```

## Comprobar `malloc` y liberar al fallar

`malloc` puede devolver `NULL`. Si lo usas sin comprobar, segfault. Y si falla a mitad de un proceso que ya había reservado otras cosas, **hay que liberar lo anterior** antes de devolver `NULL` (más en [[Control de errores en C#Liberar lo reservado antes de salir]]).

```c
buf = malloc(BUFFER_SIZE + 1);
if (!buf)
	return (NULL);
line = malloc(len + 1);
if (!line)
{
	free(buf);          /* lo de antes también */
	return (NULL);
}
```

## Errores frecuentes

> [!danger] No comprobar `malloc`
> `s = malloc(n); s[0] = 'a';` → si `malloc` devolvió `NULL`, segfault. El evaluador a veces fuerza fallos de `malloc` para comprobarlo.

> [!danger] Olvidar el `+ 1` del `'\0'`
> `malloc(ft_strlen(s))` para copiar `s`: el terminador se escribe fuera del bloque. Valgrind: `Invalid write of size 1`.

> [!warning] Liberar algo que no es tuyo
> `free` de una cadena literal (`"hola"`), de un array local o de un puntero movido (`free(s + 2)`) → `Invalid free()` o crash.

> [!warning] Reservar en la pila lo que puede ser enorme
> `char buf[10000000];` en una función: la pila suele medir 8 MB (`ulimit -s`) y el programa muere. Para tamaños grandes o que no sabes al compilar, `malloc`.

> [!question] Pregunta típica de evaluación
> **«¿Quién libera la línea que devuelve tu función?»**
> Quien la llama. Mi función reserva la línea con `malloc` y le pasa la propiedad al `main`, que hace `free` cuando ya no la necesita. Lo que mi función guarda para la siguiente llamada (el stash) lo libera ella misma al llegar al final o si hay un error.

## Relacionadas

[[Segmentos de memoria en C]] · [[Control de errores en C]] · [[Punteros en C]] · [[Valgrind y sanitizers]] · [[Variables static y ámbito]] · [[ft_strdup]] · [[ft_strjoin]] · [[ft_calloc]] · [[get_next_line - Funciones auxiliares]] · [[get_next_line - Guía paso a paso]]

## Referencias

- [malloc(3), man7.org](https://man7.org/linux/man-pages/man3/malloc.3.html): `malloc` devuelve memoria sin inicializar o `NULL`; `free(NULL)` no hace nada; liberar dos veces es comportamiento indefinido.
- [cppreference: free](https://en.cppreference.com/w/c/memory/free): solo punteros devueltos por las funciones de reserva.
- [Valgrind: Memcheck manual](https://valgrind.org/docs/manual/mc-manual.html): *Invalid read*, *Invalid free*, *definitely lost*.
- Salidas obtenidas con Ubuntu clang 12.0.1 y Valgrind en Ubuntu 22.04.
