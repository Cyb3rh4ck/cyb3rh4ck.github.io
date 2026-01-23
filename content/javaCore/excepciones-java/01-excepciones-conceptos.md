---
title: "¿Qué son las excepciones?"
description: "Definición, flujo normal vs flujo con excepción, precondiciones y postcondiciones."
weight: 10
toc: true
---

## ¿Qué es una excepción?

Una excepción representa una **situación inesperada pero posible** durante la ejecución de un programa.

Mientras que `if` y `else` controlan flujos **esperados**, las excepciones interrumpen el flujo **normal** del programa.

## Figura 4.1 — Flujo normal de un programa

```text
Inicio → Paso 1 → Paso 2 → Paso 3 → Fin
```

## Figura 4.2 — Flujo cuando se lanza una excepción

```text
Inicio → Paso 1 → Paso 2 → ❌ Excepción → (Manejo) → Continuar/Terminar
```

Joshua Bloch (autor de *Effective Java*) las llama **“condiciones excepcionales”**.

> Un password incorrecto NO es una excepción.  
> Que el sistema no pueda cumplir su contrato, sí lo es.

## Precondiciones y postcondiciones

- **Precondición:** debe cumplirse antes de ejecutar el método.
- **Postcondición:** debe cumplirse al terminar el método.

Si no se puede cumplir la postcondición → **es razonable lanzar una excepción**.

### Ejemplo: heading de una aeronave

```java
public int calculateNewHeading(int currentHeading, int correction) {
    // Precondición: heading válido
    if (currentHeading < 0 || currentHeading > 359) {
        throw new IllegalArgumentException("Heading inválido: " + currentHeading);
    }

    int newHeading = (currentHeading + correction) % 360;

    // Postcondición: resultado también válido
    if (newHeading < 0 || newHeading >= 360) {
        throw new IllegalStateException("Resultado fuera de rango: " + newHeading);
    }

    return newHeading;
}
```

En sistemas reales, estas reglas suelen ser mucho más complejas (por ejemplo, check-in de aerolínea con asientos, pagos, reglas de edad, etc.).  
En algunos escenarios, la excepción es el mecanismo más directo para indicar que **no se pudo cumplir el resultado esperado**.
