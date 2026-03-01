---
title: "🚪 Spring Cloud Gateway: API Gateway Reactivo Completo"
date: 2025-08-08
draft: false
description: "Guía completa de Spring Cloud Gateway como API Gateway reactivo con WebFlux. Incluye configuración, filtros, predicates, testing y casos de uso prácticos."
tags: ["spring-cloud-gateway", "api-gateway", "webflux", "spring-boot", "microservicios", "reactive", "proxy", "load-balancer"]
---

# Spring Cloud Gateway Demo

Este proyecto demuestra el uso de **Spring Cloud Gateway** como un API Gateway reactivo usando WebFlux.

## 🏗️ Arquitectura

```
Cliente/Navegador
       ↓
Spring Cloud Gateway (Puerto 8080)
       ↓
httpbin.org (Servicio destino)
```

## 🚀 Cómo ejecutar

### Prerrequisitos
- Java 17+
- Maven 3.6+

### Ejecutar la aplicación

```bash
# Usando Maven Wrapper (recomendado)
./mvnw spring-boot:run

# En Windows
mvnw.cmd spring-boot:run

# O ejecutando el JAR
mvn clean package
java -jar target/spring-cloud-gateway-0.0.1-SNAPSHOT.jar
```

La aplicación se ejecutará en `http://localhost:8080`

## 🧪 Pruebas

### Endpoints disponibles

| Endpoint | Descripción | Ejemplo |
|----------|-------------|---------|
| `/api/get` | Proxy a httpbin.org/get | `http://localhost:8080/api/get` |
| `/api/headers` | Ver headers enviados | `http://localhost:8080/api/headers` |
| `/api/ip` | Ver tu IP pública | `http://localhost:8080/api/ip` |
| `/api/user-agent` | Ver tu User-Agent | `http://localhost:8080/api/user-agent` |
| `/api/post` | Prueba método POST | `http://localhost:8080/api/post` |

### Endpoints de monitoreo (Actuator)

| Endpoint | Descripción |
|----------|-------------|
| `/actuator/health` | Estado de salud |
| `/actuator/gateway/routes` | Rutas configuradas |

### Pruebas con curl

```bash
# GET básico
curl -i http://localhost:8080/api/get

# GET con parámetros
curl -i "http://localhost:8080/api/get?param1=test&param2=gateway"

# POST con JSON
curl -X POST http://localhost:8080/api/post \
  -H "Content-Type: application/json" \
  -d '{"test": "data from gateway"}'

# Ver headers (verificar X-Request-Source: gateway)
curl -i http://localhost:8080/api/headers

# Estado del gateway
curl http://localhost:8080/actuator/health

# Ver rutas configuradas
curl http://localhost:8080/actuator/gateway/routes
```

### Pruebas con navegador

Simplemente visita: `http://localhost:8080/api/get`

**Respuesta esperada:**
```json
{
  "args": {},
  "headers": {
    "X-Request-Source": "gateway",  // ← Header agregado por el gateway
    // ... otros headers
  },
  "origin": "tu-ip",
  "url": "https://httpbin.org/get"  // ← URL final después del proxy
}
```

## ⚙️ Configuración explicada

### Configuración completa del `application.yml`

```yaml
server:
  port: 8080

spring:
  cloud:
    gateway:
      server:
        webflux:
          routes:
            - id: httpbin-get
              uri: https://httpbin.org
              predicates:
                - Path=/api/**        # match incoming path
              filters:
                - AddRequestHeader=X-Request-Source, gateway
                - RewritePath=/api/(?<segment>.*), /${segment}

management:
  endpoints:
    web:
      exposure:
        include: health,gateway
```

### Estructura explicada

```yaml
server:
  port: 8080                        # Puerto del gateway

spring:
  cloud:
    gateway:
      server:
        webflux:          # Usa WebFlux (reactivo)
          routes:         # Array de rutas
            - id: httpbin-get               # Identificador único
              uri: https://httpbin.org      # Destino final
              predicates:                   # Condiciones para activar la ruta
                - Path=/api/**              # Intercepta /api/**
              filters:                      # Transformaciones
                - AddRequestHeader=X-Request-Source, gateway
                - RewritePath=/api/(?<segment>.*), /${segment}

management:                         # Configuración de Actuator
  endpoints:
    web:
      exposure:
        include: health,gateway     # Endpoints expuestos para monitoreo
```

