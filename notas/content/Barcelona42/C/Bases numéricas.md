---
tags: [c, bases, binario, hexadecimal, octal, itoa, atoi]
aliases: ["Binario y hexadecimal", "Cambio de base", "itoa_base", "atoi_base"]
---
# Bases numéricas

[[Inicio|← Inicio]] · C

> Un número es una cantidad; cómo lo escribes depende de la **base**. 42 en decimal es `101010` en binario, `52` en octal y `2a` en hexadecimal: es la misma cantidad escrita con distintos símbolos. En C pasar de número a texto en una base, y al revés, son dos bucles cortos con `%` y `/`. Aparecen en `ft_putnbr`, `ft_itoa`, ft_printf (`%x`, `%X`, `%p`) y en ejercicios del examen como `ft_atoi_base` o `print_bits`.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Base** | Cuántos símbolos distintos usa el sistema: 2, 8, 10, 16… |
| **Cifra** (*digit*) | Cada símbolo: `0`–`9` en decimal; `0`–`9` y `a`–`f` en hexadecimal |
| **Peso** | Cuánto vale una posición: en base `b`, la posición `k` (desde la derecha, empezando en 0) vale `b^k` |
| **Binario** | Base 2. Lo que usa el procesador por dentro |
| **Octal** | Base 8. En C, un literal que empieza por `0` |
| **Hexadecimal** | Base 16. En C, un literal que empieza por `0x`. Una cifra = 4 bits |

## Qué significa escribir un número en una base

Cada cifra se multiplica por el peso de su posición:

```text
decimal    4 2           = 4·10 + 2·1                      = 42
binario    1 0 1 0 1 0   = 1·32 + 0·16 + 1·8 + 0·4 + 1·2 + 0·1 = 42
octal      5 2           = 5·8  + 2·1                      = 42
hex        2 a           = 2·16 + 10·1                     = 42
```

### Por qué el hexadecimal está en todas partes

Una cifra hexadecimal son exactamente 4 bits, y dos cifras son un byte. Pasar de binario a hexadecimal es agrupar de 4 en 4:

```text
255  =  1111 1111  =  f f   →  0xff     (un byte lleno)
42   =  0010 1010  =  2 a   →  0x2a
```

Por eso las direcciones de memoria (`%p`), los colores (`#ff8800`) y los volcados de memoria se escriben en hexadecimal: se lee el binario sin tener que contar 32 o 64 ceros y unos.

## Número → texto: divisiones sucesivas

La **última** cifra de `n` en base `b` es `n % b`. Para sacar la siguiente, quitas la última: `n / b`. Repites hasta que `n` sea 0.

```text
42 en base 2:
    42 % 2 = 0     42 / 2 = 21
    21 % 2 = 1     21 / 2 = 10
    10 % 2 = 0     10 / 2 = 5
     5 % 2 = 1      5 / 2 = 2
     2 % 2 = 0      2 / 2 = 1
     1 % 2 = 1      1 / 2 = 0   ← para
cifras de abajo arriba: 101010
```

Las cifras salen **al revés** (de la última a la primera). Hay tres formas de ponerlas en orden:

| Forma | Cómo | Cuándo |
|---|---|---|
| Recursiva | Primero la llamada con `n / b`, luego escribes `n % b` | Cuando escribes directamente (`ft_putnbr_base`) |
| Contar cifras y rellenar desde el final | Calculas la longitud y escribes `buf[len - 1]`, `buf[len - 2]`… | Cuando necesitas el texto antes de escribirlo (ft_printf con banderas, `ft_itoa`) |
| Llenar al revés y darle la vuelta | Escribes en orden inverso y luego inviertes el array | Funciona, pero es un paso más |

### La versión recursiva

```c
static void	put_base(unsigned long n, const char *base)
{
	unsigned long	b;

	b = 0;
	while (base[b])
		b++;
	if (n >= b)
		put_base(n / b, base);
	putchar(base[n % b]);
}
```

Llamadas y salida reales:

```text
put_base(42, "01")                 → 101010
put_base(42, "01234567")           → 52
put_base(42, "0123456789")         → 42
put_base(42, "0123456789abcdef")   → 2a
put_base(255, "0123456789ABCDEF")  → FF
```

