---
title: Contrato API
tags:
  - hackathon
  - backend
  - frontend
estado: borrador. Se congela en H3 (tarea I-05)
---

# Contrato API (Fase 1)

> [!important] Congelado en H3
> A partir de H3, un cambio se avisa en `#dev` y se actualiza aquí **antes** de programarlo. El frontend trabaja contra **datos de prueba que respetan este contrato** (`frontend/src/api/mock/`) hasta que el backend esté listo.

- **Base:** `http://<servidor>:8000/api`
- **Autenticación:** `Authorization: Bearer <jwt>`
- **Errores:** `{ "detail": "…" }`
- **Fechas:** ISO 8601 en UTC.

Acceso: 🔓 público · 👤 con sesión · 🛰️ usuario · 🎛️ Control

## REST (20 endpoints)

| # | Método | Ruta | Acceso | Entrada → salida |
|---|---|---|---|---|
| 1 | GET | `/health` | 🔓 | → `{ok: true}` |
| 2 | POST | `/auth/login` | 🔓 | `{username, password}` → `{access_token, user}` |
| 3 | GET | `/me` | 👤 | → `User` |
| 4 | GET | `/scenario` | 👤 | → `Scenario` (el activo, completo) |
| 5 | PUT | `/scenario` | 🎛️ | `{id}` → `Scenario` · emite `scenario.changed` |
| 6 | POST | `/measurements` | 🛰️ | `{indicator, value, value2?}` → `{measurement, evaluation}` |
| 7 | GET | `/measurements?user_id&indicator&days=14` | 👤 | → `Measurement[]` · **el usuario solo los suyos (si no, 403)** |
| 8 | GET | `/users/{id}/status` | 👤 | → `UserStatus` · el usuario solo el suyo |
| 9 | GET | `/crew` | 👤 | → `CrewMember[]` · **los campos dependen del rol** |
| 10 | GET | `/alerts?status=open` | 👤 | → `Alert[]` · el usuario solo las suyas |
| 11 | PATCH | `/alerts/{id}` | 🎛️ | `{status}` → `Alert` |
| 12 | GET | `/tasks?user_id` | 👤 | → `Task[]` · el usuario solo las suyas |
| 13 | POST | `/tasks` | 🎛️ | `{user_id, kind, title, description}` → `Task` |
| 14 | PATCH | `/tasks/{id}` | 🛰️ | `{status: "done"}` → `Task` |
| 15 | GET | `/messages?limit=50` | 👤 | → `Message[]` (canal general) |
| 16 | POST | `/messages` | 👤 | `{body}` → `Message` |
| 17 | PATCH | `/messages/{id}` | 🎛️ | `{pinned}` → `Message` |
| 18 | POST | `/wellness-checks` | 🎛️ | `{user_id}` → `WellnessCheck` |
| 19 | POST | `/wellness-checks/{id}/respond` | 🛰️ | `{response: "ok" \| "help"}` → `WellnessCheck` |
| 20 | POST | `/dev/reset` | 🎛️ | → `{ok}` · reinicia la BD, crea los usuarios y genera el historial |


## Tipos

```ts
type Status = 'normal' | 'attention' | 'critical' | 'no_data'
type Presence = 'online' | 'stale' | 'offline'

interface User { id: number; username: string; display_name: string; role: 'user' | 'control'; avatar: string }

interface Measurement { id: number; user_id: number; indicator: string; value: number; value2?: number;
  status: Exclude<Status,'no_data'>; source: 'manual' | 'sim'; taken_at: string }

interface Evaluation { status: Status; system: string; title: string; explanation: string;
  environment_effect: string; actions: string[]; alert_id?: number }

interface IndicatorStatus { indicator: string; status: Status; last_value?: number; last_value2?: number; last_at?: string }
interface UserStatus { user_id: number; overall: Status; systems: Record<string, Status>; indicators: IndicatorStatus[] }

// /crew → el usuario recibe solo {user_id, display_name, avatar, overall, presence}
interface CrewMember { user_id: number; display_name: string; avatar: string; overall: Status; presence: Presence;
  systems?: Record<string, Status>; open_alerts?: number }   // ← solo Control

interface Alert { id: number; user_id: number; type: 'health' | 'connection' | 'wellness'; severity: 'warning' | 'critical';
  status: 'open' | 'acknowledged' | 'resolved'; indicator?: string; title: string; message: string; created_at: string }

interface Task { id: number; user_id: number; kind: 'task' | 'recommendation'; title: string; description: string;
  status: 'pending' | 'done'; created_at: string }

interface Message { id: number; sender_id: number; sender_name: string; body: string; pinned: boolean; created_at: string }

interface WellnessCheck { id: number; user_id: number; status: 'pending' | 'ok' | 'help' | 'expired'; deadline_at: string }

interface Scenario { id: string; name: string; accent: string;
  vocabulary: { user: string; users: string; control: string };
  systems: Record<string, { label: string; icon: string }>;
  indicators: Record<string, IndicatorDef> }
interface IndicatorDef { label: string; unit: string; icon: string; system: string; dual?: boolean;
  min: number; max: number; step: number;                       // para el formulario
  bands: { status: Status; from?: number; to?: number }[];       // para las bandas de la gráfica
  how_to_measure: string[]; explanation: string; environment_effect: string;
  actions: { attention: string[]; critical: string[] } }
```

## WebSocket

`ws://<servidor>:8000/ws?token=<jwt>` · formato `{ "type": "...", "data": {...} }`

| Dirección | type | Destinatarios | data |
|---|---|---|---|
| Cliente → servidor | `ping` | — | `{}` cada 5 s |
| Servidor → cliente | `pong` | quien envió el ping | `{}` |
| Servidor → cliente | `presence.updated` | todos | `{user_id, presence}` |
| Servidor → cliente | `status.updated` | todos (el usuario recibe solo `overall`) | `{user_id, overall, systems?}` |
| Servidor → cliente | `alert.created` · `alert.updated` | el usuario afectado y Control | `Alert` |
| Servidor → cliente | `task.created` · `task.updated` | el usuario afectado y Control | `Task` |
| Servidor → cliente | `message.created` · `message.updated` | todos | `Message` |
| Servidor → cliente | `wellness.requested` | el usuario afectado | `WellnessCheck` |
| Servidor → cliente | `wellness.updated` | Control | `WellnessCheck` |
| Servidor → cliente | `scenario.changed` | todos | `{id}` → el cliente vuelve a pedir `GET /scenario` |

**Reconexión:** espera de 1, 2, 4… segundos (como máximo 10 s). Al volver, se piden de nuevo por REST `crew`, `alerts`, `tasks` y `messages`.

## Plan B (solo si fallan los WebSockets)
`POST /presence/ping` · 🛰️ · `{}` → `{ok}`. Sustituye al `ping` del WebSocket. Solo se programa si se activa el plan B ([[10 - Riesgos y plan de rescate#Plan B de los WebSockets]]).

## Fase 2 (no programar todavía)
`GET/POST /messages?channel=dm:<id>` · `POST /alerts` (manual) · `message.priority` · `task.due_at` · `/explain` · `/ground/*` (Control en Tierra) · rol `doctor`.
