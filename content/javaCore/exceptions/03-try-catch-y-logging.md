---
title: "03. Try/Catch y Logging correcto"
description: "Cómo manejar excepciones, qué loggear y cómo incluir contexto."
date: 2026-01-23T00:00:00Z
draft: false
tags: ["java", "spring-boot", "aws", "devops", "leetcode", "career", "gamification", "backend"]
categories: ["Backend", "Career", "Java"]
series: ["Backend Engineer RPG"]
toc: true
weight: 30
---

## Try-catch: el portero

Si decidimos manejar el error en lugar de dejar morir el programa, usamos `try-catch`.

{{< mermaid >}}
flowchart TD
    A[Inicio try] --> B{¿Ocurrió Excepción?}
    B -- No --> C[Fin try → sigue normal]
    B -- Sí --> D[Bloque catch]
    D --> E[Loguear y recuperar]
    E --> F[Continuar ejecución]
{{< /mermaid >}}

## Logging: lo que no debes hacer

```java
try {
    basedatos.query();
} catch (SQLException e) {
    logger.info("Catch"); // ❌ Pésimo: no hay contexto ni stack trace
}
```

Loguear "Catch" no sirve: no dice qué pasó, ni dónde, ni por qué.

## Logging: la forma correcta

Necesitamos contexto, severidad adecuada (ERROR o WARN) y pasar la excepción al logger.

```java
try {
    basedatos.query();
} catch (SQLException ex) {
    // ✅ Nivel correcto, mensaje descriptivo y pasamos 'ex'
    logger.log(Level.ERROR, "Fallo crítico al consultar usuarios", ex);
}
```

Al pasar `ex`, el framework imprimirá el stack trace completo para rastrear la línea exacta.

## Cuidado con la jerarquía al atrapar

Si tienes excepciones personalizadas, el orden importa. Ejemplo:

```java
try {
    despegarAvion();
} catch (MilitaryAircraftException e) {
    // Manejo especial para militares (FATAL)
    logger.log(Level.FATAL, "Falla crítica militar", e);
} catch (AircraftException e) {
    // Manejo genérico para avionetas (ERROR)
    logger.log(Level.ERROR, "Falla en despegue", e);
}
```

Pon primero el catch más específico; el genérico debe ir al final.
