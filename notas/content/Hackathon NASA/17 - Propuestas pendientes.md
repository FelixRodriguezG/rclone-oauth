---
title: Propuestas pendientes
tags:
  - hackathon
  - decisiones
  - propuesta
estado: pendiente de decidir en equipo
fecha: 2026-11-12
---

# Propuestas pendientes

> [!warning] Nada de esto está decidido
> Son propuestas para revisar **en equipo**. No cambian ninguna nota cerrada. Si se aprueban, se pasan a [[13 - Registro de decisiones]] y se actualizan las notas afectadas.
> Si alguna se hace en la **Fase 1**, tiene que decidirse **antes de ✅ H3**, cuando se congela el [[05 - Contrato API]].

> [!info] Contexto de capacidad
> La Fase 1 ya va justa: el backlog suma unas 97 h frente a las 75–80 h estimadas (decisión 13). Cada propuesta indica su coste para poder compararla con lo que habría que recortar.

## P1. Varios amarillos → alerta `warning`

**Problema:** en la Fase 1 solo el 🔴 avisa a Control. Alguien con varios indicadores en 🟡 a la vez es invisible para Control, aunque pueda estar peor que alguien con un solo 🔴.

**Propuesta:**
- Si un usuario tiene **N o más indicadores en 🟡**, se crea **una** alerta `warning` para Control.
- Cuenta el **último valor de cada indicador** de las últimas 24 h (lo que está en ⚪ no cuenta).
- **N va en el YAML** del escenario (`attention_alert_min`, propuesta: 2).
- **Sin duplicados:** una sola alerta `warning` abierta por usuario. Si después algo pasa a 🔴, salta la alerta `critical` normal.

**Coste:** 1 tarea S del rol 5 (después de L-07), con su test. **No cambia el contrato:** `severity: 'warning'` ya existe e `indicator` es opcional.

**A decidir:**
- [ ] ¿Se acepta para la Fase 1?
- [ ] ¿N = 2 o 3?
- [ ] ¿Los datos del simulador (L-12) la activan? Si no la activan, no aparece una alerta inesperada al reiniciar la demo. Si la activan, Control ya ve a alguien "en vigilancia" en el momento 1.

## P2. Rangos personales por usuario

**Problema:** un mismo valor no significa lo mismo en todas las personas. Ejemplo: un astronauta muy entrenado puede tener 45 lpm en reposo, que para él es normal, y con las reglas generales le sale 🟡. NASA toma valores de referencia de cada tripulante antes del vuelo.

**Propuesta:** Control define para cada usuario su **rango normal 🟢** de cada indicador. Fuera de ese rango, el valor sale 🟡. **Los límites 🔴 son los del escenario y no se tocan.**

Orden de evaluación:
1. ¿Pasa un límite 🔴 del escenario? → 🔴 (siempre primero)
2. ¿Está dentro del rango personal? → 🟢. Si no lo está → 🟡

**Reglas de seguridad (las aplica el backend):**
- El rango personal tiene que quedar **dentro de los límites críticos**. Si no, el servidor lo rechaza.
- Solo **Control** puede editarlo. El usuario lo ve, pero no lo cambia.
- Se guarda quién lo cambió y cuándo.
- Las mediciones antiguas **no se reclasifican**.
- En la tensión arterial, el rango incluye la sistólica y la diastólica.
- En la interfaz: etiqueta *"Personal range"* y el texto *"outside your personal range set by Mission Control"*.

**Coste aproximado: 4–6 h** entre los roles 2, 3, 4 y 5.

| Parte | Cambio | Rol | Tamaño |
|---|---|---|---|
| [[04 - Modelo de datos]] | Tabla nueva `user_threshold` (usuario, indicador, rango normal y de la diastólica, quién y cuándo) | 4 | S |
| [[05 - Contrato API]] | Leer, guardar y borrar los rangos de un usuario. Las bandas de la gráfica pasan a ser las del usuario | 4 | M |
| Reglas | `classify()` recibe el usuario. Tests: rango personal, críticos intocables, rechazo de rangos inválidos | 5 | S |
| Panel de Control | Editar el rango de cada indicador y volver al valor por defecto | 3 | M |
| App del usuario | Etiqueta y texto de "rango personal" | 2 | S |