### Componentes clave

#### 🆔 **Route ID (Identificador único)**

El `id` en cada ruta puede ser **cualquier nombre** que elijas, pero es recomendable seguir buenas prácticas:

##### ✅ **Flexibilidad total**
```yaml
routes:
  - id: cualquier-nombre-aqui
  - id: mi_ruta_123  
  - id: RUTA-EN-MAYUSCULAS
  - id: ruta.con.puntos
  - id: ruta-con-números-2024
```

##### 🎯 **Buenas prácticas recomendadas**

**Descriptivo y funcional:**
```yaml
- id: users-service-get          # Describe el servicio y operación
- id: orders-api-crud            # Indica el dominio y tipo
- id: payment-gateway-webhook    # Específico al propósito
- id: auth-service-login         # Claro y conciso
```

**Convenciones comunes:**
```yaml
# kebab-case (recomendado)
- id: user-management-api
- id: order-processing-service

# snake_case
- id: user_management_api
- id: order_processing_service

# camelCase
- id: userManagementApi
- id: orderProcessingService
```

**Patrones por contexto:**
```yaml
# Por servicio destino
- id: httpbin-get
- id: github-api-repos
- id: stripe-payments

# Por funcionalidad
- id: public-api
- id: admin-api
- id: webhook-handler

# Por versión
- id: api-v1-users
- id: api-v2-users
- id: legacy-orders
```

##### ⚠️ **Restricciones técnicas**

**Caracteres permitidos:**
- ✅ Letras, números, guiones, guiones bajos, puntos
- ✅ `user-api`, `api_v1`, `service.auth`, `route123`
- ❌ Espacios: `user api` (no funciona)
- ❌ Caracteres especiales: `user@api`, `api#1` (pueden causar problemas)

**Unicidad requerida:**
```yaml
routes:
  - id: duplicate-id    # ❌ Error: ID duplicado
    uri: https://api1.com
  - id: duplicate-id    # ❌ Esto sobrescribirá el anterior
    uri: https://api2.com
```

##### 🔍 **Uso del ID para debugging**

**En logs:**
```
2025-08-08 ... : Route matched: httpbin-get
2025-08-08 ... : Applying filters for route: httpbin-get
```

**En actuator:**
```bash
curl http://localhost:8080/actuator/gateway/routes
```
```json
[
  {
    "route_id": "httpbin-get",  // ← Tu ID aparece aquí
    "uri": "https://httpbin.org",
    "predicates": ["Path: /api/**"]
  }
]
```

**Para métricas:**
```bash
curl http://localhost:8080/actuator/metrics/gateway.requests?tag=routeId:httpbin-get
```

#### 🎯 **Predicates (Condiciones)**

Los **predicates** son las condiciones que determinan si una ruta debe activarse para una petición específica. Son como "filtros de entrada" que evalúan las características de la petición HTTP entrante.

##### ✅ **Path Predicate - Matching de rutas**

```yaml
predicates:
  - Path=/api/**              # Intercepta /api/**
```

**`Path=/api/**` explicado:**
- **`/api/`**: Prefijo obligatorio que debe coincidir
- **`**`**: Comodín que coincide con cualquier cantidad de caracteres y subdirectorios
- **Ant-style pattern**: Sigue la convención de patrones Ant de Spring

**Ejemplos que coinciden:**
- ✅ `http://localhost:8080/api/get`
- ✅ `http://localhost:8080/api/users/123`  
- ✅ `http://localhost:8080/api/orders/search/active`
- ✅ `http://localhost:8080/api/v1/auth/login`

**Ejemplos que NO coinciden:**
- ❌ `http://localhost:8080/health`
- ❌ `http://localhost:8080/users` 
- ❌ `http://localhost:8080/actuator/health`
- ❌ `http://localhost:8080/api` (sin la `/` final)

##### 🎯 **Otros Predicates disponibles**

**Por método HTTP:**
```yaml
predicates:
  - Path=/api/**
  - Method=GET,POST              # Solo GET y POST
```

**Por headers:**
```yaml
predicates:
  - Path=/api/**
  - Header=Authorization, Bearer.*    # Header con regex
  - Header=X-API-Version, v2         # Header con valor exacto
```

