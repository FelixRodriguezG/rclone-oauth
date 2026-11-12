---
title: Indicadores y umbrales
tags:
  - hackathon
  - contenido-medico
estado: propuesta para decidir en equipo y validar con los Resources el día 1
---

# Indicadores y umbrales

> [!important] Qué entra en cada fase
> **Fase 1:** los **6 indicadores del enunciado**: frecuencia cardíaca, tensión arterial, glucosa, temperatura, sueño y ánimo (4 sistemas: corazón, mente y sueño, metabolismo, inmunidad).
> **Fase 2:** ☢️ radiación (primera mejora) → ⚡ test de reacción → el resto (SpO₂, estrés, peso, agua, ejercicio, dolor, visión…).
> **Escalado:** en la Fase 1 solo **crítico → alerta**. Las reglas de "2 seguidas en atención", tendencias y multisistema son Fase 2.

> [!warning] Propuesta, no valores médicos validados
> Son **rangos orientativos para adultos sanos en reposo**, de fuentes clínicas generales y de valores públicos de NASA. Se validan el día 1 con los **Resources** del reto (issue C-01). Todo se configura en el YAML ([[06 - Escenarios y reglas de salud]]): **cambiar un umbral no requiere tocar código**.

## 1. Estados

| Estado | Significado | Qué hace el sistema | Quién actúa |
|---|---|---|---|
| 🟢 **Normal** | Dentro del rango esperado | Nada | — |
| 🟡 **Atención** | Fuera de rango, sin riesgo inmediato | Muestra las acciones de autocuidado y propone repetir la medición. *(Fase 2: si se repite → alerta `warning`)* | El propio astronauta |
| 🔴 **Crítico** | Riesgo para la salud o la misión | **Alerta `critical` inmediata** a Control, con protocolo de acciones y aviso "Control ha sido avisado" | Control (y Tierra, en la ampliación) |
| ⚪ **Sin datos** | Ninguna medición en 24 h | Se muestra en gris. Nunca cuenta como verde. *(Fase 2: recordatorio)* | El astronauta |

**Reglas de cálculo:**
- Estado de un **sistema** = el peor de sus indicadores. Estado **general** = el peor de sus sistemas.
- *(Fase 2)* Las **reglas de tendencia** pueden subir el estado aunque el valor puntual sea normal. Por ejemplo, perder más del 5 % del peso en 30 días.

## 2. Indicadores propuestos

Columnas: **¿Medible en el espacio?** cómo se haría a bordo · **Frecuencia** la de la rutina por defecto · **Fase** F1 = MVP (se hace) · F2 = mejora (solo si sobra tiempo; ⭐ = las primeras de la lista de [[01 - Visión y alcance|Fase 2]]).

### ❤️ Corazón y circulación
| Indicador | Unidad | ¿Medible en el espacio? | Por qué en el espacio | Frecuencia | Fase |
|---|---|---|---|---|---|
| Frecuencia cardíaca en reposo | lpm | Pulso manual, pulsioxímetro, wearable | Desacondicionamiento cardiovascular; arritmias | Diaria | **F1** |
| Tensión arterial | mmHg | Tensiómetro automático | Redistribución de fluidos; riesgo cardiovascular | Diaria | **F1** |

### 🧠 Mente y sueño
| Indicador | Unidad | ¿Medible? | Por qué | Frecuencia | Fase |
|---|---|---|---|---|---|
| Horas de sueño | h | Autoinforme o wearable | Ciclos de luz alterados, ruido, estrés | Diaria | **F1** |
| Estado de ánimo | 1–5 (caritas) | Autoinforme | Aislamiento y confinamiento | Diaria | **F1** |
| Nivel de estrés | 1–5 | Autoinforme | Carga de trabajo, confinamiento | Diaria | F2 |
| Calidad del sueño | 1–5 | Autoinforme | Complementa las horas | Diaria | F2 |
| Tiempo de reacción (**test en la app**) | ms (respecto a la línea base) | Test de toque de 30 s en la app (inspirado en el test de vigilancia psicomotora, PVT) | Fatiga, deterioro cognitivo | 2 veces por semana | F2 ⭐ |

