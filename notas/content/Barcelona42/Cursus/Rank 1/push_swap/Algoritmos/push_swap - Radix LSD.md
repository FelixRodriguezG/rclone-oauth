---
tags: [proyecto, push-swap, rank-1, algoritmos, o-n-log-n, bits]
aliases: ["Radix LSD", "Radix sort en push_swap", "Radix binario"]
---
# push_swap - Radix LSD

[[Inicio|← Inicio]] · [[push_swap - Índice|push_swap]] · [[push_swap - Catálogo de algoritmos|Algoritmos]] · O(n log n)

> **Radix sort LSD** (*Least Significant Digit*) no compara números: los reparte según sus **dígitos**, empezando por el de menos peso. En push_swap los «dígitos» son los **bits** de los índices normalizados. En cada pasada, los que tienen ese bit a 0 van a `b` y los que lo tienen a 1 se rotan. Luego se juntan. Tras ⌈log₂ n⌉ pasadas, todo está ordenado.

## Ficha

| | |
|---|---|
| Familia | O(n log n): «adaptación del orden radix (LSD o MSD)» del subject |
| Operaciones | **Θ(n log n)**: siempre el mismo número, sea cual sea la entrada |
| Memoria extra | O(1) |
| 100 aleatorios | 1 084 (siempre) |
| 500 aleatorios | 6 784 (siempre); es la cifra del ejemplo del subject |
| Al revés / casi ordenada (500) | 6 784 / 6 784 |
| Uso recomendado | `--complex` si priorizas una demostración trivial sobre el número de operaciones |

## La idea original

Ordenar por el último dígito, luego por el penúltimo… sin desordenar lo anterior en cada paso (el reparto es **estable**):

```text
170 45 75 90 802 24 2 66
por unidades: 170 90 802 2 24 45 75 66
por decenas:  802 2 24 45 66 170 75 90
por centenas: 2 24 45 66 75 90 170 802   ✅
```

## Adaptación a dos pilas

Requisito: **índices normalizados** 0…n−1 (ver [[push_swap - Guía paso a paso#4. Normalizar a índices]]). Sin eso, los negativos rompen el bit de signo y harían falta 32 pasadas.

```text
bit = 0
mientras a no esté ordenada y (n − 1) >> bit != 0:
    repetir n veces:
        si el bit «bit» del top de a vale 0:  pb
        si no:                                 ra
    pa hasta vaciar b
    bit++
```

`(x >> bit) & 1` lee el bit (ver [[Operadores de bits]]). El código en C está en [[push_swap - Estrategias#Radix binario (LSD)]].

Por qué funciona: radix LSD solo es correcto si cada pasada es **estable**, es decir, si no cambia el orden relativo de los que tienen el mismo bit. Y aquí lo es:

- Los de bit **1** se rotan con `ra` uno detrás de otro: conservan su orden.
- Los de bit **0** se invierten **dos veces**: al apilarlos en `b` (el primero queda abajo) y al devolverlos con `pa` (el de arriba sale primero). Dos inversiones = orden original.

Al final de la pasada, `a` = [los de bit 0, en su orden] + [los de bit 1, en su orden].

## Traza real

Índices `2 4 1 3 0` (2 = 010, 4 = 100, 1 = 001, 3 = 011, 0 = 000):

```text
--- bit 0 ---
pb    a: 4 1 3 0          b: 2           2 → bit 0 = 0
pb    a: 1 3 0            b: 4 2         4 → 0
ra    a: 3 0 1            b: 4 2         1 → 1
ra    a: 0 1 3            b: 4 2         3 → 1
pb    a: 1 3              b: 0 4 2       0 → 0
pa × 3                    a: 2 4 0 1 3
--- bit 1 ---
ra    a: 4 0 1 3 2                       2 = 010 → 1
pb, pb, pb                b: 1 0 4       4, 0, 1 → 0
ra    a: 2 3                             3 → 1
pa × 3                    a: 4 0 1 2 3
--- bit 2 ---
ra                        a: 0 1 2 3 4   4 = 100 → 1
pb × 4, pa × 4            a: 0 1 2 3 4   (ya ordenada, pero la pasada termina)
total 25
```

## Complejidad

Con índices 0…n−1 hacen falta `⌈log₂ n⌉` bits: 7 para 100 (99 = 1100011), 9 para 500. Cada pasada hace exactamente **n** operaciones (`pb` o `ra`) más tantos `pa` como `pb`:

```text
operaciones ≈ ⌈log₂ n⌉ · (n + n/2)  ≈ 1,5 · n · log₂ n      →  Θ(n log n)
n = 100:  7 · (100 + ~55) ≈ 1 084
```

Es **Θ** porque no depende de la entrada: incluso una lista casi ordenada paga todas las pasadas (salvo que compruebes `is_sorted` entre pasadas, como en el pseudocódigo).

## Mejoras conocidas

| Mejora | Idea |
|---|---|
| No rotar en la última pasada | En el último bit, los de bit 1 ya están en su orden; si solo quedan bits 1, puedes parar antes |
| Radix en base 3 con las dos pilas | Tres «cubetas»: arriba de `b`, fondo de `b` (`pb` + `rb`) y `a`. ⌈log₃ n⌉ pasadas, pero cada una más cara |
| Pasar a [[push_swap - Radix MSD\|MSD]] | Empezar por el bit **más** significativo, y recursivamente: muchas menos operaciones |

## Pros y contras

| ✅ | ❌ |
|---|---|
| Demostración de complejidad inmediata | No pasa de «bueno» (1 084 y 6 784) |
| ~20 líneas de código | No aprovecha ningún orden previo |
| Imposible que dé KO si la normalización está bien | |

## Preguntas de evaluación

> [!question] ¿Por qué normalizas antes del radix?
> «Para que los números sean 0…n−1: sin negativos (el bit de signo rompería el orden) y con solo log₂ n bits en vez de 32.»

## Relacionadas

[[push_swap - Catálogo de algoritmos]] · [[push_swap - Radix MSD]] · [[Operadores de bits]] · [[push_swap - Estrategias#3. Compleja O(n log n)]] · [[Notación Big-O#O, Ω y Θ]]

## Referencias

- [Wikipedia: Radix sort](https://en.wikipedia.org/wiki/Radix_sort) — variantes LSD y MSD, y el ejemplo de 170 45 75…
- Subject de push_swap v1.0, VI.3.3 («Adaptación del orden radix (LSD o MSD)») y VI.5.1 (500 números → 6 784).
- Cifras y traza: simulador propio, contrastado con un prototipo en C (ver [[push_swap - Catálogo de algoritmos]]).
