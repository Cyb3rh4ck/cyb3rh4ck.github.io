---
title: "Debugging vs Logs (por qué los logs importan)"
description: "Diferencias entre depurar y analizar stack traces en producción."
weight: 80
toc: true
---

## Debugging

- Tiempo real
- Breakpoints
- Step into / step over
- Puedes inspeccionar y modificar estado

👉 Es como “tiempo presente”.

## Logs

- Evidencia del pasado
- Producción (no puedes pausar servicios)
- La única forma de reconstruir incidentes

👉 Es como “tiempo pasado”.

## Conclusión

Cuando estás local con código y control total → **debugging**.  
Cuando estás en producción sin posibilidad de detener nada → **logs**.

> Por eso la calidad del logging es un tema de ingeniería, no de estilo.