### 🩸 Metabolismo e hidratación
| Indicador | Unidad | ¿Medible? | Por qué | Frecuencia | Fase |
|---|---|---|---|---|---|
| Glucosa | mg/dL | Glucómetro capilar | Nutrición, estrés, sedentarismo | Diaria (o 3 por semana) | **F1** |
| Peso / masa corporal | kg (cambio respecto a la línea base) | En la ISS existen dispositivos de medida de masa | Nutrición inadecuada, pérdida muscular | Semanal | F2 |
| Agua ingerida | L/día | Autoinforme | Deshidratación; cálculos renales (por la pérdida ósea) | Diaria | F2 |

### 🌡️ Inmunidad
| Indicador | Unidad | ¿Medible? | Por qué | Frecuencia | Fase |
|---|---|---|---|---|---|
| Temperatura | °C | Termómetro | Inmunidad alterada; infecciones en un entorno cerrado | Diaria | **F1** |
| Síntomas (checklist) | sí/no por síntoma | Autoinforme | Detección precoz de infecciones | Si hay fiebre o malestar | F2 |

### 🫁 Respiración
| Indicador | Unidad | ¿Medible? | Por qué | Frecuencia | Fase |
|---|---|---|---|---|---|
| Saturación de oxígeno (SpO₂) | % | Pulsioxímetro | Entorno cerrado (CO₂), fallos del soporte vital | Diaria | F2 |
| Frecuencia respiratoria | rpm | Contar 30 s × 2 | Infección, estrés, CO₂ | Diaria | F2 |

### 🦴 Huesos y músculos
| Indicador | Unidad | ¿Medible? | Por qué | Frecuencia | Fase |
|---|---|---|---|---|---|
| Ejercicio diario | min | Autoinforme o registro de la máquina | Contramedida principal frente a la pérdida ósea y muscular (en la ISS, unas 2 h al día) | Diaria | F2 |
| Dolor | 0–10, con la zona | Autoinforme | Lesiones, dolor de espalda (muy común en microgravedad) | Si hay dolor | F2 |

### ☢️ Radiación (exclusivo del espacio, gran diferenciador)
| Indicador | Unidad | ¿Medible? | Por qué | Frecuencia | Fase |
|---|---|---|---|---|---|
| Dosis diaria | mSv/día | Dosímetro personal (en la demo, **datos simulados** de un sensor) | Cáncer, sistema nervioso central, cataratas; tormentas solares | Automática y diaria | F2 ⭐ |
| Dosis acumulada en la misión | % del límite de la carrera | Calculada | Límite de carrera de NASA (unos 600 mSv; verificar) | Calculada | F2 ⭐ |

### 👁️ Visión (F2)
| Indicador | Unidad | ¿Medible? | Por qué | Frecuencia | Fase |
|---|---|---|---|---|---|
| Visión de cerca, cuestionario | sí/no | Autoinforme o test en pantalla | SANS (síndrome neuro-ocular asociado al vuelo espacial) | Semanal | F2 |

> [!success] Decidido
> **Fase 1:** frecuencia cardíaca, tensión arterial, glucosa, temperatura, sueño y ánimo (exactamente los 6 del enunciado).
> **Fase 2, primeras mejoras:** ⭐ radiación (simulada) y ⭐ test de reacción. El orden completo está en [[01 - Visión y alcance]].

## 3. Umbrales propuestos (escenario Marte, e ISS en la Fase 2; adulto sano)

