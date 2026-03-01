¡Claro! Aquí tienes un esquema de curso con ejemplos para entender la Arquitectura Hexagonal en Java.

## Curso: Entendiendo la Arquitectura Hexagonal en Java

### Módulo 1: Introducción a la Arquitectura Hexagonal

**Objetivo:** Comprender los fundamentos y la motivación detrás de la Arquitectura Hexagonal.

**Temas:**
1.  **¿Qué es la Arquitectura Hexagonal?**
    *   Problemas que resuelve (acoplamiento, dificultad de testing).
    *   Conceptos clave: Puertos (Ports) y Adaptadores (Adapters).
    *   La metáfora del "Hexágono".
2.  **Principios de Diseño Implicados:**
    *   Separación de preocupaciones.
    *   Inversión de Dependencias (DIP).
    *   Principio de Abierto/Cerrado (OCP).
3.  **Beneficios:**
    *   Mayor testabilidad.
    *   Mayor flexibilidad y adaptabilidad a cambios.
    *   Aislamiento de la lógica de negocio.

**Ejemplo Práctico 1.1: El Problema del Acoplamiento (Demo)**
*   Crear una aplicación Java simple sin Arquitectura Hexagonal que persista datos directamente a una base de datos y muestre cómo los cambios en la capa de persistencia afectan a la lógica de negocio.

### Módulo 2: Diseñando el Dominio (El Core del Hexágono)

**Objetivo:** Definir la lógica de negocio pura e independiente de cualquier tecnología externa.

**Temas:**
1.  **Identificando el Dominio:**
    *   Entidades (Entities).
    *   Objetos de Valor (Value Objects).
    *   Agregados (Aggregates).
    *   Servicios de Dominio (Domain Services).
2.  **Definiendo Puertos (Ports):**
    *   **Puertos de Entrada (Driving Ports / Inbound Ports):** Interfaces que la aplicación ofrece a los adaptadores externos para interactuar con el dominio (ej. `CrearProductoService`).
    *   **Puertos de Salida (Driven Ports / Outbound Ports):** Interfaces que el dominio requiere para interactuar con sistemas externos (ej. `ProductoRepository`, `EmailService`).
3.  **Implementando la Lógica de Negocio:**
    *   Clases que implementan la lógica principal, orquestando las interacciones a través de los puertos.

**Ejemplo Práctico 2.1: Diseño de Dominio para un Sistema de Gestión de Productos**
*   **Dominio:** `Producto` (Entity), `ProductoId` (Value Object).
*   **Puerto de Entrada:** `CrearProductoUseCase` (interfaz).
*   **Puerto de Salida:** `ProductoRepository` (interfaz).
*   **Implementación de Lógica de Negocio:** `DefaultCrearProductoUseCase` (implementa `CrearProductoUseCase` y usa `ProductoRepository`).

### Módulo 3: Conectando el Exterior (Adaptadores)

**Objetivo:** Implementar los mecanismos para que sistemas externos interactúen con el dominio y para que el dominio interactúe con sistemas externos.

**Temas:**
1.  **Adaptadores de Entrada (Driving Adapters / Inbound Adapters):**
    *   Implementan los puertos de entrada.
    *   Ejemplos: REST Controllers, CLI, UI, Message Listeners.
    *   Traducen las peticiones externas al lenguaje del dominio.
2.  **Adaptadores de Salida (Driven Adapters / Outbound Adapters):**
    *   Implementan los puertos de salida.
    *   Ejemplos: Implementaciones de bases de datos (JPA, JDBC), Clientes REST, Colas de mensajes (Kafka, RabbitMQ).
    *   Traducen las llamadas del dominio al formato del sistema externo.

**Ejemplo Práctico 3.1: Implementación de Adaptadores para el Sistema de Productos**
*   **Adaptador de Entrada (REST):** Un `ProductoController` (Spring MVC) que expone un endpoint `/productos` para crear productos, llamando a `CrearProductoUseCase`.
*   **Adaptador de Salida (JPA):** Una implementación de `ProductoRepository` usando Spring Data JPA para persistir `Producto` en una base de datos.
    *   *Opcional:* Implementación de `ProductoRepository` con una base de datos en memoria para testing.

### Módulo 4: Estructura de Proyectos y Dependencias

**Objetivo:** Organizar el código Java de forma que refleje la Arquitectura Hexagonal y gestione las dependencias correctamente.

