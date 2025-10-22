---
title: "GitHub Copilot Agents @"
date: 2025-10-22
weight: 3
---


---

## 🧠 ¿Qué son los GitHub Copilot Agents?

A partir del **30 de noviembre de 2023**, GitHub introdujo un concepto llamado ***Copilot Agents***, que son **“mini asistentes especializados”** dentro del **chat de GitHub Copilot**.

En lugar de que el chat solo responda preguntas generales, ahora puedes invocar **agentes especializados** usando el símbolo **@** (arroba) directamente en el cuadro de texto del chat, dentro de **Visual Studio Code** o **GitHub Copilot Chat**.

Estos agentes tienen **roles distintos** y pueden ayudarte en tareas más específicas relacionadas con tu entorno de desarrollo, tu código y tus repositorios.

---

## ⚙️ Cómo funcionan

Cuando estás en el panel de **Copilot Chat** dentro de VS Code, puedes escribir:

```
@workspace
```

y Copilot entenderá que quieres hablar con el **agente de workspace**, que se enfoca en tu proyecto actual.

A medida que GitHub ha ido expandiendo Copilot, se han ido agregando **diferentes tipos de agentes**, cada uno con un enfoque distinto.

---

## 🧩 Tipos principales de Copilot Agents (a noviembre 2023 – actualidad)

1. ### 🗂️ **@workspace**

   * Es el agente más importante.
   * Te ayuda a **entender la estructura de tu proyecto**, encontrar archivos, buscar clases, funciones o métodos dentro del workspace.
   * También puede ayudarte a **generar o refactorizar código** considerando múltiples archivos del proyecto.
   * 👉 Ejemplo de uso:

     ```
     @workspace summarize my project structure
     ```

     o

     ```
     @workspace show all files related to user authentication
     ```

   ⚠️ Sin embargo, **por seguridad**, GitHub **no le da acceso directo a tu sistema de archivos local**.
   Por eso, si escribes algo como:

   ```
   @workspace what's my current folder structure?
   ```

   Copilot responderá algo como:

   > "I don’t have access to your local file system."

   Esto se debe a que Copilot solo ve **los archivos abiertos en el editor** o **los que forman parte del contexto de VS Code**, no tu sistema completo.
   En otras palabras, **no puede escanear todo tu disco duro ni ver archivos fuera del proyecto abierto.**

---

2. ### 🧠 **@vscode**

   * Este agente te permite interactuar con **Visual Studio Code** mismo.
   * Puede responder preguntas sobre comandos, atajos de teclado, extensiones o configuraciones.
   * Ejemplo:

     ```
     @vscode how do I change the editor font size?
     ```

---

3. ### 💡 **@terminal**

   * Te ayuda con **comandos de terminal o consola**, explicando qué hacen o generándolos por ti.
   * Ejemplo:

     ```
     @terminal explain what "git rebase --continue" does
     ```
   * Nota: por seguridad, **no ejecuta comandos**, solo los explica o sugiere.

---

4. ### 🔍 **@github**

   * Conecta con la información de tu cuenta de GitHub o repositorios (si tienes permisos configurados).
   * Ejemplo:

     ```
     @github find open pull requests assigned to me
     ```
   * Este agente puede ayudarte a automatizar flujos de trabajo de repos (PRs, issues, etc.), aunque algunas funciones todavía están en beta.

---

## 🔒 ¿Por qué no puede ver tu sistema de archivos?

GitHub **diseñó Copilot bajo un principio de seguridad y privacidad fuerte**.
Copilot Chat **solo puede ver:**

* Archivos **abiertos en el editor**,
* Archivos **referenciados** por el contexto actual del chat,
* Y el contenido **dentro del workspace activo** de VS Code.

Esto evita que el modelo acceda a archivos personales o confidenciales fuera del proyecto.

Por eso, cuando escribes:

> "What's my current folder structure?"

El modelo responde algo como:

> "I don’t have access to your file system."

Significa que **Copilot Chat no tiene permiso de exploración** como un file explorer —solo puede analizar lo que VS Code le expone.

---

## 🚀 En resumen

| Concepto                          | Descripción                                                                |
| --------------------------------- | -------------------------------------------------------------------------- |
| **Copilot Agents**                | Subasistentes especializados de Copilot Chat.                              |
| **Símbolo @**                     | Sirve para invocar un agente específico, como @workspace o @vscode.        |
| **Acceso limitado**               | No pueden acceder directamente al sistema de archivos local por seguridad. |
| **@workspace**                    | Analiza tu proyecto actual y te ayuda a navegar o entenderlo.              |
| **@vscode / @terminal / @github** | Permiten interactuar con el IDE, la terminal o tu cuenta GitHub.           |

---

💡 **Dato curioso:**
Esta arquitectura modular de agentes es parte de la evolución hacia **“Copilot Workspace”**, una experiencia que GitHub está integrando donde el chat, el código y la IA trabajan como un entorno autónomo que entiende el contexto del desarrollador.

