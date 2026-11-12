---
title: Resumen en 5 minutos
tags:
  - hackathon
  - resumen
aliases:
  - Resumen
  - TL;DR
fecha: 2026-11-12
estado: resumen del plan a 2026-11-12, incluidas las propuestas pendientes
---

# ⏱️ Resumen en 5 minutos

> [!abstract] Si solo lees una nota, que sea esta
> Resume **todo el plan** en una página. Cada apartado enlaza a la nota completa por si quieres más detalle. Qué leer según tu rol: [[00 - Empieza aquí#1. Qué leer según tu rol]].

## 1. Qué hacemos

**NASA Space Apps 2026** · reto *Health Monitoring Software for Astronauts* · sábado 14 y domingo 15 de noviembre · 42 Barcelona · equipo de 6.

Una **app web local, sin internet**, donde:
- el **astronauta** registra 6 indicadores de salud y ve al momento si está bien (🟢🟡🔴⚪), qué significa y qué hacer;
- **Control** supervisa a toda la tripulación en tiempo real y recibe alertas.

**Por qué:** lejos de la Tierra no se puede esperar 20 minutos a que conteste el médico. La tripulación tiene que evaluarse y actuar sola.

> *Diseñado para Marte, útil en la Tierra*: el mismo sistema sirve en una **residencia de mayores** (cambio de escenario).

## 2. Cómo funciona

**Dos roles:** `user` (astronauta, en el móvil) y `control` (en el portátil).

**6 indicadores (Fase 1):** frecuencia cardíaca · tensión arterial · glucosa · temperatura · horas de sueño · estado de ánimo. Cada uno tiene su "cómo medir" paso a paso. **No hay sensores:** se mide a mano y se escribe el valor, o los genera el simulador para la demo.

**4 estados:**

| Estado | Significa | Qué pasa |
|---|---|---|
| 🟢 Normal | Dentro del rango | Nada |
| 🟡 Atención | Fuera de rango, sin riesgo inmediato | Consejos al astronauta. Control **no** recibe alerta (en la Fase 1) |
| 🔴 Crítico | Riesgo | **Alerta inmediata** a Control |
| ⚪ Sin datos | Sin medir en 24 h | Gris, **nunca** cuenta como verde |

**Quién decide:** un **motor de reglas**, no una IA. Cada valor se compara con los umbrales del escenario, que están en un YAML y se pueden cambiar sin tocar código. Es fiable, rápido, se puede testear y siempre explica el porqué. Un LLM solo serviría para *explicar* resultados (Fase 2): **las reglas deciden, la IA explica**.

**3 tipos de alerta:**

| Tipo | Cuándo salta |
|---|---|
| `health` | Una medición en 🔴 |
| `connection` | El dispositivo deja de enviar su señal de vida (cada 5 s): 🟡 "sin señal" a los 15 s, 🔴 "desconectado, salud desconocida" a los 30 s |
| `wellness` | Control pregunta "¿estás bien?" y el astronauta pide ayuda o no responde en 60 s |

Una alerta pasa por `open` → `acknowledged` (reconocida) → `resolved` (resuelta). **Solo Control puede cambiar su estado.** En la de desconexión está escrito que no se cierra sola al reconectar. *Para `health` y `wellness` no está definido si se cierran solas. Hay una propuesta: P4, en el apartado 9.*

**Además:** Control envía **tareas y recomendaciones** y hay un **canal general** con mensajes fijados.

Detalle: [[02 - Requisitos]] · [[15 - Indicadores y umbrales]] · [[06 - Escenarios y reglas de salud]].

## 3. La demo manda (7 momentos, unos 2 minutos)

1. Control ve a la tripulación (4) con su historial.
2. Una astronauta mide **39,2 °C** → 🔴 con qué hacer.
3. A Control le llega la alerta con sonido en menos de 1 s → la reconoce.
4. Control le envía una recomendación → ella la marca como hecha.
5. Control fija un mensaje en el canal general.
6. Se desconecta otro astronauta → 🟡 a los 15 s → 🔴 a los 30 s.
7. Control lanza una comprobación de bienestar → "Estoy bien". Final: **cambio a Residencia**.

> [!tip] Regla de oro
> **Solo se construye lo que sale en la demo** (Fase 1). Lo demás es Fase 2 y solo empieza si en H20 la demo funciona entera.

Guion con frases: [[11 - Pitch y entrega]].

## 4. Qué nos diferencia

1. **Varios usuarios con roles**, no un dashboard individual más.
2. **Distingue "está mal" de "no sabemos nada de él"** (señal de vida y bienestar).
3. **Escenarios configurables:** Marte ↔ Residencia, con impacto social en la Tierra.
4. **Reglas explicables sin conexión**, no una caja negra.

## 5. Alcance

| ✅ Fase 1 (sí o sí) | ⏳ Fase 2 (por orden, solo si sobra tiempo) |
|---|---|
| Todo lo del apartado 2 · dashboard con tarjetas y **silueta** · detalle con gráfica · simulador de 14 días · escenarios Marte y Residencia · solo modo oscuro | ☢️ radiación → 💬 chats individuales → 🌍 Control en Tierra con retraso → ⚡ test de reacción → 📅 agenda → 📈 escalado → 🎮 gamificación → 🤖 LLM → extras |

Fuera de alcance: sensores reales, registro de usuarios, app nativa, datos médicos reales. **Es un prototipo educativo, no un dispositivo médico.**

Detalle: [[01 - Visión y alcance]].

## 6. Tecnología (en una línea)

**Vue 3 + TypeScript + Tailwind** (frontend) · **FastAPI + SQLite** (backend) · **WebSocket** para el tiempo real (plan B: consultar cada 3 s) · tests con **pytest** y **Vitest** · se arranca con `./dev.sh`.

Detalle: [[03 - Arquitectura y stack]] · Contrato entre frontend y backend: [[05 - Contrato API]] (**se congela en H3**).

## 7. Equipo y calendario

**6 roles:** 1 coordinación e integración · 2 app del usuario · 3 panel de Control · 4 API y tiempo real · 5 reglas y simulador · 6 diseño, contenido y pitch. *Pendiente de asignar.* Detalle: [[07 - Equipo y roles]].

| Hito | Cuándo (aprox.) | Qué tiene que funcionar |
|---|---|---|
| ✅ H3 | sáb 12:00 | El repo arranca y el **contrato queda congelado** |
| ✅ H10 | sáb 19:00 | Medir → alerta en Control en vivo, en dos pantallas |
| ✅ H14 | sáb 23:00 | Si vamos más de 3 h tarde → [[10 - Riesgos y plan de rescate\|plan de rescate]] |
| ✅ H20 | dom 05:00 | **Fase 1 completa** |
| ✅ H22 | dom 07:00 | Congelar funcionalidades |
| ✅ H28 | dom 13:00 | Demo ensayada ×2; lo no fusionado se queda fuera |
| 📦 | dom ~17:00 | **Entregado 2 horas antes** del límite |

Detalle: [[08 - Fases y cronograma]] · Tareas: [[09 - Backlog]].

## 8. Cómo trabajamos

- Cada uno toca **solo su carpeta** (tabla en `AGENTS.md`). Para tocar la de otro, se avisa antes.
- **PRs pequeños** (menos de 300 líneas), fusionados cada 2–3 h, con 1 revisión.
- **La IA parte de `AGENTS.md`** y del contrato: no inventa endpoints ni reescribe lo que funciona. Lee lo que genera antes de hacer commit.
- Un bloqueo de más de 20 minutos se cuenta en `#bloqueos`.

Detalle: [[12 - Convenciones de trabajo]] · [[16 - Uso de IA]].

## 9. 🆕 Propuestas nuevas (pendientes de decidir en equipo)

> [!warning] Todavía no forman parte del plan
> Detalle, costes y preguntas abiertas en [[17 - Propuestas pendientes]]. Si alguna entra en la Fase 1, hay que decidirla **antes de H3**.

| # | Propuesta | En una frase | Coste |
|---|---|---|---|
| P1 | **Varios amarillos → alerta** | Si alguien tiene 2 o más indicadores en 🟡, Control recibe **una** alerta `warning` | Bajo (1 tarea, no cambia el contrato) |
| P2 | **Rangos personales** | Control ajusta lo que es normal para cada persona (por ejemplo, 45 lpm en un deportista). **El 🔴 no se toca**: es igual para todos | Medio (4–6 h, cambia el modelo y el contrato) |
| P3 | **Escenario frente a persona** | El escenario pone el contexto y la seguridad (qué se mide, frecuencia, textos, alertas, límites 🔴 y rangos por defecto); la persona pone su rango normal | Es el marco de P2 |
| P4 | **Cierre de alertas con plazo** | Si el valor vuelve a normal en **60 s** o menos, la alerta se cierra sola con registro. Si dura más, se queda abierta hasta que el astronauta confirma con una medición normal y Control la cierra | Bajo-medio (toca el contrato) |
| P5 | **Dispositivos simulados en directo** | El servidor se hace pasar por un reloj o parche y envía valores cada 5 s. Si falla el sensor → ⚪ *"Sensor offline"* y se mide a mano | Medio-alto (obliga a tener alertas sin duplicados) |

## 10. Lo que falta decidir

- [ ] Asignar los 6 roles y quién presenta el pitch.
- [ ] Hora real de inicio (H0) y hora límite de entrega.
- [ ] Capacidad real: el backlog suma unas 97 h frente a las 75–80 h estimadas.
- [ ] Nombre y logotipo (ideas: HALO, Vitalis, OrbitCare, Pulse Base).
- [ ] Estilo de la silueta.
- [ ] Leer las reglas oficiales (qué se puede preparar antes, uso de IA, marcas de NASA).
- [ ] Las propuestas P1–P5.

Lista completa: [[00 - Empieza aquí#6. Decisiones pendientes]].
