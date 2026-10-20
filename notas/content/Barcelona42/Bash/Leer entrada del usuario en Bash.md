---
tags: [bash, read, entrada, select, menus, interactivo]
aliases: ["read en Bash", "Preguntar al usuario", "select en Bash", "Confirmar s/n"]
---
# Leer entrada del usuario en Bash

[[Inicio|← Inicio]] · [[Bash - Índice|Bash]]

> Para que un script **pregunte** algo se usa `read`: espera a que el usuario escriba y pulse Enter, y guarda el texto en una variable. Con sus opciones puedes mostrar una pregunta, ocultar contraseñas, leer una sola tecla o poner un tiempo límite. `select` crea menús numerados en una línea.

## Vocabulario

| Término | Qué significa |
|---|---|
| **Entrada estándar** (*stdin*) | Por donde llega lo que escribes; descriptor 0 (ver [[Descriptores de fichero y read]]) |
| **Prompt** | El texto que pide el dato: `Nombre: ` |
| **Interactivo** | El script espera a una persona; no sirve si se ejecuta desde otro programa |
| **Timeout** | Tiempo máximo de espera |

## `read` básico

```bash
read -r -p "¿Cómo te llamas? " nombre
echo "Hola, $nombre"
```

- `-p "texto"` → muestra la pregunta en la misma línea.
- `-r` → no interpreta las `\` (ponlo **siempre**, como en [[Bucles en Bash]]).
- Sin nombre de variable, el texto queda en `$REPLY`.

Con varias variables, `read` parte por los espacios y la **última** se queda con el resto:

```bash
read -r -p "Nombre y apellidos: " nombre apellidos
# escribes: Ana García López  →  nombre=Ana  apellidos="García López"
```

## Opciones útiles

| Opción | Qué hace | Ejemplo |
|---|---|---|
| `-s` | No muestra lo que escribes (*silent*) | Contraseñas |
| `-n 1` | Vuelve tras **1** carácter, sin esperar Enter | «Pulsa una tecla» |
| `-t 5` | Se rinde a los 5 segundos (sale con código > 128) | Valor por defecto si no contestan |
| `-e` | Permite editar con flechas y autocompletar con Tab | Pedir rutas |
| `-a nombre` | Guarda las palabras en un array (ver [[Arrays en Bash]]) | Listas |

```bash
read -r -s -p "Contraseña: " clave
echo                                  # -s no imprime el Enter: salta de línea tú
echo "La clave tiene ${#clave} caracteres"

read -r -n 1 -s -p "Pulsa cualquier tecla para seguir..."
echo

if read -r -t 5 -p "Puerto [8080]: " puerto; then
    echo "Has escrito: ${puerto:-8080}"
else
    echo; echo "Sin respuesta: uso 8080"
fi
```

## Valor por defecto

Muestra el valor entre corchetes y usa `${var:-defecto}` (ver [[Expansiones en Bash]]):

```bash
read -r -p "Carpeta destino [$HOME/backup]: " destino
destino="${destino:-$HOME/backup}"
```

## Validar en bucle

Repite la pregunta hasta que la respuesta sirva:

```bash
while true; do
    read -r -p "Edad: " edad || exit 1      # sin esto, al llegar a EOF (Ctrl+D) el bucle no acaba nunca
    if [[ "$edad" =~ ^[0-9]+$ ]] && (( edad <= 120 )); then
        break
    fi
    echo "Escribe un número entre 0 y 120." >&2
done
echo "Edad válida: $edad"
```

## Confirmar s/n

```bash
confirmar() {
    local resp
    read -r -p "$1 [s/N] " resp
    [[ "${resp,,}" == s || "${resp,,}" == si || "${resp,,}" == sí ]]
}

if confirmar "¿Borrar todas las tareas?"; then
    echo "Borrando..."
else
    echo "Cancelado."
