---
title: Equipo y roles
tags:
  - hackathon
  - equipo
---

# Equipo y roles

> [!todo] Asignar
> Los perfiles con experiencia web, en los roles 1, 2 y 3. Copiad el resultado en [[00 - Empieza aquí#1. Qué leer según tu rol]].

**3 en frontend (1, 2 y 3), 2 en backend (4 y 5) y 1 en diseño, contenido y pitch (6).** La interfaz es lo que ve el jurado.

| # | Rol | Persona | Pareja de apoyo |
|---|---|---|---|
| 1 | Coordinación, integración y piezas comunes del frontend | | 4 |
| 2 | Frontend: app del usuario | | 3 |
| 3 | Frontend: panel de Control y canal general | | 2 |
| 4 | Backend: API, autenticación y WebSocket | | 5 |
| 5 | Backend: reglas, YAML, simulador y tarea periódica | | 4 |
| 6 | Diseño, contenido médico, pruebas y pitch | | 1 |

## Rol 1: coordinación, integración y piezas comunes
- Repo, tablero, `AGENTS.md`, `dev.sh`, arranque de Vue.
- Congelar el [[05 - Contrato API]] en H3 y **los datos de prueba** del frontend.
- Piezas comunes: `http.ts`, `ws.ts`, store de autenticación, toasts y aviso de conexión.
- Revisar y fusionar los PRs. Reuniones breves cada 4 horas. Decide los recortes.
- Comprobar **pronto** (en E1) que la red del evento permite que el móvil y el portátil se vean.

## Rol 2: app del usuario
Diseño base del usuario · `StatusBadge` · `IndicatorCard` · Dashboard · Medir (3 pasos) · Resultado · Detalle con `TrendChart` · Recomendaciones · `WellnessModal`.

## Rol 3: panel de Control y canal general
Diseño base de Control · `PresenceDot` · `CrewCard` · vista de tripulación · `AlertItem` y lista en vivo · sonido · detalle del usuario · enviar tarea · botón de bienestar · `ChatPanel` (que se usa en las dos aplicaciones) · mensajes fijados · selector de escenario y reinicio.

## Rol 4: API, autenticación y WebSocket
Arranque de FastAPI · modelos · `seed.py` · autenticación · los 20 endpoints · `ConnectionManager` del WebSocket · eventos.

## Rol 5: reglas, YAML, simulador y tarea periódica
Esquema y cargador del escenario · `classify`, `aggregate`, `evaluation` · tests · alerta automática · `jobs.tick()` (presencia y bienestar) · simulador · volcar los textos del rol 6 al YAML.

## Rol 6: diseño, contenido, pruebas y pitch
Wireframes · paleta · silueta SVG · validar umbrales con los Resources · textos de cada indicador · narrativa de la demo (nombres y avatares) · **probar la app en cada punto de control** · vídeo, presentación, README y entrega.

## Turnos de descanso (noche del sábado al domingo)
Los turnos están pensados para que **nadie duerma durante un punto de control**: ✅ H20 (dom 05:00) con todos despiertos y ✅ H22 (dom 07:00) con el rol 1 (decide los recortes) y el rol 6 (prueba la app) despiertos.

| Franja | Horas | Descansan | Despiertos |
|---|---|---|---|
| sáb 23:00 – dom 02:00 | H14–H17 (justo después de ✅ H14) | 2 y 4 | 1, 3, 5, 6 |
| dom 02:00 – 05:00 | H17–H20 | 3 y 5 | 1, 2, 4, 6 |
| dom 07:30 – 10:00 | H22,5–H25 (tras congelar en ✅ H22) | 1 y 6 | 2, 3, 4, 5 |

> [!warning] Quien presenta el pitch, duerme.
> Mejor que presente alguien de los roles 2–5 (duermen 3 h). Se decide antes del evento: [[00 - Empieza aquí#6. Decisiones pendientes]].
