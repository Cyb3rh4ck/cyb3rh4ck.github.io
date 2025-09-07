---
title: "Arquitectura de Autenticación con Token Opaco, Gateway y JWT"
date: 2025-08-08
draft: false
tags: ["ChatGPT", "Negocios", "Innovación", "Tecnología", "IA"]
categories: ["Inteligencia Artificial", "Estrategia Empresarial"]
---

# Arquitectura de Autenticación con Token Opaco, Gateway y JWT

¡Va! Te lo explico como arquitecto (AWS + Java) y te dejo **diagramas** + **tips prácticos** + **fuentes oficiales** para cada punto clave.

# Arquitectura (visión general)

* **access-gateway** (Spring Cloud Gateway): entrada única; lee la cookie de sesión, valida con *identity-service* y reinyecta un **JWT** en `Authorization: Bearer ...` hacia los microservicios. ([Home][1])
* **identity-service** (Spring Boot): valida **token opaco** con el *Authorization Server* (introspección), crea **sesión** en **ElastiCache Redis**, firma un **JWT** y setea **cookie** segura. ([IETF Datatracker][2])
* **utils-service / fraud-ins-service** (Spring Boot 3.5.x): **stateless**; validan localmente el JWT contra el **JWKS** de *identity-service* (cacheado). ([Home][3], [IETF Datatracker][4])

# Diagrama 1 — “Handoff”: login vía token opaco (desde app)

<div class="mermaid" style="text-align: center;">
sequenceDiagram
    autonumber
    participant App as App origen (ya logueada)
    participant Browser as Navegador
    participant GW as access-gateway
    participant ID as identity-service
    participant AS as Authorization Server (token opaco)
    participant Redis as ElastiCache Redis

    App->>AS: (2.1) Solicita token opaco (con datos del usuario)
    AS-->>App: (2.2) token_opaco
    App->>Browser: (2.3) Redirige a nueva app: https://newapp/...?token=token_opaco
    Browser->>GW: GET /?token=token_opaco
    GW->>ID: (3) Enruta a /exchange?token=...
    ID->>AS: (4.1) Introspección RFC 7662 (valida token opaco)
    AS-->>ID: Respuesta activa + claims del usuario
    ID->>Redis: (4.3) Crea sesión (SESSION_ID -> atributos)
    ID-->>Browser: (4.4) Set-Cookie: __Host-SESSION_ID=...; HttpOnly; Secure; SameSite=...
    Note right of ID: (4.3) También firma un JWT para uso interno
    Browser->>GW: (5) Peticiones subsecuentes (cookie incluida)
</div>

**Notas clave**

* La validación de tokens **opacos** se hace con **token introspection** (RFC 7662). ([IETF Datatracker][2], [RFC Editor][5])
* Evita que el `token` viaje mucho tiempo en la URL (logs, referers, history). Si lo usas, que sea **de un solo uso y cortísima vida**, y haz **redirect** inmediato a una URL limpia. (Buenas prácticas OWASP sobre *query strings* sensibles). ([OWASP Foundation][6])
* Para cookies seguras: `Secure`, `HttpOnly`, `SameSite` adecuado y **prefijo `__Host-`** (defensa contra session fixation). Si necesitas **cross-site**, usa `SameSite=None; Secure`. ([MDN Web Docs][7])

# Diagrama 2 — Peticiones subsecuentes (propagación de identidad)

<div class="mermaid" style="text-align: center;">
sequenceDiagram
    autonumber
    participant Browser as Navegador
    participant GW as access-gateway
    participant ID as identity-service
    participant Biz as Microservicio de negocio
    participant Redis as ElastiCache Redis

    Browser->>GW: (5.1) GET /biz/endpoint (Cookie: __Host-SESSION_ID=...)
    GW->>ID: (5.1.2) /validate-session (SESSION_ID)
    ID->>Redis: (5.1.3) Consulta sesión
    Redis-->>ID: OK + info usuario
    ID-->>GW: (5.1.3) Responde con JWT firmado (roles, permisos, etc.)
    GW->>GW: (5.1.4) Inserta header Authorization: Bearer <JWT>
    GW->>Biz: (5.1.5) Enruta con contexto de autenticación
    Biz->>Biz: Valida firma y claims del JWT (JWKs cacheado)
