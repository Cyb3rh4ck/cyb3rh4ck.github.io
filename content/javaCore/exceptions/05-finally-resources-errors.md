---
title: "05. Finally, Resources y Errores Fatales"
description: "Cómo envolver checked en unchecked sin perder información."
date: 2026-01-23T00:00:00Z
draft: false
tags: ["java", "spring-boot", "aws", "devops", "leetcode", "career", "gamification", "backend"]
categories: ["Backend", "Career", "Java"]
series: ["Backend Engineer RPG"]
toc: true
weight: 40
---

## El bloque finally

`finally` siempre se ejecuta, haya error o no. Se usaba para cerrar conexiones o archivos.

- Problema: loguear en `finally` puede mentir ("Conexión cerrada") aunque nunca se abrió.

## La evolución: try-with-resources

Desde Java 7, si el recurso implementa `AutoCloseable`, colócalo en el paréntesis del `try` y Java lo cerrará solo.

```java
// ✅ Forma moderna y limpia
try (Connection conn = obtenerConexion()) {
    // Usa la conexión
    // Java la cerrará automáticamente al salir, pase lo que pase.
} catch (SQLException e) {
    logger.error("Error", e);
}
```

Menos código y logs más honestos.

## Errors: el lado oscuro

`Error` es el hermano malvado de `Exception`:

- `StackOverflowError`: recursión infinita.
- `OutOfMemoryError`: sin RAM.

Regla: son dominio de la JVM; no los atrapes. Si ocurren, el proceso está comprometido. Loguear quizá ni funcione; deja morir la app y corrige la causa.

{{< mermaid >}}
flowchart TD
    A[Problema detectado] --> B{¿Es Exception?}
    B -- Sí --> C[Try/Catch o Throws]
    B -- No (es Error) --> D[☠️ Dejar morir la app]
{{< /mermaid >}}

## Resumen final

- Distingue checked vs unchecked.
- No uses `e.printStackTrace()`: usa un logger.
- Evita el antipatrón "log & throw".
- Modernízate con try-with-resources.