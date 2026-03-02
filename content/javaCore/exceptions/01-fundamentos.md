---
title: "01. Fundamentos: Cuando el código se rompe"
date: 2026-01-23T00:00:00Z
draft: false
description: "Definición, flujo normal vs flujo con excepción, precondiciones y postcondiciones."
tags: ["java", "spring-boot", "aws", "devops", "leetcode", "career", "gamification", "backend"]
categories: ["Backend", "Career", "Java"]
series: ["Backend Engineer RPG"]
toc: true
weight: 1
---


## 🛀 La analogía de la bañera

Imagínate que es martes en la noche. Estás en la bañera, cantando a todo pulmón, relajado. De repente, se abre la puerta y entra tu pareja con una visita.

¡Pánico! Esto no estaba en el script. Es una excepción en tu programa personal. Si no manejas bien esa situación (handle), el resultado va a ser incómodo.

En el código pasa lo mismo: si ignoras las excepciones, tu programa crashea o queda en un estado inestable (como tú, escondido bajo la espuma). Y ojo: manejarlas sin loguearlas es como gritar bajo el agua; nadie se entera de qué pasó realmente.

## ¿Qué es una excepción?

Mientras que el `if` y el `else` nos dan control, las excepciones rompen el flujo normal de forma agresiva. Por eso los juniors les tienen miedo.

Para entenderlas, miremos el flujo normal:

- Empieza.
- Ejecuta pasos.
- Termina feliz.

{{< mermaid >}}
graph LR
    A[Inicio] --> B[Paso 1]
    B --> C[Paso 2]
    C --> D[Fin Exitoso]
    style A fill:#9f9,stroke:#333,stroke-width:2px
    style D fill:#9f9,stroke:#333,stroke-width:2px
{{< /mermaid >}}


Una excepción es una "condición excepcional" (como diría Joshua Bloch). Ojo: un password incorrecto NO es una excepción, es un flujo alternativo esperado. Una excepción es algo que no debería pasar, como que se caiga la base de datos o se acabe la memoria RAM.

## Precondiciones y postcondiciones

Para saber qué es excepcional, hay que saber qué es "normal".

- **Precondición:** Lo que debe ser verdad antes de correr el método.
  - Ejemplo: Si calculas el rumbo de un avión, la entrada debe ser 0-359 grados.
- **Postcondición:** Lo que debe ser verdad después de correr el método.
  - Ejemplo: El resultado también debe ser 0-359 grados.

Si no puedes cumplir la postcondición (ej. otro usuario compró el boleto de avión milisegundos antes que tú), lanzar una excepción es la forma honesta de decir: "No pude hacer mi trabajo".

```java
public int calcularRumbo(int rumboActual, int correccion) {
    // Validación de precondición
    if (rumboActual < 0 || rumboActual > 359) {
        throw new IllegalArgumentException("Rumbo actual invalido");
    }

    int nuevoRumbo = (rumboActual + correccion) % 360;

    // Validación de postcondición
    if (nuevoRumbo < 0 || nuevoRumbo >= 360) {
        throw new IllegalStateException("Calculo fuera de rango"); // Esto no deberia pasar nunca
    }
    return nuevoRumbo;
}
```
