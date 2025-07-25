---
title: "🔐 OAuth 2.0: Resumen Profesional (RFC 6749)"
date: 2025-07-24
draft: false
description: "Resumen técnico visual y profesional del estándar OAuth 2.0 (RFC 6749) enfocado en aprendizaje visual y arquitectura de seguridad moderna."
tags: ["oauth2", "seguridad", "autenticación", "api", "diagramas", "visual"]
---

# 🔐 OAuth 2.0 - Resumen Visual (RFC 6749)

OAuth 2.0 es un marco de autorización que permite a aplicaciones de terceros acceder de forma limitada y segura a recursos protegidos, **sin compartir las credenciales del usuario**.

---

## 🧩 Actores del Protocolo

| Actor               | Descripción                                                         |
|--------------------|----------------------------------------------------------------------|
| 👤 **Resource Owner**     | Usuario que posee los recursos                                 |
| 🌐 **Client**             | App que quiere acceder a recursos                              |
| 🔐 **Authorization Server** | Autentica usuarios y entrega tokens                          |
| 📦 **Resource Server**     | API que expone los recursos protegidos                       |

---

## 🔁 Flujo General (Authorization Code)

```text
[Usuario] --> (Autorización) --> [Authorization Server]
    ↓                                   ↓
[Client App] <-- Código de Autorización -- [Authorization Server]
    ↓
Token Exchange
    ↓
[Client App] --> Token de Acceso --> [Resource Server]
```

---

## 🧾 Tipos de "Grants" (Concesiones)

| Grant Type              | Uso común                           | Seguridad  |
|-------------------------|-------------------------------------|------------|
| 🔐 Authorization Code   | Apps Web / Móviles                  | ⭐⭐⭐⭐⭐      |
| 💳 Client Credentials    | Sistemas máquina a máquina          | ⭐⭐⭐⭐       |
| ⚠️ Implicit             | SPAs (obsoleto)                     | ❌          |
| 🔑 Password Credentials | Legacy / Testing                    | ❌          |

---

## 🎫 Tokens

- **Access Token**: Caduca rápido, limitado en alcance.
- **Refresh Token**: Opcional. Renueva el Access Token sin reautenticación.
- **Formatos**: Generalmente JWT o token opaco.

---

## 🔐 Seguridad Requerida

- Uso obligatorio de **HTTPS (TLS)**
- Tokens deben tratarse como **credenciales sensibles**
- Revocación y expiración controladas desde el servidor

---

## 🧱 Arquitectura con OAuth 2.0

```text
     +-------------------------+
     |     Resource Owner      |
     +-------------------------+
                 |
      (autorización vía UI)
                 ↓
     +-------------------------+
     |  Authorization Server   |----+
     +-------------------------+    |
                 ↓                 ↓
     Código     Token            API Call
     ↓           ↓                ↓
+---------+  +-----------+    +------------+
| Client  |->| Access    |--->| Resource   |
| (App)   |  | Token     |    | Server     |
+---------+  +-----------+    +------------+
```

---

## 📚 Extensiones Comunes

| RFC       | Nombre                       | Propósito                           |
|-----------|------------------------------|-------------------------------------|
| RFC 6750  | Bearer Token                 | Define uso de tokens tipo Bearer    |
| RFC 7636  | PKCE                         | Seguridad adicional en móviles      |
| RFC 7662  | Token Introspection          | Validación de tokens                |
| RFC 9068  | JWT Profile                  | Estandariza uso de JWT              |

---

## ✅ Buenas Prácticas

- Usa **PKCE** en apps móviles y SPAs
- Expira tokens en minutos
- Usa scopes específicos (`read`, `write`, etc.)
- Implementa logs, revocación y rotating refresh tokens

---

## 🧠 Conclusión

OAuth 2.0 no es solo un protocolo de autenticación, sino un **marco flexible para autorización moderna**.

Contribuye a:

- 🔒 Separación entre usuario y recursos
- 🛡️ Control de acceso granular
- 🧭 Trazabilidad y auditoría

Es la base de soluciones modernas como **OpenID Connect**, **API Gateway Security**, **SSO empresarial**, etc.

---