</div>

* El **gateway** puede implementar un **GlobalFilter** para leer la cookie, llamar a `/validate-session` y **añadir** `Authorization: Bearer <jwt>` antes de rutear. ([Home][1], [Baeldung on Kotlin][8])
* Los **microservicios** validan **localmente** el JWT (firma, `exp`, `iat`, `iss`, `aud`, `scope/roles`) apoyándose en **Spring Security Resource Server** + **JWKS** (cacheado). ([Home][3])

# Implementación (puntos finos)

## 1) Cookie de sesión (HTTP-only, Secure, SameSite)

Ejemplo de cabecera **correcta** (ojo: es `HttpOnly`, no “HTTPonly”, y conviene el prefijo `__Host-`):

```
Set-Cookie: __Host-SESSION_ID=<id>; Path=/; Secure; HttpOnly; SameSite=Lax
```

* `Strict` es muy restrictivo (rompe flujos tipo link-in), `Lax` suele ser el sweet spot; para **cross-site** usa `SameSite=None; Secure`. ([MDN Web Docs][9])
* **Prefijos** `__Host-`/`__Secure-` endurecen el setting de la cookie a nivel de navegador. ([MDN Web Docs][9])

## 2) access-gateway (Spring Cloud Gateway)

**YAML** (ejemplo simple de ruta y filtro de cabecera):

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: biz
          uri: http://biz-service
          predicates:
            - Path=/biz/**
          filters:
            - AddRequestHeader=X-Trace-Id, "#{T(java.util.UUID).randomUUID()}"
```

([Home][10])

**GlobalFilter** (pseudo-código) para validar sesión y añadir `Authorization`:

```java
@Bean
public GlobalFilter sessionToJwtFilter(WebClient.Builder webClient) {
  return (exchange, chain) -> {
    var sessionId = extractCookie(exchange.getRequest(), "__Host-SESSION_ID");
    if (sessionId == null) return chain.filter(exchange);

    return webClient.build()
      .get()
      .uri("http://identity-service/validate-session")
      .header("Cookie", "__Host-SESSION_ID=" + sessionId)
      .retrieve()
      .bodyToMono(String.class) // aquí recibes el JWT
      .defaultIfEmpty("") 
      .flatMap(jwt -> {
        if (!jwt.isEmpty()) {
          var mutated = exchange.mutate()
            .request(r -> r.headers(h -> h.setBearerAuth(jwt)))
            .build();
          return chain.filter(mutated);
        } else {
          return chain.filter(exchange);
        }
      });
  };
}
```

Referencias sobre **Global Filters** y filtros personalizados: ([Home][1], [Home][11], [Baeldung on Kotlin][8])

## 3) identity-service

* **Introspección** de token opaco (RFC 7662) contra el Authorization Server (`/introspect`) para confirmar que está **activo** y leer *claims*. ([IETF Datatracker][2])
* **Sesión** en **ElastiCache Redis** (TTL acorde a riesgo; usa Spring Session Redis). ([Home][12], [AWS Documentation][13])
* **JWT** interno firmado (por ejemplo con RSA/ECDSA). Expón **JWKS** en `GET /.well-known/jwks.json` para validación descentralizada. (JWK/JWKS en RFC 7517; la ruta de *well-known* es la práctica estandarizada por OIDC Discovery). ([IETF Datatracker][4], [OpenID Foundation][14])

**Spring Session Redis** (pom y propiedades) — guía oficial: ([Home][12])
**ElastiCache prácticas** (tamaño/escala/cluster-mode y TLS en tránsito): ([AWS Documentation][15], [Amazon Web Services, Inc.][16])

## 4) Microservicios de negocio (validación **stateless** del JWT)

Config mínima con **Spring Security Resource Server** (valida firma con tu JWKS):

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          jwk-set-uri: https://identity-service/.well-known/jwks.json
```

Spring Security **auto-configura** la validación de **Bearer JWT** y **cachea** el JWKS; puedes ajustar caché/timeout si lo necesitas. ([Home][3])

**Sobre el caché del JWKS** (Nimbus): por defecto se cachea y se refresca si llega un `kid` desconocido; puedes tunear TTL (ej. 5–15 min). ([javadoc.io][17])

**Verificaciones recomendadas** en tu filtro/interceptor o configuración:

* Firma (`alg`, `kid`) con clave pública correcto.
* `exp`, `iat`, opcional `nbf`; `iss`/`aud` esperado; scopes/roles. ([Home][3])

## 5) Seguridad del “token en query param”

Tu flujo lo usa solo **una vez** (handoff). Aun así, **mitiga**:

* Vida muy corta (**segundos**), **un solo uso**, y **redirect** inmediato a URL limpia.
* `Cache-Control: no-store` en respuestas intermedias.
* Si puedes, usa **código de intercambio** en lugar de token directamente (patrones de OIDC/BCP).
  Motivo: la URL puede filtrarse por logs, histórico, *Referer*, etc. ([OWASP Foundation][6])

## 6) Encabezado Authorization: Bearer

La forma estándar de portar tokens es **Authorization: Bearer <token>** (RFC 6750). ([IETF Datatracker][18], [Swagger][19])

# AWS (piezas y tips)

* **ElastiCache for Redis** como *session store* (cluster-mode enabled para escalar; definir TTL de sesión). ([AWS Documentation][15])
* Habilita **TLS** en tránsito y cifrado en reposo del clúster. (Buenas prácticas generales de ElastiCache). ([AWS Documentation][13])

# Resumen del flujo que pediste (con ajustes recomendados)

1. **Usuario** ya autenticado en la app origen.
2. La app origen obtiene **token opaco** (RFC 7662), redirige a la nueva app con `?token=...` (**one-shot, TTL corto; redirect a URL limpia**). ([IETF Datatracker][2], [OWASP Foundation][6])
3. **access-gateway** detecta el `token` y enruta a **identity-service** (endpoint interno de intercambio). ([Home][1])
4. **identity-service**:

   * **Introspección** del token opaco.
   * Recupera datos de usuario.
   * Crea **sesión** en **Redis**, firma un **JWT**, y setea **cookie** `__Host-SESSION_ID` (`Secure; HttpOnly; SameSite=` adecuado). ([IETF Datatracker][2], [Home][12], [MDN Web Docs][9])
5. **Peticiones subsecuentes**:

   * **Gateway** lee cookie, llama a `/validate-session` en **identity-service**, recibe **JWT** y lo **inyecta en `Authorization: Bearer`**. ([Home][1])
   * Los **microservicios** validan **localmente** el JWT (JWKS) y quedan **stateless**. ([Home][3])

---

# Enlaces/Fuentes

* **JWT / JWK / JWKS**: RFC 7517 (JWK); JWKS autoconsumo vía OIDC Discovery (`/.well-known/openid-configuration` y `jwks_uri`). ([IETF Datatracker][4], [OpenID Foundation][14], [Auth0][20])
* **Bearer Tokens** (header `Authorization`): RFC 6750. ([IETF Datatracker][18], [RFC Editor][21])
* **Token opaco / Introspección**: RFC 7662. ([IETF Datatracker][2])
* **Spring Security Resource Server (JWT + JWK)**: docs oficiales. ([Home][3])
* **Spring Cloud Gateway (GlobalFilter / mutate headers)**: docs + ejemplos. ([Home][1], [Home][11], [Baeldung on Kotlin][8])
* **Cookies seguras** (`Secure`, `HttpOnly`, `SameSite`, prefijos `__Host-`/`__Secure-`)—MDN: guías y `Set-Cookie`. ([MDN Web Docs][9])
* **Riesgos de poner tokens en URL** (OWASP). ([OWASP Foundation][6])
* **ElastiCache Redis** (mejores prácticas; Spring Session Redis). ([AWS Documentation][15], [Home][12])

---

¿Quieres que te deje un **esqueleto de proyecto** (gateway + identity + dos servicios) con estas piezas (Boot 3.5.x, Java 17) y *Mermaid* en el README? Te lo preparo y lo afinamos a tu naming de Seguros360.

[1]: https://docs.spring.io/spring-cloud-gateway/reference/spring-cloud-gateway/global-filters.html?utm_source=chatgpt.com "Global Filters :: Spring Cloud Gateway"
[2]: https://datatracker.ietf.org/doc/html/rfc7662?utm_source=chatgpt.com "RFC 7662 - OAuth 2.0 Token Introspection"
[3]: https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/jwt.html?utm_source=chatgpt.com "OAuth 2.0 Resource Server JWT"
[4]: https://datatracker.ietf.org/doc/html/rfc7517?utm_source=chatgpt.com "RFC 7517 - JSON Web Key (JWK)"
[5]: https://www.rfc-editor.org/info/rfc7662?utm_source=chatgpt.com "Information on RFC 7662"
[6]: https://owasp.org/www-community/vulnerabilities/Information_exposure_through_query_strings_in_url?utm_source=chatgpt.com "Information exposure through query strings in url"
[7]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie?utm_source=chatgpt.com "Set-Cookie header - HTTP - MDN Web Docs"
[8]: https://www.baeldung.com/spring-cloud-custom-gateway-filters?utm_source=chatgpt.com "Writing Custom Spring Cloud Gateway Filters"
[9]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies?utm_source=chatgpt.com "Using HTTP cookies - MDN Web Docs - Mozilla"
[10]: https://docs.spring.io/spring-cloud-gateway/reference/spring-cloud-gateway/gatewayfilter-factories/addrequestheader-factory.html?utm_source=chatgpt.com "AddRequestHeader GatewayFilter Factory"
[11]: https://spring.io/blog/2022/08/26/creating-a-custom-spring-cloud-gateway-filter/?utm_source=chatgpt.com "Creating a custom Spring Cloud Gateway Filter"
[12]: https://docs.spring.io/spring-session/reference/guides/boot-redis.html?utm_source=chatgpt.com "Spring Session - Spring Boot"
[13]: https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/BestPractices.html?utm_source=chatgpt.com "ElastiCache best practices and caching strategies"
[14]: https://openid.net/specs/openid-connect-discovery-1_0.html?utm_source=chatgpt.com "OpenID Connect Discovery 1.0 incorporating errata set 2"
[15]: https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/WorkingWithRedis.html?utm_source=chatgpt.com "Overall best practices - Amazon ElastiCache"
[16]: https://aws.amazon.com/blogs/database/best-practices-for-sizing-your-amazon-elasticache-for-redis-clusters/?utm_source=chatgpt.com "Best practices for sizing your Amazon ElastiCache ..."
[17]: https://www.javadoc.io/doc/com.nimbusds/nimbus-jose-jwt/latest/com/nimbusds/jose/jwk/source/RemoteJWKSet.html?utm_source=chatgpt.com "RemoteJWKSet - nimbus-jose-jwt 10.4 javadoc"
[18]: https://datatracker.ietf.org/doc/html/rfc6750?utm_source=chatgpt.com "RFC 6750 - The OAuth 2.0 Authorization Framework"
[19]: https://swagger.io/docs/specification/v3_0/authentication/bearer-authentication/?utm_source=chatgpt.com "Bearer Authentication | Swagger Docs"
[20]: https://auth0.com/docs/get-started/applications/configure-applications-with-oidc-discovery?utm_source=chatgpt.com "Configure Applications with OIDC Discovery"
[21]: https://www.rfc-editor.org/info/rfc6750?utm_source=chatgpt.com "Information on RFC 6750"






Este documento describe un flujo de autenticación basado en **token opaco** para intercambio inicial, **Spring Cloud Gateway** como punto de entrada, **JWT** para propagación de identidad y **Redis** como store de sesión.

## 🏗 Diagrama General

<div class="mermaid" style="text-align: center;">
graph TD
sequenceDiagram
    autonumber
    participant App as App origen (ya logueada)
    participant Browser as Navegador
    participant GW as access-gateway
    participant ID as identity-service
    participant AS as Authorization Server (token opaco)
    participant Redis as ElastiCache Redis

    App->>AS: (2.1) Solicita token opaco (con datos del usuario)
    AS-->>App: (2.2) token_opaco
    App->>Browser: (2.3) Redirige a nueva app: https://newapp/...?token=token_opaco
    Browser->>GW: GET /?token=token_opaco
    GW->>ID: (3) Enruta a /exchange?token=...
    ID->>AS: (4.1) Introspección RFC 7662 (valida token opaco)
    AS-->>ID: Respuesta activa + claims del usuario
    ID->>Redis: (4.3) Crea sesión (SESSION_ID -> atributos)
    ID-->>Browser: (4.4) Set-Cookie: __Host-SESSION_ID=...; HttpOnly; Secure; SameSite=...
    Note right of ID: (4.3) También firma un JWT para uso interno
    Browser->>GW: (5) Peticiones subsecuentes (cookie incluida)
</div>

## ⚙ Flujo de Peticiones Subsecuentes

<div class="mermaid" style="text-align: center;">
graph TD
sequenceDiagram
    autonumber
    participant Browser as Navegador
    participant GW as access-gateway
    participant ID as identity-service
    participant Biz as Microservicio de negocio
    participant Redis as ElastiCache Redis

    Browser->>GW: (5.1) GET /biz/endpoint (Cookie: __Host-SESSION_ID=...)
    GW->>ID: (5.1.2) /validate-session (SESSION_ID)
    ID->>Redis: (5.1.3) Consulta sesión
    Redis-->>ID: OK + info usuario
    ID-->>GW: (5.1.3) Responde con JWT firmado (roles, permisos, etc.)
    GW->>GW: (5.1.4) Inserta header Authorization: Bearer <JWT>
    GW->>Biz: (5.1.5) Enruta con contexto de autenticación
    Biz->>Biz: Valida firma y claims del JWT (JWKs cacheado)
</div>

---

## 📂 Estructura del Proyecto

```
seguros360-auth-arch/
├── access-gateway/
│   ├── src/main/java/com/seguros360/gateway/GatewayApplication.java
│   ├── src/main/resources/application.yml
│   └── pom.xml
├── identity-service/
│   ├── src/main/java/com/seguros360/identity/IdentityApplication.java
│   ├── src/main/resources/application.yml
│   └── pom.xml
├── utils-service/
│   ├── src/main/java/com/seguros360/utils/UtilsApplication.java
│   ├── src/main/resources/application.yml
│   └── pom.xml
├── fraud-ins-service/
│   ├── src/main/java/com/seguros360/fraud/FraudApplication.java
│   ├── src/main/resources/application.yml
│   └── pom.xml
└── pom.xml (padre)
```

---

## 📌 Dependencias Comunes (Spring Boot 3.5.x, Java 17)

```xml
<properties>
    <java.version>17</java.version>
    <spring.boot.version>3.5.0</spring.boot.version>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>${spring.boot.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

---

## 🛠 access-gateway (Spring Cloud Gateway)

**application.yml** ejemplo:

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: utils
          uri: http://utils-service
          predicates:
            - Path=/utils/**
        - id: fraud
          uri: http://fraud-ins-service
          predicates:
            - Path=/fraud/**
```

---

## 🔑 identity-service

- Valida token opaco con introspección (RFC 7662)
- Crea sesión en Redis (Spring Session Redis)
- Firma JWT (RSA/ECDSA) y expone JWKS

**application.yml** ejemplo:

```yaml
spring:
  redis:
    host: localhost
    port: 6379
  session:
    store-type: redis
```

---

## 📜 utils-service & fraud-ins-service

Config para validar JWT vía JWKS:

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          jwk-set-uri: http://identity-service/.well-known/jwks.json
```

---

## 🔗 Referencias

- [Spring Cloud Gateway](https://spring.io/projects/spring-cloud-gateway)
- [Spring Security Resource Server](https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/jwt.html)
- [RFC 7662 - OAuth 2.0 Token Introspection](https://www.rfc-editor.org/rfc/rfc7662)
- [RFC 6750 - Bearer Tokens](https://www.rfc-editor.org/rfc/rfc6750)
- [MDN - Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)



<script src="https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.min.js"></script>
<script>
  mermaid.initialize({ startOnLoad: true });
</script>