**Por parámetros de consulta:**
```yaml
predicates:
  - Path=/api/**
  - Query=version, 2             # Query param version=2
  - Query=debug                  # Query param debug (cualquier valor)
```

**Por host:**
```yaml
predicates:
  - Path=/api/**
  - Host=api.example.com         # Dominio específico
  - Host=*.example.com           # Subdominio con comodín
```

**Por cookies:**
```yaml
predicates:
  - Path=/api/**
  - Cookie=sessionId, [a-f0-9]+  # Cookie con regex
```

**Por tiempo:**
```yaml
predicates:
  - Path=/api/**
  - After=2025-01-01T00:00:00+00:00    # Después de una fecha
  - Before=2025-12-31T23:59:59+00:00   # Antes de una fecha
  - Between=2025-01-01T00:00:00+00:00,2025-12-31T23:59:59+00:00
```

**Por IP remota:**
```yaml
predicates:
  - Path=/api/**
  - RemoteAddr=192.168.1.0/24    # Rango de IPs CIDR
```

##### 🔗 **Combinando múltiples predicates (AND lógico)**

```yaml
routes:
  - id: secure-api
    uri: https://secure-api.example.com
    predicates:
      - Path=/secure/**                    # Y debe empezar con /secure/
      - Method=GET,POST                    # Y debe ser GET o POST
      - Header=Authorization, Bearer.*     # Y debe tener header Authorization con Bearer
      - Query=version, v2                  # Y debe tener query param version=v2
    filters:
      - StripPrefix=1
```

**Para que la ruta se active, TODOS los predicates deben cumplirse:**
- ✅ `POST /secure/data?version=v2` + `Authorization: Bearer xyz123`
- ❌ `POST /secure/data?version=v1` + `Authorization: Bearer xyz123` (version incorrecta)
- ❌ `DELETE /secure/data?version=v2` + `Authorization: Bearer xyz123` (método no permitido)

##### 🧪 **Ejemplos prácticos de Path patterns**

```yaml
routes:
  # Exacto
  - id: exact-match
    predicates:
      - Path=/api/health          # Solo /api/health

  # Comodín simple
  - id: single-wildcard
    predicates:
      - Path=/api/*              # /api/users pero NO /api/users/123

  # Comodín doble (recursivo)
  - id: recursive-wildcard
    predicates:
      - Path=/api/**             # /api/users Y /api/users/123/orders

  # Múltiples patrones
  - id: multiple-patterns
    predicates:
      - Path=/api/users/**,/api/orders/**  # Ambos patrones

  # Con variables de path
  - id: path-variables
    predicates:
      - Path=/api/users/{userId}/**    # Captura userId como variable
```

##### 🔍 **Testing y debugging de predicates**

**Ver qué rutas coinciden:**
```bash
curl http://localhost:8080/actuator/gateway/routes
```

**Logs detallados:**
```yaml
logging:
  level:
    org.springframework.cloud.gateway.handler.predicate: DEBUG
```

**En logs verás:**
```
2025-08-08 ... : Predicate PathRoutePredicateFactory matched: /api/get
2025-08-08 ... : Route matched: httpbin-get
```

**Probar predicates manualmente:**
```bash
# Debería funcionar (coincide con /api/**)
curl -i http://localhost:8080/api/test

# No debería funcionar (no coincide)
curl -i http://localhost:8080/other/test
```

##### ⚡ **Predicate personalizado (avanzado)**

```java
@Component
public class CustomPredicateFactory 
    extends AbstractRoutePredicateFactory<CustomPredicateFactory.Config> {
    
    @Override
    public Predicate<ServerHttpRequest> apply(Config config) {
        return request -> {
            // Tu lógica personalizada aquí
            return request.getHeaders().containsKey("X-Custom-Header");
        };
    }
}
```

Los **predicates son la puerta de entrada** a tu gateway - determinan qué tráfico entra por cada ruta. 🚪

#### 🔧 **Filters (Transformaciones)**

Los **filters** son las transformaciones que se aplican a las peticiones HTTP antes de enviarlas al servicio destino, o a las respuestas antes de devolverlas al cliente. Son como "middleware" que modifica el tráfico.

##### 🏷️ **AddRequestHeader - Agregar headers**

