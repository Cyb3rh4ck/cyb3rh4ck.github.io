---
title: "Uso de Records en Java y Spring Boot"
date: 2026-03-01T12:00:00Z
draft: false
description: "Aprende qué son los Records en Java, cómo funcionan y sus casos de uso principales en aplicaciones Spring Boot."
tags: ["java", "spring-boot", "records", "dto", "backend"]
categories: ["Java", "Backend"]
toc: true
pin: false
---

# Uso de Records en Java y Spring Boot

En Java 16 se introdujeron de forma oficial los **Records**, una característica del lenguaje diseñada para crear clases inmutables que actúan principalmente como portadores de datos ("data carriers").

## El escenario: Mapear un JSON anidado

Supongamos que estamos construyendo un endpoint de registro en Spring Boot. El cliente nos envía el siguiente *payload* JSON:

```json
{
    "user": {
        "name": {
            "name": "Evan",
            "firstName": "Hernandez",
            "secondName": "Angeles"
        }
    },
    "login": {
        "email": "myemail@codigojava.net",
        "password": "mypassword"
    }
}
```

Históricamente, para mapear este objeto anidado crearíamos múltiples clases (DTOs) llenas de *getters*, *setters*, constructores por defecto, `equals()`, y `hashCode()`. Con los **Records**, podemos hacerlo en unas pocas líneas, de manera inmutable y muy expresiva.

## Arquitectura Hexagonal: ¿Dónde ubicamos los Records?

En una **Arquitectura Hexagonal** (Ports and Adapters), nuestros `Records` actúan como **DTOs** (Data Transfer Objects) que viven única y exclusivamente en la capa de adaptadores de entrada (*Inbound Adapters* / *Driving Adapters*). Su propósito es recibir la petición, validarla en la entrada y mapearla hacia un objeto puro en la capa de dominio.

Aquí tienes un vistazo de la estructura de directorios y el flujo de los componentes:

```text
src/main/java/com/codigojava/
├── domain/                      # Capa de Dominio (Core)
│   └── User.java                # Entidad de negocio
├── application/                 # Capa de Aplicación (Ports)
│   └── port/in/
│       └── RegisterUserUseCase.java
└── infrastructure/              # Capa de Infraestructura (Adapters)
    ├── input/
    │   └── rest/
    │       ├── RegistrationController.java
    │       ├── exception/
    │       │   └── GlobalExceptionHandler.java
    │       └── dto/             # ¡Aquí viven nuestros Records!
    │           ├── NameRequest.java
    │           ├── UserRequest.java
    │           ├── LoginRequest.java
    │           └── UserRegistrationRequest.java
    └── validation/              # Validaciones personalizadas
        └── EmpresarialEmail.java
```

{{< mermaid >}}
flowchart LR
    A[Cliente Rest] -->|JSON| B(RegistrationController)
    B -->|Valida DTO Record| C{Es Valido?}
    C -- No --> D[GlobalExceptionHandler]
    C -- Si --> E[RegisterUserUseCase]
    E --> F[Domain Model]
    
    classDef adapter fill:#f9f,stroke:#333,stroke-width:2px;
    classDef domain fill:#bbf,stroke:#333,stroke-width:2px;
    class B,D adapter;
    class E,F domain;
{{< /mermaid >}}

---

## Validación Personalizada: Email Empresarial

Antes de definir nuestros records, queremos añadir una regla de negocio estricta en nuestro Entrypoint: **el correo electrónico debe ser empresarial (`@codigojava.io`)**. Para esto, creamos una anotación de validación personalizada.

```java
import jakarta.validation.Constraint;
import jakarta.validation.Payload;
import java.lang.annotation.*;

@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = EmpresarialEmailValidator.class)
@Documented
public @interface EmpresarialEmail {
    String message() default "El correo debe ser empresarial y terminar en @codigojava.io";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

Y su respectivo validador:

```java
import jakarta.validation.ConstraintValidator;
import jakarta.validation.ConstraintValidatorContext;

public class EmpresarialEmailValidator implements ConstraintValidator<EmpresarialEmail, String> {
    
    private static final String DOMAIN = "@codigojava.io";

    @Override
    public boolean isValid(String email, ConstraintValidatorContext context) {
        if (email == null || email.isBlank()) {
            return false;
        }
        return email.endsWith(DOMAIN);
    }
}
```

## Definiendo los Records con Validaciones (Jakarta Validation)

Para asegurarnos de que los datos de entrada son correctos, combinaremos los `record` con las anotaciones estándar de Jakarta y nuestra nueva anotación personalizada.

Podemos definir estos componentes por separado para inyectar validaciones estructuradas a cada capa de nuestro JSON:

```java
import jakarta.validation.Valid;
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Size;

