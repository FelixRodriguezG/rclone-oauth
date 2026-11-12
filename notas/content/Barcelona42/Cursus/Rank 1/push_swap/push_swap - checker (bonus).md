---
tags: [proyecto, push-swap, rank-1, bonus, checker]
aliases: ["checker", "checker de push_swap"]
---
# push_swap - checker (bonus)

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · Bonus

> El bonus es escribir **tu propio `checker`**: un programa que recibe los números, **lee operaciones por la entrada estándar**, las ejecuta y dice `OK` si al final `a` está ordenada y `b` vacía. Es el mismo trabajo que push_swap al revés: en vez de **generar** operaciones, las **interpreta**.

> [!warning] Solo cuenta si la parte obligatoria es perfecta
> En push_swap eso incluye **pasar todas las pruebas de rendimiento** sin excepción.

## Qué pide el subject

| Aspecto | Subject v1.0 |
|---|---|
| Programa | `checker` |
| Entrega | `*.h`, `*.c`, con sufijo `_bonus` |
| Makefile | Regla `bonus` |
| Funciones | `read`, `write`, `malloc`, `free`, `exit`, `ft_printf` (o equivalente) |
| libft | Permitida |
| Sin argumentos | Termina sin mostrar nada |
| Correcto | `OK\n` en stdout si `a` está ordenada **y** `b` vacía |
| Incorrecto | `KO\n` en stdout en cualquier otro caso |
| Error | `Error\n` en stderr: argumento no entero, fuera de `int`, duplicado, instrucción que no existe o mal formateada |

```text
$ ./checker 3 2 1 0
rra
pb
sa
rra
pa
OK
$ ./checker 3 2 1 0
sa
rra
pb
KO
$ ./checker 3 2 one 0
Error
$ ./checker "" 1
Error
```

Al teclear a mano, terminas la entrada con **Ctrl+D** (fin de fichero).

## Cómo funciona

```text
argv ──► mismo parseo que push_swap (¡reutilízalo!) ──► pila a
                                                          │
stdin ──► leer línea a línea ──► ¿es una op válida? ──no──► Error (liberar todo)
                                        │ sí
                                        ▼
                                  ejecutar sobre a y b (sin imprimir)
                                        │
                              EOF ──────┘
                                        ▼
                       a ordenada y b vacía ? "OK" : "KO"
```

| Pieza | De dónde sale |
|---|---|
| Parseo y errores de los números | El mismo código de push_swap |
| Operaciones | Las mismas funciones básicas, **sin imprimir** (por eso conviene que `op()` sea la única que imprime) |
| Leer líneas | [[get_next_line - Índice\|get_next_line]] (lo hicisteis en este mismo rank; podéis meterlo en vuestra libft) |
| Pasar de `"rra\n"` a la función | Una [[Tablas de punteros a función\|tabla de despacho]] |

> [!note] ¿Leer todo y luego ejecutar, o ejecutar mientras lees?
> El subject dice que se ejecutan cuando se han leído todas. El resultado es el mismo en los dos casos, salvo en uno: si hay una instrucción inválida al final. Leyendo todo primero, das `Error` sin ejecutar nada; ejecutando sobre la marcha, también das `Error`, solo que después de haber movido las pilas. El subject deja la gestión a criterio de cada grupo.

## Las líneas: qué es válido

get_next_line devuelve la línea **con** su `'\n'`. Compara contra `"sa\n"`, o quita el `'\n'` y compara contra `"sa"`.

| Línea recibida | ¿Válida? |
|---|---|
| `sa\n` | ✅ |
| `rrr\n` | ✅ |
| `sa` (última línea, sin `\n`) | Decisión vuestra; lo habitual es aceptarla |
| `sa \n` (espacio) | ❌ Error |
| `SA\n` | ❌ Error |
| `\n` (línea vacía) | ❌ Error |
| `rra\n` vs `rr` | Ojo: compara la cadena **entera**, no un prefijo |

## Errores frecuentes

> [!danger] Leaks con get_next_line al salir por `Error`
> Si encuentras una instrucción inválida y sales, libera la línea actual y deja que get_next_line libere su `static` (por ejemplo, leyendo hasta `NULL`). Valgrind lo comprobará.

> [!warning] `OK` con `b` no vacía
> `a` puede estar ordenada con elementos aún en `b`. Eso es `KO`.

> [!warning] Cero instrucciones
> `echo -n "" | ./checker 1 2 3` → `OK` (ya ordenado). `echo -n "" | ./checker 2 1` → `KO`.

## Relacionadas

[[push_swap - Índice]] · [[push_swap - Pruebas y depuración]] · [[Tablas de punteros a función]] · [[get_next_line - Índice]] · [[Descriptores de fichero y read]] · [[argc y argv]]

## Referencias

- Subject de push_swap v1.0, capítulo VIII («Parte bonus») — requisitos y ejemplos copiados de ahí.
