---
title: "🚀 Bootcamp: Lambdas en Java con JDK 17"
date: 2025-08-10
draft: false
description: "Bootcamp de Lambdas en Java (JDK 17) desde cero, en varias fases"
tags: ["java", "lambdas"]
---

**bootcamp de Lambdas en Java (JDK 17)** desde cero, en varias fases, para que empieces sin saber nada y termines aplicándolas en casos reales de trabajo.

---

## **🚀 Bootcamp: Lambdas en Java con JDK 17**

### **Módulo 1 – Fundamentos**

🎯 Objetivo: Entender qué es una lambda y por qué existe.

1. **Teoría básica**

   * ¿Qué es una expresión lambda?
   * Diferencia entre *clase anónima* y *lambda*.
   * Sintaxis y partes de una lambda.
2. **Interfaces funcionales**

   * `@FunctionalInterface`
   * Uso de `Runnable`, `Callable`, `Comparator`.
3. **Primeros pasos**

   * Crear una interfaz funcional y pasarla como parámetro a un método.
   * Ejercicio: Implementar un `Saludador` que imprima "Hola, <nombre>".

---

### **Módulo 2 – Lambdas con API de Java**

🎯 Objetivo: Aprender a usar las lambdas con interfaces funcionales estándar.
4\. **`java.util.function`**

* `Predicate<T>` (filtrado)
* `Function<T,R>` (transformación)
* `Consumer<T>` (procesar datos)
* `Supplier<T>` (proveer datos)

5. **Ejercicios prácticos**

   * Filtrar nombres que empiecen por "A".
   * Mapear enteros a su doble.
   * Imprimir lista en mayúsculas.

---

### **Módulo 3 – Streams y Programación Funcional**

🎯 Objetivo: Usar lambdas para manipular colecciones de forma declarativa.
6\. **Operaciones intermedias**: `map`, `filter`, `sorted`.
7\. **Operaciones terminales**: `collect`, `forEach`, `reduce`.
8\. **Ejercicios**:

* Multiplicar todos los números de una lista por 3.
* Sumar todos los elementos de una lista.
* Agrupar cadenas por longitud (`Collectors.groupingBy`).

---

### **Módulo 4 – Casos Reales**

🎯 Objetivo: Integrar lambdas en situaciones de backend real.
9\. **Ordenamiento de objetos** con `Comparator.comparing`.
10\. **Procesar JSON**: Filtrar, mapear y agrupar datos con streams.
11\. **Ejercicio de integración**:
\- Lista de `Persona(nombre, edad)` → filtrar mayores de 18, ordenar por nombre, unir en un string separado por comas.

---

### **Módulo 5 – Avanzado (JDK 11–17)**

🎯 Objetivo: Usar características modernas con lambdas.
12\. **`var` en parámetros de lambdas**
13\. **Métodos de referencia** (`Class::method`)
14\. **Lambdas en `Optional`** (`ifPresent`, `map`, `flatMap`)
15\. **Parallel Streams**: cuándo y cuándo no usarlos.

---

### **Módulo 6 – Proyecto Final**

🎯 Objetivo: Aplicar todo en un microproyecto.

* Crear un pequeño sistema de ventas que:

  * Cargue productos en memoria.
  * Filtre por disponibilidad.
  * Ordene por precio.
  * Calcule el total con `reduce`.
  * Use `map` para aplicar descuentos.

---

📌 **Recomendaciones**:

* Practica cada ejercicio primero con **for/while** y luego con **lambdas** para ver la diferencia.
* Usa **IntelliJ IDEA** o **VS Code** con Java Extension Pack para tener autocompletado y depuración.
* Trabaja con **JUnit 5** para probar tus lambdas en cada módulo.

---

Perfecto Efren 😏
Vamos a desmenuzar el **Módulo 1 – Fundamentos** para que entiendas **desde cero** qué es una **lambda** y por qué existe en Java.

---

## **Módulo 1 – Fundamentos**

### 🎯 Objetivo

Que entiendas:

* Qué es una **expresión lambda**.
* En qué se diferencia de una **clase anónima**.
* Cómo se escribe y cuáles son sus partes.

---

## **1️⃣ ¿Qué es una expresión lambda?**

En Java, una **expresión lambda** es una **función anónima** que puedes usar como si fuera un **objeto**.

📌 Características:

* **No tiene nombre** (es “anónima”).
* Se puede **pasar como parámetro** a un método.
* Se usa para **implementar interfaces funcionales** (interfaces con **un solo método abstracto**).

Ejemplo sencillo:

```java
// Interface funcional
@FunctionalInterface
interface Saludador {
    void saludar(String nombre);
}

// Uso con lambda
Saludador s = (nombre) -> System.out.println("Hola, " + nombre);
s.saludar("Efren");
```

---

## **2️⃣ Diferencia entre Clase Anónima y Lambda**

Antes de Java 8, si querías pasar “comportamiento” como argumento, usabas **clases anónimas**.

### **Clase anónima**

```java
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hola desde clase anónima");
    }
};
```

### **Lambda**

```java
Runnable r = () -> System.out.println("Hola desde lambda");
```

💡 **Diferencias clave**:

| Clase Anónima                                          | Lambda                                                        |
| ------------------------------------------------------ | ------------------------------------------------------------- |
| Código más **largo** y repetitivo.                     | Sintaxis **corta** y limpia.                                  |
| Crea una **nueva clase interna**.                      | No crea clase interna, usa **invokedynamic** (más eficiente). |
| Puede tener múltiples métodos (si es interfaz normal). | Solo se usa con **interfaces funcionales**.                   |
| `this` hace referencia a la clase anónima.             | `this` hace referencia a la clase envolvente.                 |

---

## **3️⃣ Sintaxis y Partes de una Lambda**

La forma general es:

```java
(parametros) -> { cuerpo }
```

### **Partes**:

1. **Parámetros** → `(x, y)`

   * Si solo hay 1 parámetro, puedes omitir los paréntesis: `x -> ...`
2. **Operador flecha** → `->`

   * Separa parámetros y cuerpo.
3. **Cuerpo** → `{ ... }`

   * Si es una sola línea, puedes omitir llaves y `return`.

---

### **Ejemplos de sintaxis válidas**

```java
// Sin parámetros
() -> System.out.println("Hola mundo");

// Con un parámetro
x -> System.out.println(x);

// Con tipo explícito
(String nombre) -> System.out.println("Hola " + nombre);

// Múltiples parámetros
(a, b) -> a + b;

// Con llaves y return
(a, b) -> {
    int suma = a + b;
    return suma;
};
```

---

💡 **Regla de oro:**
Una lambda **siempre** se asocia a una **interface funcional**, aunque sea una de las que ya trae Java (`Runnable`, `Predicate`, etc.).

---




