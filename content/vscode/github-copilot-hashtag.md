---
title: "GitHub Copilot #"
date: 2025-10-22
weight: 4
---


---
⚡
Vamos a desmenuzar **el uso del símbolo `#` en GitHub Copilot Chat**, aplicado específicamente a un **proyecto Java 17 con Spring Boot 3.4.x**, y cómo puedes aprovecharlo en tu flujo real de desarrollo.

---

## 🧠 ¿Qué hace el símbolo `#` en Copilot Chat?

El **símbolo `#` (hash)** sirve para **añadir contexto de código o archivos específicos** a tu conversación con Copilot Chat.
Básicamente, le estás diciendo al modelo:

> “Mira este archivo, clase o método específico, y respóndeme con base en eso.”

Esto ayuda a que la respuesta sea **más precisa y localizada**, especialmente cuando trabajas en proyectos grandes o con muchas capas (como los de Spring Boot).

---

## ⚙️ Cómo funciona el `#` en VS Code

Cuando estás en el chat de Copilot, puedes escribir algo como:

```
#PolicyService.java
```

o

```
#savePolicy
```

y Copilot entiende que **ese archivo o método** es el contexto sobre el que quieres preguntar o trabajar.

🔹 También puedes usar el comando **“Add Context”** (botón con ícono de clip 📎 en VS Code) para adjuntar archivos, métodos o carpetas completas al chat.
Esto internamente se traduce igual: Copilot los referencia con `#`.

---

## 🔒 Seguridad y alcance

Copilot **no escanea tu disco duro completo**, solo:

* Archivos abiertos en el editor.
* Archivos o métodos que tú referencias con `#`.
* El workspace activo (si usas `#codebase`, aunque este es más general).

Así que, cuando usas `#`, estás dándole permiso explícito de **ver ese fragmento de código**.

---

## 🚀 Ejemplos prácticos para proyectos Java + Spring Boot

### 🧩 1. Revisar la lógica de un método específico

Supón que tienes un servicio así:

```java
@Service
public class PolicyService {

    public BigDecimal calculatePremium(Policy policy) {
        BigDecimal base = policy.getAmount();
        if (policy.isHighRisk()) {
            base = base.multiply(BigDecimal.valueOf(1.2));
        }
        return base;
    }
}
```

👉 Prompt:

```
#calculatePremium refactoriza este método para usar Optional y mejorar legibilidad.
```

💡 Copilot entenderá que `calculatePremium` está dentro de `PolicyService.java` y devolverá algo así:

```java
public BigDecimal calculatePremium(Policy policy) {
    return Optional.ofNullable(policy)
        .map(Policy::getAmount)
        .map(amount -> policy.isHighRisk() ? amount.multiply(BigDecimal.valueOf(1.2)) : amount)
        .orElse(BigDecimal.ZERO);
}
```

---

### 🧩 2. Generar una prueba unitaria para un método concreto

👉 Prompt:

```
#generaTest crea una prueba unitaria para #calculatePremium usando JUnit 5.
```

Copilot ubicará el método dentro de `PolicyService.java` y generará algo como:

```java
@ExtendWith(MockitoExtension.class)
class PolicyServiceTest {

    private final PolicyService service = new PolicyService();

    @Test
    void debeCalcularPrimaParaPolizaDeAltoRiesgo() {
        Policy policy = new Policy(BigDecimal.valueOf(1000), true);
        BigDecimal result = service.calculatePremium(policy);
        assertEquals(BigDecimal.valueOf(1200), result);
    }
}
```

---

### 🧩 3. Explicar un archivo completo

👉 Prompt:

```
#PolicyController.java explica qué hace este controlador y qué endpoints expone.
```

Copilot analizará solo ese archivo y podría responder:

> “El `PolicyController` maneja solicitudes REST para crear y listar pólizas.
> Tiene dos endpoints: `GET /api/policies` y `POST /api/policies`.
> Depende del servicio `PolicyService` para la lógica de negocio.”

---

### 🧩 4. Revisar dependencias o repositorios

👉 Prompt:

```
#PolicyRepository.java revisa si esta interfaz cumple las buenas prácticas de JPA y si necesita anotaciones adicionales.
```

Copilot leerá ese archivo y revisará si estás extendiendo correctamente `JpaRepository`, si faltan anotaciones como `@Repository` o si hay imports redundantes.

---

### 🧩 5. Comparar versiones de una clase

Si tienes dos versiones abiertas del mismo archivo:

```
#PolicyService.java
#PolicyService_old.java
Compara ambas versiones y dime qué cambió en la lógica del método calculatePremium.
```

Copilot usará ambas referencias `#` y generará un diff con las diferencias relevantes.

---

## 🧠 Buenas prácticas con `#`

| Situación                 | Qué hacer                                                                                         |
| :------------------------ | :------------------------------------------------------------------------------------------------ |
| Clase o método grande     | Usa `#` solo en el método clave para que Copilot no se pierda en el contexto.                     |
| Revisión de código        | Adjunta varios archivos con `#` y pídele a Copilot un resumen o sugerencias.                      |
| Generar documentación     | Usa `#` sobre una clase y pide “genera JavaDocs para todos sus métodos públicos”.                 |
| Refactorizaciones grandes | Combina `#codebase` con uno o más `#archivo` para mantener el contexto global y el detalle local. |

---

## 💬 En resumen

| Símbolo         | Significado                                          | Uso principal                             |
| :-------------- | :--------------------------------------------------- | :---------------------------------------- |
| **`#archivo`**  | Contexto de un archivo específico                    | “Mira solo este código y trabaja con él.” |
| **`#método`**   | Contexto de un método dentro del archivo abierto     | “Refactoriza o explica esta parte.”       |
| **`#carpeta`**  | Contexto de todos los archivos dentro de esa carpeta | “Analiza el módulo completo.”             |
| **`#codebase`** | Contexto global del proyecto abierto                 | “Toma en cuenta todo el proyecto.”        |

---


