---
title: "Automatiza tu código con Copilot en IntelliJ"
date: 2025-09-06
description: "Guía paso a paso para instalar, configurar y usar GitHub Copilot en Intellij"
tags: ["Copilot", "Intellij", "Java", "Productividad", "Spring Boot"]
categories: ["IA para desarrolladores"]
toc: true
---

Aquí tienes cómo exprimir **IntelliJ IDEA Community** con **GitHub Copilot** para Java/Spring/Kafka como un pro 👇

# 1) Deja fino el setup

* **Plugins**: GitHub Copilot + Copilot Chat + SonarLint + Lombok + Maven Helper.
* **Keymap**: ve a *Settings → Keymap → GitHub Copilot* y asigna atajos cómodos para:

  * *Trigger Inline Suggestion* (invocar propuesta)
  * *Next/Previous Suggestion* (ciclar)
  * *Open Chat* (abrir chat lateral)
  * *Accept Word/Line* (aceptar por partes)
* **Scope**: activa Copilot sólo en los lenguajes que usas (Java, YAML, Markdown) para reducir ruido.

# 2) Flujo de trabajo que rinde

1. **Comentario → código**
   Escribe el comentario exacto de lo que quieres y luego invoca Copilot:

   ```java
   // Spring Boot 3: POST /v1/policies {policy} -> valida, guarda, produce evento Kafka "audit.customer.updated"
   ```
2. **TDD rápido**
   Pide primero las *pruebas* (JUnit 5 + Mockito) y luego genera la implementación. Copilot entiende muy bien los métodos esperados.
3. **Itera en línea**
   Acepta por palabra o por línea, y cicla hasta ver una variante con buenas firmas y excepciones.
4. **Chat > inline** cuando:

   * Quieres *explicación* de código legado.
   * Necesitas *refactor* guiado (“extrae puerto hexagonal”, “aplica DTO y mapper”).
   * Pides *tests de borde* y *propiedad parametrizada* (ParameterizedTest).

# 3) Prompts “cirujanos” para tu stack

Pégalos como comentario encima del método/clase o en Copilot Chat.

**Hexagonal + Kafka (Seguros360)**

* “Crea puerto `PolicyAuditPort` y adaptador Kafka con `spring-kafka`. Produce evento `AuditCustomerUpdated` con headers `correlationId`, `actor`, `reason`. Retrys idempotentes y DLQ.”
* “Valida JWT con `kid` firmado por AWS KMS (clave RSA). Extrae claims y roles. Maneja expiración y clockSkew.”

**Endpoints y validación**

* “Controller `PolicyController` POST `/v1/policies` con Bean Validation (campos obligatorios), `@ControllerAdvice` para errores 400/422 con payload estándar.”

**Persistencia**

* “Repositorio JPA para `Policy` con índice compuesto (customerId, productCode). Método `findActiveByCustomerId` paginado.”

**Observabilidad**

* “Integra `micrometer-tracing` y logs estructurados JSON (traceId, spanId, userId, route, latencyMs).”

**Resiliencia**

* “Configura Resilience4j para el cliente externo: retry exponencial (maxAttempts=3), timeout=2s, circuit breaker.”

# 4) Chat rápido (atajos útiles)

En Copilot Chat, pide acciones claras:

* “Explain this file in Spanish and list risky parts.”
* “Generate edge-case tests for `PolicyService#update`.”
* “Refactor to hex architecture: extract ports, move infra into adapters.”
* “Add MapStruct mapper + tests.”
* “Create .http requests for the controller with sample bodies.”

# 5) YML, Regex, SQL y .http: oro con Community

* **application.yml**: “Genera config para `spring-kafka` con SSL, timeouts y serializers.”
* **Regex**: “Escribe regex para validar RFC mexicano y prueba 5 casos.”
* **SQL**: “Crea migración Flyway para tabla policies con índices y `created_at`/`updated_at`.”
* **HTTP Client**: crea `requests.http` y pide a Copilot ejemplos con `GET/POST` (ideal para probar APIs sin salir de IntelliJ).

# 6) Plantillas y Live Templates (turbo)

* Define **File Templates** para:

  * `Controller`, `Service`, `Port`, `Adapter`, `Record DTO`, `ErrorResponse`.
* Crea **Live Templates** tipo `hexport`, `hexadapter`, `dto`, `err` y deja comentarios instructivos; Copilot completará el resto.

# 7) Calidad y seguridad sin fricción

* **SonarLint** en modo *Connected* si puedes; si no, local. Pídele a Copilot:
  “Fix issues Sonar: S2095, S112, S1452 in this class.”
* **Format & Inspections**: guarda con *Reformat + Optimize imports* para que Copilot “aprenda” tu estilo.

# 8) Estrategia para respuestas mejores

* Contexto primero: di el *qué* + *restricciones* (versiones, librerías, patrones).
* Pide *arquitectura previa* (interfaces, excepciones, DTOs) antes que la implementación.
* Da **ejemplos de entrada/salida** concretos (payload JSON y respuesta esperada).
* “No uses librerías X” o “Debes usar Y” para forzar el stack correcto.
* Repite *nombres exactos* (paquetes, topics, endpoints). Evita ambigüedad.

# 9) Recetas express (lista corta)

* “Create unit tests with Mockito and parameterized inputs for null/invalid cases.”
* “Document public methods with Javadoc including examples.”
* “Generate mapping between `Policy` and `PolicyDTO` with MapStruct and test it.”
* “Produce/consume Kafka with headers and error handler to DLQ.”
* “Write an exception hierarchy: domain, application, infra with `@ControllerAdvice` mapping.”

# 10) Qué NO delegar ciegamente

* Seguridad (JWT/KMS), transacciones y concurrencia: usa Copilot para *borradores*, valida tú las decisiones.
* Config sensible en YAML: revísala siempre y añade tests de integración.

---
