---
tags: [proyecto, push-swap, rank-1, pruebas, valgrind, depuracion]
aliases: ["Probar push_swap", "Tests de push_swap"]
---
# push_swap - Pruebas y depuración

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · Pruebas

> Que tu push_swap ordene una entrada no demuestra nada. En la evaluación se prueban errores, flags, tamaños pequeños, leaks y, sobre todo, el **número de operaciones con muchas entradas aleatorias**. Esta nota reúne los comandos para comprobarlo todo antes de que lo haga otra persona.

> [!warning] Ejecuta los scripts con bash
> Los ejemplos usan `ARG="…"; ./push_swap $ARG`. En **zsh**, `$ARG` llega como un único argumento (ver [[argc y argv#Lo que hace la shell antes que tu programa]]). Lánzalos con `bash script.sh` o escribe `${=ARG}` en zsh.

## 1. El checker de la intra

La intra da `checker_linux` (y `checker_Mac`): lee tus operaciones y dice si ordenan.

```bash
chmod +x checker_linux
ARG="4 67 3 87 23"
./push_swap $ARG | ./checker_linux $ARG      # OK / KO
./push_swap $ARG | wc -l                     # cuántas operaciones
```

## 2. Errores

```bash
#!/bin/bash
# errores.sh — cada línea debe imprimir "Error" (por stderr) y nada por stdout
casos=("0 one 2 3" "3 2 3" "2147483648" "-2147483649" "1 + 2" "1 - 2"
       "-0 0" "1 2a" "99999999999999999999" "--fast 1 2")
for c in "${casos[@]}"; do
	out=$(./push_swap $c 2>/dev/null)
	err=$(./push_swap $c 2>&1 >/dev/null)
	printf '%-25s stdout=[%s] stderr=[%s]\n' "$c" "$out" "$err"
done
./push_swap "" 1 2>&1 | cat -e          # argumento vacío → Error$
./push_swap | wc -c                     # sin argumentos → 0
./push_swap 1 2 3 | wc -c               # ya ordenado → 0
```

`cat -e` muestra un `$` al final de cada línea: así compruebas que hay `\n` y que no sobra nada.

## 3. Tamaños pequeños: todas las permutaciones

```bash
#!/bin/bash
# pequenos.sh — prueba todas las permutaciones de 3, 4 y 5 números
perms() { python3 -c "import itertools,sys
for p in itertools.permutations(range(1,int(sys.argv[1])+1)): print(*p)" "$1"; }
for n in 3 4 5; do
	max=0
	while read -r ARG; do
		[ "$(./push_swap $ARG | ./checker_linux $ARG)" = "OK" ] || echo "KO: $ARG"
		ops=$(./push_swap $ARG | wc -l)
		[ "$ops" -gt "$max" ] && max=$ops
	done < <(perms $n)
	echo "n=$n máximo $max operaciones"
done
```

Esperado: como máximo 2 con 3 números, 6 con 4 y 10 con 5 (ver [[push_swap - Estrategias#Atajos antes de elegir]]). Si no tenéis `checker_linux` y el `[ "$(…)" = "OK" ]` falla con «command not found», el script imprimirá `KO` para todo.

## 4. Rendimiento: el peor caso, no uno

```bash
#!/bin/bash
# rendimiento.sh N VECES [flag]   ej.: bash rendimiento.sh 500 50 --complex
n=$1; veces=$2; flag=$3
peor=0; suma=0
for i in $(seq 1 "$veces"); do
	ARG=$(shuf -i 0-99999 -n "$n" | tr '\n' ' ')
	ok=$(./push_swap $flag $ARG | ./checker_linux $ARG)
	ops=$(./push_swap $flag $ARG | wc -l)
	[ "$ok" = "OK" ] || echo "KO con: $ARG"
	suma=$((suma + ops)); [ "$ops" -gt "$peor" ] && peor=$ops
done
echo "n=$n $flag  media=$((suma / veces))  peor=$peor"
```

`shuf -i 0-99999 -n N` saca N números **sin repetir** del rango; `tr` cambia los saltos de línea por espacios.

Ejecuta las cuatro estrategias:

```bash
for f in --simple --medium --complex --adaptive; do bash rendimiento.sh 100 50 $f; done
for f in --medium --complex --adaptive; do bash rendimiento.sh 500 20 $f; done
```

Objetivo con `--adaptive` (el que se evalúa por defecto): el **peor** < 700 con 100 y < 5 500 con 500 para «excelente», o como mínimo < 2 000 y < 12 000.

> [!tip] Mira qué tramo usa cada prueba
> Con `--bench` sabes si una entrada fue al tramo medio o al alto: `./push_swap --bench $ARG 2>&1 >/dev/null | head -2`. Si el peor caso sale siempre con desorden < 0,5, el que hay que optimizar es el tramo medio.

## 5. Entradas con desorden controlado

Para probar la adaptativa en los tres tramos:

```bash
# casi ordenada: 0..499 con 5 elementos cambiados de sitio (desorden muy bajo)
ARG=$(python3 -c "import random
v=list(range(500))
for _ in range(5): v.insert(random.randrange(500), v.pop(random.randrange(500)))
print(*v)")
./push_swap --bench $ARG 2>&1 >/dev/null | head -2

# al revés (desorden 1)
ARG=$(seq 500 -1 1 | tr '\n' ' ')
./push_swap --bench $ARG | ./checker_linux $ARG
```

Prueba también negativos y valores extremos: `./push_swap 2147483647 -2147483648 0`.

## 6. Bench

```bash
ARG=$(shuf -i 0-9999 -n 500 | tr '\n' ' ')
./push_swap --bench $ARG 2> bench.txt | ./checker_linux $ARG   # stdout limpio → OK
cat bench.txt
./push_swap --bench $ARG 2>/dev/null | wc -l                   # igual que total_ops
```

Comprueba que la suma de los 11 contadores es igual a `total_ops` y que este es igual a `wc -l`.

## 7. Memoria

```bash
valgrind --leak-check=full --show-leak-kinds=all ./push_swap 3 2 1 > /dev/null
valgrind --leak-check=full ./push_swap 1 2 a                   # ruta de error
valgrind --leak-check=full ./push_swap --bench $(seq 100 -1 1) > /dev/null
```

Debe acabar con `All heap blocks were freed`, **también** en las rutas de error. Más en [[Valgrind y sanitizers]].

## 8. Depurar un KO

1. Busca la entrada **más pequeña** que falle (empieza con 5, 10, 20 números).
2. Ejecuta tus operaciones a mano con un `main` de depuración que **imprima las dos pilas** después de cada `op()` (detrás de un `#ifdef DEBUG`, ver [[Preprocesador y headers]]).
3. O para en `op` con [[Depurar con gdb|gdb]]: `break op`, `display *s->a@s->size_a`.

| Síntoma | Causa típica |
|---|---|
| `KO` con `b` no vacía | Te falta devolver elementos con `pa` |
| `KO` con todo en `a` pero mal | Falta el giro final (subir el 0) |
| `Error` del checker | Algo que no es una operación sale por stdout (debug o bench) |
| Segfault con 1 o 2 números | Una estrategia asume `size ≥ 3` |
| Muchas más operaciones de lo esperado | Rotas siempre en el mismo sentido en vez de por el lado corto |

## Relacionadas

[[push_swap - Índice]] · [[push_swap - Estrategias]] · [[push_swap - checker (bonus)]] · [[Valgrind y sanitizers]] · [[Depurar con gdb]] · [[argc y argv]]

## Referencias

- Subject de push_swap v1.0, VI.5.1 (ejemplos de uso con `checker_linux`, `shuf` y `wc -l`) y VI.6 (prueba de rendimiento).
- [shuf(1) — man7.org](https://man7.org/linux/man-pages/man1/shuf.1.html) — `-i` (rango) y `-n` (cantidad) sin repetir.
- [cat(1) — man7.org](https://man7.org/linux/man-pages/man1/cat.1.html) — `-e` para ver el final de línea.
