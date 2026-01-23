---
title: "finally vs try-with-resources"
description: "Cómo cerrar recursos sin logs engañosos."
weight: 60
toc: true
---

## finally

`finally` se ejecuta casi siempre (salvo `System.exit()`), haya o no excepción.

Uso típico: cerrar recursos (conexiones, archivos).

❌ Problema: es fácil producir logs engañosos y errores durante `close()`.

---

## try-with-resources (recomendado)

Java ofrece `try-with-resources` para cerrar recursos automáticamente.

```java
public void readFromDatabase() {
    try (Connection connection = getConnection()) {
        Statement statement = connection.createStatement();
        statement.executeQuery("SELECT * FROM blackbox");
    } catch (SQLException e) {
        logger.log(System.Logger.Level.ERROR, "Error de DB", e);
    }
}
```

Ventajas:
- menos código
- cierre garantizado
- menos tentación de loggear “Connection closed” cuando nunca abrió

> Regla: evita loggear en `finally` si puedes.  
> Prefiere `try-with-resources` para mantener logs limpios y veraces.