- **La base es una cadena**: su longitud es la base y sus caracteres son las cifras. `%x` y `%X` solo se diferencian en la cadena.
- **`unsigned long`**: así no hay negativos que gestionar dentro. El signo se trata fuera (ver más abajo).
- Cómo funciona la recursión por dentro, en [[Recursión]]. La versión que cuenta cifras y rellena desde el final está en [[ft_printf - Funciones auxiliares#`ft_utoa_base`: número a texto en cualquier base|ft_printf - Funciones auxiliares]].

## Texto → número: multiplicar y sumar

Al revés: recorres el texto de izquierda a derecha y por cada cifra haces `n = n * base + valor_de_la_cifra`. (Es el método de Horner: evita calcular potencias).

```text
"2a" en base 16:
    n = 0
    '2' vale 2    n = 0  · 16 + 2  = 2
    'a' vale 10   n = 2  · 16 + 10 = 42
```

```c
static int	ft_index(char c, const char *base)
{
	int	i;

	i = 0;
	while (base[i] && base[i] != c)
		i++;
	if (!base[i])
		return (-1);
	return (i);
}

static long	ft_atoi_base(const char *s, const char *base)
{
	long	n;
	int		len;
	int		d;

	len = 0;
	while (base[len])
		len++;
	n = 0;
	d = ft_index(*s, base);
	while (*s && d != -1)
	{
		n = n * len + d;
		d = ft_index(*++s, base);
	}
	return (n);
}
```

```text
ft_atoi_base("101010", "01")             → 42
ft_atoi_base("2a", "0123456789abcdef")   → 42
ft_atoi_base("ff", "0123456789abcdef")   → 255
ft_atoi_base("52", "01234567")           → 42
ft_atoi_base("12z9", "0123456789")       → 12    ← para en la primera cifra que no es de la base
```

- **El valor de una cifra es su posición en la cadena de la base**: `'a'` está en la posición 10 de `"0123456789abcdef"`. En base 10 eso es lo mismo que `c - '0'`, que es lo que hace `ft_atoi`.
- Compila con `-Wall -Wextra -Werror` y pasa norminette. Un `ft_atoi_base` de examen suele pedir además espacios iniciales, signo y validar la base (sin repetidos, sin `+` ni `-`).

## Negativos

Las bases no tienen signo: el signo es aparte. Hay dos formas de tratar un negativo:

1. **Como hace `%d`**: escribes `-` y luego el valor absoluto. Cuidado con `INT_MIN`: su valor absoluto no cabe en un `int`, hay que pasarlo a `long` antes (ver [[Comportamiento indefinido]]).
2. **Como hacen `%x` y `%u`**: tratas los bits tal cual como un número **sin signo**. En complemento a dos, `-1` son todos los bits a 1:

```text
printf("%x", -1)             → ffffffff
printf("%u", -1)             → 4294967295
printf("%x", -42)            → ffffffd6
printf("%lx", (unsigned long)-1) → ffffffffffffffff
```

Más sobre complemento a dos en [[Tipos enteros y conversiones]] y sobre los bits uno a uno en [[Operadores de bits]].

## Literales en C

```c
printf("%d %d %d\n", 42, 052, 0x2a);    /* 42 42 42 */
printf("%x %X %o\n", 255, 255, 8);      /* ff FF 10 */
```

| Escribes | Base | Vale |
|---|---|---|
| `42` | 10 | 42 |
| `0x2a` o `0X2A` | 16 | 42 |
| `052` | 8 | 42 |
| `0b101010` | 2 | 42, pero **no es C estándar** hasta C23: extensión de gcc y clang. Evítalo en 42 |

> [!danger] El cero de delante cambia el número
> En C, `010` **no es diez**: es octal, vale 8. `printf("%d\n", 010 + 1)` imprime `9`. No pongas ceros delante para alinear números en tu código.

## Errores frecuentes

> [!danger] Los que más aparecen
> - Escribir las cifras en el orden en que salen del `%`: el número sale al revés.
> - Que el `0` no imprima nada: el bucle `while (n > 0)` no entra. Hace falta al menos una cifra.
> - Usar `int` para direcciones o para `%x`: los negativos y los números grandes salen mal. Usa `unsigned int` / `unsigned long`.
> - `-n` con `INT_MIN` antes de pasar a la base.
> - Buffer demasiado pequeño: un `unsigned long` en **binario** tiene hasta 64 cifras, no 20.
> - Poner ceros delante de un literal (`010`) creyendo que es decimal.

## Relacionadas

[[Tipos enteros y conversiones]] · [[Operadores de bits]] · [[Recursión]] · [[Comportamiento indefinido]] · [[printf]] · [[ft_printf - Funciones auxiliares]] · [[ft_printf - Guía paso a paso]] · [[libft - Índice]]

## Referencias

- [cppreference: Integer constant](https://en.cppreference.com/w/c/language/integer_constant): prefijos `0`, `0x` y `0b` (este último desde C23).
- [printf(3), man7.org](https://man7.org/linux/man-pages/man3/printf.3.html): conversiones `o`, `u`, `x`, `X` con argumento `unsigned int`.
- [Wikipedia: Horner's method](https://en.wikipedia.org/wiki/Horner%27s_method).
- Todo el código de esta nota está compilado con `cc -Wall -Wextra` (Ubuntu clang 12.0.1) y las salidas son reales.
