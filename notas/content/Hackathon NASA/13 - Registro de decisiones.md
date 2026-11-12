---
title: Registro de decisiones
tags:
  - hackathon
  - decisiones
---

# Registro de decisiones

> [!info] Cómo leerlo
> Lo ~~tachado~~ quedó anulado por una decisión posterior (indicada con →). La versión vigente del alcance está en [[01 - Visión y alcance]].

| # | Fecha | Decisión | Motivo |
|---|---|---|---|
| 1 | 2026-11-11 | **Escenarios configurables** (Marte por defecto, ~~ISS~~, residencia) | Se puede replicar en otros entornos y tiene impacto social en la Tierra. *ISS pasa a la Fase 2 (→ 15)* |
| 2 | 2026-11-11 | **Entrada manual con datos simulados**, sin hardware | Menos riesgo; el foco está en la interfaz y la lógica |
| 3 | 2026-11-11 | **Reglas explicables sin conexión, con LLM opcional** | Coherente con "sin internet en Marte"; fiable; muchos equipos apuestan solo por la IA |
| 4 | 2026-11-11 | **Vue 3 + FastAPI + SQLite** (descartado Astro) | Vue para una app interactiva; un servidor local encaja con el reto; permite repartir el trabajo entre 6 |
| 5 | 2026-11-11 | **Silueta como mapa de estado**, con la interacción en tarjetas con iconos | Usabilidad en móvil; cumple la sugerencia del enunciado |
| 6 | 2026-11-11 | **Varios usuarios: Usuario y Control** (Doctor como ampliación) | Una misión real tiene tripulación y Control |
| 7 | 2026-11-11 | Control: ve todo, mensajes públicos ~~y privados~~, tareas y recomendaciones, alertas (~~crear y~~ cambiar estado) | Supervisión completa. *Mensajes privados y alertas manuales pasan a la Fase 2 (→ 15)* |
| 8 | 2026-11-11 | **Privacidad:** el usuario ve el detalle de sus datos y solo un resumen de estado del resto | Confidencialidad médica |
| 9 | 2026-11-11 | **Sistema local**, con latido a 5, 15 y 30 s y comprobación de bienestar (~~rutinaria o urgente~~; en la Fase 1, un solo tipo con plazo de 60 s → 16) | Un estado tiene que ser genuino y no un fallo de conexión; en un accidente, 2 minutos es demasiado |
| 10 | 2026-11-11 | **Canal general:** todos escriben; Control fija mensajes ~~y los marca como prioritarios~~ | Que no se pierda la información relevante. *Mensajes prioritarios pasan a la Fase 2 (→ 15)* |
| 11 | 2026-11-11 | Chat privado entre usuarios y Control en Tierra con retraso: **ampliación** | Prioridad baja frente al MVP |
| 12 | 2026-11-11 | Kanban en **GitHub Projects**; documentación en Obsidian dentro de `docs/` del repo | Une tareas y PRs; la documentación se versiona con el código |
| 13 | 2026-11-11 | **Revisión crítica: el plan era de 1,5 a 2 veces la capacidad real.** Se divide en **Fase 1 (MVP)** y **Fase 2 (mejoras)** | Unas 75–80 horas reales de desarrollo frente a unas 125 que hacían falta |
| 14 | 2026-11-11 | La Fase 1 se define por la **demo de 7 momentos** | Menos es más; solo se construye lo que se enseña |
| 15 | 2026-11-11 | Pasan a la Fase 2: chats individuales, mensajes prioritarios, agenda con fechas y recordatorios, escalado de alertas, escenario ISS, modo claro, rol Doctor y Docker | Reducen mucha complejidad (tareas periódicas, permisos, casos límite) |
| 16 | 2026-11-11 | Valores fijos para el latido (5, 15 y 30 s) y el bienestar (60 s) | No hace falta configurarlos por escenario en el MVP |
| 17 | 2026-11-11 | Equipo de **3 en frontend, 2 en backend y 1 en diseño y pitch** | La interfaz es lo que puntúa |
| 18 | 2026-11-11 | `AGENTS.md` común y responsable por carpeta para trabajar con IA | Coherencia entre 6 personas generando código |
| 19 | 2026-11-11 | Cronograma por **etapas E0–E6**, con el recorrido completo en H10 y la congelación de funcionalidades en H22 | Dar margen realista |
| 20 | 2026-11-11 | **Testing en backend y frontend:** pytest + Vitest + CI. Solo se testea lo que rompería la demo y no se ve (reglas, permisos, tiempos, tiempo real); la interfaz, a mano. E2E en la Fase 2 | Detectar fallos de integración pronto sin gastar el poco tiempo en tests de interfaz |