fi
```

La `N` mayúscula indica qué pasa si solo pulsas Enter. `${resp,,}` pasa a minúsculas: vale `S` y `s`.

## Menús con `select`

`select` muestra una lista numerada (`1) Ver tareas`, `2) Añadir`…), lee el número y pone la opción elegida en la variable. Repite hasta que hagas `break`.

```bash
PS3="Elige una opción: "            # PS3 = el prompt de select
select opcion in "Ver tareas" "Añadir" "Salir"; do
    case "$opcion" in
        "Ver tareas") echo "Mostrando..." ;;
        "Añadir")     echo "Añadiendo..." ;;
        "Salir")      break ;;
        *)            echo "Número no válido: $REPLY" ;;   # $REPLY = lo escrito
    esac
done
```

`select` es perfecto para menús rápidos. Para algo más bonito (flechas, colores, cuadros) mira [[TUIs en Bash - dialog, whiptail y gum]].

## Cuando no hay nadie al teclado

Si el script se ejecuta desde otro programa o con la entrada redirigida, `read` lee del archivo o de la tubería, no del teclado. Puedes comprobarlo:

```bash
if [[ -t 0 ]]; then          # -t 0: ¿la entrada estándar es una terminal?
    read -r -p "Nombre: " nombre
else
    nombre="anónimo"
fi
```

> [!warning] `read` dentro de un `while read`
> Dentro de `while ... done < archivo`, un `read` interno lee **del archivo**, no del teclado. Para preguntar al usuario ahí, lee de la terminal: `read -r -p "¿Seguro? " resp < /dev/tty`.

## `tareas.sh`: modo interactivo

Un menú para usar el script sin recordar los subcomandos. Usa `listar`, `anadir` y `borrar` de [[Funciones en Bash]] y `confirmar` de arriba:

```bash
menu() {
    local PS3="¿Qué hacemos? " opcion texto num
    select opcion in "Listar" "Añadir" "Borrar" "Salir"; do
        case "$opcion" in
            Listar) listar ;;
            Añadir) read -r -p "Tarea: " texto && anadir "$texto" ;;
            Borrar) read -r -p "Número: " num
                    confirmar "¿Borrar la $num?" && borrar "$num" ;;
            Salir)  break ;;
        esac
    done
}
```

Para que se abra al lanzar el script sin argumentos, en el `case` de [[Funciones en Bash]] cambia `""|lista) listar ;;` por dos ramas: `"") menu ;;` y `lista) listar ;;`. Los subcomandos (`añadir`, `lista`, `borrar`, `help`) siguen funcionando igual.

Siguiente: [[Subshells y procesos en Bash]].

## Errores típicos

| Error | Causa |
|---|---|
| Las `\` desaparecen | Falta `-r` |
| Tras `-s` el siguiente texto sale pegado | Falta un `echo` para el salto de línea |
| El script «no espera» | La entrada viene de un archivo o una tubería |
| `select` repite el menú sin fin | Falta `break` en la opción de salir |
| `(( edad <= 120 ))` da error con letras | Valida primero con `=~ ^[0-9]+$` |

## Relacionadas

[[Funciones en Bash]] · [[Condiciones en Bash]] · [[Expansiones en Bash]] · [[Redirecciones y tuberías]] · [[Subshells y procesos en Bash]] · [[Argumentos y subcomandos en Bash]] · [[TUIs en Bash - dialog, whiptail y gum]] · [[Descriptores de fichero y read]]

## Referencias

- [GNU Bash Manual: Bash Builtins (read)](https://www.gnu.org/software/bash/manual/bash.html#index-read) — opciones `-r`, `-p`, `-s`, `-n`, `-t`, `-e`, `-a`.
- [GNU Bash Manual: Conditional Constructs (select)](https://www.gnu.org/software/bash/manual/bash.html#index-select) — `select`, `PS3` y `REPLY`.
- [GNU Bash Manual: Bash Conditional Expressions](https://www.gnu.org/software/bash/manual/bash.html#Bash-Conditional-Expressions) — `-t fd` para saber si hay terminal.
- [Greg's Wiki: BashGuide/InputAndOutput](https://mywiki.wooledge.org/BashGuide/InputAndOutput) — `read`, entrada estándar y redirecciones.
