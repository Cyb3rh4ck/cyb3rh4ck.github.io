---
title: "Frases y prompts útiles para Copilot en VS Code (Java + Spring Boot)"
date: 2025-07-27
description: "Lista profesional de comentarios para usar con GitHub Copilot en VS Code, con enfoque en Java, Spring Boot y arquitectura limpia."
tags: ["Copilot", "Java", "VS Code", "Productividad", "Spring Boot"]
categories: ["IA para desarrolladores"]

---

![Copilot Prompts](/images/post/banner-copilot-prompts.png)

> 🎯 Usa estos comentarios dentro de tu código en VS Code para que **Copilot entienda lo que necesitas** y te sugiera código útil, limpio y contextualizado.

---

## 📌 Frases ideales para Copilot en VS Code (como `//`)

### 🔹 Generación de código básico

```java
// Create a method to calculate the total price with tax
// Implement a REST controller for customer registration
// Generate a model class for CreditCard with fields: number, holderName, limit
// Write a unit test for this service method
```

---

### 🔹 Limpieza y refactorización

```java
// Refactor this method to reduce complexity
// Replace System.out with Lombok logger (@Slf4j)
// Convert this logic to use Java 8 streams
// Extract this logic into a separate utility class
```

---

### 🔹 Lógica de negocio

```java
// Validate customer age is over 18 before approving the card
// Check if the credit card limit is sufficient before proceeding
// Implement retry logic if the external API fails
// Add audit trail when user updates personal data
```

---

### 🔹 Arquitectura hexagonal / DDD

```java
// Create a use case to handle credit card application
// Define a port interface for customer repository
// Implement an adapter for Oracle database using JdbcTemplate
// Map domain entity to DTO for external API
```

---

### 🔹 Test y TDD

```java
// Write a JUnit test for CreditCardService
// Mock the customer repository and return a fake customer
// Verify that the audit event is sent when data is updated
```

---

### 🔹 Desempeño y calidad

```java
// Optimize this loop for performance
// Add null checks and input validation
// Suggest improvements for readability
// Handle edge cases for this validation
```

---

## 💡 Ejemplos prácticos con Copilot

### 🧪 Antes:
```java
System.out.println("Cliente aprobado");
```

### ✨ Comentario Copilot:
```java
// Replace System.out with Lombok logger
```

### ✅ Resultado sugerido por Copilot:
```java
@Slf4j
public class ClienteService {
  public void notificarAprobacion() {
    log.info("Cliente aprobado");
  }
}
```

---

### 🧪 Antes:
```java
// lógica mezclada en un método gigante
```

### ✨ Comentario Copilot:
```java
// Refactor this method to reduce complexity
```

### ✅ Resultado sugerido por Copilot:
```java
public BigDecimal calcularTotal(Pedido pedido) {
  BigDecimal subtotal = calcularSubtotal(pedido);
  BigDecimal impuestos = calcularImpuestos(pedido);
  return subtotal.add(impuestos);
}
```

---

