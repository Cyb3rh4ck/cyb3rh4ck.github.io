---
title: "Ejemplo práctico de Arquitectura Hexagonal con Spring Boot"
date: 2025-08-09
draft: false
tags: ["Spring Boot", "Arquitectura Hexagonal"]
categories: ["Spring Boot", "Arquitectura Hexagonal"]
---


Mini‑bootcamp práctico de Arquitectura Hexagonal aplicado a Spring Boot (con Commands, diseño, carpetas, y código listo para copiar/pegar). Iremos de conceptos → diseño → implementación → pruebas.

# 1) Qué es (de verdad) la arquitectura hexagonal

* **Hexágono = núcleo independiente**: tu dominio + casos de uso (application) no conocen frameworks ni detalles de I/O.
* **Puertos (Ports)**: interfaces que el núcleo expone o necesita.

  * **Inbound ports**: lo que tu app ofrece (casos de uso). Ej: `CreatePolicyUseCase`.
  * **Outbound ports**: lo que tu app necesita del exterior. Ej: `PolicyRepository`, `PaymentGateway`.
* **Adaptadores (Adapters)**: implementaciones concretas de puertos.

  * **Inbound adapters**: REST, gRPC, mensajería, CLI.
  * **Outbound adapters**: JPA, Mongo, S3, SMTP, Kafka, etc.
* **Dependencias**: `infra → application → domain` (nunca al revés). Spring y librerías “viven” en infra.

# 2) Mapa de capas (y responsabilidades)

* **domain**

  * Entidades, Value Objects, reglas de negocio, invariantes.
  * Sin anotaciones de Spring/JPA si puedes evitarlo.
* **application**

  * Casos de uso (Commands/Queries), orquestación, transacciones, seguridad de caso de uso.
  * Define **Ports** (interfaces) que necesita del exterior.
* **infrastructure**

  * Adapters, mapeos (DTO↔domain), persistencia (JPA/Mongo), config de Spring, wiring.

# 3) Commands y Queries (CQRS light)

* **Command**: intención que cambia estado. Pure data (idempotencia definida en handler).
  Ej: `CreatePolicyCommand { policyId, productCode, holderId }`.
* **Command Handler / Use Case**: servicio de aplicación que ejecuta la regla usando puertos.
* **Query**: lectura sin efectos (puede ir por otro puerto/repo optimizado).

# 4) Estructura de proyecto (maven multi‑módulo)

```
seguros360/
  pom.xml               <-- packaging: pom
  seguros360-domain/
  seguros360-application/
  seguros360-infrastructure/
```

**POM raíz (resumen)**

```xml
<packaging>pom</packaging>
<modules>
  <module>seguros360-domain</module>
  <module>seguros360-application</module>
  <module>seguros360-infrastructure</module>
</modules>
```

**Domain POM**

```xml
<dependencies>
  <!-- opcional: lombok para boilerplate, pero evita JPA aquí -->
</dependencies>
```

**Application POM**

```xml
<dependencies>
  <dependency>
    <groupId>io.cyb3rh4ck.github</groupId>
    <artifactId>seguros360-domain</artifactId>
    <version>${project.version}</version>
  </dependency>
</dependencies>
```

**Infrastructure POM**

```xml
<dependencies>
  <dependency>
    <groupId>io.cyb3rh4ck.github</groupId>
    <artifactId>seguros360-application</artifactId>
    <version>${project.version}</version>
  </dependency>
  <!-- Spring Boot, JPA/Mongo, Web, Kafka, etc. -->
</dependencies>
```

# 5) Código de ejemplo (Seguros360)

## 5.1 Domain

**Value Objects y Entidad**

```java
// seguros360-domain
package io.cyb3rh4ck.github.seguros360.domain.policy;

import java.util.Objects;

public record PolicyId(String value) {
  public PolicyId {
    if (value == null || value.isBlank()) throw new IllegalArgumentException("PolicyId blank");
  }
}

public enum PolicyStatus { DRAFT, ACTIVE, CANCELLED }

public record ProductCode(String value) {
  public ProductCode {
    if (value == null || value.isBlank()) throw new IllegalArgumentException("ProductCode blank");
  }
}

public final class Policy {
  private final PolicyId id;
  private final ProductCode product;
  private PolicyStatus status;

  public Policy(PolicyId id, ProductCode product, PolicyStatus status) {
    this.id = Objects.requireNonNull(id);
    this.product = Objects.requireNonNull(product);
    this.status = Objects.requireNonNull(status);
  }
  public PolicyId id() { return id; }
  public ProductCode product() { return product; }
  public PolicyStatus status() { return status; }

  public void changeStatus(PolicyStatus newStatus) { this.status = newStatus; }
}
```

