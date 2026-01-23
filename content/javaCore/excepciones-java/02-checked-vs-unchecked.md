---
title: "Checked vs Unchecked Exceptions"
description: "Diferencias, reglas del compilador, jerarquía y ejemplos."
weight: 20
toc: true
---

## Jerarquía (mental model)

```text
Throwable
├── Error
└── Exception
    ├── RuntimeException (unchecked)
    └── Otras excepciones (checked)
```

En Java, **todas** las excepciones heredan de `java.lang.Exception`.  
Pero si heredan de `java.lang.RuntimeException`, se consideran **unchecked**.

## Figura 4.3 — Regla práctica

- Si **extiende `RuntimeException`** → **unchecked**
- Si **NO extiende `RuntimeException`** → **checked**

---

## Checked Exceptions (regla: handle or declare)

Los checked exceptions existen para **obligar al caller** a decidir qué hacer.

Reglas:
1. Si llamas a un método que puede lanzar un checked exception → debes **manejarlo** con `try/catch`.
2. Si no lo manejas ahí → debes **declararlo** con `throws`.

### Ejemplo: declarar `SQLException`

```java
public ResultSet read(Connection connection) throws SQLException {
    Statement stmt = connection.createStatement();
    return stmt.executeQuery("SELECT * FROM my_table");
}
```

Quien llame a `read()` ahora debe manejar o declarar el `SQLException`.

---

## Unchecked Exceptions (no obligan al compilador)

Un unchecked exception:
- NO requiere `throws`
- NO requiere `try/catch`
- Si nadie lo captura → se propaga hasta `main()` y puede terminar el programa.

Ejemplos:
- `NullPointerException`
- `ArrayIndexOutOfBoundsException`

### Ejemplo: compila, pero truena en runtime

```java
public static void main(String[] args) {
    String[] passengers = {"Christian", "Linda", "Olivier"};
    System.out.println(passengers[4]); // ArrayIndexOutOfBoundsException
}
```

> En general, los unchecked exceptions suelen apuntar a **errores de programación**.  
> En vez de llenar el código de `try/catch`, normalmente se corrigen con **tests** y mejores contratos (validaciones).
