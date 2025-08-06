---
title: "🌿 Convenciones de Ramas en Proyectos Limpios"
date: 2025-08-06
tags: ["git", "github", "best practices", "branching"]
categories: ["DevOps", "Workflows"]
---

# 🌿 Convenciones de Ramas en Proyectos Limpios

Cuando inicias un **proyecto nuevo** (clean project), tener un **flujo de ramas claro** facilita la colaboración y mantiene la historia del repositorio **organizada y profesional**.  

A continuación te muestro las **convenciones más utilizadas** en la industria para nombrar ramas:

---

## **📌 1. Tipos de ramas y su uso**

| Tipo de rama   | Uso principal                                         | Ejemplo                             |
|----------------|------------------------------------------------------|-------------------------------------|
| `feature/`     | Nueva funcionalidad                                   | `feature/register-user`              |
| `bugfix/`      | Corrección de errores menores                         | `bugfix/fix-login-nullpointer`       |
| `hotfix/`      | Corrección **urgente** en producción                  | `hotfix/jwt-expired-issue`           |
| `chore/`       | Configuración, refactor, CI/CD, sin cambiar lógica    | `chore/add-ci-cd`                    |
| `release/`     | Preparación de una nueva versión                      | `release/v1.0.0`                     |

---

## **📊 2. Flujo visual de ramas (Branch Flow)**

![Branch Flow](https://raw.githubusercontent.com/github/explore/main/topics/git/git.png)

```plaintext
         main
          │
          ├── chore/project-setup
          │
          ├── feature/register-user
          │        \
          │         └── bugfix/fix-email-validation
          │
          └── release/v1.0.0
                   \
                    └── hotfix/jwt-expired-issue
```

**Explicación del diagrama:**

1. **`main`** → rama principal siempre estable  
2. **`chore/*`** → tareas iniciales de setup o refactor  
3. **`feature/*`** → desarrollo de nuevas funcionalidades  
4. **`bugfix/*`** → pequeñas correcciones sobre features  
5. **`release/*`** → preparación de versiones estables  
6. **`hotfix/*`** → correcciones críticas directamente a producción  

---

## **💡 Recomendaciones profesionales**

1. **Usa nombres descriptivos:**  
   Evita `feature1` o `fix2`. Mejor:  
   ```plaintext
   feature/user-authentication
   bugfix/handle-null-tokens
   ```

2. **Mantén ramas cortas y claras:**  
   Cada rama debe atacar **un único objetivo**.  
   - ✅ `feature/upload-profile-picture`  
   - ❌ `feature/upload-picture-and-fix-login`  

3. **Usa commits claros y consistentes:**  
   Ejemplo de primer commit en un proyecto limpio:  
   ```plaintext
   chore: initial project setup with hexagonal structure
   ```

---

## **🚀 Ejemplo de inicio de proyecto limpio**

![Hexagonal Structure](https://raw.githubusercontent.com/github/explore/main/topics/spring-boot/spring-boot.png)

```plaintext
seguro360/
 └─ src/
     └─ main/
         ├─ java/io/cyb3rh4ck/github/seguros360/
         │         ├─ application/
         │         ├─ domain/
         │         └─ infrastructure/
         └─ resources/
             ├─ application.yml
             └─ logback-spring.xml
```

- Primer branch recomendado:
  ```bash
  git checkout -b chore/project-setup
  git push -u origin chore/project-setup
  ```

- Primer commit recomendado:
  ```plaintext
  chore: initial project setup with hexagonal structure
  ```

---

## **🎯 Beneficios de seguir esta convención**

- **Historia clara:** cualquiera entiende qué hace cada rama  
- **Facilita PRs y code reviews**  
- **Previene conflictos** al tener ramas separadas por tipo  
- **Alineado con Git Flow y GitHub Flow**  

---

> **Tip extra:**  
> Si usas **GitHub Projects** o **Jira**, nombra las ramas con el ID de la tarea:
> ```plaintext
> feature/SEGUROS360-12-register-user
> ```
> Así podrás trazar commits directamente con issues o historias de usuario.
---
