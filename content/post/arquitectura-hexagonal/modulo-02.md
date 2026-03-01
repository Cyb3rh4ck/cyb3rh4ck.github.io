¡Perfecto! Una vez comprendidos los fundamentos, es hora de adentrarnos en el corazón de la Arquitectura Hexagonal: el dominio.

## Módulo 2: Diseñando el Dominio (El Core del Hexágono)

**Objetivo:** Definir la lógica de negocio pura e independiente de cualquier tecnología externa.

En este módulo, aprenderemos a identificar, modelar y construir la parte más valiosa de nuestra aplicación: la lógica de negocio. Esta parte debe ser agnóstica a la infraestructura y contener las reglas, datos y comportamientos fundamentales que definen el propósito de nuestro software.

### 1. Identificando el Dominio

El dominio es el "qué" de tu aplicación, no el "cómo". Es donde se encuentran las reglas que hacen que tu negocio funcione. Para modelarlo correctamente, nos apoyamos en conceptos clave del Domain-Driven Design (DDD), aunque no profundizaremos completamente en DDD aquí.

*   **Entidades (Entities):** Son objetos que tienen una identidad única y un ciclo de vida, y cuyas características pueden cambiar con el tiempo. Su identidad es lo que los hace distintos, no sus atributos.
    *   **Ejemplos:** `Producto`, `Usuario`, `Pedido`, `CuentaBancaria`.
    *   **En Java:** Generalmente son clases con un ID único.

*   **Objetos de Valor (Value Objects):** Son objetos que representan un concepto descriptivo del dominio y no tienen una identidad conceptual propia. Se definen por sus atributos y son inmutables. Dos objetos de valor con los mismos atributos se consideran iguales.
    *   **Ejemplos:** `Dinero` (cantidad y moneda), `Direccion`, `Email`, `NombreCompleto`.
    *   **En Java:** Clases inmutables, a menudo con un constructor y *getters*, y sobreescribiendo `equals()` y `hashCode()`.

*   **Agregados (Aggregates):** Son un clúster de entidades y objetos de valor que son tratados como una unidad transaccional. Tienen una **raíz agregada (Aggregate Root)**, que es una entidad que actúa como puerta de entrada para todas las operaciones dentro del agregado. Solo la raíz agregada puede ser referenciada desde fuera del agregado. Esto asegura la consistencia de los datos dentro del clúster.
    *   **Ejemplos:** Un `Pedido` (raíz agregada) con sus `LineasDePedido` (entidades) y `DireccionDeEnvio` (objeto de valor).
    *   **En Java:** La clase raíz contendrá referencias a las otras entidades y objetos de valor, y será responsable de orquestar sus cambios.

*   **Servicios de Dominio (Domain Services):** Son operaciones que no encajan naturalmente en una entidad u objeto de valor. Coordinan la lógica de negocio que involucra múltiples entidades o agregados, o que realiza alguna operación significativa del dominio. Deben ser sin estado (stateless) y su comportamiento es la clave.
    *   **Ejemplos:** `TransferenciaDeFondosService`, `CalculadorDeEnvio`, `ValidadorDePromociones`.
    *   **En Java:** Clases de servicio que operan sobre una o más entidades o agregados.

### 2. Definiendo Puertos (Ports)

Los Puertos son el contrato entre tu dominio y el mundo exterior. Son simplemente interfaces Java que describen las capacidades que el dominio ofrece o las dependencias que el dominio necesita.

*   **Puertos de Entrada (Driving Ports / Inbound Ports):**
    *   **Propósito:** Definen cómo el mundo exterior (los adaptadores de entrada) puede interactuar con la lógica de negocio del hexágono. Son las "APIs" que tu aplicación expone.
    *   **Características:** Son interfaces que el dominio implementa (o un servicio de dominio dentro del hexágono implementa).
    *   **Nombres comunes:** Suelen terminar en `UseCase`, `Service`, `CommandBus`, `QueryBus`. Representan una intención o un caso de uso del negocio.
    *   **Ejemplo:**
        ```java
        // Puerto de Entrada: Define el caso de uso para crear un producto
        package com.example.hexagonal.application.port.in; // Ubicado dentro del 'application' o 'domain'

        public interface CreateProductUseCase {
            // El dominio interactúa con objetos de dominio, no con DTOs de infraestructura
            Product createProduct(CreateProductCommand command);

            // Objeto de Comando, parte del contrato del puerto
            class CreateProductCommand {
                private final String name;
                private final String description;
                private final double price;

                public CreateProductCommand(String name, String description, double price) {
                    this.name = name;
                    this.description = description;
                    this.price = price;
                }

                public String getName() { return name; }
                public String getDescription() { return description; }
                public double getPrice() { return price; }
            }
        }
        ```
        *Nota:* El `CreateProductCommand` es un objeto de valor específico para el comando, parte del contrato del puerto.

