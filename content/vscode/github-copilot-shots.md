---
title: "GitHub Copilot shoots"
date: 2025-10-22
weight: 6
---


---
Excelente, Efren ⚡
Vamos a desglosar **cómo aplicar las técnicas de Zero-Shot, One-Shot y Few-Shot Learning** directamente en el contexto de **GitHub Copilot Chat** con un proyecto **Java 17 + Spring Boot 3.4.x**, usando ejemplos reales de prompts.

---

## 🧠 Primero: ¿qué significan esos tres enfoques?

Son **estrategias de entrenamiento y prompting** que provienen del *Machine Learning*, pero también se aplican al **prompt engineering** —es decir, a *cómo le das instrucciones a Copilot*.

| Técnica       | Qué significa                                                | Cuándo usarla                                                          |
| :------------ | :----------------------------------------------------------- | :--------------------------------------------------------------------- |
| **Zero-Shot** | No das ningún ejemplo. Solo explicas la tarea.               | Cuando el resultado es simple o Copilot ya entiende el contexto.       |
| **One-Shot**  | Das **un solo ejemplo** antes de pedir la tarea.             | Cuando quieres que Copilot imite un formato o estilo específico.       |
| **Few-Shot**  | Das **varios ejemplos (2-5)** antes de la instrucción final. | Cuando quieres consistencia o seguir un patrón de arquitectura/código. |

---

## ⚙️ Ejemplo base del proyecto

Supón que tienes un microservicio **PolicyService** en tu app de seguros, y quieres generar métodos o pruebas relacionados con él.

---

## 🧩 1. Zero-Shot Prompting

> 🟢 *Copilot actúa solo con la instrucción general.*

**Prompt:**

```
Crea un endpoint REST en Spring Boot que reciba una póliza en JSON y la guarde en la base de datos usando PolicyService.
```

**Resultado típico:**
Copilot genera un controlador como este:

```java
@RestController
@RequestMapping("/api/policies")
public class PolicyController {

    private final PolicyService policyService;

    public PolicyController(PolicyService policyService) {
        this.policyService = policyService;
    }

    @PostMapping
    public ResponseEntity<Policy> createPolicy(@RequestBody Policy policy) {
        Policy saved = policyService.save(policy);
        return ResponseEntity.status(HttpStatus.CREATED).body(saved);
    }
}
```

**Ventaja:** rápido y directo.
**Desventaja:** puede variar el estilo o faltar documentación.

---

## 🧩 2. One-Shot Prompting

> 🟡 *Le das un ejemplo y pides que lo replique con otra entidad.*

**Prompt:**

```
Aquí tienes un ejemplo de cómo luce mi controlador estándar:
@GetMapping("/customers")
public ResponseEntity<List<Customer>> getAllCustomers() {
    return ResponseEntity.ok(customerService.findAll());
}

Crea un controlador similar para manejar pólizas.
```

**Resultado típico:**
Copilot aprende tu estilo y produce:

```java
@GetMapping("/policies")
public ResponseEntity<List<Policy>> getAllPolicies() {
    return ResponseEntity.ok(policyService.findAll());
}
```

**Ventaja:** mantiene tu formato y convención.
**Desventaja:** depende de la calidad del ejemplo.

---

## 🧩 3. Few-Shot Prompting

> 🔵 *Le das varios ejemplos para que entienda tu patrón arquitectónico completo.*

**Prompt:**

```
Ejemplo 1:
@GetMapping("/customers")
public ResponseEntity<List<Customer>> getAllCustomers() {
    return ResponseEntity.ok(customerService.findAll());
}

Ejemplo 2:
@PostMapping("/customers")
public ResponseEntity<Customer> createCustomer(@RequestBody Customer customer) {
    return ResponseEntity.status(HttpStatus.CREATED).body(customerService.save(customer));
}

Ahora crea los mismos dos endpoints para la entidad Policy.
```

**Resultado típico:**
Copilot genera ambos endpoints con la misma estructura, convenciones y estilo de respuesta:

```java
@GetMapping("/policies")
public ResponseEntity<List<Policy>> getAllPolicies() {
    return ResponseEntity.ok(policyService.findAll());
}

@PostMapping("/policies")
public ResponseEntity<Policy> createPolicy(@RequestBody Policy policy) {
    return ResponseEntity.status(HttpStatus.CREATED).body(policyService.save(policy));
}
```

**Ventaja:** el resultado es consistente y perfectamente alineado con tus prácticas internas.
**Desventaja:** el prompt es más largo, pero vale la pena para tareas repetitivas o generación masiva.

---

## ⚡ Aplicaciones reales en un proyecto Spring Boot

| Caso                           | Tipo de Prompt | Ejemplo                                                                                    |
| :----------------------------- | :------------- | :----------------------------------------------------------------------------------------- |
| Generar DTOs y Mappers         | **Zero-Shot**  | “Genera un DTO para Policy con id, amount y customerName.”                                 |
| Replicar estructura de pruebas | **One-Shot**   | “Basado en esta prueba de CustomerService, crea una para PolicyService.”                   |
| Crear endpoints uniformes      | **Few-Shot**   | “Siguiendo los 3 ejemplos de controllers que te muestro, genera uno para ClaimController.” |

---

## 🧠 Tip Pro: combinar con `#`

Puedes combinar estas técnicas con el **hash `#`** para dar contexto adicional sin pegar todo el código.
Ejemplo:

```
#CustomerServiceTest.java
Usa este archivo como referencia (one-shot) y crea una prueba similar para #PolicyService.java.
```

---

## 🔍 En resumen

| Técnica       | Contexto        | Ideal para                  | Nivel de control |
| :------------ | :-------------- | :-------------------------- | :--------------- |
| **Zero-Shot** | Sin ejemplos    | Código nuevo, rápido        | 🔹 Bajo          |
| **One-Shot**  | Un ejemplo      | Reutilizar formato o estilo | 🔸 Medio         |
| **Few-Shot**  | Varios ejemplos | Establecer patrones o guías | 🔺 Alto          |

---

💬 **Cyberhack  tip:**

> “Cuando trabajes con Copilot Chat, piensa en ti como quien le enseña al modelo ‘cómo se programa en tu equipo’.
> Zero-Shot es como darle una orden, One-Shot es como mostrarle una plantilla, y Few-Shot es como entrenarlo en tu estilo.” ⚡

---
