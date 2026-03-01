---
title: "Automatiza tu código con Copilot en VS Code"
date: 2025-07-26
description: "Guía paso a paso para instalar, configurar y usar GitHub Copilot en Visual Studio Code con un ejemplo en Java y Spring Boot."
tags: ["Copilot", "VS Code", "Java", "Productividad", "Spring Boot"]
categories: ["IA para desarrolladores"]
toc: true
---

## 🧠 Introducción

GitHub Copilot es un asistente de programación impulsado por IA que sugiere código y funciones completas directamente desde tu editor. Esta guía te enseñará cómo instalarlo, configurarlo y sacarle el máximo provecho dentro de Visual Studio Code con un ejemplo práctico en Java y Spring Boot.

---

## 🛠️ Requisitos previos

- Tener una cuenta de GitHub
- Tener instalado [Visual Studio Code](https://code.visualstudio.com/)
- Tener instalado [Java 17](https://adoptium.net/)
- Conexión a internet
- Git instalado

---

## 🔌 Instalación de Copilot en VS Code

1. Abre VS Code
2. Ve a la extensión "GitHub Copilot"
3. Haz clic en instalar o usa el siguiente comando:

{{< highlight bash >}}
code --install-extension GitHub.copilot
{{< /highlight >}}

4. Una vez instalado, VS Code te pedirá iniciar sesión con GitHub.
5. Acepta los permisos y espera la confirmación.

---

## ⚙️ Configuración básica

- Asegúrate de que Copilot esté activado en la barra de estado.
- Configura el comportamiento de sugerencias:
  - Aceptar: `Tab`
  - Siguiente sugerencia: `Ctrl + ]`
  - Anterior sugerencia: `Ctrl + [`
- Puedes desactivar Copilot por archivo o carpeta desde la configuración.

---

## 🤖 Cómo utilizarlo: primeros ejemplos

Crea un archivo `Main.java` y escribe:

{{< highlight java >}}
public class Main {
    public static void main(String[] args) {
        // Escribe aquí: System.
        // Copilot sugerirá: System.out.println("Hola mundo");
    }
}
{{< /highlight >}}

También puedes probar creando una clase de modelo o DTO:

{{< highlight java >}}
public class Persona {
    private String nombre;
    private int edad;

    // Copilot sugiere automáticamente constructor, getters y setters
}
{{< /highlight >}}

---

## 🚀 Ejemplo completo (con Spring Boot)

### Crear un proyecto nuevo

Usa Spring Initializr desde VS Code:
- Ctrl+Shift+P > "Spring Initializr: Create Maven Project"
- Lenguaje: Java
- Versión: 17
- Dependencias: Spring Web

### Clase generada con ayuda de Copilot

{{< highlight java >}}
@RestController
@RequestMapping("/api")
public class GreetingController {

    @GetMapping("/hello")
    public String sayHello() {
        return "Hola desde Copilot!";
    }
}
{{< /highlight >}}

---

## 🧪 Tips y buenas prácticas

- Revisa siempre el código sugerido: puede tener errores o no aplicar a tu contexto.
- Úsalo como apoyo, no como reemplazo.
- Ideal para tareas repetitivas: getters/setters, controladores REST, tests.

---

## 📦 Recursos y documentación

- [Documentación oficial de Copilot](https://docs.github.com/en/copilot)
- [Extensión en VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot)
- [Guía Spring Boot](https://spring.io/guides)

---

## 📌 Conclusión

Copilot puede aumentar tu productividad como desarrollador Java, especialmente en tareas repetitivas. Aprende a sacarle jugo sin perder el control del código que escribes. ¿Ya lo probaste?