*   **Puertos de Salida (Driven Ports / Outbound Ports):**
    *   **Propósito:** Definen lo que el hexágono necesita del mundo exterior (los adaptadores de salida) para realizar sus operaciones. Son las "dependencias" que el dominio necesita.
    *   **Características:** Son interfaces que el dominio **usa**, pero que serán implementadas por la infraestructura (los adaptadores de salida).
    *   **Nombres comunes:** Suelen terminar en `Repository`, `Port`, `Sender`, `Publisher`, `Client`.
    *   **Ejemplo:**
        ```java
        // Puerto de Salida: Define la interfaz para persistir productos
        package com.example.hexagonal.application.port.out; // Ubicado dentro del 'application' o 'domain'

        import com.example.hexagonal.domain.model.Product;
        import com.example.hexagonal.domain.model.ProductId;

        import java.util.Optional;

        public interface ProductRepository {
            void save(Product product);
            Optional<Product> findById(ProductId id);
            // Otros métodos como delete, update, findAll...
        }
        ```

### 3. Implementando la Lógica de Negocio

Esta es la implementación real de los casos de uso definidos por los puertos de entrada, utilizando las capacidades proporcionadas por los puertos de salida. Estas clases residen completamente dentro del hexágono.

*   No deben tener dependencias directas a frameworks o tecnologías específicas (Spring, JPA, etc.).
*   Orquestan las entidades y objetos de valor del dominio.
*   Utilizan los puertos de salida para interactuar con el exterior.

**Ejemplo Práctico 2.1: Diseño de Dominio para un Sistema de Gestión de Productos**

Vamos a construir el dominio para un sistema simple de gestión de productos.

**1. Definir Entidades y Objetos de Valor:**

```java
// Archivo: com/example/hexagonal/domain/model/ProductId.java
package com.example.hexagonal.domain.model;

import java.util.Objects;
import java.util.UUID;

// Objeto de Valor: Representa la identidad de un producto
public class ProductId {
    private final UUID value;

    public ProductId(UUID value) {
        this.value = Objects.requireNonNull(value, "ProductId value cannot be null");
    }

    public static ProductId generate() {
        return new ProductId(UUID.randomUUID());
    }

    public UUID getValue() {
        return value;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        ProductId productId = (ProductId) o;
        return value.equals(productId.value);
    }

    @Override
    public int hashCode() {
        return Objects.hash(value);
    }

    @Override
    public String toString() {
        return "ProductId{" + "value=" + value + '}';
    }
}
```

```java
// Archivo: com/example/hexagonal/domain/model/Product.java
package com.example.hexagonal.domain.model;

import java.util.Objects;

// Entidad: Representa un producto con su identidad y atributos
public class Product {
    private final ProductId id; // Identidad del producto
    private String name;
    private String description;
    private double price;
    private boolean available;

    // Constructor para crear un nuevo producto
    public Product(String name, String description, double price) {
        this.id = ProductId.generate(); // Genera un ID al crear
        this.name = Objects.requireNonNull(name, "Product name cannot be null");
        this.description = Objects.requireNonNull(description, "Product description cannot be null");
        this.price = price;
        this.available = true; // Por defecto, disponible al crear
        validatePrice(price);
    }

    // Constructor para reconstruir un producto existente desde la persistencia
    public Product(ProductId id, String name, String description, double price, boolean available) {
        this.id = Objects.requireNonNull(id, "ProductId cannot be null");
        this.name = Objects.requireNonNull(name, "Product name cannot be null");
        this.description = Objects.requireNonNull(description, "Product description cannot be null");
        this.price = price;
        this.available = available;
        validatePrice(price);
    }

    // Lógica de negocio encapsulada dentro de la entidad
    public void updateDetails(String newName, String newDescription, double newPrice) {
        this.name = Objects.requireNonNull(newName, "Product name cannot be null");
        this.description = Objects.requireNonNull(newDescription, "Product description cannot be null");
        validatePrice(newPrice);
        this.price = newPrice;
    }

    public void makeUnavailable() {
        this.available = false;
    }

    public void makeAvailable() {
        this.available = true;
    }

    private void validatePrice(double price) {
        if (price < 0) {
            throw new IllegalArgumentException("Price cannot be negative.");
        }
    }

    // Getters
    public ProductId getId() { return id; }
    public String getName() { return name; }
    public String getDescription() { return description; }
    public double getPrice() { return price; }
    public boolean isAvailable() { return available; }

    // No setters públicos directos para mantener el control sobre los cambios de estado.
    // Los cambios se realizan a través de métodos con lógica de negocio (ej. updateDetails).
}
```

