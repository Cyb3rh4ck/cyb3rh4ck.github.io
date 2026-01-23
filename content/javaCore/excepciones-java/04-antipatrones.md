---
title: "Antipatrones comunes"
description: "Loggear todo, log & throw, y problemas de seguridad/ruido."
weight: 40
toc: true
---

## Antipatón 1: Loggear TODO (especialmente input sin validar)

❌ Peligroso:

```java
logger.log(System.Logger.Level.INFO, "Input: " + input);
if (input.isEmpty() || input.length() > 34) {
    throw new IllegalArgumentException("Bad input: " + input);
}
```

Riesgos:
- **log injection**
- PII/secretos en logs
- logs gigantes (DoS de observabilidad)
- ruido que tapa señales reales

✔️ Recomendación: **valida primero**. Si debes lanzar excepción, evita meter el input crudo en el mensaje.

---

## Antipatrón 2: Log and Throw (duplicas logs)

❌ Mala práctica:

```java
catch (NullPointerException npe) {
    log.error("NPE", npe);
    throw npe;
}
```

Problemas:
- entradas duplicadas en logs
- confusión operativa
- stack traces repetidos

> Regla de oro: **quien maneja, loggea**.  
> Si solo vas a re-lanzar, normalmente NO deberías loggear ahí.
