---
title: "☕ Uso de Cache Local con Caffeine"
date: 2025-07-24
draft: false
description: "Cómo usar Caffeine para caching local en microservicios con Spring Boot y el framework Darwin."
tags: ["caching", "caffeine", "spring-boot", "performance"]
---

# ☕ Caffeine Cache - Guía Rápida

**Caffeine** es una librería Java de alto rendimiento para caching en memoria local. Es una alternativa moderna a Ehcache y Guava, recomendada para microservicios que requieren acceso rápido a datos sin necesidad de sincronización entre instancias.

---

## ✅ ¿Cuándo usar Caffeine?

- Datos que no cambian frecuentemente (ej. catálogos, configuraciones)
- Objetos pequeños a consultar muchas veces
- Cuando no necesitas compartir cache entre instancias

---

## ⚙️ Configuración en Spring Boot

1. **Agrega la dependencia en tu `pom.xml`:**

```xml
<dependency>
  <groupId>com.github.ben-manes.caffeine</groupId>
  <artifactId>caffeine</artifactId>
</dependency>
```

2. **Habilita caching en tu aplicación:**

```java
@SpringBootApplication
@EnableCaching
public class MyApp { ... }
```

3. **Usa la anotación `@Cacheable`:**

```java
@Cacheable("clientes")
public Cliente getClienteById(String id) {
    return clienteRepository.findById(id);
}
```

---

## 🧠 Políticas de Evicción

Puedes configurar políticas como:

```java
@Bean
public CacheManager cacheManager() {
    CaffeineCacheManager manager = new CaffeineCacheManager("clientes", "productos");
    manager.setCaffeine(Caffeine.newBuilder()
        .expireAfterWrite(10, TimeUnit.MINUTES)
        .maximumSize(500));
    return manager;
}
```

| Política               | Descripción                                      |
|------------------------|--------------------------------------------------|
| `expireAfterWrite`     | Expira X tiempo después de escribir              |
| `expireAfterAccess`    | Expira X tiempo después del último acceso        |
| `maximumSize`          | Máximo de elementos en cache                     |
| `refreshAfterWrite`    | Refresca entrada tras cierto tiempo              |

---

## 🧪 Verificando Caching

- Verifica con logs si el método anotado con `@Cacheable` se ejecuta o se evita.
- Usa endpoints de Actuator (`/actuator/caches`) para inspeccionar el cache.

---

## 🚫 Precauciones

- No usar para datos críticos o compartidos entre servicios.
- Definir bien los tiempos de vida (`TTL`) para evitar inconsistencias.
- Cuidado con almacenar grandes volúmenes que saturen la memoria.

---

## ✅ Conclusión

Caffeine mejora el rendimiento al reducir llamadas repetidas a base de datos o servicios. Es ideal como **cache local** en microservicios Spring y puede integrarse fácilmente con el **framework Darwin** para cubrir escenarios comunes.