## 5.2 Application (Ports + Commands + UseCases)

**Outbound Port**

```java
// seguros360-application
package io.cyb3rh4ck.github.seguros360.application.ports;

import io.cyb3rh4ck.github.seguros360.domain.policy.*;

import java.util.Optional;

public interface PolicyRepository {
  Optional<Policy> findById(PolicyId id);
  void save(Policy policy);
  boolean exists(PolicyId id);
}
```

**Commands / Use cases**

```java
package io.cyb3rh4ck.github.seguros360.application.commands;

import io.cyb3rh4ck.github.seguros360.domain.policy.*;

public record CreatePolicyCommand(String policyId, String productCode) {}

package io.cyb3rh4ck.github.seguros360.application.usecases;

import io.cyb3rh4ck.github.seguros360.application.commands.CreatePolicyCommand;
import io.cyb3rh4ck.github.seguros360.application.ports.PolicyRepository;
import io.cyb3rh4ck.github.seguros360.domain.policy.*;

public class CreatePolicyUseCase {
  private final PolicyRepository repo;

  public CreatePolicyUseCase(PolicyRepository repo) { this.repo = repo; }

  public void handle(CreatePolicyCommand cmd) {
    var id = new PolicyId(cmd.policyId());
    if (repo.exists(id)) throw new IllegalStateException("Policy already exists: " + id.value());
    var product = new ProductCode(cmd.productCode());
    var policy = new Policy(id, product, PolicyStatus.DRAFT);
    repo.save(policy);
  }
}
```

> Nota: Los **use cases no llevan @Service** (puedes ponerlos como beans en infra via `@Bean`), evitando coupling a Spring.

## 5.3 Infrastructure (Adapters + Web + Persistence)

**JPA Entity + Mapper**

```java
// seguros360-infrastructure
package io.cyb3rh4ck.github.seguros360.infrastructure.persistence.jpa;

import jakarta.persistence.*;

@Entity @Table(name = "policies")
public class PolicyJpaEntity {
  @Id private String id;
  private String productCode;
  @Enumerated(EnumType.STRING) private PolicyStatus status;

  // getters/setters (o Lombok)
}
```

**Spring Data repository**

```java
package io.cyb3rh4ck.github.seguros360.infrastructure.persistence.jpa;

import org.springframework.data.jpa.repository.JpaRepository;

public interface SpringDataPolicyRepository extends JpaRepository<PolicyJpaEntity, String> { }
```

**Adapter del puerto**

```java
package io.cyb3rh4ck.github.seguros360.infrastructure.persistence;

import io.cyb3rh4ck.github.seguros360.application.ports.PolicyRepository;
import io.cyb3rh4ck.github.seguros360.domain.policy.*;
import io.cyb3rh4ck.github.seguros360.infrastructure.persistence.jpa.*;
import org.springframework.stereotype.Component;

import java.util.Optional;

@Component
public class PolicyRepositoryJpaAdapter implements PolicyRepository {

  private final SpringDataPolicyRepository jpa;

  public PolicyRepositoryJpaAdapter(SpringDataPolicyRepository jpa) { this.jpa = jpa; }

  @Override public Optional<Policy> findById(PolicyId id) {
    return jpa.findById(id.value()).map(this::toDomain);
  }
  @Override public void save(Policy policy) {
    jpa.save(toEntity(policy));
  }
  @Override public boolean exists(PolicyId id) { return jpa.existsById(id.value()); }

  private Policy toDomain(PolicyJpaEntity e) {
    return new Policy(new PolicyId(e.getId()), new ProductCode(e.getProductCode()), e.getStatus());
  }
  private PolicyJpaEntity toEntity(Policy p) {
    var e = new PolicyJpaEntity();
    e.setId(p.id().value());
    e.setProductCode(p.product().value());
    e.setStatus(p.status());
    return e;
  }
}
```

**Wiring de UseCases como Beans**

```java
package io.cyb3rh4ck.github.seguros360.infrastructure.config;

import io.cyb3rh4ck.github.seguros360.application.ports.PolicyRepository;
import io.cyb3rh4ck.github.seguros360.application.usecases.CreatePolicyUseCase;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class UseCaseConfig {
  @Bean CreatePolicyUseCase createPolicyUseCase(PolicyRepository repo) {
    return new CreatePolicyUseCase(repo);
  }
}
```

