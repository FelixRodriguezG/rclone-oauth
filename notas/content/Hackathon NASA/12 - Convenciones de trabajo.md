---
title: Convenciones de trabajo
tags:
  - hackathon
  - proceso
---

# Convenciones de trabajo

## Git
- `main` está **protegida**: nada de `push` directo, solo PRs.
- Ramas: `feat/<id-issue>-descripcion-corta`, `fix/<id>-…`, `docs/…` (por ejemplo, `feat/B-07-alerts-api`).
- **PRs pequeños** (menos de 300 líneas cuando se pueda) y fusionados **a menudo** (como mínimo cada 2 o 3 horas).
- El PR enlaza la issue (`Closes #12`) para que el tablero se mueva solo.
- Revisión: **1 aprobación** (el rol 1 o la pareja de apoyo). En la etapa E6, solo arreglos críticos.
- Fusión con **squash merge**.
- Commits en inglés con *Conventional Commits*: `feat(alerts): add acknowledge endpoint`.

## Responsables por área (para evitar conflictos)
La tabla de qué carpeta es de cada rol está **solo en `AGENTS.md`** (raíz del repo), para que la lean igual las personas y las IAs. Si cambia, se cambia allí.

Para tocar una carpeta de otro: avisar antes, o hacer el PR y pedirle la revisión.

## IA
- Todas las herramientas de IA parten de `AGENTS.md` (raíz del repo). Reglas en [[16 - Uso de IA]].
- Un PR es una tarea del [[09 - Backlog]] (el ID va en el nombre de la rama: `feat/F-07-indicator-card`).

## Código
- **Backend:** `ruff` (lint y formato) y anotaciones de tipo. Variables de configuración en `.env` (con `.env.example` en el repo).
- **Frontend:** ESLint + Prettier, `<script setup lang="ts">`, componentes en PascalCase.
- Interfaz en **inglés**, documentación en español (ver [[02 - Requisitos#Requisitos no funcionales|RNF-6]]).

## Tests
- **Qué se testea con código:** lo que rompería la demo y no se ve a simple vista: umbrales y `classify`, permisos (403), alertas automáticas, tiempos del latido y del bienestar, eventos WebSocket, stores. **La interfaz se prueba a mano** (tareas Q-01…Q-03).
- **Backend:** `pytest` en `backend/tests/` (SQLite en memoria, `TestClient`). **Frontend:** `vitest`, con el test junto al archivo (`StatusBadge.spec.ts`).
- Un PR que toca reglas, permisos o tiempos **lleva su test**. El resto, no es obligatorio.
- La CI (I-18) tiene que estar en verde para fusionar. **En la etapa E6, un test roto no bloquea un arreglo crítico**: se avisa en `#dev`.

## Comunicación
- **Canal del equipo** (Discord o Slack): `#general`, `#dev`, `#bloqueos`, `#pitch`.
- Un **bloqueo de más de 20 minutos** se comunica en `#bloqueos`.
- Las decisiones importantes se apuntan en [[13 - Registro de decisiones]].

## Obsidian
- Este vault es la carpeta `docs/` del repo: abridla en Obsidian con "Open folder as vault".
- Añadid `docs/.obsidian/workspace*.json` al `.gitignore` para evitar conflictos.
