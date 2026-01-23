---
title: "Logging en Backend Java (Guía Senior)"
date: 2026-01-23T00:00:00Z
draft: false
description: "Un sistema tipo videojuego para subir de nivel como Backend Senior: XP, skill trees, misiones y boss fights, enfocado a entrevistas Big Tech."
tags: ["java", "spring-boot", "aws", "devops", "leetcode", "career", "gamification", "backend"]
categories: ["Backend", "Career", "Java"]
series: ["Backend Engineer RPG"]
toc: true
---


## Logging en Backend Java (Guía Senior)

El logging **no es imprimir texto**.  
Es una herramienta crítica de **observabilidad, diagnóstico y operación en producción**.


### Niveles de Logging (ordenados por severidad)

De **menor** a **mayor** severidad:



- TRACE
- DEBUG
- INFO
- WARN
- ERROR



> En Java moderno (`SLF4J` + `Logback` / `Log4j2`), el nivel `FATAL` prácticamente no se usa y suele tratarse como `ERROR`.

---

### TRACE — Detalle microscópico
- Flujo interno extremadamente detallado
- Paso a paso de algoritmos
- Diagnóstico profundo

```java
log.trace("Entering method calculatePrice with input {}", input);
````

**Uso real:**
Solo debugging profundo. Nunca activo en producción normal.

---

### DEBUG — Diagnóstico de desarrollo

* Estados internos
* Decisiones de código
* Valores intermedios

```java
log.debug("User {} loaded from database", userId);
```

**Uso real:**
Desarrollo y QA. En producción solo de forma temporal y controlada.

---

### INFO — Flujo normal del sistema (nivel más importante)

* Eventos esperados
* Flujo de negocio
* Estado saludable del sistema

```java
log.info("Order {} successfully processed", orderId);
```

**Regla senior:**

> Los logs INFO deben contar la historia de lo que hace el sistema.

---

### WARN — Algo no ideal, pero el sistema continúa

* Situaciones anómalas
* Reintentos
* Configuraciones sospechosas
* Datos inesperados pero tolerables

```java
log.warn("Retrying payment for order {}", orderId);
```

**No es un error**, pero **merece atención**.

---

### ERROR — Fallo real

* Excepciones
* Requests fallidos
* Transacciones abortadas

```java
log.error("Failed to process order {}", orderId, ex);
```

**En producción:**

* ERROR = alerta
* ERROR = ticket
* ERROR = análisis post-mortem

---

## Jerarquía de niveles (clave)

Si configuras el logger en:

```
INFO
```

Se registran:

* INFO
* WARN
* ERROR

Se ignoran:

* DEBUG
* TRACE

> Los niveles son **acumulativos hacia arriba**.

---

## Error común (rechazado en code review)

❌ Mal uso:

```java
log.error("User not found");
```

Si “usuario no encontrado” es un escenario esperado → **NO es ERROR**.
Debe ser `INFO` o `WARN`.

---

## Regla de oro (memorízala)

> **INFO cuenta historias**
> **WARN pide atención**
> **ERROR pide acción**
> **DEBUG explica**
> **TRACE disecciona**

---

## Buenas prácticas senior

* No loggear datos sensibles
* No abusar de `ERROR`
* No usar logging como debugger permanente
* Preferir logs estructurados
* Pensar siempre en **producción y soporte**

---

## Resumen

El logging correcto:

* facilita troubleshooting
* reduce MTTR
* evita falsos positivos
* y refleja madurez técnica

> Un backend senior no escribe más logs.
> Escribe **los logs correctos**.

```