```yaml
filters:
  - AddRequestHeader=X-Request-Source, gateway
```

**`AddRequestHeader=X-Request-Source, gateway` explicado:**
- **Propósito**: Agrega un header HTTP a la petición saliente
- **Sintaxis**: `AddRequestHeader=nombre-header, valor`
- **Resultado**: Toda petición hacia el servicio destino incluirá `X-Request-Source: gateway`

**Ejemplo práctico:**
```
Petición original:  GET /api/get HTTP/1.1
                    Host: localhost:8080
                    User-Agent: curl/7.68.0

Petición transformada: GET /get HTTP/1.1
                       Host: httpbin.org
                       User-Agent: curl/7.68.0
                       X-Request-Source: gateway  ← Agregado por el filter
```

**Casos de uso:**
- ✅ **Identificación**: Saber que la petición pasó por el gateway
- ✅ **Autenticación**: Agregar tokens o credenciales
- ✅ **Tracking**: Headers para trazabilidad y monitoreo
- ✅ **Versionado**: Indicar versión del gateway o API

**Variaciones del AddRequestHeader:**
```yaml
filters:
  # Header simple
  - AddRequestHeader=X-Gateway-Version, 1.0
  
  # Header con timestamp
  - AddRequestHeader=X-Request-Time, #{T(System).currentTimeMillis()}
  
  # Header condicional usando SpEL
  - AddRequestHeader=X-User-Type, #{request.headers['Authorization'][0].contains('admin') ? 'admin' : 'user'}
  
  # Múltiples headers
  - AddRequestHeader=X-Gateway, spring-cloud
  - AddRequestHeader=X-Environment, production
```

##### 🔄 **RewritePath - Reescribir rutas**

```yaml
filters:
  - RewritePath=/api/(?<segment>.*), /${segment}
```

**`RewritePath=/api/(?<segment>.*), /${segment}` explicado:**

**Parte 1 - Patrón de captura: `/api/(?<segment>.*)`**
- **`/api/`**: Prefijo literal que debe coincidir
- **`(?<segment>.*)`**: Grupo de captura con nombre `segment`
  - **`?<segment>`**: Nombra el grupo como "segment"
  - **`.*`**: Captura cualquier carácter, cualquier cantidad de veces

**Parte 2 - Reemplazo: `/${segment}`**
- **`/`**: Prefijo de la nueva ruta
- **`${segment}`**: Referencia al contenido capturado en el grupo `segment`

**Ejemplos de transformación:**

| URL Original | Patrón capturado | Resultado Final |
|--------------|------------------|-----------------|
| `/api/get` | `segment = "get"` | `/get` |
| `/api/users/123` | `segment = "users/123"` | `/users/123` |
| `/api/v1/orders/search` | `segment = "v1/orders/search"` | `/v1/orders/search` |
| `/api/auth/login?redirect=/dashboard` | `segment = "auth/login?redirect=/dashboard"` | `/auth/login?redirect=/dashboard` |

**Flujo completo:**
```
1. Cliente solicita:     http://localhost:8080/api/users/123
2. Gateway captura:      segment = "users/123"  
3. Gateway reescribe:    /users/123
4. Envía al destino:     https://httpbin.org/users/123
```

**Otros patrones de RewritePath:**
```yaml
filters:
  # Remover prefijo simple
  - RewritePath=/old-api/(?<segment>.*), /new-api/${segment}
  
  # Agregar versión
  - RewritePath=/api/(?<segment>.*), /v1/api/${segment}
  
  # Reemplazar completamente
  - RewritePath=/legacy/(?<path>.*), /modern/services/${path}
  
  # Capturar múltiples segmentos
  - RewritePath=/api/(?<version>v[0-9]+)/(?<resource>.*), /${version}/${resource}
```

##### 🛠️ **Otros Filters comunes**

**Manipulación de headers:**
```yaml
filters:
  # Request headers
  - AddRequestHeader=Authorization, Bearer token123
  - SetRequestHeader=Content-Type, application/json    # Sobrescribe si existe
  - RemoveRequestHeader=X-Internal-Header              # Remueve header
  
  # Response headers  
  - AddResponseHeader=X-Gateway-Response, processed
  - SetResponseHeader=Cache-Control, no-cache
  - RemoveResponseHeader=Server                        # Oculta información del servidor
```

