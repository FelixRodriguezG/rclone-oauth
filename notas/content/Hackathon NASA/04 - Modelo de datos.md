---
title: Modelo de datos
tags:
  - hackathon
  - backend
---

# Modelo de datos (Fase 1)

Siete tablas, sin campos que no se usen en la Fase 1. Los campos de la Fase 2 están al final.

## `user`
| Campo | Tipo | Notas |
|---|---|---|
| id | int PK | |
| username | str, único | |
| password_hash | str | |
| display_name | str | |
| role | `user` \| `control` | |
| avatar | str | emoji o ruta de imagen |

## `measurement`
| Campo | Tipo | Notas |
|---|---|---|
| id | int PK | |
| user_id | FK user | |
| indicator | str | `heart_rate`, `blood_pressure`, `glucose`, `temperature`, `sleep_hours`, `mood` |
| value | float | sistólica en la tensión |
| value2 | float? | diastólica |
| status | `normal` \| `attention` \| `critical` | se calcula al guardar |
| source | `manual` \| `sim` | |
| taken_at | datetime | |

## `alert`
| Campo | Tipo | Notas |
|---|---|---|
| id | int PK | |
| user_id | FK user | |
| type | `health` \| `connection` \| `wellness` | |
| severity | `warning` \| `critical` | |
| status | `open` \| `acknowledged` \| `resolved` | |
| indicator | str? | solo en `health` |
| title · message | str | |
| created_at | datetime | |
| updated_by | FK user? | quien la reconoció o resolvió por última vez |
| updated_at | datetime? | |

## `task`
| Campo | Tipo | Notas |
|---|---|---|
| id | int PK | |
| user_id | FK user | destinatario |
| kind | `task` \| `recommendation` | |
| title · description | str | |
| status | `pending` \| `done` | |
| created_at · done_at | datetime | |

## `message` (solo el canal general en la Fase 1)
| Campo | Tipo | Notas |
|---|---|---|
| id | int PK | |
| sender_id | FK user | |
| body | str | |
| pinned | bool | solo lo cambia Control |
| created_at | datetime | |

## `wellness_check`
| Campo | Tipo | Notas |
|---|---|---|
| id | int PK | |
| user_id | FK user | |
| status | `pending` \| `ok` \| `help` \| `expired` | |
| deadline_at | datetime | ahora + 60 s |
| answered_at | datetime? | |

## `setting`
| key | value |
|---|---|
| `active_scenario` | `mars` |

## En memoria
- `last_seen: dict[user_id, datetime]` y `presence: dict[user_id, "online" | "stale" | "offline"]`.

## Fase 2 (no crear todavía)
- `message.channel` (`general` o `dm:<id>`), `message.priority`.
- `task.due_at`, `task.reminded`.
- `alert.type += manual, task_overdue`, `alert.severity += info`.
- `measurement.note`. Indicador `radiation`. Rol `doctor`.
