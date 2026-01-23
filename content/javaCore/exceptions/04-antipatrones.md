---
title: "04. Throwing y Antipatrones Comunes"
description: "Loggear todo, log & throw, y problemas de seguridad/ruido."
date: 2026-01-23T00:00:00Z
draft: false
tags: ["java", "spring-boot", "aws", "devops", "leetcode", "career", "gamification", "backend"]
categories: ["Backend", "Career", "Java"]
series: ["Backend Engineer RPG"]
toc: true
weight: 40
---

## Programación defensiva (fail fast)

A veces debes detener el tren: si te pasan basura, no la proceses. Falla rápido.

- Usa `throw` para disparar excepciones y reutiliza las estándar:
  - `IllegalArgumentException`: argumentos inválidos.
  - `NullPointerException`: `null` donde no debe (sí, puedes lanzarla explícitamente; Effective Java lo avala).

```java
public void escribirRegistro(String input) {
    // Validar primero
    if (input == null) {
        throw new NullPointerException("Input no puede ser null");
    }
    if (input.length() > 34) {
        throw new IllegalArgumentException("Input muy largo: " + input.length());
    }
    // ... procesar
}
```

## 🚫 Antipatrón 1: log everything (el chismoso)

Nunca loguees los datos de entrada antes de validarlos.

```java
// ❌ Peligroso: podría ser un script o 10MB de texto
logger.info("Input recibido: " + input);
if (input.length() > 10) throw ...;
```

Esto abre la puerta a log injection. Valida primero; si el input es inválido, lanza y deja que el manejador adecuado loguee.

## 🚫 Antipatrón 2: log and throw (el eco)

```java
try {
    algo();
} catch (Exception e) {
    logger.error("Fallo ejecutando algo", e); // ❌ Lo logueas aquí
    throw e;                                    // ❌ Y lo vuelves a lanzar
}
```

Duplicas el mismo error en los logs. Regla de oro: o lo manejas (y logueas) o lo lanzas, no ambas.

## Wrapping (exception chaining)

Envuelve excepciones técnicas en una de negocio para simplificar firmas sin perder la causa.

```java
try {
    db.query();
} catch (SQLException e) {
    throw new IllegalArgumentException("Error procesando datos", e); // chaining
}
```

Los frameworks de logging muestran la causa original (`Caused by...`), así que el rastro se conserva.