**Manipulación de paths:**
```yaml
filters:
  - StripPrefix=1                    # Remueve 1 segmento: /api/users → /users
  - StripPrefix=2                    # Remueve 2 segmentos: /v1/api/users → /users
  - PrefixPath=/legacy              # Agrega prefijo: /users → /legacy/users
```

**Redirecciones y status:**
```yaml
filters:
  - RedirectTo=302, https://new-domain.com/api        # Redirige temporalmente
  - SetStatus=200                                     # Fuerza status code
  - SetStatus=404                                     # Simula not found
```

**Rate limiting y circuit breaker:**
```yaml
filters:
  - name: RequestRateLimiter
    args:
      rate-limiter: "#{@myRateLimiter}"
      key-resolver: "#{@userKeyResolver}"
  
  - name: CircuitBreaker
    args:
      name: myCircuitBreaker
      fallbackUri: forward:/fallback
```

**Request/Response modification:**
```yaml
filters:
  - name: ModifyRequestBody
    args:
      modify-request-body-class: java.lang.String
      modify-request-body-function: "#{@requestBodyModifier}"
  
  - name: ModifyResponseBody  
    args:
      modify-response-body-class: java.lang.String
      modify-response-body-function: "#{@responseBodyModifier}"
```

##### ⚡ **Orden de ejecución de filters**

Los filters se ejecutan en el orden que aparecen en la configuración:

```yaml
filters:
  - AddRequestHeader=Step, 1        # Se ejecuta PRIMERO
  - RewritePath=/api/(?<segment>.*), /${segment}  # Se ejecuta SEGUNDO  
  - AddRequestHeader=Step, 2        # Se ejecuta TERCERO
```

**Para casos complejos, puedes especificar el orden:**
```yaml
filters:
  - name: AddRequestHeader
    args:
      name: X-Step
      value: first
    order: 1
      
  - name: RewritePath  
    args:
      regexp: /api/(?<segment>.*)
      replacement: /${segment}
    order: 2
```

##### 🧪 **Testing y debugging de filters**

**Ver headers agregados:**
```bash
curl -i http://localhost:8080/api/headers
```

**Respuesta esperada:**
```json
{
  "headers": {
    "X-Request-Source": "gateway",    // ← Header agregado por AddRequestHeader
    "Host": "httpbin.org",           // ← Host cambiado por el proxy
    // ... otros headers
  }
}
```

**Ver transformación de path:**
```bash
curl -i http://localhost:8080/api/get
# La URL final en httpbin será: https://httpbin.org/get (sin /api/)
```

**Logs detallados:**
```yaml
logging:
  level:
    org.springframework.cloud.gateway.filter: DEBUG
```

**En logs verás:**
```
2025-08-08 ... : Applying filter: AddRequestHeaderGatewayFilterFactory
2025-08-08 ... : Applying filter: RewritePathGatewayFilterFactory  
2025-08-08 ... : Rewritten path from /api/get to /get
```

##### 🎯 **Ejemplo completo con múltiples filters**

```yaml
routes:
  - id: complete-example
    uri: https://api.backend.com
    predicates:
      - Path=/public/api/**
      - Method=GET,POST
    filters:
      # 1. Autenticación y headers de seguridad
      - AddRequestHeader=X-Gateway-Auth, gateway-token-123
      - AddRequestHeader=X-Forwarded-Proto, https
      
      # 2. Transformación de path
      - RewritePath=/public/api/(?<version>v[0-9]+)/(?<resource>.*), /api/${version}/${resource}
      
      # 3. Rate limiting
      - name: RequestRateLimiter
        args:
          rate-limiter: "#{@redisRateLimiter}"
          
      # 4. Headers de respuesta
      - AddResponseHeader=X-Processed-By, spring-cloud-gateway
      - AddResponseHeader=X-Response-Time, "#{T(System).currentTimeMillis()}"
      
      # 5. Remover headers internos en respuesta
      - RemoveResponseHeader=X-Internal-Token
```

Los **filters son el corazón de la transformación** en tu gateway - modifican tanto peticiones como respuestas según tus necesidades. 🔧✨

