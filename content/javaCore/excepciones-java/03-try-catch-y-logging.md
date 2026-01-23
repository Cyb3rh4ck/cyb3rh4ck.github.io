---
title: "Try/Catch y Logging correcto"
description: "Cómo manejar excepciones, qué loggear y cómo incluir contexto."
weight: 30
toc: true
---

## Manejar excepciones con try/catch

`try` contiene el código que puede fallar.  
`catch` contiene el código para manejar el fallo.

## Figura 4.4 — Excepción capturada y flujo continúa

```text
Caller → try { Paso 1 → Paso 2 → ❌ throw } → catch { manejar + log } → continuar
```

### Ejemplo

```java
public void read(Statement statement) {
    try {
        logger.log(System.Logger.Level.INFO, "Start");
        statement.executeQuery("SELECT * FROM my_table");
        logger.log(System.Logger.Level.INFO, "End");
    } catch (SQLException e) {
        // manejar + log con contexto
        logger.log(System.Logger.Level.ERROR, "Error consultando la base de datos", e);
    }
}
```

---

## Logging de excepciones: qué es “bien hecho”

❌ Mal ejemplo (ruidoso e inútil):

```java
catch (SQLException e) {
    logger.log(System.Logger.Level.INFO, "Catch");
}
```

✔️ Buen ejemplo (severidad correcta + contexto + stack trace):

```java
catch (SQLException e) {
    logger.log(System.Logger.Level.ERROR, "Error consultando la base de datos (query falló)", e);
}
```

### Por qué incluir el exception como parámetro

Los frameworks de logging imprimen el **stack trace** y ayudan a diagnosticar:
- dónde falló
- en qué método
- con qué cadena de llamadas

> Regla práctica: un `catch` vacío es deuda técnica instantánea.  
> Si no puedes manejar, no captures.
