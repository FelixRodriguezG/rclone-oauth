---
title: Escenarios y reglas de salud
tags:
  - hackathon
  - contenido-medico
  - backend
estado: borrador. Validar con los Resources el día 1
---

# Escenarios (formato YAML)

Qué se monitoriza y con qué valores: [[15 - Indicadores y umbrales]]. Esta nota define **cómo se escribe en YAML**. Cambiar un umbral o un texto **no requiere tocar código**.

## Fase 1: dos escenarios
| Escenario | Clave | Diferencias |
|---|---|---|
| 🟠 Base en Marte (por defecto) | `mars` | Vocabulario de misión ("Astronaut", "Crew", "Mission Control"); textos sobre los efectos del espacio |
| 🟣 Residencia de mayores | `care_home` | "Resident", "Residents", "Nursing"; umbrales de tensión, temperatura y sueño adaptados; textos más sencillos |

*Fase 2: `iss`, retraso de comunicaciones, rutina diaria y tiempos por escenario.*

## Formato

```yaml
# backend/scenarios/mars.yaml
id: mars
name: "Mars Surface Base"
accent: "#F97316"
vocabulary:
  user: "Astronaut"
  users: "Crew"
  control: "Mission Control"
systems:
  heart:     { label: "Heart & circulation", icon: "healthicons:heart-organ" }
  brain:     { label: "Mind & sleep",        icon: "healthicons:brain" }
  metabolic: { label: "Metabolism",          icon: "healthicons:blood-drop" }
  immune:    { label: "Immunity",            icon: "healthicons:thermometer" }
indicators:
  heart_rate:
    label: "Heart rate"
    unit: "bpm"
    icon: "lucide:heart-pulse"
    system: heart
    min: 30
    max: 200
    step: 1                       # límites y paso del campo en el formulario
    thresholds:                   # se evalúan en orden: gana la primera que encaja
      - { status: critical,  lt: 40 }
      - { status: critical,  gt: 120 }
      - { status: attention, lt: 50 }
      - { status: attention, gt: 100 }
      - { status: normal }
    how_to_measure:
      - "Rest seated for 5 minutes."
      - "Place two fingers on your wrist."
      - "Count beats for 30 seconds and multiply by 2."
    explanation: "Heart rate is the number of heartbeats per minute."
    environment_effect: "In reduced gravity the heart works less and can decondition."
    actions:
      attention: ["Rest 10 minutes and measure again.", "Drink 500 ml of water."]
      critical:  ["Stop any physical activity and sit down.", "Measure again in 5 minutes.", "Mission Control has been notified."]

  blood_pressure:
    label: "Blood pressure"
    unit: "mmHg"
    dual: true                    # dos valores: value (sistólica) y value2 (diastólica)
    system: heart
    # thresholds con `value2_gt`, `value2_lt`. Gana la condición peor de las dos.
    # …
```

**Reglas de `thresholds`:**
- Condiciones posibles: `lt`, `gt`, `lte`, `gte`, y sus equivalentes `value2_*` para la diastólica.
- Se evalúan **en orden** y gana la primera que encaja. La última, sin condición, es `normal`.
- En la tensión se ponen primero todas las condiciones críticas (de las dos cifras), después las de atención y al final la de normal.
- Las **bandas** de la gráfica (`bands` en el contrato) las calcula el backend a partir de `thresholds`.

## Fuentes para validar
- [NASA Human Research Program](https://www.nasa.gov/hrp/)
- NASA-STD-3001 (vol. 1 y 2)
- La pestaña **Resources** del reto (día 1)