// 1. Record para el Nombre
public record NameRequest(
    @NotBlank(message = "El nombre es obligatorio")
    String name,
    
    @NotBlank(message = "El apellido paterno es obligatorio")
    String firstName,
    
    String secondName // Opcional, sin validación estricta
) {}

// 2. Record para el Usuario
public record UserRequest(
    @NotNull(message = "Los datos del nombre son requeridos")
    @Valid 
    NameRequest name
) {}

// 3. Record para el Login
public record LoginRequest(
    @NotBlank(message = "El email no puede estar vacío")
    @Email(message = "Formato de email inválido")
    @EmpresarialEmail // Nuestra validación personalizada
    String email,
    
    @NotBlank(message = "La contraseña es obligatoria")
    @Size(min = 8, message = "La contraseña debe tener al menos 8 caracteres")
    String password
) {}

// 4. Record Raíz (El Payload Completo evaluado en el Endpoint)
public record UserRegistrationRequest(
    @NotNull(message = "El objeto user es obligatorio")
    @Valid 
    UserRequest user,
    
    @NotNull(message = "El objeto login es obligatorio")
    @Valid 
    LoginRequest login
) {}
```

> **Nota sobre `@Valid`**: Es crucial añadir la anotación `@Valid` dentro de los registros padre (`UserRegistrationRequest` y `UserRequest`) para que Spring propague la validación en cascada a todos los objetos anidados. Sin esto, sólo validaría si el objeto padre existe, ignorando el contenido interno.

## Uso en un RestController de Spring Boot

Ahora que tenemos nuestros modelos inmutables validados, podemos utilizarlos en nuestro controlador usando la anotación `@RequestBody` y detonando el proceso de validación en la firma del método con `@Valid`.

```java
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import jakarta.validation.Valid;

@RestController
@RequestMapping("/api/v1/users")
public class RegistrationController {

    @PostMapping("/register")
    public ResponseEntity<String> registerUser(
            @Valid @RequestBody UserRegistrationRequest request) {
        
        // Accediendo a los datos del Record
        // Nota: Los records exponen métodos con el mismo nombre que sus atributos (no hay prefijos "get" o "set")
        String userEmail = request.login().email();
        String firstName = request.user().name().firstName();

        // Lógica de registro...
        return ResponseEntity.ok("Usuario " + firstName + " registrado exitosamente con el email " + userEmail);
    }
}
```

## Manejo Global de Excepciones (@ControllerAdvice)

Cuando una validación falla en nuestro controlador (porque un campo es nulo o el correo no cumple con `@EmpresarialEmail`), Spring internamente rechaza la petición y lanza una excepción `MethodArgumentNotValidException`. 

Dentro de nuestra capa de infraestructura REST, podemos implementar un manejador global de excepciones para atrapar automáticamente estos errores y devolverle al cliente una respuesta limpia (por ejemplo, los puristas del clean code suelen mapearlo a un estándar como [Problem Details para HTTP APIs - RFC 7807](https://www.rfc-editor.org/rfc/rfc7807)):

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.util.HashMap;
import java.util.Map;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationExceptions(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        
        ex.getBindingResult().getFieldErrors().forEach(error -> {
            String fieldName = error.getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });

        // Retorna un objeto JSON con los errores mapeados por campo
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(errors);
    }
}
```

Si el cliente consume el endpoint enviando el correo `evan@gmail.com` y dejando la contraseña de 3 digitos, nuestro `ControllerAdvice` interceptará el objeto y responderá con algo asÍ:

```json
{
    "login.email": "El correo debe ser empresarial y terminar en @codigojava.io",
    "login.password": "La contraseña debe tener al menos 8 caracteres"
}
```

## ¿Por qué esto es mejor que una Clase normal (`class`) en Java?

1. **Inmutabilidad por defecto**: Una vez que Spring Boot deserializa el JSON y crea el Record, los campos no pueden ser alterados (`final` por defecto). Esto resulta perfecto para la capa Web ya que un Request enviado al Backend no debería cambiar en medio del proceso (previene mutaciones accidentales en los controladores o servicios).
2. **Cero Boilerplate**: Nos libramos de abusar de la anotación `@Data` de Lombok o de teclear a mano constructores, *getters* y `toString()`. El código es más legible y puro Java.
3. **Lectura directa**: El acceso a los datos se hace llamando al nombre de la propiedad en sí (`request.login().email()`), una sintaxis más limpia y directa.


