---
title: "Shortcuts de GitHub Copilot en VS Code"
date: 2025-07-26
description: "Aprende los atajos de teclado y comandos del chat más útiles para trabajar eficientemente con GitHub Copilot en Visual Studio Code."
tags: ["Copilot", "VS Code", "Productividad", "Shortcuts", "GitHub"]
categories: ["IA para desarrolladores"]
toc: true
---

GitHub Copilot mejora tu productividad como desarrollador, pero combinarlo con **shortcuts** y **comandos de chat** en VS Code lleva la experiencia a otro nivel. Aquí te dejamos los más esenciales.

---

## ⌨️ Shortcuts esenciales de Copilot

| Acción                                     | Windows / Linux           | macOS                      |
|-------------------------------------------|----------------------------|----------------------------|
| **Aceptar sugerencia**                    | `Tab`                     | `Tab`                      |
| **Aceptar la palabra siguiente**          | `Ctrl` + `→`              | `⌃` + `→`                  |
| **Aceptar la línea siguiente**            | `Ctrl` + `↓`              | `⌃` + `↓`                  |
| **Ver siguiente sugerencia**              | `Alt` + `]`               | `⌥` + `]`                  |
| **Ver sugerencia anterior**               | `Alt` + `[`               | `⌥` + `[`                  |
| **Abrir panel de sugerencias (Copilot)**  | `Ctrl` + `Enter`          | `⌘` + `Enter`              |
| **Descartar sugerencia actual**           | `Esc`                     | `Esc`                      |
| **Forzar una nueva sugerencia**           | `Ctrl` + `Alt` + `\`     | `⌘` + `⌥` + `\`           |

---

## 💬 Comandos del chat de Copilot

Estos comandos pueden usarse dentro del panel de **Copilot Chat** (`Ctrl+I` o ícono del robot en la barra lateral) para interactuar con el código de forma contextual:

| Comando       | Descripción                                                       |
|---------------|-------------------------------------------------------------------|
| `/explain`    | Explica el bloque de código seleccionado                          |
| `/fix`        | Intenta corregir errores en el bloque de código                   |
| `/tests`      | Genera pruebas unitarias para el código seleccionado              |
| `/optimize`   | Mejora el rendimiento o legibilidad del código                    |
| `/doc`        | Agrega documentación (JavaDoc, comentarios, etc.)                 |
| `/translate`  | Traduce el código o comentario al idioma indicado                 |
| `/steps`      | Describe paso a paso lo que hace el código                        |

> 💡 Usa estos comandos luego de seleccionar un fragmento de código o posiciónate dentro del método/clase deseada.

---

## 🧠 Buenas prácticas al usar Copilot

- **Escribe comentarios claros** para obtener mejores sugerencias.
- **Acepta solo lo que entiendes.** No confíes ciegamente.
- **Combina con testing.** Asegúrate de validar el código generado.

---

## ⚙️ Personalización de shortcuts

Puedes personalizar los shortcuts de Copilot desde:

```
File → Preferences → Keyboard Shortcuts
```

Busca `copilot` y edita los accesos directos según tu gusto.

---

## 📌 Tip extra

Para pedirle a Copilot una función más compleja, escribe un comentario como:

```java
// Función para validar una tarjeta de crédito usando algoritmo de Luhn
```

Y presiona `Enter`. Copilot generará una sugerencia basada en el comentario.

---

¿Quieres aprender más? Visita la [documentación oficial de GitHub Copilot](https://docs.github.com/copilot).