**2. Definir Puertos (ya vistos, pero los colocamos en su lugar conceptual):**

```java
// Puerto de Entrada: com/example/hexagonal/application/port/in/CreateProductUseCase.java
// (Código igual al mostrado anteriormente)
package com.example.hexagonal.application.port.in;

import com.example.hexagonal.domain.model.Product; // Importamos la entidad de nuestro dominio

public interface CreateProductUseCase {
    Product createProduct(CreateProductCommand command);

    class CreateProductCommand {
        private final String name;
        private final String description;
        private final double price;

        public CreateProductCommand(String name, String description, double price) {
            this.name = name;
            this.description = description;
            this.price = price;
        }

        public String getName() { return name; }
        public String getDescription() { return description; }
        public double getPrice() { return price; }
    }
}
```

```java
// Puerto de Salida: com/example/hexagonal/application/port/out/ProductRepository.java
// (Código igual al mostrado anteriormente)
package com.example.hexagonal.application.port.out;

import com.example.hexagonal.domain.model.Product;
import com.example.hexagonal.domain.model.ProductId;

import java.util.Optional;

public interface ProductRepository {
    void save(Product product);
    Optional<Product> findById(ProductId id);
}
```

**3. Implementar la Lógica de Negocio (Servicio de Dominio / Caso de Uso):**

Esta clase implementa el puerto de entrada `CreateProductUseCase` y utiliza el puerto de salida `ProductRepository`.

```java
// Archivo: com/example/hexagonal/application/service/DefaultCreateProductService.java
package com.example.hexagonal.application.service;

import com.example.hexagonal.application.port.in.CreateProductUseCase;
import com.example.hexagonal.application.port.out.ProductRepository;
import com.example.hexagonal.domain.model.Product; // Importamos la entidad de nuestro dominio

// Esta clase es el "motor" que implementa la lógica del caso de uso
public class DefaultCreateProductService implements CreateProductUseCase {

    private final ProductRepository productRepository; // Dependencia del puerto de salida

    // El constructor recibe el puerto de salida (abstracción), no una implementación concreta
    public DefaultCreateProductService(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    @Override
    public Product createProduct(CreateProductCommand command) {
        // Validación de negocio adicional (si aplica)
        if (command.getPrice() <= 0) {
            throw new IllegalArgumentException("Product price must be positive.");
        }

        // Crear una nueva instancia de la entidad de dominio
        Product newProduct = new Product(
                command.getName(),
                command.getDescription(),
                command.getPrice()
        );

        // Usar el puerto de salida para persistir el producto
        productRepository.save(newProduct);

        System.out.println("Product created in domain: " + newProduct.getName() + " with ID: " + newProduct.getId().getValue());

        return newProduct;
    }
}
```

**Estructura de Directorios/Paquetes (hasta ahora):**

```
src/main/java
└── com/example/hexagonal
    └── application
        ├── port
        │   ├── in    // Puertos de Entrada
        │   │   └── CreateProductUseCase.java
        │   └── out   // Puertos de Salida
        │       └── ProductRepository.java
        └── service   // Implementaciones de los Puertos de Entrada (Lógica de Negocio)
            └── DefaultCreateProductService.java
    └── domain
        └── model     // Entidades y Objetos de Valor
            ├── Product.java
            └── ProductId.java
```

**Análisis de la Arquitectura Hexagonal en este módulo:**

*   **Dominio Puro:** Las clases `Product` y `ProductId` no tienen ninguna referencia a Spring, JPA, o cualquier otra tecnología. Son POJOs (Plain Old Java Objects) que representan el negocio.
*   **Puertos como Contratos:** `CreateProductUseCase` y `ProductRepository` son interfaces. Definen "qué" hacer, no "cómo".
*   **Lógica de Negocio Aislada:** `DefaultCreateProductService` implementa la lógica de crear un producto. Toma un `CreateProductCommand` y utiliza un `ProductRepository`. No le importa cómo se persiste el producto ni cómo se le llama desde la interfaz de usuario.
*   **Inversión de Dependencias:** El `DefaultCreateProductService` (módulo de alto nivel) depende de la interfaz `ProductRepository` (abstracción), no de una implementación concreta de base de datos. Esto es clave.

Con esto, tenemos nuestro dominio central bien definido y desacoplado. En el próximo módulo, veremos cómo conectar este dominio con el mundo exterior utilizando Adaptadores.

¿Alguna pregunta sobre el diseño del dominio, los puertos o la lógica de negocio en este módulo?