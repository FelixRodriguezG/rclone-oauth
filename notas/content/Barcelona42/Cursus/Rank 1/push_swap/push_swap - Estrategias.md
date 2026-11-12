---
tags: [proyecto, push-swap, rank-1, algoritmos, complejidad]
aliases: ["Estrategias de push_swap", "Algoritmos de push_swap"]
---
# push_swap - Estrategias

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · Estrategias

> [!tip] Una nota por algoritmo
> Esta nota resume. Cada algoritmo del subject (y algunos más) tiene su nota con traza y cifras en [[push_swap - Catálogo de algoritmos]].

> El subject pide **cuatro** estrategias dentro del mismo binario: una O(n²), una O(n√n), una O(n log n) y una **adaptativa** que elige según el [[push_swap - Índice de desorden|índice de desorden]]. Esta nota explica cada una con dibujos, da cifras **medidas** y prepara la justificación de complejidad que tendréis que escribir en el README y defender. Repasa antes [[Notación Big-O]].

> [!important] La regla de oro del subject
> La complejidad se mide en **número de operaciones de push_swap impresas**, no en tiempo de CPU (subject v1.0, VI.3.1). Calcular mucho es gratis; imprimir es lo que cuesta.

## Antes de cualquier estrategia: normalizar

Cambia cada número por su **posición en el orden final** (su índice, de 0 a n−1):

```text
valores:   -50   1000    3    42
índices:     0      3    1     2
```

