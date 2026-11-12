---
title: Backlog
tags:
  - hackathon
  - tareas
---

# Backlog: tareas pequeñas y en paralelo

> [!info] Cómo está pensado
> - **Cada tarea dura como máximo unas 2 horas**: **S** ≤ 1 h · **M** 1–2 h. Una tarea es **una issue, una rama y un PR** (y un prompt de IA, ver [[16 - Uso de IA]]).
> - **Trabajo en paralelo desde H0:** el frontend trabaja con **datos de prueba** (I-05) y el backend se prueba con `/docs` y `pytest`. Nadie espera a nadie.
> - **"Dep."** solo indica dependencias **duras**. Si la dependencia es un endpoint, el frontend puede empezar con los datos de prueba y conectar la API real después (tarea "Conectar…").
> - **Etiquetas:** `fase-1` `fase-2` · `frontend` `backend` `design` `content` `pitch` `infra` · `user-app` `control` · `S` `M`
> - **Testing:** se testea con código lo que rompería la demo y no se ve a simple vista (reglas, permisos, tiempos, tiempo real). La interfaz se prueba a mano (Q-01…Q-03). Ver [[12 - Convenciones de trabajo#Tests]].
> - **Columnas:** `Backlog` → `Todo` → `In progress` → `Review` → `Done`. **Como máximo 2 tareas en curso por persona.**
> - **Campos de GitHub Projects:** Rol (1–6) · Etapa (E0–E6) · Tamaño (S/M) · Fase (1/2)

## 🚦 Primeras tareas de cada persona (H0)
| Rol | Tareas de arranque (en paralelo)               |
| --- | ---------------------------------------------- |
| 1   | I-01 → I-04 → I-06 → I-05 → I-02               |
| 2   | (espera unos 20 min a I-04) F-01 → F-02 → F-04 → F-03 (cuando esté I-06) |
| 3   | (espera unos 20 min a I-04) K-01 → K-02 → K-03 |
| 4   | B-01 → B-02 → B-03 → B-04                      |
| 5   | L-01 → L-02 → L-03 → L-04                      |
| 6   | D-03 → D-01 → D-02 → C-01                      |

*Orden pensado para que nadie se bloquee: el rol 1 hace I-06 pronto porque el login (F-03) lo necesita; el rol 2 empieza los colores (F-01) con la paleta provisional de [[14 - Diseño de la app]], y el rol 6 cierra la paleta (D-03) primero.*

## 🧩 Hitos de integración
*Son los mismos puntos de control del cronograma (✅ H3, H10, H20, H28) con nombre de hito.*

| Hito | Cuándo | Prueba de "hecho" |
|---|---|---|
| **INT-0** Arranque | H3 | `./dev.sh` levanta los dos servidores; el frontend muestra el login con datos de prueba |
| **INT-1** Recorrido completo | H10 | Login real → medir 39,2 °C → 🔴 → la alerta aparece en Control por WebSocket → reconocer |
| **INT-2** Fase 1 completa | H20 | La demo de 7 momentos funciona sin datos de prueba |
| **INT-3** Lista para presentar | H28 | La demo se ha ensayado 2 veces sin fallos y se puede reiniciar con un botón |

---

# ✅ Fase 1

## Infraestructura y piezas comunes (rol 1)
| ID | Tarea | Hecho cuando… | T | Etapa | Dep. |
|---|---|---|---|---|---|
| I-01 | Crear repo, proteger `main`, plantillas de PR e issue | Existe el repo y `main` exige PR | S | E1 | — |
| I-02 | Etiquetas y tablero de GitHub Projects con sus campos; cargar estas tareas | Están todas las tareas de la Fase 1 en el tablero | S | E1 | I-01 |
| I-03 | `AGENTS.md`, `.gitignore`, `.editorconfig`, esqueleto del README | Están en `main` | S | E1 | I-01 |
| I-04 | Arranque de Vue 3, Vite, TypeScript, Tailwind, Pinia, Router y ESLint | `pnpm dev` muestra una página | S | E1 | I-01 |
| I-05 | **Datos de prueba**: JSON según el contrato y `http.ts` con `VITE_MOCK=1` | Los GET del contrato devuelven datos de prueba | M | E1 | I-04 |
| I-06 | `http.ts`: envoltorio de `fetch` con token, `baseURL` y errores | Login con datos de prueba guarda el token | S | E1 | I-04 |
| I-07 | Store de autenticación y protección de rutas por rol (`/app`, `/control`) | Usuario → `/app` y Control → `/control` | S | E2 | I-06 |
| I-08 | `dev.sh`: arranca backend y frontend | `./dev.sh` levanta los dos | S | E1 | I-04, B-01 |
| I-09 | `ws.ts`: conexión con token y `ping` cada 5 s | El backend registra los pings | S | E2 | B-09 |
| I-10 | `ws.ts`: reconexión con espera creciente y volver a pedir por REST al reconectar | Al reiniciar el backend, el cliente se reconecta solo | S | E3 | I-09 |
| I-11 | `ws.ts`: bus de eventos `on(type, fn)` conectado a los stores | Un `alert.created` actualiza el store de alertas | S | E2 | I-09 |
| I-12 | **Prueba de red del evento**: el móvil abre el frontend del portátil | Funciona en la wifi de la sede, o hay plan B | S | E1 | I-04 |
| I-13 | Toasts globales (`Toast`) | Un evento muestra un toast | S | E3 | I-11 |
| I-14 | Aviso de "Sin conexión con Control" (estado de `ws.ts`) | Al cortar el wifi aparece el aviso | S | E3 | I-10 |
| I-15 | Prueba desde cero en un portátil limpio, siguiendo el README | Funciona en menos de 10 minutos | S | E5 | — |
| I-16 | **Vitest** + `@vue/test-utils` + jsdom configurados; script `pnpm test` | `pnpm test` pasa con un test de ejemplo | S | E1 | I-04 |
| I-17 | Tests de `ws.ts`: bus de eventos y reconexión (WebSocket simulado) | `pnpm test` en verde | S | E3 | I-10, I-16 |
| I-18 | **CI en GitHub Actions**: `ruff` + `pytest` + ESLint + `vitest` en cada PR | Un PR muestra los checks en verde | S | E2 | I-16, L-16 |

## Backend: API (rol 4)
| ID | Tarea | Hecho cuando… | T | Etapa | Dep. |
|---|---|---|---|---|---|
| B-01 | Arranque de FastAPI, CORS y `/api/health` | `/docs` funciona | S | E1 | — |
| B-02 | `db.py` y modelos `User` y `Measurement` | Se crean las tablas | S | E1 | B-01 |
| B-03 | Modelos `Alert`, `Task`, `Message`, `WellnessCheck` y `Setting` | Se crean las tablas | S | E1 | B-02 |
| B-04 | `seed.py`: Control y 4 astronautas | `python seed.py` los crea | S | E1 | B-02 |
| B-05 | `POST /auth/login` con JWT y `GET /me` | Login desde `/docs` | S | E2 | B-04 |
| B-06 | Dependencias `current_user` y `require_control` | Un usuario recibe 403 en una ruta de Control | S | E2 | B-05 |
| B-07 | `POST /measurements` (llama a `rules.evaluate`) | Devuelve `{measurement, evaluation}` | S | E2 | B-06, L-06 |
| B-08 | `GET /measurements` con privacidad | Un usuario recibe 403 al pedir los datos de otro | S | E2 | B-06 |
| B-09 | WS `/ws?token`: `ConnectionManager` (conectar y desconectar) | Hay 2 clientes conectados | M | E2 | B-05 |
| B-10 | WS: `ping` → `pong`, `last_seen` y helpers `send_to_user`, `send_to_control`, `broadcast` | Los helpers funcionan en una prueba manual | S | E2 | B-09 |
| B-11 | Emitir `status.updated` tras cada medición | Control recibe el evento | S | E2 | B-07, B-10 |
| B-12 | `GET /alerts` y `PATCH /alerts/{id}`, con eventos | Reconocer emite `alert.updated` | S | E2 | B-10 |
| B-13 | `GET /users/{id}/status` | Devuelve `UserStatus` | S | E2 | L-05 |
| B-14 | `GET /crew` con los campos según el rol | Un usuario no recibe `systems` | S | E3 | L-05, B-10 |
| B-15 | Tareas: GET, POST y PATCH, con eventos | Control crea una tarea y el usuario la recibe por WebSocket | M | E3 | B-10 |
| B-16 | Mensajes: GET y POST del canal general, con evento | El mensaje llega a todos | S | E3 | B-10 |
| B-17 | `PATCH /messages/{id}` (fijar), solo Control | Emite `message.updated` | S | E3 | B-16 |
| B-18 | Bienestar: `POST` y `/respond`, con eventos | "help" crea una alerta crítica | S | E3 | B-10 |
| B-19 | `GET` y `PUT /scenario`, con `scenario.changed` | Cambiar a `care_home` funciona | S | E3 | L-01 |
| B-20 | `POST /dev/reset` (BD, usuarios y simulador) | Un clic deja la demo como nueva | S | E3 | L-11 |
| B-21 | Tests de API: login y **permisos** (403 en ruta de Control y al pedir datos ajenos) | `pytest` en verde | S | E2 | L-16, B-08 |
| B-22 | Test del **recorrido completo**: medición crítica → alerta creada → evento `alert.created` por WebSocket (`TestClient`) | `pytest` en verde | S | E2 | L-16, B-11, L-07 |

## Backend: lógica (rol 5)
| ID | Tarea | Hecho cuando… | T | Etapa | Dep. |
|---|---|---|---|---|---|
| L-01 | Esquema Pydantic del escenario y `load_scenario(id)` | Carga un YAML de ejemplo | S | E1 | — |
| L-02 | `mars.yaml`: los 6 indicadores con umbrales (textos provisionales) | Pasa la validación | S | E1 | L-01 |
| L-03 | `classify(indicator, value, value2)` → estado | Funciona en la consola | S | E1 | L-02 |
| L-04 | Tests de `classify`: todos los límites de los 6 indicadores | `pytest` en verde | S | E1 | L-03 |
| L-05 | `aggregate(user)`: estado por sistema y general, y `no_data` a las 24 h | Test en verde | S | E2 | L-03 |
| L-06 | `evaluate()` → `Evaluation` (título, textos, acciones según el estado) | Devuelve el texto correcto | S | E2 | L-03 |
| L-07 | Alerta automática al recibir una medición crítica | Una medición de 39,2 crea la alerta y emite `alert.created` | S | E2 | B-07, B-12 |
| L-08 | `bands` derivadas de los umbrales, para la gráfica | `GET /scenario` incluye `bands` | S | E2 | L-02 |
| L-09 | `jobs.tick()` cada 1 s: presencia `stale` y `offline` y `presence.updated` | A los 15 y 30 s se emite el evento | S | E3 | B-10 |
| L-10 | Alerta `connection` al pasar a `offline` | A los 30 s se crea la alerta | S | E3 | L-09 |
| L-11 | Simulador: 14 días × 4 tripulantes, valores normales con ruido | Las gráficas tienen historial | M | E3 | B-03 |
| L-12 | Simulador: un tripulante con tendencia leve a atención | Se ve en la gráfica | S | E3 | L-11 |
| L-13 | `tick()`: bienestar vencido → `expired` y alerta crítica | A los 60 s sin respuesta, alerta | S | E3 | B-18, L-09 |
| L-14 | Volcar los textos del rol 6 a `mars.yaml` | Los textos definitivos están en el YAML | S | E3 | C-02…C-07 |
| L-15 | `care_home.yaml` (umbrales y textos de C-08) | El cambio de escenario funciona | S | E3 | C-08 |
| L-16 | `conftest.py`: SQLite en memoria, `TestClient` y fixtures de usuario y Control con token | Un test de `/health` pasa | S | E2 | B-06 |
| L-17 | Tests de `tick()` con reloj simulado: presencia 15/30 s y bienestar vencido a los 60 s | `pytest` en verde | S | E3 | L-10, L-13, L-16 |

## Frontend: app del usuario (rol 2)
| ID | Tarea | Hecho cuando… | T | Etapa | Dep. |
|---|---|---|---|---|---|
| F-01 | Colores en `tailwind.config` (estados, neutros, acento) y fuentes empaquetadas | Las clases `bg-status-crit`… funcionan | S | E1 | I-04 (paleta provisional de la nota 14; D-03 la ajusta después) |
| F-02 | Diseño base del usuario: cabecera (escenario, conexión) y navegación inferior o lateral | Se navega entre 4 pestañas vacías | S | E1 | I-04 |
| F-03 | Pantalla de login | Login con datos de prueba → `/app` | S | E1 | I-06 |
| F-04 | `StatusBadge` (4 estados, 3 tamaños; color, icono y texto) | Se ve en una página de pruebas | S | E1 | F-01 |
| F-05 | `IndicatorCard` (icono, valor, unidad, estado y hace cuánto) | Se ve con datos de prueba | S | E2 | F-04 |
| F-06 | Stores `scenario` y `me` (estado y mediciones) | Cargan los datos de prueba | S | E2 | I-05 |
| F-07 | Dashboard: cuadrícula de tarjetas desde el store | Salen las 6 tarjetas | S | E2 | F-05, F-06 |
| F-08 | Medir, paso 1: selector de indicador (cuadrícula de iconos) | Lleva al paso 2 | S | E2 | F-06 |
| F-09 | Medir, paso 2: "cómo medir" desde el escenario | Muestra los pasos | S | E2 | F-08 |
| F-10 | Medir, paso 3: campo numérico (doble en la tensión) con validación | Hace el POST con datos de prueba | S | E2 | F-09 |
| F-11 | Pantalla de resultado: estado grande, qué significa, qué hacer y "Control avisado" | Aparece tras el POST | S | E2 | F-10 |
| F-12 | **Conectar con la API real**: login, medición y estado | INT-1 desde el lado del usuario | S | E2 | B-05, B-07, B-13 |
| F-13 | `TrendChart` (ECharts): línea más bandas de umbral | Se ve con datos de prueba | M | E3 | F-06 |
| F-14 | Detalle del indicador: gráfica y secciones (qué es, entorno, acciones, cómo medir) | Navegable desde la tarjeta | S | E3 | F-13 |
| F-15 | Mini gráfica de tendencia en `IndicatorCard` | Se ve en el dashboard | S | E3 | F-13 |
| F-16 | Franja de tripulación (avatar, `StatusBadge` y `PresenceDot`) | Usa `/crew` | S | E3 | K-02, K-03 |
| F-17 | Tareas: lista y "marcar como hecha" | El PATCH funciona | S | E3 | B-15 |
| F-18 | `WellnessModal` a pantalla completa, con cuenta atrás y los 2 botones | Aparece con `wellness.requested` | S | E3 | I-11, B-18 |
| F-19 | Pestaña Canal (usa `ChatPanel` de K-12) | Se envían y reciben mensajes | S | E3 | K-12 |
| F-20 | Escuchar `scenario.changed` y aplicar textos y acento nuevos | Cambia en directo | S | E3 | B-19 |
| F-21 | Tests: `StatusBadge` (4 estados con icono y texto) y validación del formulario de medición (tensión doble) | `pnpm test` en verde | S | E3 | F-04, F-10, I-16 |

## Frontend: panel de Control (rol 3)
| ID | Tarea | Hecho cuando… | T | Etapa | Dep. |
|---|---|---|---|---|---|
| K-01 | Diseño base de Control: barra superior (escenario, críticas, conectados) | Se ve vacía | S | E1 | I-04 |
| K-02 | `PresenceDot` (online, stale, offline) | Se ve en la página de pruebas | S | E1 | F-01 |
| K-03 | `CrewCard` (nombre, presencia, estado, sistemas, número de alertas) | Se ve con datos de prueba | S | E1 | F-04, K-02 |
| K-04 | Store `crew` y vista de tripulación (cuadrícula) | Salen 4 tarjetas | S | E2 | K-03, I-05 |
| K-05 | `AlertItem` y store `alerts` | Se ve la lista con datos de prueba | S | E2 | F-04 |
| K-06 | Panel de alertas en vivo con botones de reconocer y resolver | El PATCH con datos de prueba funciona | S | E2 | K-05 |
| K-07 | **Conectar con la API real**: crew, alerts y eventos WebSocket | INT-1 desde el lado de Control | S | E2 | B-12, B-14, I-11 |
| K-08 | Sonido en las alertas críticas y botón de silenciar | Suena al llegar la alerta | S | E3 | K-07 |
| K-09 | Detalle del usuario: reutiliza `IndicatorCard` y `TrendChart` | Se navega desde `CrewCard` | S | E3 | F-05, F-13 |
| K-10 | Detalle del usuario: formulario de enviar tarea o recomendación | Llega al usuario | S | E3 | B-15 |
| K-11 | Detalle del usuario: botón de bienestar y estado (pendiente, ok, ayuda, vencida) | Funciona de principio a fin | S | E3 | B-18 |
| K-12 | `ChatPanel` compartido: lista, enviar y desplazamiento automático | Funciona en Control | M | E3 | B-16 |
| K-13 | Mensajes fijados: franja superior y fijar o desfijar (solo Control) | Fijar se ve en las dos apps | S | E3 | K-12, B-17 |
| K-14 | Selector de escenario y botón "Reset demo" en la barra superior | Funciona | S | E3 | B-19, B-20 |
| K-15 | Tests del store `alerts`: `alert.created` y `alert.updated` actualizan la lista y el contador de críticas | `pnpm test` en verde | S | E3 | K-07, I-16 |

## Diseño y contenido (rol 6)
| ID | Tarea | Hecho cuando… | T | Etapa | Dep. |
|---|---|---|---|---|---|
| D-01 | Wireframes en FigJam de la app del usuario (Dashboard, Medir, Resultado, Detalle) | Validados por el rol 2 | M | E1 | — |
| D-02 | Wireframes de Control (Tripulación, Detalle del usuario) | Validados por el rol 3 | S | E1 | — |
| D-03 | Paleta final, tipografía y lista de iconos (nombres de iconify) | Está en [[14 - Diseño de la app]] | S | E1 | — |
| D-04 | Silueta SVG con 4 anclas (cabeza, pecho, abdomen, cuello) | El SVG está en `assets/` | M | E2 | D-03 |
| D-05 | `BodySilhouette.vue`: colorear las anclas según los sistemas (con el rol 2) | Se ve en el dashboard | S | E3 | D-04, F-06 |
| D-06 | Pulso CSS en lo crítico y `prefers-reduced-motion` | Late en rojo | S | E3 | D-05 |
| C-01 | Revisar los **Resources** y validar los umbrales | [[15 - Indicadores y umbrales]] actualizado | M | E1 | — |
| C-02 | Textos: frecuencia cardíaca (cómo medir, qué es, entorno, acciones, fuente) | Cumple la ficha de la nota 15 | S | E2 | C-01 |
| C-03 | Textos: tensión arterial | Ídem | S | E2 | C-01 |
| C-04 | Textos: glucosa | Ídem | S | E2 | C-01 |
| C-05 | Textos: temperatura | Ídem | S | E2 | C-01 |
| C-06 | Textos: sueño | Ídem | S | E2 | C-01 |
| C-07 | Textos: ánimo | Ídem | S | E2 | C-01 |
| C-08 | Umbrales y textos de Residencia (`care_home`) | Entregado al rol 5 | S | E3 | C-01 |
| C-09 | Aviso legal (texto y ubicación en la app) | Se ve en el login o el pie | S | E3 | — |
| C-10 | Narrativa: nombres de la tripulación, avatares y guion del evento | Está en `seed.py` y [[11 - Pitch y entrega]] | S | E2 | — |
| Q-01 | Prueba guiada de INT-1 y abrir los errores como issues | Errores en el tablero | S | E2 | INT-1 |
| Q-02 | Prueba guiada de la demo completa (INT-2) | Errores en el tablero | S | E4 | INT-2 |
| Q-03 | Prueba en un móvil real (tamaños y toques) | Errores en el tablero | S | E5 | INT-2 |

## Pitch y entrega (rol 6, con apoyo de todos)
| ID | Tarea | T | Etapa | Dep. |
|---|---|---|---|---|
| P-01 | Nombre e imagen (sin marcas de NASA) | S | E2 | — |
| P-02 | Guion del vídeo y la presentación | S | E5 | INT-2 |
| P-03 | Capturas y grabación de la demo | M | E5 | INT-3 |
| P-04 | Montaje del vídeo | M | E6 | P-03 |
| P-05 | Presentación | M | E6 | P-02 |
| P-06 | README final con capturas | S | E6 | P-03 |
| P-07 | Formulario de Space Apps y entrega | S | E6 | P-04, P-05 |
| P-08 | Ensayo del pitch ×3 | S | E6 | P-05 |

**Total de la Fase 1: 120 tareas** (108 S y 12 M) → **unas 97 horas** (incluye 9 tareas de testing), unas 15–18 por persona.

> [!warning] Capacidad por confirmar
> Aquí se estimaban unas 130 horas disponibles, pero la [[13 - Registro de decisiones|decisión 13]] calcula 75–80 h reales de desarrollo. Si es la segunda cifra, 90 h no caben. Se decide antes del evento ([[00 - Empieza aquí#6. Decisiones pendientes]]).

---

# ⏳ Fase 2 (solo después de INT-2, cada mejora en su rama)

### X1. ☢️ Radiación (prioridad 1)
| ID | Tarea | T | Rol |
|---|---|---|---|
| X1-1 | Indicador `radiation` y sistema en `mars.yaml` (umbrales de la nota 15) | S | 5 |
| X1-2 | Simulador: dosis diaria y evento "tormenta solar" | S | 5 |
| X1-3 | Alerta para toda la tripulación si la dosis supera 5 veces la esperada | S | 5 |
| X1-4 | Tarjeta de radiación y dosis acumulada (% del límite) | S | 2 |
| X1-5 | Aura de radiación en la silueta | S | 6 |
| X1-6 | Botón "Simular tormenta solar" en Control | S | 3 |

### X2. 💬 Chats individuales Control ↔ usuario
| ID | Tarea | T | Rol |
|---|---|---|---|
| X2-1 | `message.channel` y permisos por canal | S | 4 |
| X2-2 | Selector de canal en `ChatPanel` | S | 3 |
| X2-3 | Chat del usuario en el detalle de Control | S | 3 |

### X3. 🌍 Control en Tierra con retraso
| ID | Tarea | T | Rol |
|---|---|---|---|
| X3-1 | Rol `ground` y `deliver_at` (retraso según el escenario) | S | 4 |
| X3-2 | Cola con retraso para alertas y mensajes hacia Tierra | M | 5 |
| X3-3 | Vista de Control en Tierra con cuenta atrás "llega en…" | M | 3 |

### X4. ⚡ Test de reacción
| ID | Tarea | T | Rol |
|---|---|---|---|
| X4-1 | Componente de test de 30 s (toques y tiempos) | M | 2 |
| X4-2 | Línea base y clasificación en % respecto a ella | S | 5 |

### X5 a X9. Resto (por orden)
| ID | Mejora | Tareas principales |
|---|---|---|
| X5 | 📅 Agenda | `task.due_at`, recordatorios en `tick()`, `task_overdue`, rutina diaria |
| X6 | 📈 Escalado | 2 seguidas en atención → `warning`, tendencias, multisistema, alertas sin duplicados |
| X7 | 🎮 Gamificación | Rachas por mediciones diarias, puntos, insignias |
| X8 | 🤖 LLM | `/explain` con textos predefinidos sin conexión, y panel "Pregunta" |
| X9 | Extras | Modo claro, rol Doctor, ISS, SpO₂, estrés, mensajes prioritarios, chat entre usuarios, Docker |
| X10 | 🧪 Test E2E | Playwright con la demo de 7 momentos (instalar los navegadores en `/goinfre`: ocupan mucho disco) |

> [!tip] Crear las issues
> Cuando exista el repo, estas tablas se pueden convertir en issues automáticamente con `gh issue create` (etiquetas, fase y rol) y añadirlas al tablero con `gh project item-add`.