---

⚡
Aquí tienes una **tabla completa** con **ejemplos reales y prácticos** de cómo usar los **GitHub Copilot Agents** desde el chat 
Incluye los **cuatro agentes principales** (`@workspace`, `@vscode`, `@terminal`, `@github`) y ejemplos de **prompts reales**, con **descripción, propósito y posible respuesta del agente**.

---

## 🎓 **Tabla de ejemplos: GitHub Copilot Agents (para curso Zelda Core)**

| 🧩 Agente      | 🗣️ Ejemplo de prompt                                             | 🎯 Qué hace / Propósito                                                        | 💬 Respuesta típica del agente                                                                                                               |
| :------------- | :---------------------------------------------------------------- | :----------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------- |
| **@workspace** | `@workspace show me all classes that use RestController`          | Analiza tu proyecto actual y busca clases o archivos que tengan esa anotación. | “I found 3 files that contain @RestController: PolicyController.java, UserController.java, and HealthCheckController.java.”                  |
| **@workspace** | `@workspace summarize what this project does`                     | Lee los archivos abiertos y te da un resumen del propósito del proyecto.       | “This project is a Spring Boot microservice for managing insurance policies with endpoints for CRUD operations.”                             |
| **@workspace** | `@workspace explain the structure of my service layer`            | Analiza carpetas y clases relacionadas con el servicio.                        | “Your service layer consists of three main classes: PolicyService, CustomerService, and AuditService. Each one interacts with a repository.” |
| **@workspace** | `@workspace generate a unit test for PolicyService using Mockito` | Crea una prueba unitaria adaptada al contexto del proyecto.                    | “Here’s a JUnit 5 test using Mockito for PolicyService.savePolicy().” *(+código generado)*                                                   |
| **@workspace** | `@workspace find all TODO comments in the project`                | Escanea el workspace en busca de comentarios pendientes.                       | “I found 5 TODOs in PolicyController, PaymentService, and PolicyRepository.”                                                                 |

---

| 🧠 **@vscode** |                                                     |                                                |                                                                                                    |
| :------------- | :-------------------------------------------------- | :--------------------------------------------- | :------------------------------------------------------------------------------------------------- |
| **@vscode**    | `@vscode how do I change my theme to dark mode?`    | Explica o guía configuraciones del editor.     | “You can change your theme by pressing Ctrl+K Ctrl+T, or going to Preferences → Color Theme.”      |
| **@vscode**    | `@vscode open settings.json`                        | Abre directamente la configuración del editor. | *VS Code abre el archivo settings.json.*                                                           |
| **@vscode**    | `@vscode what shortcut formats code automatically?` | Explica atajos útiles.                         | “You can format code with Shift+Alt+F or Cmd+Shift+I on macOS.”                                    |
| **@vscode**    | `@vscode recommend extensions for Java development` | Sugiere extensiones relevantes.                | “Here are 3 recommended extensions: Language Support for Java, Debugger for Java, Maven for Java.” |

---

| 💻 **@terminal** |                                                            |                                                      |                                                                                               |                
| :--------------- | :--------------------------------------------------------- | :--------------------------------------------------- | :-------------------------------------------------------------------------------------------- | 
| **@terminal**    | `@terminal explain what git rebase --continue does`        | Explica un comando sin ejecutarlo.                   | “git rebase --continue continues the current rebase process after resolving merge conflicts.” |                
| **@terminal**    | `@terminal what does mvn clean install do?`                | Explica un comando Maven (útil para proyectos Java). | “It cleans previous builds and compiles your project, packaging it into a JAR or WAR.”        |                
| **@terminal**    | `@terminal show me how to kill a process on port 8080`     | Sugiere comandos útiles.                             | “You can run: lsof -ti:8080                                                                   | 
| **@terminal**    | `@terminal generate a command to list all branches in git` | Crea un comando con explicación.                     | “You can list all local branches using: git branch -a.”                                       |                

---

| 🧭 **@github** |                                                     |                                                              |                                                                                                        |
| :------------- | :-------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------------------------------------------------- |
| **@github**    | `@github list my open pull requests`                | Consulta información de tu cuenta GitHub (repos conectados). | “You currently have 2 open PRs in repo seguros360-backend.”                                            |
| **@github**    | `@github summarize recent activity on main branch`  | Resume actividad reciente del repositorio.                   | “There have been 5 commits in the last 24 hours, mostly related to refactoring controllers and tests.” |
| **@github**    | `@github show issues labeled as bug`                | Lista issues con etiquetas específicas.                      | “Found 3 issues labeled ‘bug’ in your repository.”                                                     |
| **@github**    | `@github generate release notes for the last 3 PRs` | Crea un resumen tipo changelog.                              | “Here’s a summary of the last 3 merged pull requests…” *(+markdown con detalles)*                      |

---