| Indicador | 🟢 Normal | 🟡 Atención | 🔴 Crítico |
|---|---|---|---|
| Frecuencia cardíaca (lpm) | 50–100 | 40–49 · 101–120 | < 40 · > 120 |
| Tensión arterial (mmHg) | sist. 90–129 **y** diast. 60–84 | sist. 130–159 **o** diast. 85–99 | sist. ≥ 160 **o** diast. ≥ 100 **o** sist. < 90 **o** diast. < 60 |
| Glucosa (mg/dL) | 70–140 | 54–69 · 141–250 | < 54 · > 250 |
| Temperatura (°C) | 36,0–37,5 | 35,0–35,9 · 37,6–38,9 | < 35,0 · ≥ 39,0 |
| Sueño (h) | ≥ 7 | 5–6,9 | < 5 · *(F2: < 6 h tres días seguidos)* |
| Ánimo (1–5) | 4–5 | 3 · *(F2: 2 días seguidos en 3)* | 1–2 · *(F2: bajada de 2 o más puntos en 3 días)* |
| Estrés (1–5) | 1–2 | 3–4 | 5 |
| Calidad del sueño (1–5) | 4–5 | 2–3 | 1 |
| Tiempo de reacción | ≤ 10 % peor que la línea base | 10–25 % peor | > 25 % peor |
| Peso (respecto a la línea base) | variación < 2 % | 2–5 % | > 5 % (en 30 días) |
| Agua (L/día) | ≥ 2,0 | 1,0–1,9 | < 1,0 |
| SpO₂ (%) | ≥ 95 | 91–94 | ≤ 90 |
| Frecuencia respiratoria (rpm) | 12–20 | 10–11 · 21–24 | < 10 · > 24 |
| Ejercicio (min/día) | ≥ 90 | 45–89 | < 45 tres días seguidos |
| Dolor (0–10) | 0–3 | 4–6 | 7–10 |
| Radiación diaria | ≤ 1,5 × la dosis esperada del entorno | 1,5–5 × | > 5 × (posible tormenta solar → **ir al refugio**) |
| Radiación acumulada | < 50 % del límite | 50–80 % | > 80 % |

> [!info] Dosis de referencia (aproximadas, verificar con los Resources)
> - **Superficie de Marte:** unos 0,6–0,7 mSv/día (medido por el instrumento RAD del rover Curiosity).
> - **Tránsito a Marte:** unos 1,8 mSv/día.
> - **ISS:** unos 0,3–0,8 mSv/día.
> - **Tierra:** unos 0,01 mSv/día.

### Diferencias en el escenario residencia (`care_home`)
| Indicador | Cambio | Motivo |
|---|---|---|
| Tensión arterial | Normal hasta 139/89 | En personas mayores el objetivo de tensión suele ser algo más amplio |
| Temperatura | Atención desde 37,3 °C y crítico desde 38,0 °C | Las personas mayores suelen tener menos fiebre ante una infección |
| Sueño | Normal ≥ 6 h | Patrón distinto en la vejez |
| Radiación, reacción | No se usan | No aplican |
| Nuevos (F2) | Caídas (sí/no) e ingesta | Riesgos típicos de una residencia |

## 4. Reglas de escalado

**Fase 1:** solo la fila "1 medición 🔴". El resto es Fase 2.

| Situación | Alerta | A quién |
|---|---|---|
| 1 medición 🟡 | — (solo las acciones de autocuidado) | Usuario |
| 2 mediciones 🟡 seguidas del mismo indicador, o 24 h en 🟡 | `warning` | Control |
| 1 medición 🔴 | `critical` inmediata | Control (y Tierra, en la ampliación) |
| 2 o más sistemas en 🔴 a la vez | `critical` con la etiqueta "**multisistema**" | Control |
| Indicador ⚪ (sin datos) más de 2 veces su frecuencia | `info` | Control |
| Radiación > 5 × | `critical` **para toda la tripulación** | Todos |

## 5. Ficha de contenido por indicador (lo redacta el rol 6)
Por cada indicador del MVP hay que rellenar:
1. **¿Qué es?** (1 o 2 frases sencillas)
2. **¿Se puede medir en el espacio, y cómo?** (paso a paso, de 3 a 5 pasos)
3. **¿Por qué importa en esta misión?** (efecto del entorno)
4. **Acciones en atención** (de 2 a 3)
5. **Acciones en crítico** (de 2 a 4, incluido avisar a Control)
6. **Prevención** (de 1 a 2 hábitos)
7. **Fuente** (enlace)

Este contenido va al YAML (`how_to_measure`, `explanation`, `environment_effect`, `actions`).
