---
title: Claude - Diseño Fase 1
tags:
  - hackathon
  - diseño
fecha: 2026-11-11
---

# 🟠 Claude: Diseño Fase 1

Volver al [[Diseños - Índice]].

Hecho con el **lienzo de diseño de Claude** el 11 de noviembre de 2026. Sigue los colores y tokens de [[14 - Diseño de la app]]: fondo `#0B1020`, superficie `#131A2E`, estados `status-ok` / `status-warn` / `status-crit`, y el acento naranja de Marte `#F97316`.

**Datos de la demo (ficticios):** base en Marte, sol 28, tripulación Ana, Leo, Kai y Sam. Recorrido: Dashboard → Medir → Resultado (39,2 °C, crítico) → Detalle.

> [!info] Pantallas pendientes
> El lienzo tenía previstas 4 pantallas más que **no llegaron a dibujarse**: comprobación de bienestar, Control (tripulación), Control (detalle de un tripulante) y guía de estilo. Las 11 pantallas de Stitch sí incluyen Control: [[Stitch - Pantallas]].

## 1 · Dashboard

Silueta como mapa de estado, estado general con "Mission Control notified", las 6 tarjetas de indicadores con mini gráfica, la franja de la tripulación (Kai "no signal") y la navegación inferior.

![[claude-1-dashboard.png|Dashboard de la app del astronauta]]

## 2 · Medir (paso 3 de 3)

"How to measure" paso a paso, el valor en grande (39.2 °C), el rango esperado de la misión y el botón de guardar.

![[claude-2-medir.png|Pantalla de medir temperatura, paso 3]]

## 3 · Resultado crítico

Estado en rojo con icono y texto, aviso "Mission Control has been notified", qué significa y qué hacer ahora, en 4 pasos.

![[claude-3-resultado-critico.png|Resultado crítico de 39,2 °C]]

## 4 · Detalle del indicador

Gráfica de 14 días con las bandas de color de los umbrales, selector de periodo y los textos "How it is affected on Mars", "What to do" y "What is it?".

![[claude-4-detalle-indicador.png|Detalle de la temperatura con gráfica de 14 días]]
