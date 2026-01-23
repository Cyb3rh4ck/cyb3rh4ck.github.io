---
title: "02. Checked vs Unchecked Exceptions"
date: 2026-01-23T00:00:00Z
draft: false
description: "Diferencias, reglas del compilador, jerarquía y ejemplos."
tags: ["java", "spring-boot", "aws", "devops", "leetcode", "career", "gamification", "backend"]
categories: ["Backend", "Career", "Java"]
series: ["Backend Engineer RPG"]
toc: true
weight: 2

---
## La gran division

En Java, todas las excepciones heredan de `java.lang.Exception`, pero existen dos ramas que dictan como reacciona el compilador. Conocerlas evita sorpresas.

- **Checked exceptions:** Heredan directamente de `Exception`. El compilador exige que las manejes o declares.
- **Unchecked exceptions:** Heredan de `RuntimeException`. El compilador te deja continuar sin try-catch ni `throws`.

Si dudas, abre la clase en el IDE: si sube hasta `RuntimeException` es unchecked; de lo contrario, es checked.

## Mapa de herencia

Así se relacionan las principales clases de excepciones en Java:

{{< mermaid >}}
classDiagram
    class Throwable {
        +getMessage()
        +printStackTrace()
    }
    class Error["Error (Fatal)"]
    class Exception["Exception (Checked)"]
    class RuntimeException["RuntimeException (Unchecked)"]
    class SQLException
    class IOException
    class NullPointerException
    class IllegalArgumentException

    Throwable <|-- Error
    Throwable <|-- Exception
    Exception <|-- SQLException
    Exception <|-- IOException
    Exception <|-- RuntimeException
    RuntimeException <|-- NullPointerException
    RuntimeException <|-- IllegalArgumentException
{{< /mermaid >}}

- **Ejemplo checked:** `SQLException` obliga a reaccionar.
- **Ejemplo unchecked:** `NullPointerException` delata un bug en tu logica.

## Las reglas del juego

### Checked exceptions

Se reservan para condiciones recuperables (por ejemplo, la base de datos no responde). El compilador te fuerza a tomar una decision:

1. **Handle:** Envolver con `try-catch` y tomar accion.
2. **Declare:** Propagar usando `throws` en la firma.

```java
// "throws SQLException" advierte al invocador que maneje el problema
public ResultSet leerDatos(Connection conn) throws SQLException {
    logger.log(Level.INFO, "Inicio");
    Statement stmt = conn.createStatement();
    return stmt.executeQuery("SELECT * FROM tabla");
}
```

### Unchecked exceptions

Son sintomas de errores de programacion. El compilador no exige `try-catch` ni `throws`.

- Normalmente no se atrapan; se previenen con pruebas y validaciones (`if (obj != null)`).
- Si se disparan, escalan por la pila hasta interrumpir el programa, exponiendo el stack trace.