**Temas:**
1.  **Estructura de Carpetas/Módulos:**
    *   `application` (o `core`, `domain`): Contiene el dominio y los puertos.
    *   `adapters`: Contiene las implementaciones de los adaptadores, subdivididos por tipo (ej. `in.web`, `out.persistence`, `out.messaging`).
    *   `bootstrap` (o `main`): El punto de entrada de la aplicación, donde se cablean los componentes.
2.  **Gestión de Dependencias (Maven/Gradle):**
    *   Asegurar que el dominio no dependa de los adaptadores.
    *   Inversión de Dependencia aplicada en el "wiring" de la aplicación.
    *   Demostrar cómo las dependencias fluyen hacia el centro y cómo el centro define las interfaces.

**Ejemplo Práctico 4.1: Organizando el Proyecto Java con Maven/Gradle**
*   Refactorizar el proyecto anterior en módulos Maven o paquetes siguiendo la estructura hexagonal.
*   Demostrar cómo las dependencias entre módulos/paquetes se alinean con los principios.
    *   `pom.xml` del módulo `application` no depende de `adapters`.
    *   `pom.xml` de `adapters` sí depende de `application`.

### Módulo 5: Testing en Arquitectura Hexagonal

**Objetivo:** Aprovechar la Arquitectura Hexagonal para realizar pruebas unitarias, de integración y end-to-end de manera efectiva.

**Temas:**
1.  **Testing del Dominio (Unit Tests):**
    *   Probar la lógica de negocio pura sin dependencias externas.
    *   Uso de mocks para puertos de salida.
2.  **Testing de Adaptadores de Entrada:**
    *   Probar el mapeo de entrada y la llamada al puerto de entrada.
    *   Ej: Tests de controladores REST con MockMvc.
3.  **Testing de Adaptadores de Salida:**
    *   Probar la interacción con sistemas externos (bases de datos, otros servicios).
    *   Uso de bases de datos en memoria o Testcontainers.
4.  **Testing de Integración/End-to-End:**
    *   Probar el flujo completo a través de todos los componentes.

**Ejemplo Práctico 5.1: Pruebas del Sistema de Productos**
*   **Test Unitario de `DefaultCrearProductoUseCase`:** Mockear `ProductoRepository`.
*   **Test de Integración de `ProductoController`:** Usar `MockMvc` e inyectar una implementación real (o mockeada) del `UseCase`.
*   **Test de Integración de `ProductoRepositoryJPA`:** Usar una base de datos en memoria (H2) o Testcontainers para probar la persistencia.

### Módulo 6: Integración con Frameworks Modernos (Spring Boot)

**Objetivo:** Aplicar la Arquitectura Hexagonal en un proyecto real usando Spring Boot.

**Temas:**
1.  **Configuración de Spring Boot:**
    *   Inyección de dependencias (`@Autowired`, `@Service`, `@Repository`).
    *   Cableado de Puertos y Adaptadores.
2.  **Manejo de Transacciones.**
3.  **Manejo de Excepciones.**
4.  **Consideraciones de Rendimiento y Escalabilidad.**

**Ejemplo Práctico 6.1: Aplicación Completa con Spring Boot y Arquitectura Hexagonal**
*   Construir una aplicación Spring Boot con los módulos definidos, integrando JPA para la persistencia, REST para la interfaz de usuario, y siguiendo estrictamente la separación de capas.
*   Demostrar cómo Spring facilita el "cableado" sin comprometer la arquitectura.

### Conclusiones y Siguientes Pasos

*   Recapitulación de los beneficios.
*   Cuándo y cuándo no usar Arquitectura Hexagonal.
*   Recursos adicionales y patrones relacionados (DDD, CQRS).

---

### Ilustraciones (a lo largo del curso)

Puedes insertar imágenes en puntos clave para visualizar los conceptos:

*   **El Hexágono:** Una representación clara del hexágono con el dominio en el centro y los adaptadores alrededor.
*   **Flujo de Peticiones:** Diagrama mostrando cómo una petición externa entra por un adaptador, pasa por el puerto de entrada, llega al dominio, usa un puerto de salida y sale por un adaptador de salida.
*   **Estructura de Paquetes:** Un diagrama de la estructura de paquetes/módulos en un proyecto Java.
*   **Diagrama de Dependencias:** Flechas mostrando cómo las dependencias fluyen siempre hacia el centro del hexágono.

---

Este esquema de curso proporciona una base sólida para entender y aplicar la Arquitectura Hexagonal en Java, con un enfoque práctico a través de ejemplos incrementales.

¿Te gustaría que profundicemos en algún módulo o ejemplo en particular?