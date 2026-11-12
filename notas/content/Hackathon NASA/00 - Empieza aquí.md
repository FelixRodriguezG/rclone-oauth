---
title: Empieza aquí
tags:
  - hackathon
  - indice
  - plan
aliases:
  - Inicio
  - Home
  - MOC
  - Plan en una página
  - Resumen
evento: NASA Space Apps Challenge 2026
fechas: 2026-11-14 / 2026-11-15
reto: Create Health Monitoring Software for Astronauts on Space Missions
estado: planificación
---

# 🚀 Empieza aquí

> [!abstract] En una frase
> Plataforma **local, sin internet**, donde los **astronautas** registran 6 indicadores de salud y ven su estado (🟢🟡🔴⚪) con explicación y qué hacer, y **Control** supervisa a la tripulación en tiempo real. El escenario cambia (Marte ↔ Residencia) para demostrar el impacto en la Tierra.
>
> *Diseñado para Marte, útil en la Tierra.*

**Reto:** [Health Monitoring Software](https://www.spaceappschallenge.org/2026/challenges/health-monitoring-software/?tab=details) · **Fechas:** sábado 14 y domingo 15 de noviembre de 2026 · **Sede:** 42 Barcelona (presencial)

> [!tip] Regla de oro
> **Menos es más. La demo manda.** Solo se construye lo que sale en la demo (Fase 1). Las mejoras (Fase 2) empiezan **únicamente** si en ✅ H20 la demo funciona de principio a fin.

## 1. Qué leer según tu rol

No hace falta leerlo todo. **Esta nota + las de tu rol** (≈ 20 minutos).

| Rol | Persona | Lee esto |
|---|---|---|
| 1 · Coordinación e integración | *por asignar* | Todo por encima; a fondo [[05 - Contrato API]], [[09 - Backlog]], [[12 - Convenciones de trabajo]] |
| 2 · Frontend: app del usuario | *por asignar* | [[05 - Contrato API]], [[14 - Diseño de la app]], tu sección de [[09 - Backlog]] |
| 3 · Frontend: panel de Control | *por asignar* | [[05 - Contrato API]], [[14 - Diseño de la app]], tu sección de [[09 - Backlog]] |
| 4 · Backend: API, auth y WebSocket | *por asignar* | [[04 - Modelo de datos]], [[05 - Contrato API]], tu sección de [[09 - Backlog]] |
| 5 · Backend: reglas, YAML y simulador | *por asignar* | [[02 - Requisitos]], [[06 - Escenarios y reglas de salud]], [[15 - Indicadores y umbrales]], tu sección de [[09 - Backlog]] |
| 6 · Diseño, contenido, pruebas y pitch | *por asignar* | [[14 - Diseño de la app]], [[15 - Indicadores y umbrales]], [[11 - Pitch y entrega]] |

**Todos:** [[12 - Convenciones de trabajo]] y [[16 - Uso de IA]] (5 minutos). Detalle de cada rol: [[07 - Equipo y roles]].


## 2. La demo en 7 momentos

1. Control ve a la tripulación (4) con su salud, su conexión y un historial simulado.
2. Una astronauta registra **39,2 °C** siguiendo el "cómo medir" → 🔴, con explicación y qué hacer.
3. A Control le llega la **alerta en tiempo real** con sonido → la reconoce.
4. Control le envía una **recomendación** y ella la marca como hecha.
5. Control **fija** un mensaje en el **canal general**.
6. Se **desconecta** un astronauta → 🟡 sin señal (15 s) → 🔴 alerta "desconectado, salud desconocida" (30 s).
7. Control lanza una **comprobación de bienestar** → "Estoy bien". Final: **cambio a Residencia**.

Guion detallado con frases: [[11 - Pitch y entrega#Guion de la demo (Fase 1, unos 2 minutos, móvil y portátil)]].

## 3. Alcance

| ✅ Fase 1 (MVP: se hace sí o sí) | ⏳ Fase 2 (mejoras, por prioridad) |
|---|---|
| Login con usuarios de demo · roles Usuario y Control | 1. ☢️ Radiación (simulada) |
| 6 indicadores: frecuencia cardíaca, tensión, glucosa, temperatura, sueño y ánimo | 2. 💬 Chats individuales Control ↔ usuario |
| Medir con "cómo medir" → resultado y qué hacer | 3. 🌍 Control en Tierra con retraso |
| Dashboard: tarjetas, silueta y tripulación · detalle con gráfica | 4. ⚡ Test de reacción |
| Control: tripulación, alertas en vivo, detalle del usuario | 5. 📅 Agenda · 6. 📈 Escalado · 7. 🎮 Gamificación · 8. 🤖 LLM |
| Crítico → alerta automática · reconocer y resolver | 9. ☀️ Modo claro · 🩺 Doctor · 🛰️ ISS · más indicadores |
| Recomendaciones y tareas · canal general con mensajes fijados | |
| Latido 5 s → sin señal a los 15 s → alerta a los 30 s | |
| Bienestar: plazo de 60 s; "ayuda" o sin respuesta → alerta | |
| Escenarios Marte y Residencia (YAML) · simulador · solo modo oscuro | |

Detalle y motivos: [[01 - Visión y alcance]].

## 4. Cronograma

H0 = sábado 09:00 (confirmar). H = horas desde el inicio. Detalle: [[08 - Fases y cronograma]].

| Etapa | Horas | Reloj aprox. | Meta | ✅ Punto de control |
|---|---|---|---|---|
| E0 | antes del día 14 | — | Formación, entorno, cuentas, roles | Todos con el entorno funcionando |
| E1 | H0–H3 | sáb 09–12 | Arranque, contrato, wireframes, datos de prueba | **✅ H3:** repo arranca y contrato congelado |
| E2 | H3–H10 | sáb 12–19 | Recorrido completo | **✅ H10:** medir → alerta en Control en vivo |
| E3 | H10–H20 | sáb 19 – dom 05 | Resto de la Fase 1 | **✅ H14:** ¿plan de rescate? · **✅ H20:** Fase 1 completa |
| E4 | H20–H22 | dom 05–07 | Margen y errores | **✅ H22:** congelar funcionalidades |
| E5 | H22–H28 | dom 07–13 | Pulido, ensayos, Fase 2 en ramas | **✅ H28:** demo ensayada ×2; lo no fusionado se queda fuera |
| E6 | H28–H34 | dom 13–19 | Vídeo, presentación, entrega | **Entregado 2 horas antes** |

Turnos de sueño: [[07 - Equipo y roles#Turnos de descanso (noche del sábado al domingo)]].

## 5. Glosario (todos los códigos del plan)

| Código | Significa | Dónde |
|---|---|---|
| **Fase 1 / Fase 2** (F1 / F2) | MVP obligatorio / mejoras opcionales | [[01 - Visión y alcance]] |
| **E0–E6** | Etapa del fin de semana | [[08 - Fases y cronograma]] |
| **H**n | Hora *n* desde el inicio (H0) | [[08 - Fases y cronograma]] |
| **✅ H3, H10, H20, H28** | Puntos de control. En el backlog se llaman **INT-0, INT-1, INT-2, INT-3** (son lo mismo) | [[09 - Backlog#🧩 Hitos de integración]] |
| **Rol 1–6** | Puesto en el equipo (tabla de arriba) | [[07 - Equipo y roles]] |
| **I-** · **B-** · **L-** | Tareas de infraestructura (rol 1) · API backend (rol 4) · lógica backend (rol 5) | [[09 - Backlog]] |
| **F-** · **K-** | Tareas de **F**rontend usuario (rol 2) · panel de **K**ontrol (rol 3). *Ojo: `F-07` es una tarea, no la Fase 1* | [[09 - Backlog]] |
| **D-** · **C-** · **Q-** · **P-** | Tareas de diseño · contenido · pruebas · pitch (rol 6). *Ojo: `C-01` es una tarea, no la historia C1* | [[09 - Backlog]] |
| **X1–X9** | Mejoras de la Fase 2 | [[09 - Backlog]] |
| **U1…, C1…** | Historias de usuario (Usuario, Control) | [[02 - Requisitos]] |
| **RF-n / RNF-n** | Requisito funcional / no funcional | [[02 - Requisitos]] |
| **S / M** | Tamaño de tarea: ≤ 1 h / 1–2 h | [[09 - Backlog]] |

## 6. Decisiones pendientes

- [ ] **Asignar los 6 roles** y escribir los nombres en la tabla de arriba.
- [ ] **Quién presenta el pitch** (esa persona tiene que dormir: mejor uno de los roles 2–5).
- [ ] **Hora real de H0 y hora límite de entrega** (sede y global).
- [ ] **Capacidad real:** la decisión 13 estima 75–80 h de desarrollo; el backlog suma ~97 h de tareas (con testing) y habla de ~130 h disponibles. Acordar una cifra.
- [ ] **Nombre y logotipo** (ideas en [[11 - Pitch y entrega#Nombre]]).
- [ ] **Estilo de la silueta:** contorno de líneas o relleno plano.
- [ ] Leer las **reglas oficiales** de Space Apps: qué se puede preparar antes, formato de entrega, uso de IA, marcas de NASA.
- [ ] Crear el repo y el tablero de GitHub Projects con [[09 - Backlog]].
- [ ] El día 1: revisar los **Resources** y ajustar [[15 - Indicadores y umbrales]].

## 7. Mapa de la documentación

| Nota | Para qué sirve | Estado |
|---|---|---|
| [[01 - Visión y alcance]] | Problema, propuesta, Fase 1 y Fase 2 | ✅ cerrado |
| [[02 - Requisitos]] | Historias de usuario y requisitos | ✅ cerrado |
| [[03 - Arquitectura y stack]] | Tecnologías, diagrama, estructura del repo | ✅ cerrado |
| [[04 - Modelo de datos]] | Tablas SQLite | ✅ cerrado |
| [[05 - Contrato API]] | Endpoints REST y eventos WebSocket | 📝 borrador (se congela en ✅ H3) |
| [[06 - Escenarios y reglas de salud]] | Formato del YAML de escenarios | 📝 borrador (validar con los Resources) |
| [[07 - Equipo y roles]] | Quién hace qué y turnos de sueño | ❓ faltan nombres |
| [[08 - Fases y cronograma]] | Etapas por horas y puntos de control | ❓ falta H0 real |
| [[09 - Backlog]] | 120 tareas pequeñas para GitHub Projects (incluido testing) | ✅ cerrado |
| [[10 - Riesgos y plan de rescate]] | Qué puede salir mal y el MVP mínimo | ✅ cerrado |
| [[11 - Pitch y entrega]] | Guion de la demo, vídeo, presentación | ❓ falta nombre |
| [[12 - Convenciones de trabajo]] | Git, PRs, comunicación | ✅ cerrado |
| [[13 - Registro de decisiones]] | Qué decidimos y por qué | ✅ al día |
| [[14 - Diseño de la app]] | Principios, colores, pantallas, componentes | 📝 propuesta (pasar a FigJam) |
| [[15 - Indicadores y umbrales]] | Qué se mide y valores de cada estado | 📝 propuesta (validar con los Resources) |
| [[16 - Uso de IA]] | Cómo usar la IA sin romper la integración | ✅ cerrado |
