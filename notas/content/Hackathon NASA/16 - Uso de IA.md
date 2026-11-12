---
title: Uso de IA
tags:
  - hackathon
  - proceso
  - ia
---

# Uso de IA en el equipo

> [!abstract] Idea clave
> La IA acelera **escribir** código, pero no **integrarlo**. Con 6 personas generando código a la vez, el cuello de botella son la coherencia y las fusiones. Estas reglas existen para eso.

## Reglas
1. **Un contexto común:** `AGENTS.md` (o `CLAUDE.md`) en la raíz del repo, con el stack, las convenciones, las carpetas de cada uno y el enlace al contrato. Todas las IAs del equipo lo leen.
2. **El contrato manda:** se le pasa a la IA [[05 - Contrato API]] y los tipos. No inventa endpoints ni campos.
3. **Cada uno en su carpeta:** la IA solo toca los archivos de tu tarea. Si propone cambiar un archivo común (`ws.ts`, `models.py`, `rules.py`…), **para y avisa** a su responsable.
4. **Tareas pequeñas, PRs pequeños:** una tarea del backlog es un prompt y un PR (menos de 300 líneas).
5. **Lee lo que cambia antes de hacer commit.** Si no lo entiendes, no lo fusionas.
6. **Nada de reescrituras:** no se le pide "mejora este archivo" sobre algo que ya funciona.
7. **Prueba antes del PR:** arranca, haz clic y comprueba que funciona.
8. Se declara el uso de IA en la entrega si las reglas lo piden.

## Dónde rinde más
| Uso | Ejemplo |
|---|---|
| Código repetitivo | Modelos SQLModel, routers CRUD, stores de Pinia |
| Datos de prueba y del simulador | Datos de prueba en JSON según el contrato, historial con ruido |
| Componentes de interfaz | Primera versión de `IndicatorCard` según el wireframe |
| Tests | Casos límite de `classify` |
| Contenido | Primer borrador de los textos de los indicadores (**el rol 6 los revisa y cita la fuente**) |
| Depuración | Pegar el error y el archivo concreto |
| Tutor | "Explícame cómo funciona `defineProps` en Vue" |

## Plantilla de prompt para una tarea
```
Lee AGENTS.md y docs/05 - Contrato API.md.
Tarea: <ID> <título> (de docs/09 - Backlog.md).
Archivos que puedes tocar: <lista>.
No modifiques otros archivos. Usa los tipos del contrato.
Criterio de hecho: <de la tarea>.
```

## Skills de Claude Code
Quien use Claude Code tiene estas órdenes en el repo (`.claude/skills/`). `CLAUDE.md` importa `AGENTS.md`, así que todas las IAs siguen las mismas reglas.

| Orden | Para qué | Quién y cuándo |
|---|---|---|
| `/tarea <ID>` | Hace la plantilla de arriba automáticamente: lee la tarea, comprueba las dependencias y los archivos de tu rol, crea la rama, escribe el test si toca y comprueba "Hecho cuando…". No hace commit sin pedírselo | Todos, en cada tarea |
| `/check-contrato` | Compara endpoints, permisos, tipos y eventos WebSocket con [[05 - Contrato API]]. Solo lectura | Rol 1, en ✅ H10 y ✅ H20, y antes de fusionar PRs de API |
| `/check-demo [H3\|H10\|H20\|H28]` | Tests, lint, build, el recorrido de la API y la checklist manual del punto de control. Pregunta antes de reiniciar la BD | Rol 6, en cada punto de control |
| `/escenario` | Valida los YAML y prueba los umbrales en los valores límite de [[15 - Indicadores y umbrales]] | Rol 5 (y rol 6 al volcar textos) |

> [!warning] Si cambian los umbrales
> `/escenario` comprueba los YAML con los valores límite de la nota 15, copiados en `CASOS` dentro de `.claude/skills/escenario/validar.py`. Si se cambia un umbral en la nota 15, hay que actualizar también `CASOS`.

Con otras herramientas de IA, usad la plantilla de prompt de arriba.

## Borrador de `AGENTS.md`
Ver el archivo `AGENTS.md` en la raíz del repo (lo crea y mantiene el rol 1).
