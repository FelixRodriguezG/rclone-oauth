---
title: Visión y alcance
tags:
  - hackathon
  - producto
---

# Visión y alcance

## El problema

En misiones largas, los astronautas están expuestos a los **5 riesgos del vuelo espacial** (NASA Human Research Program):

| Riesgo | Efectos en la salud |
|---|---|
| Radiación espacial | Sistema inmune, cáncer, sistema nervioso central |
| Aislamiento y confinamiento | Sueño, estado de ánimo, rendimiento cognitivo |
| Distancia de la Tierra | Retraso en las comunicaciones, autonomía médica obligatoria |
| Campos gravitatorios alterados | Pérdida ósea y muscular, desacondicionamiento cardiovascular, redistribución de fluidos |
| Entorno cerrado y hostil | CO₂, contaminantes, microbiología, nutrición |

Lejos de la Tierra, **el astronauta debe evaluarse a sí mismo** y la tripulación debe actuar sin esperar al médico en Tierra.

## Nuestra propuesta

> [!tip] Propuesta de valor
> **Una plataforma local de salud para tripulaciones** que convierte mediciones en **estados claros** (🟢🟡🔴⚪), **explica** qué significan y **qué hacer**, y conecta al astronauta con **Control** en tiempo real, **sin depender de internet**.

Lo que nos diferencia:
1. **Varios usuarios con roles** (Usuario y Control): no es un dashboard individual más.
2. **Un estado de salud real, no un fallo de conexión.** El latido de conexión y la comprobación de bienestar separan "está mal" de "no sabemos nada de él".
3. **Escenarios configurables** (Marte ↔ Residencia): el mismo motor sirve en la Tierra, lo que da **impacto social**.
4. **Reglas explicables que funcionan sin conexión**, no una caja negra de IA.

## Usuarios

| Rol | Espacio | Tierra | Fase |
|---|---|---|---|
| **Usuario** | Astronauta | Residente o paciente | 1 |
| **Control** | Control de misión a bordo | Enfermería | 1 |
| **Doctor** | Oficial médico | Médico | 2 |
| **Control en Tierra** | Médico de vuelo en Tierra (con retraso) | — | 2 |

## ✅ Fase 1: MVP

> [!important] Criterio
> Entra solo lo que aparece en la **demo de 7 momentos** ([[00 - Empieza aquí#2. La demo en 7 momentos]]). Todo lo demás es Fase 2.

**Usuario (astronauta)**
- Iniciar sesión (usuarios de demo ya creados).
- **Registrar medición** de los 6 indicadores, con un "cómo medir" paso a paso.
- Ver el **resultado** al guardar: estado, qué significa y qué hacer ("Control ha sido avisado" si es crítico).
- **Dashboard:** tarjetas por indicador, **silueta** como mapa de estado, franja de la **tripulación** (solo estado y conexión).
- **Detalle del indicador:** gráfica temporal, explicación, efecto del entorno y acciones.
- **Recomendaciones:** lista de lo que envía Control y marcar como hecho.
- **Canal general:** leer, escribir y ver los mensajes fijados.
- **Bienestar:** modal "Estoy bien" / "Necesito ayuda".
- Aviso de **"Sin conexión con Control"**.

**Control**
- **Tripulación:** tarjeta por persona con estado, conexión y alertas, más el panel de **alertas en vivo** (reconocer y resolver, con sonido en las críticas) y el canal general.
- **Detalle del usuario:** sus tarjetas y gráficas, sus alertas, enviar **tarea o recomendación**, lanzar **bienestar**.
- **Fijar mensajes** en el canal general.
- Cambiar de **escenario** y **reiniciar la demo**.

**Sistema**
- Motor de reglas: valor → 🟢🟡🔴; **crítico → alerta automática**; sin medición en 24 h → ⚪.
- **Latido de conexión** con valores fijos (5 s, 15 s y 30 s) y alerta "desconectado".
- **Bienestar** con plazo fijo (60 s); "ayuda" o sin respuesta → alerta.
- **Escenarios** `mars` (por defecto) y `care_home`, en YAML.
- **Simulador** de historial (4 tripulantes, unos 14 días).
- Todo en **red local**, sin internet. **Solo modo oscuro.**

## ⏳ Fase 2: mejoras (por orden de prioridad)

| #   | Mejora                                                                                                                                    | Coste | Por qué                             |
| --- | ----------------------------------------------------------------------------------------------------------------------------------------- | ----- | ----------------------------------- |
| 1   | ☢️ **Radiación** (dosis simulada, tarjeta y aura en la silueta, alerta para toda la tripulación)                                          | Bajo  | Es lo más "espacial" y luce mucho   |
| 2   | 💬 **Chats individuales** Control ↔ usuario                                                                                               | Medio | Comunicación privada                |
| 3   | 🌍 **Control en Tierra con retraso** (Marte, unos 20 minutos)                                                                             | Medio | Refuerza el mensaje de la autonomía |
| 4   | ⚡ **Test de reacción** en la app (fatiga cognitiva)                                                                                       | Medio | Medición interactiva y original     |
| 5   | 📅 **Agenda**: fechas límite, recordatorios, rutina diaria                                                                                | Medio | Rutina de la misión                 |
| 6   | 📈 **Escalado**: 2 seguidas en atención, tendencias, multisistema, alertas sin duplicados                                                 | Medio | Más inteligencia                    |
| 7   | 🎮 **Gamificación**: rachas, puntos                                                                                                       | Medio | Motivación                          |
| 8   | 🤖 **Asistente LLM** con textos predefinidos sin conexión                                                                                 | Medio | Explicaciones abiertas              |
| 9   | ☀️ Modo claro · 🩺 rol Doctor · 🛰️ escenario ISS · indicadores extra (SpO₂, estrés, peso…) · chat entre usuarios · mensajes prioritarios | —     | Pulido y extensión                  |

> [!warning] Cómo entra una mejora de la Fase 2
> 1. Solo después del ✅ de H20 (la Fase 1 funciona de principio a fin).
> 2. En **su propia rama**.
> 3. Se fusiona solo si funciona al 100 % **y** no rompe la demo. Si no está lista en H28, se queda fuera.

## ❌ Fuera de alcance
Hardware y sensores reales, registro y recuperación de contraseña, figura 3D, datos médicos reales, aplicación nativa (será una web adaptable).

> [!danger] Aviso legal (en la app y en el pitch)
> Es un **prototipo educativo**, no un dispositivo médico. Los umbrales son orientativos.
