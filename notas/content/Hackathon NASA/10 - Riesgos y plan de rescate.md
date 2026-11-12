---
title: Riesgos y plan de rescate
tags:
  - hackathon
  - riesgos
---

# Riesgos y plan de rescate

## Riesgos

| Riesgo | Prob. | Impacto | Mitigación |
|---|---|---|---|
| Integración tardía | Alta | Alto | Contrato en H3, datos de prueba, recorrido completo en H10, PRs pequeños y frecuentes |
| Código de IA incoherente entre 6 personas | Alta | Medio | `AGENTS.md` común, responsable por carpeta, revisar lo que cambia ([[16 - Uso de IA]]) |
| Los WebSockets se complican | Media | Alto | Plan B por consultas periódicas (abajo) |
| El alcance crece durante el evento | Alta | Alto | Si no está en la Fase 1, va a la Fase 2. El rol 1 dice "no" |
| Poca experiencia con Vue o FastAPI | Media | Medio | Formación en E0, parejas de apoyo, IA como tutor |
| La red del evento aísla los dispositivos | Media | Alto | **Probar en E1** (tarea I-12). Plan B: zona wifi propia o varias ventanas en un portátil |
| Disco lleno en 42 | Media | Medio | `/goinfre`, `pnpm` |
| Cansancio | Alta | Alto | Turnos; congelar en H22 |
| La demo falla en directo | Media | Muy alto | **El vídeo es la pieza principal**; botón de reinicio; ensayos |

## 🛟 MVP mínimo de rescate

> [!danger] Se activa en H14 si vamos más de 3 horas tarde o el recorrido completo no es estable
> Se queda:
> 1. Login, medir, resultado, dashboard con tarjetas y silueta, detalle con gráfica.
> 2. Control: tripulación con alertas en vivo (reconocer y resolver).
> 3. **Latido con alerta de desconexión** (es el diferenciador; no se recorta).
> 4. Simulador.
> 5. Escenario Marte (Residencia se enseña en el pitch con una captura del YAML).
>
> **Se recorta (lo primero que se quita, primero):** escenario Residencia (pasa a captura) → canal general → recomendaciones → bienestar.

## Plan B de los WebSockets
- `ws.ts` expone `on(type, handler)`. Con `VITE_REALTIME=polling` consulta `GET /crew`, `/alerts`, `/messages` y `/tasks` cada 3 s y emite los mismos eventos.
- El latido pasa a `POST /api/presence/ping` cada 5 s. **Este endpoint no está en el contrato:** si se activa el plan B, el rol 1 lo añade a [[05 - Contrato API#Plan B (solo si fallan los WebSockets)]] y lo avisa en `#dev` antes de programarlo.
