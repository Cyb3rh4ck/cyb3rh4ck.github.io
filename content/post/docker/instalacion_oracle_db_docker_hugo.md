---
title: "Instalación de Oracle DB en Docker Desktop (desde cero)"
date: 2025-07-26
#type: "docs"
draft: false
#weight: 1
description: "Guía paso a paso para instalar Oracle Database en Docker Desktop usando la imagen oficial del Oracle Container Registry."
---

# 🐳 Instalación de Oracle Database en Docker Desktop

Esta guía te explica cómo instalar **Oracle Database Enterprise Edition** usando Docker Desktop en Windows o Linux, **utilizando la imagen oficial desde el Oracle Container Registry**.

---

## ✅ Prerrequisitos

- Tener instalado **Docker Desktop**
- Contar con una cuenta activa en [https://profile.oracle.com](https://profile.oracle.com)

---

## 🔐 Paso 1: Aceptar los términos de la imagen

1. Ir al sitio: [https://container-registry.oracle.com](https://container-registry.oracle.com)
2. Iniciar sesión con tu cuenta Oracle
3. Buscar: `database/enterprise`
4. Entrar al resultado y **aceptar los términos de licencia**

---

## 🔑 Paso 2: Generar un Auth Token (opcional pero recomendado)

1. Ve a: [https://profile.oracle.com](https://profile.oracle.com)
2. En el menú izquierdo, selecciona **"Security" → "Auth Tokens"**
3. Crea un nuevo token con nombre `docker-token`
4. Guarda el token (se muestra solo una vez)

---

## 🔐 Paso 3: Login en Oracle Container Registry

Abre una terminal o PowerShell y ejecuta:

```bash
docker login container-registry.oracle.com
```

🔹 Ingresa tu **email de Oracle como usuario**  
🔹 Usa el **Auth Token como contraseña**

---

## 📦 Paso 4: Descargar la imagen oficial de Oracle Database

```bash
docker pull container-registry.oracle.com/database/enterprise:21.3.0.0
```

> Puedes reemplazar `21.3.0.0` por otra versión como `19.3.0.0` si lo prefieres.

---

## 🚀 Paso 5: Ejecutar el contenedor

```bash
docker run -d --name oracle21c ^
  -p 1521:1521 -p 5500:5500 ^
  -e ORACLE_PWD=MiPass123 ^
  container-registry.oracle.com/database/enterprise:21.3.0.0
```

🔸 Detalles:
- `1521`: puerto para conexiones SQL*Net (SQL Developer, JDBC)
- `5500`: Oracle Enterprise Manager
- `ORACLE_PWD`: password para usuarios `SYS` y `SYSTEM`

---

## 🔍 Paso 6: Verificar que el contenedor está corriendo

```bash
docker ps
```

Debes ver algo como:

```
CONTAINER ID   IMAGE                                                 PORTS
abc123456789   container-registry.oracle.com/database/enterprise...  0.0.0.0:1521->1521/tcp
```

---

## 📥 Acceder a Oracle desde SQL Developer

- **Host:** `localhost`
- **Puerto:** `1521`
- **Usuario:** `system`
- **Password:** `MiPass123`
- **SID:** `ORCLCDB`

---

## 🧼 Para detener y eliminar el contenedor

```bash
docker stop oracle21c
docker rm oracle21c
```

---

## ✅ Resultado

Ya tienes Oracle Database Enterprise Edition corriendo en Docker Desktop, listo para desarrollo y pruebas locales.

> 🧪 También puedes usar la imagen comunitaria `gvenzl/oracle-xe` si deseas evitar el login o usar una versión más ligera.