### 📊 **Management/Actuator**
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,gateway
```

Expone endpoints específicos para monitoreo y debugging del gateway:
- **`health`**: Estado de salud de la aplicación
- **`gateway`**: Información específica del gateway (rutas, filtros, métricas)

## 🔄 Flujo de una petición

1. **Cliente envía**: `GET http://localhost:8080/api/get`
2. **Gateway evalúa**: `Path=/api/**` → ✅ Coincide con la ruta `httpbin-get`
3. **Aplica filtros**:
   - Agrega header: `X-Request-Source: gateway`
   - Reescribe path: `/api/get` → `/get`
4. **Envía a destino**: `GET https://httpbin.org/get` con header adicional
5. **Respuesta regresa**: httpbin.org responde y el gateway reenvía al cliente

## 📁 Estructura del proyecto

```
spring-cloud-gateway/
├── README.md
├── pom.xml
├── mvnw / mvnw.cmd
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── io/cyb3rh4ck/github/spring_cloud_gateway/
│   │   │       └── SpringCloudGatewayApplication.java
│   │   └── resources/
│   │       └── application.yml
│   └── test/
│       └── java/
│           └── io/cyb3rh4ck/github/spring_cloud_gateway/
│               └── SpringCloudGatewayApplicationTests.java
├── target/
└── .vscode/
    └── launch.json
```

## 🛠️ Dependencias principales

```xml
<dependencies>
    <!-- Spring Cloud Gateway (WebFlux/Reactivo) -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>
    
    <!-- Actuator para monitoreo -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

## 🧩 Casos de uso comunes

### 1. **API Gateway empresarial**
```yaml
routes:
  - id: user-service
    uri: lb://user-service
    predicates:
      - Path=/users/**
    filters:
      - StripPrefix=1
      - AddRequestHeader=X-Service, users
```

### 2. **Rate Limiting**
```yaml
filters:
  - name: RequestRateLimiter
    args:
      redis-rate-limiter.replenishRate: 10
      redis-rate-limiter.burstCapacity: 20
```

### 3. **Circuit Breaker**
```yaml
filters:
  - name: CircuitBreaker
    args:
      name: myCircuitBreaker
      fallbackUri: forward:/fallback
```

### 4. **Load Balancing con Service Discovery**
```yaml
- id: load-balanced-service
  uri: lb://my-service
  predicates:
    - Path=/service/**
```

## 🔍 Debugging

### Ver logs detallados
```yaml
logging:
  level:
    org.springframework.cloud.gateway: DEBUG
    reactor.netty: DEBUG
```

### Verificar rutas activas
```bash
curl http://localhost:8080/actuator/gateway/routes | jq
```

### Verificar métricas
```bash
curl http://localhost:8080/actuator/metrics/gateway.requests
```

## 🚨 Solución de problemas comunes

### 1. **Error 404 en /api/***
- Verificar que la aplicación esté ejecutándose en puerto 8080
- Verificar sintaxis del predicate `Path=/api/**`
- Revisar logs para errores de configuración

### 2. **Gateway no agrega headers**
- Verificar sintaxis: `AddRequestHeader=nombre, valor`
- Probar con `/api/headers` para ver headers enviados

### 3. **Reescritura de path no funciona**
- Verificar regex: `/api/(?<segment>.*)`
- Probar la regex en herramientas online
- Revisar logs de Spring Cloud Gateway

### 4. **Aplicación no inicia**
- Verificar que puerto 8080 esté libre: `netstat -an | findstr 8080`
- Verificar dependencias de Maven: `mvn dependency:tree`
- Revisar versiones de Spring Boot y Spring Cloud

## 📚 Referencias

- [Spring Cloud Gateway Docs](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/)
- [HTTPBin.org](https://httpbin.org/) - Servicio de prueba HTTP
- [Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)

## 🎯 Próximos pasos

1. **Agregar autenticación**: JWT, OAuth2
2. **Implementar rate limiting**: Redis-based rate limiter
3. **Agregar circuit breaker**: Resilience4j integration
4. **Service Discovery**: Eureka, Consul
5. **Observabilidad**: Distributed tracing, metrics
6. **Seguridad**: CORS, CSRF protection

---

**Autor**: GitHub Portfolio Labs  
**Tecnologías**: Spring Boot 3.5.4, Spring Cloud 2025.0.0, Java