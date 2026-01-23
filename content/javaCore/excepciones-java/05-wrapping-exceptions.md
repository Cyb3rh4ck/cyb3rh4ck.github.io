---
title: "Wrapping (Exception Chaining)"
description: "Cómo envolver checked en unchecked sin perder información."
weight: 50
toc: true
---

## Wrapping de excepciones (exception chaining)

En sistemas modernos, es común envolver checked exceptions en unchecked para evitar boilerplate.

Ejemplo:

```java
try {
    statement.executeQuery("SELECT * FROM blackbox");
} catch (SQLException e) {
    throw new IllegalArgumentException("Error al consultar blackbox", e);
}
```

Aquí, `SQLException` queda como **cause** del `IllegalArgumentException`.

## Figura 4.8 — Cadena de excepciones (sushi roll)

```text
IllegalArgumentException: "Error al consultar blackbox"
└── Caused by: SQLException: ...
```

Los frameworks de logging recorren `getCause()` e imprimen la cadena completa.

> No necesitas loggear cada capa.  
> Basta con loggear donde realmente se **maneja** la excepción.