Ventajas: no hay negativos, sabes que el mínimo es 0 y el máximo n−1, los chunks son rangos de índices y radix necesita solo ⌈log₂ n⌉ bits. Cómo se hace: [[push_swap - Guía paso a paso#4. Normalizar a índices]].

## Atajos antes de elegir

Valen para todas las estrategias y ahorran mucho:

| Situación | Qué hacer | Operaciones |
|---|---|---|
| Ya ordenada | Nada | 0 |
| 2 números | `sa` si están al revés | ≤ 1 |
| 3 números | Tabla de abajo | ≤ 2 |
| 4 o 5 números | Pasar los mínimos a `b`, ordenar 3, devolver | ≤ 6 con 4, ≤ 10 con 5 |
| Ordenada pero rotada (`3 4 5 1 2`) | Llevar el 0 arriba por el lado más corto | ≤ n/2 |

### 3 números

Hay 6 órdenes posibles y todos se resuelven en **2 operaciones como máximo**:

| Orden (top primero) | Operaciones |
|---|---|
| 1 2 3 | — |
| 2 1 3 | `sa` |
| 3 2 1 | `sa` `rra` |
| 3 1 2 | `ra` |
| 1 3 2 | `sa` `ra` |
| 2 3 1 | `rra` |

Regla práctica: si el mayor está arriba, `ra`; si está en medio, `rra`; después, si los dos primeros están al revés, `sa`.

### 4 y 5 números

1. Sube el mínimo por el camino más corto (`ra` o `rra`) y haz `pb`.
2. Con 5, repítelo con el siguiente mínimo.
3. Ordena los 3 que quedan.
4. `pa` las veces necesarias.

Probado con las 24 permutaciones de 4 y las 120 de 5: como máximo **6** y **10** operaciones.

> [!question] ¿Las estrategias forzadas deben usar estos atajos?
> El subject solo pide que cada flag ordene correctamente. Usar los atajos en todas es razonable y fácil de justificar; decidlo en pareja y escribidlo en el README.

---

## 1. Simple O(n²)

**Selection sort con pilas**: el mínimo a `b`, repetir, devolver todo.

```text
mientras a no esté ordenada:
    pos = posición del mínimo en a
    si pos <= tamaño/2:  ra × pos           si no:  rra × (tamaño − pos)
    pb
pa × (todo lo que hay en b)
```

Ejemplo con `5 4 3 2 1` (es exactamente la salida del ejemplo `--simple` del subject):

```text
a = 5 4 3 2 1    mínimo 1 en pos 4 → rra      → a = 1 5 4 3 2 → pb
a = 5 4 3 2      mínimo 2 en pos 3 → rra      → a = 2 5 4 3   → pb
a = 5 4 3        mínimo 3 en pos 2 → rra      → a = 3 5 4     → pb
a = 5 4          mínimo 4 en pos 1 → ra       → a = 4 5       → pb
a = 5                                                         → pb
pa × 5                                                  total: 14
```

**Complejidad:** cada uno de los n elementos cuesta como mucho n/2 rotaciones + 1 `pb` + 1 `pa`:

```text
operaciones ≤ n·(n/2 + 2)  =  O(n²)          memoria extra: O(1)
```

Otras opciones válidas para este tramo: insertion sort (insertar cada elemento en su sitio de `b`), bubble sort con `sa` + `ra`.

---

## 2. Intermedia O(n√n)

**Chunks** (bloques de índices). Con n = 100, unos √100 = 10 chunks de 10 índices cada uno: 0–9, 10–19…

**Fase 1, de `a` a `b`, chunk a chunk:**

```text
para cada chunk [lo, hi):
    mientras quede en a algún índice del chunk:
        busca el más cercano al top, por arriba (ra) o por abajo (rra)
        súbelo y haz pb
        si es de la mitad baja del chunk:  rb     ← lo hunde en b
```

**Fase 2, de `b` a `a`:** sube el **máximo** de `b` por el lado más corto y haz `pa`. Repite hasta vaciar `b`. Como entra siempre el mayor de los que quedan, `a` se construye ordenada desde abajo.

```text
Fase 1 terminada (chunks de 4: 0–3, 4–7, 8–11):

 b, de top a fondo:
   11 10 │ 7 6 │ 3 2 │ 0 1 │ 4 5 │ 8 9
   └─────┴─────┴─────┘     └─────┴─────┘
   mitades altas (pb)      mitades bajas (pb + rb, se hunden al fondo)
   el último chunk arriba  el último chunk al fondo

Fase 2: los números más grandes que quedan están siempre cerca del top
o cerca del fondo, así que se alcanzan con pocos rb o rrb.
```

(El orden exacto dentro de cada grupo depende de la entrada; lo importante es la estructura en capas.)

**Complejidad:**

- Fase 1: hay √n chunks, y para vaciar cada uno basta como mucho una vuelta a `a` (≤ n rotaciones). En total, ≤ √n · n.
- Fase 2: el máximo de `b` está en el grupo de arriba o en el del fondo del chunk más alto que queda (≈ √n elementos), así que cuesta del orden de √n rotaciones por elemento: n · √n.

```text
operaciones = O(n·√n)          memoria extra: O(1) (solo rangos)
```

> [!tip] Cuántos chunks
> El subject sugiere √n. Muchos usan una cantidad fija (5 con 100 números, 11 con 500) y bajan alguna operación más. Si cambias la regla, cambia también el argumento de complejidad: con un número **fijo** de chunks, la fase 2 ya no es O(n√n).

Medido (media / peor de 40 entradas de 100 y 12 de 500): **776 / 811** con 100 y **7 377 / 7 473** con 500. Pasa «bueno» en los dos tamaños, pero no llega a «excelente».

Otras opciones del subject: partición por bloques, buckets con √n cubos, estrategias por rangos.

---

## 3. Compleja O(n log n)

### Radix binario (LSD)

Con los índices normalizados, mira un bit cada vez, del menos significativo al más:

```text
para cada bit (0, 1, 2…) mientras a no esté ordenada:
    repite n veces:
        si el bit del top de a es 0:  pb
        si es 1:                       ra
    pa hasta vaciar b
```

```text
índices: 2 0 3 1          (2 = 10, 0 = 00, 3 = 11, 1 = 01)

bit 0:   2(0)→pb  0(0)→pb  3(1)→ra  1(1)→ra   a = 3 1   b = 0 2
         pa pa                                 a = 2 0 3 1
bit 1:   2(1)→ra  0(0)→pb  3(1)→ra  1(0)→pb   a = 2 3   b = 1 0
         pa pa                                 a = 0 1 2 3  ✅
```

El código, con `(x >> bit) & 1` para leer el bit (ver [[Operadores de bits]]):

```c
static void	radix_sort(t_stacks *s)
{
	int	bit;
	int	i;
	int	size;

	bit = 0;
	size = s->size_a;
	while (!is_sorted(s) && (size - 1) >> bit != 0)
	{
		i = 0;
		while (i++ < size)
		{
			if (((s->a[0] >> bit) & 1) == 0)
				op(s, PB);
			else
				op(s, RA);
		}
		while (s->size_b > 0)
			op(s, PA);
		bit++;
	}
}
```

**Complejidad:** ⌈log₂ n⌉ pasadas (7 con 100, 9 con 500). Cada pasada hace n operaciones (`pb` o `ra`) más los `pa` de vuelta (≤ n):

```text
operaciones ≤ 2n·⌈log₂ n⌉  =  Θ(n log n)       memoria extra: O(1)
```

Es **Θ**: siempre el mismo número de pasadas, sea como sea la entrada. Es el algoritmo más fácil de **demostrar**. Medido: **1 084** con 100 y **6 784** con 500 (el subject usa justo 6 784 como ejemplo de «input largo»).

### Algoritmo de coste («turco»)

Da muchas menos operaciones. La idea: en cada paso, calcula **cuánto costaría** colocar cada elemento en su sitio de la otra pila y mueve el **más barato**.

```text
1. pb, pb.
2. Mientras a tenga más de 3:
     para cada x de a (posición i):
        objetivo en b: el mayor de b menor que x (si no hay, el máximo de b)
        j = posición de ese objetivo
        coste(x) = el mínimo de:
           max(i, j)                         ← los dos con rotaciones hacia arriba (rr)
           max(len_a − i, len_b − j)         ← los dos hacia abajo (rrr)
           i + (len_b − j)                   ← mixto
           (len_a − i) + j                   ← mixto
     ejecuta el x más barato (aprovechando rr/rrr) y pb
3. Ordena los 3 de a.
4. Devuelve cada elemento de b encima del menor de a que sea mayor que él.
5. Sube el 0 arriba.
```

Medido: **558** de media (604 el peor) con 100 y **5 084** (5 259) con 500. Nivel **excelente** en los dos.

> [!warning] Cuidado al llamarlo O(n log n)
> En la práctica funciona como O(n log n), pero en el peor caso teórico cada inserción puede costar ~n/2 rotaciones, y eso daría O(n²). Si lo presentáis como `--complex`, preparad el argumento con honestidad («cota práctica medida» frente a «cota demostrada»). Una opción segura: radix para `--complex` (demostrable) y el de coste dentro de la adaptativa, documentando por qué.

Otras opciones del subject: merge sort con dos pilas, quicksort con partición por pilas, heap sort, árbol binario indexado.

---

## 4. Adaptativa

Mide el desorden y elige. El subject fija los tramos y la complejidad mínima exigida; los métodos internos son libres:

| Desorden | Complejidad exigida | Propuesta |
|---|---|---|
| < 0,2 | O(n) | Conservar la subsecuencia creciente más larga (ver abajo) |
| 0,2 – 0,5 | O(n√n) | Chunks |
| ≥ 0,5 | O(n log n) | Radix o coste |

Orden recomendado:

```text
1. ¿ordenada?                → nada
2. ¿n ≤ 5?                   → atajos
3. ¿ordenada pero rotada?    → rotar (≤ n/2)
4. calcular el desorden y elegir el tramo
```

### Por qué esos umbrales

Argumentos para el README:

- **0,5 es el desorden de una lista aleatoria** (ver [[push_swap - Índice de desorden#Lo que vale con números aleatorios]]). Por encima de 0,5, la entrada es «tan mala o peor que el azar»: no hay estructura que aprovechar, y lo mejor es el algoritmo general más eficiente.
- **Por debajo de 0,2**, la mayoría de los pares ya están en orden: hay mucha estructura que se puede conservar sin moverla.
- **Entre medias**, hay algo de orden, y los chunks lo aprovechan bien.

### El tramo O(n) y su trampa

> [!warning] O(n) exacto es imposible para *cualquier* entrada con desorden < 0,2
> Hay demasiadas permutaciones con menos de un 20 % de inversiones. Por el argumento de conteo de [[Notación Big-O#El límite que nadie puede romper: Ω(n log n)]], algunas necesitan Ω(n log n) operaciones. Lo honesto es decir **para qué tipo de entradas** es O(n) tu método y cuál es su cota general.

Método propuesto: **LIS** (*longest increasing subsequence*, subsecuencia creciente más larga).

```text
1. Calcula la subsecuencia creciente más larga de a (en CPU, no cuesta operaciones).
2. Recorre a una vez: si el top está en la LIS, ra; si no, pb.     ← ≈ n operaciones
3. Inserta cada elemento de b en su sitio de a, eligiendo el más barato (como el de coste).
4. Sube el 0 arriba.
```

Si hay **k** elementos fuera de la LIS, el coste es `n + k·(n/2 + 1)` en el peor caso: **O(n)** cuando k es pequeño (constante), y como mucho O(n·k) en general. Medido con n = 500:

| Entrada (n = 500) | Desorden | LIS | Coste | Chunks | Radix |
|---|---|---|---|---|---|
| 1 elemento fuera de sitio | 0,003 | 831 | 998 | 1 511 | 6 784 |
| 5 fuera de sitio | 0,009 | 842 | 1 005 | 2 099 | 6 784 |
| 50 fuera de sitio | 0,060 | 1 146 | 1 588 | 3 799 | 6 784 |
| Bloques de 10 barajados | 0,009 | 2 159 | 2 322 | 2 212 | 6 784 |
| Bloques de 200 barajados | 0,181 | 3 702 | 3 984 | 4 532 | 6 784 |

Con pocos elementos fuera de sitio, el coste es ~1,7·n (lineal). Con bloques barajados, el desorden es bajo pero hay muchos elementos fuera de la LIS, y el coste crece. Es justo el matiz que hay que explicar.

### Cotas para el README

| Tramo | Método | Tiempo (operaciones) | Espacio extra |
|---|---|---|---|
| Bajo | LIS + inserción | O(n + k·n), O(n) si k es acotado | O(n) (marcar la LIS) |
| Medio | Chunks √n | O(n√n) | O(1) |
| Alto | Radix | Θ(n log n) | O(1) |

---

## Comparativa medida

Entradas aleatorias (desorden ≈ 0,5). Media y peor caso de 40 entradas de 100 y 12 de 500:

| Algoritmo | 100: media (peor) | 500: media (peor) | Clase |
|---|---|---|---|
| Selection | 1 456 (1 592) | 32 377 (33 781) | O(n²) |
| Chunks √n | 776 (811) | 7 377 (7 473) | O(n√n) |
| Radix | 1 084 (1 084) | 6 784 (6 784) | Θ(n log n) |
| Coste («turco») | 558 (604) | 5 084 (5 259) | O(n log n) en la práctica |
| **Límite «excelente»** | **< 700** | **< 5 500** | |

```text
Operaciones con 500 números (escala: █ = 1 000)
Selection   ████████████████████████████████ 32 377
Chunks      ███████▍                          7 377
Radix       ██████▊                           6 784
Coste       █████                             5 084
```

> [!summary] Qué elegir
> - **Para aprobar con buena nota**: chunks en el tramo medio, radix en el alto.
> - **Para «excelente»**: [[push_swap - Rangos (butterfly)|rangos]] en el tramo medio y [[push_swap - Quicksort|quicksort con mediana]] (O(n log n) demostrable) o el algoritmo de coste en el alto. [[push_swap - LIS|LIS]] fue el mejor en todas las pruebas del [[push_swap - Catálogo de algoritmos|catálogo]].
> - Sea como sea, **mide el peor caso** con muchas entradas ([[push_swap - Pruebas y depuración]]).

## Relacionadas

[[push_swap - Índice]] · [[push_swap - Índice de desorden]] · [[push_swap - Guía paso a paso]] · [[push_swap - Defensa y evaluación]] · [[Notación Big-O]] · [[Algoritmos de ordenación y complejidad]] · [[Operadores de bits]] · [[Pilas y colas]]

## Referencias

- Subject de push_swap v1.0, VI.3.3 («Estrategias requeridas») y VI.6 («Prueba de rendimiento»).
- [Wikipedia: Radix sort](https://en.wikipedia.org/wiki/Radix_sort) — variante LSD.
- [Wikipedia: Longest increasing subsequence](https://en.wikipedia.org/wiki/Longest_increasing_subsequence) — algoritmo O(n log n) de cálculo (en CPU).
- [Wikipedia: Selection sort](https://en.wikipedia.org/wiki/Selection_sort) — base de la estrategia simple.
- Todas las cifras: simulador propio de las dos pilas (Python, `random.seed(42)` para las aleatorias y `random.seed(7)` para las de desorden bajo). Selection y radix contrastados con un prototipo en C (Ubuntu clang 12.0.1). Los casos de 3, 4 y 5 números se han comprobado con todas las permutaciones.