**Controller (Inbound Adapter REST)**

```java
package io.cyb3rh4ck.github.seguros360.infrastructure.web;

import io.cyb3rh4ck.github.seguros360.application.commands.CreatePolicyCommand;
import io.cyb3rh4ck.github.seguros360.application.usecases.CreatePolicyUseCase;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/policies")
public class PolicyController {

  private final CreatePolicyUseCase createPolicy;

  public PolicyController(CreatePolicyUseCase createPolicy) { this.createPolicy = createPolicy; }

  @PostMapping
  public ResponseEntity<Void> create(@RequestBody CreatePolicyCommand cmd) {
    createPolicy.handle(cmd);
    return ResponseEntity.accepted().build();
  }
}
```

# 6) Transacciones, validación, errores

* **Transacciones** en la **capa application**, iniciadas desde el **adapter inbound** o con un *wrapping service* en infra:

  * Opción simple: marcar el **controller** o un **@Service façade** con `@Transactional`.
  * Opción más pura: un **TransactionalUseCaseExecutor** en infra que envuelva `useCase.handle()`.
* **Validación**: anota `Command` con `jakarta.validation` y valida en controller (`@Valid`) o en un pipeline.
* **Errores**: maneja excepciones de dominio y de aplicación en un `@ControllerAdvice`.

# 7) Tests (punto crítico)

* **domain**: unit tests puros (sin Spring).
* **application**: tests del caso de uso con **repos fake/in‑memory**.
* **infrastructure**: tests de adapter (con **Testcontainers** para DB), y **slice tests** (`@DataJpaTest`, `@WebMvcTest`).
* **end‑to‑end**: `@SpringBootTest` + base de datos efímera.

# 8) Scripts para arrancar rápido

## 8.1 Crear el multi‑módulo (Maven)

```bash
mvn -q -DgroupId=io.cyb3rh4ck.github -DartifactId=seguros360 -Dversion=0.0.1-SNAPSHOT -Dpackaging=pom archetype:generate -DinteractiveMode=false -DarchetypeArtifactId=maven-archetype-quickstart

cd seguros360
mvn -q -DgroupId=io.cyb3rh4ck.github.seguros360 -DartifactId=seguros360-domain -Dversion=0.0.1-SNAPSHOT -Dpackage=io.cyb3rh4ck.github.seguros360.domain -DinteractiveMode=false -DarchetypeArtifactId=maven-archetype-quickstart archetype:generate

mvn -q -DgroupId=io.cyb3rh4ck.github.seguros360 -DartifactId=seguros360-application -Dversion=0.0.1-SNAPSHOT -Dpackage=io.cyb3rh4ck.github.seguros360.application -DinteractiveMode=false -DarchetypeArtifactId=maven-archetype-quickstart archetype:generate

mvn -q -DgroupId=io.cyb3rh4ck.github.seguros360 -DartifactId=seguros360-infrastructure -Dversion=0.0.1-SNAPSHOT -Dpackage=io.cyb3rh4ck.github.seguros360.infrastructure -DinteractiveMode=false -DarchetypeArtifactId=maven-archetype-quickstart archetype:generate
```

## 8.2 Agregar Spring Boot sólo en **infra**

En `seguros360-infrastructure/pom.xml` añade:

```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>3.3.2</version>
</parent>

<dependencies>
  <dependency>
    <groupId>io.cyb3rh4ck.github.seguros360</groupId>
    <artifactId>seguros360-application</artifactId>
    <version>${project.version}</version>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>
  <dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
  </dependency>
  <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
  </dependency>
</dependencies>
```

**Clase main** en infra:

```java
package io.cyb3rh4ck.github.seguros360;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Seguros360Application {
  public static void main(String[] args) {
    SpringApplication.run(Seguros360Application.class, args);
  }
}
```

# 9) Checklist de “pureza” hexagonal

* [ ] Domain sin dependencias de Spring/JPA/HTTP.
* [ ] Application sólo conoce **ports** y **use cases**.
* [ ] Infra implementa outbound ports y expone inbound adapters.
* [ ] Mappers en infra (no contamines domain con DTOs).
* [ ] Transacciones fuera del dominio (app/infra).
* [ ] Tests por capa (rápidos en domain/application).