**A decidir:**
- [ ] ¿Fase 1 o Fase 2? Si es Fase 2, ¿se deja el hueco en el contrato desde H3?
- [ ] ¿Sale en la demo? Por ejemplo, en el momento 1: la astronauta deportista en 🟢 con la etiqueta.

## P3. Reparto entre escenario y persona

**Idea:** el escenario pone el **contexto y la seguridad**; la persona pone **lo que es normal para ella**.

| Lo define el escenario | Lo define la persona |
|---|---|
| Qué indicadores se miden y cómo | Su rango normal 🟢 de cada indicador |
| Frecuencia de medición | Dónde empieza su 🟡 |
| Textos, vocabulario y recomendaciones | |
| Qué genera alerta y de qué tipo | |
| **Límites críticos 🔴** | |
| **Rangos por defecto** (el punto de partida de alguien nuevo) | |

**Consecuencias:**
- Los umbrales distintos de la Residencia (ver [[15 - Indicadores y umbrales]]) pasan a ser **rangos por defecto**: una residente nueva empieza con valores pensados para personas mayores y Control los ajusta.
- `user_threshold` **no necesita la columna de escenario**.
- La frecuencia por escenario solo se nota en la Fase 2 (recordatorios y alerta `info` por falta de datos). En la Fase 1, ⚪ salta a las 24 h igual para todos.

**A decidir:**
- [ ] **Cambio de escenario en la demo:** las 4 cuentas de demo son las mismas en Marte y en la Residencia. Hay que elegir entre:
  - que `POST /dev/reset` cargue los rangos según el escenario elegido;
  - tener usuarios de demo distintos para cada escenario.

## P4. Cierre de alertas de salud según cuánto dura el valor anómalo

**Problema:** los docs solo dicen que la alerta de desconexión **no** se cierra sola (RF-3). Para `health` no está definido. Si los valores vuelven a la normalidad, tener la alerta abierta para siempre solo genera ruido en Control. Pero un 🔴 tampoco debería desaparecer sin control.

**Propuesta (plazo: 60 s, configurable en el YAML del escenario):**
- La alerta `health` se crea **al momento**, como ahora. Control la recibe en menos de 1 s.
- **El valor vuelve a normal en 60 s o menos** → la alerta se **cierra sola** y queda registro: autor "sistema" y una nota del tipo *"Auto-resolved: back to 37.2 °C after 40 s"*.
- **El valor anómalo dura más de 60 s** → la alerta **se queda abierta** hasta que haya confirmación:
  1. una medición nueva normal del mismo indicador;
  2. el astronauta pulsa *"I feel better"*;
  3. la alerta queda como "recuperada" y **Control la cierra** con un clic.
- Todas las alertas cerradas, también las automáticas, se ven en el historial del usuario. No se borran.

**Qué significa "dura" en cada caso:**
- **Medición manual:** el tiempo entre la medición crítica y la siguiente medición normal del mismo indicador. Volver a medir a mano lleva minutos, así que en la práctica el cierre automático cubre sobre todo los **errores al escribir** (39,2 en vez de 37,2, corregido al momento).
- **Dispositivo (ver P5):** el tiempo que el valor se mantiene fuera de rango. Aquí la regla funciona tal cual: un pico de pocos segundos se cierra solo.

**Coste:** pequeño, pero **toca el contrato y el modelo**: autor "sistema" en `updated_by`, una nota de cierre, el estado o marca "recuperada" y un endpoint para el *"I feel better"* del usuario.

