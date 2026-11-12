---
title: Requisitos
tags:
  - hackathon
  - requisitos
---

# Requisitos

**F1** = Fase 1 (MVP) · **F2** = Fase 2 (mejoras). Alcance en [[01 - Visión y alcance]].

## Historias de usuario: Fase 1

### Usuario
| ID | Historia |
|---|---|
| U1 | Como usuario, quiero **iniciar sesión** para acceder a mis datos. |
| U2 | Quiero **registrar una medición** de los 6 indicadores en 3 toques como máximo. |
| U3 | Quiero ver **cómo se mide** cada indicador antes de registrarlo. |
| U4 | Quiero ver el **resultado**: estado, qué significa y qué hacer. |
| U5 | Quiero ver **mi estado general** de un vistazo (tarjetas y silueta). |
| U6 | Quiero ver la **evolución** de un indicador en una gráfica. |
| U7 | Quiero ver el **estado de mis compañeros** (solo colores y conexión). |
| U8 | Quiero ver las **recomendaciones** de Control y marcarlas como hechas. |
| U9 | Quiero **escribir y leer en el canal general**, y ver los mensajes fijados. |
| U10 | Quiero **responder a la comprobación de bienestar** con un toque. |
| U11 | Quiero saber si **he perdido la conexión** con Control. |

### Control
| ID  | Historia                                                                       |
| --- | ------------------------------------------------------------------------------ |
| C1  | Como Control, quiero ver a **toda la tripulación**: salud, conexión y alertas. |
| C2  | Quiero recibir **alertas en tiempo real**, con sonido en las críticas.         |
| C3  | Quiero **reconocer y resolver** alertas.                                       |
| C4  | Quiero ver el **detalle médico** de cada usuario.                              |
| C5  | Quiero **enviar tareas o recomendaciones** a un usuario.                       |
| C6  | Quiero **fijar mensajes** en el canal general.                                 |
| C7  | Quiero lanzar una **comprobación de bienestar** y saber si responden.          |
| C8  | Quiero **cambiar de escenario** y **reiniciar la demo**.                       |

### Fase 2
U12 chat individual · U13 radiación · U14 test de reacción · U15 recordatorios y agenda · U16 gamificación · U17 asistente · C9 Control en Tierra con retraso · C10 alertas manuales · C11 mensajes prioritarios · D1 rol Doctor.

## Requisitos funcionales: Fase 1

### RF-1. Clasificación
- Cada medición se clasifica en `normal`, `attention` o `critical` con los umbrales del **escenario activo** ([[15 - Indicadores y umbrales]]).
- Estado del **sistema** = el peor de sus indicadores. Estado **general** = el peor de sus sistemas.
- Sin medición en **24 h** → `no_data` (⚪), nunca verde.

### RF-2. Alertas
- **Tipos:** `health` (medición crítica), `connection` (desconexión) y `wellness` (pide ayuda o no responde).
- **Gravedad:** `warning` o `critical`.
- **Estado:** `open` → `acknowledged` → `resolved`. Se guarda quién y cuándo.
- `attention` **no** genera alerta en la Fase 1 (solo se muestra en la app del usuario). *El escalado es Fase 2.*

### RF-3. Latido de conexión (valores fijos)
| Estado | Condición | Acción |
|---|---|---|
| 🟢 `online` | Último latido hace menos de 15 s | — |
| 🟡 `stale` | Entre 15 y 30 s sin latido | Aviso visual en Control |
| 🔴 `offline` | Más de 30 s sin latido | **Alerta `connection` crítica** |

- El cliente envía `ping` cada **5 s**.
- Control lo ve como "**desconectado, salud desconocida**", no como un rojo médico.
- Al reconectar, la alerta **no** se cierra sola: la resuelve Control.

### RF-4. Bienestar
- Control la lanza a **un usuario** (en la Fase 1 no hay opción "todos"), con un **plazo de 60 s**.
- El usuario ve un modal: "Estoy bien" / "Necesito ayuda".
- "Necesito ayuda" → alerta `wellness` **crítica**. Sin respuesta en 60 s → alerta `wellness` **crítica**.

### RF-5. Canal general
- Todos leen y escriben. **Control puede fijar** mensajes (franja superior).
- Tiempo real por WebSocket; el historial se carga por REST.

### RF-6. Tareas y recomendaciones
- Control crea `{kind: task | recommendation, title, description}` para un usuario.
- El usuario las ve en su lista y las marca como `done`. **Sin fechas** en la Fase 1.

### RF-7. Escenarios
- Archivos YAML `mars` y `care_home`. Hay **un escenario activo** y lo cambia Control.
- Definen: nombre, color de acento, vocabulario, indicadores, umbrales y textos.

### RF-8. Privacidad (se comprueba en el backend)
- El usuario solo ve el detalle de **sus** datos. De los demás, solo el estado general y la conexión.
- Control lo ve todo.

### RF-9. Simulador
- Genera unos **14 días** de historial para 4 tripulantes (valores normales con ruido; uno de ellos con una leve tendencia a atención).
- `POST /dev/reset` reinicia la BD, crea los usuarios y genera el historial.

## Requisitos no funcionales
| ID | Requisito |
|---|---|
| RNF-1 | **Sin conexión:** sin CDNs; fuentes e iconos empaquetados. |
| RNF-2 | **Adaptable:** desde 360 px; zonas táctiles de 44 px como mínimo. La app del usuario se diseña primero para móvil y Control primero para escritorio. |
| RNF-3 | **Accesible:** el estado siempre con color, icono y texto; contraste AA. |
| RNF-4 | **Tiempo real:** una alerta llega a Control en menos de 1 s en la red local. |
| RNF-5 | **Arranque con un comando** (`./dev.sh` o `make dev`). |
| RNF-6 | **Idioma:** interfaz en inglés y documentación en español. |
| RNF-7 | Contraseñas con hash y JWT; permisos en cada endpoint. |