**A decidir:**
- [ ] ¿Se acepta? ¿60 s es un buen plazo?
- [ ] ¿"Recuperada" es un estado nuevo de la alerta (`open` → `acknowledged` → `recovered` → `resolved`) o una marca dentro de `acknowledged`?
- [ ] ¿Se aplica también a `wellness`, por ejemplo con un "Estoy bien" tardío? La alerta `connection` sigue sin cerrarse sola (RF-3).
- [ ] Con dispositivo: ¿la alerta salta al momento y se cierra si se recupera en 60 s, o **solo salta si el valor dura más de 60 s**? La segunda opción evita que suene la alarma por picos pasajeros, pero retrasa el aviso de un 🔴 real.

## P5. Dispositivos simulados en directo, con medición manual como plan B

**Problema:** en el plan actual, todas las mediciones del momento son **manuales**. El simulador (L-11, L-12) solo genera **historial** (14 días) al reiniciar la demo. En una misión real, muchos indicadores llegan de dispositivos (reloj, parche), y la medición manual es el plan B si fallan.

**Propuesta:** no hay hardware. Un programa del servidor **se hace pasar por el dispositivo**:
- Se cuelga de la tarea periódica que ya existe (`jobs.tick()`, L-09). **Cada 5 s**, por cada tripulante, genera su valor habitual con un poco de ruido.
- Se guarda como medición con `source: "device"` y pasa por **las mismas reglas** que una manual.
- Se envía por WebSocket: las tarjetas y las gráficas se mueven en directo.
- **Botones para la demo** (en Control o en un panel de desarrollo): *forzar anomalía* · *desconectar sensor* · *volver a la normalidad*. Así cada momento sale cuando toca.
- **Sensor desconectado** → el indicador sale ⚪ *"Sensor offline"* → la app pide **medir a mano**. Es un estado distinto de la desconexión de la app.

**Qué indicadores encajan:**

| Indicador | Dispositivo |
|---|---|
| Frecuencia cardíaca | ✅ Continuo (reloj o banda) |
| Temperatura | ✅ Continuo (parche) |
| Glucosa | ✅ Cada pocos minutos (sensor continuo) |
| Tensión arterial | 🟡 Periódica (tensiómetro automático) |
| Sueño | 🟡 Una vez al día (lo calcula el reloj) |
| Ánimo | ❌ Siempre a mano |

**Versión mínima:** solo la **frecuencia cardíaca** desde un reloj simulado, con el sensor desconectable y el paso a medición manual. El resto sigue a mano.

**Coste:** simular los valores es lo fácil (unas pocas horas del rol 5). Lo que cuesta es lo que trae tener datos continuos:
- **Alertas sin duplicados obligatorias.** Sin ellas, un valor crítico continuo crearía una alerta cada 5 s. Hoy es una mejora de la Fase 2 (X6).
- La regla del plazo de **P4**.
- **Estado de conexión por sensor**, separado del de la app.
- **Contrato:** `source: 'device'`, el estado del sensor y los eventos nuevos.
- **Gráficas** con muchos más puntos.
- **Demo:** el momento 2 (39,2 °C a mano) sigue valiendo; habría que añadir un momento con el dispositivo.

**A decidir:**
- [ ] ¿Fase 1 (versión mínima) o Fase 2 prioritaria? Si es Fase 2, ¿sustituye a alguna mejora de la lista?
- [ ] ¿Qué indicadores tienen dispositivo?
- [ ] ¿Dónde van los botones de la demo: en el panel de Control o en uno de desarrollo aparte?

## Cómo se relacionan

- **P3** es el marco; **P2** es su implementación.
- **P1** y **P2** se combinan bien: "en amarillo" significaría fuera del rango **personal**.
- **P1** se puede hacer sola y es la más barata.
- **P4** sirve sin **P5** (sobre todo para corregir errores al escribir), pero cobra todo su sentido con datos continuos.
- **P5** obliga a tener alertas sin duplicados (como en P1) y la regla de P4.
- Para el pitch valen aunque no se construyan: *"un rojo avisa al momento, y varios amarillos juntos también"* · *"lo normal lo define cada persona; la seguridad, la misión"* · *"si falla el sensor, la tripulación no se queda a ciegas: mide a mano"*.
