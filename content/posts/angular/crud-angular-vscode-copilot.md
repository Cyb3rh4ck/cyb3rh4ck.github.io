---
title: "Crear un CRUD en Angular con VS Code y Copilot (Arquitectura Hexagonal)"
date: 2025-07-28
description: "Aprende a crear un CRUD en Angular usando Visual Studio Code y GitHub Copilot, aplicando principios de arquitectura hexagonal en el frontend"
tags: ["Angular", "Frontend", "Java", "Spring Boot", "API REST", "Arquitectura Hexagonal", "Desarrollo Web"]
categories: ["Frontend"]
toc: true
---

## 🧠 Introducción

Crear una aplicación Angular (versión recomendada: **18.x**) que consuma un API REST con operaciones CRUD para una entidad (por ejemplo: `Cliente` o `Producto`). Se aplicará una estructura inspirada en **arquitectura hexagonal** para mejorar la mantenibilidad del frontend. El backend puede estar hecho en Java + Spring Boot, pero esta guía se centra en el frontend.

---

## 🔧 Requisitos del entorno

- Node.js v20.19 o superior (`node -v`)
- Angular CLI (`npm install -g @angular/cli`)
- Angular v18.x (`ng version`)
- Un backend REST disponible (puerto, endpoints y CORS habilitado)
- Editor recomendado: **Visual Studio Code** con extensión GitHub Copilot

---

## 🏁 Paso 1: Crear la aplicación Angular

```bash
ng new frontend-crud-app
cd frontend-crud-app
ng serve --open
```

Selecciona **Routing = Yes**, **Estilo = CSS o SCSS** según tu preferencia.

---

## 🧱 Paso 2: Estructura basada en arquitectura hexagonal

```text
src/
├── app/
│   ├── domain/               # Modelos y lógica de negocio
│   │   └── cliente.model.ts
│   ├── application/          # Casos de uso (servicios de aplicación)
│   │   └── cliente.usecase.ts
│   ├── infrastructure/       # Servicios HTTP, adaptadores externos
│   │   └── cliente.service.ts
│   ├── ui/                   # Componentes y vistas
│   │   ├── cliente-list/
│   │   ├── cliente-form/
```

---

## 🔌 Paso 3: Crear modelo de dominio

`src/app/domain/cliente.model.ts`

```ts
export interface Cliente {
  id: number;
  nombre: string;
  email: string;
  telefono: string;
}
```

---

## 🧠 Paso 4: Lógica de aplicación

`src/app/application/cliente.usecase.ts`

```ts
import { Injectable } from '@angular/core';
import { ClienteService } from '../infrastructure/cliente.service';
import { Cliente } from '../domain/cliente.model';

@Injectable({ providedIn: 'root' })
export class ClienteUseCase {
  constructor(private clienteService: ClienteService) {}

  listar() {
    return this.clienteService.getAll();
  }

  guardar(cliente: Cliente) {
    return cliente.id
      ? this.clienteService.update(cliente)
      : this.clienteService.create(cliente);
  }

  eliminar(id: number) {
    return this.clienteService.delete(id);
  }
}
```

---

## 🌐 Paso 5: Servicio de infraestructura

`src/app/infrastructure/cliente.service.ts`

```ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Cliente } from '../domain/cliente.model';
import { Observable } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class ClienteService {
  private apiUrl = 'http://localhost:8080/api/clientes';

  constructor(private http: HttpClient) {}

  getAll(): Observable<Cliente[]> {
    return this.http.get<Cliente[]>(this.apiUrl);
  }

  getById(id: number): Observable<Cliente> {
    return this.http.get<Cliente>(`${this.apiUrl}/${id}`);
  }

  create(cliente: Cliente): Observable<Cliente> {
    return this.http.post<Cliente>(this.apiUrl, cliente);
  }

  update(cliente: Cliente): Observable<Cliente> {
    return this.http.put<Cliente>(`${this.apiUrl}/${cliente.id}`, cliente);
  }

  delete(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}
```

---

## 📦 Paso 6: Importar `HttpClientModule`

Edita `app.module.ts`:

```ts
import { HttpClientModule } from '@angular/common/http';

@NgModule({
  imports: [
    HttpClientModule,
    ...
  ]
})
```

---

## 🧱 Paso 7: Crear componentes

```bash
ng generate component ui/cliente-list
ng generate component ui/cliente-form
```

Ejemplo de `cliente-list.component.ts`:

```ts
import { Component, OnInit } from '@angular/core';
import { ClienteUseCase } from 'src/app/application/cliente.usecase';
import { Cliente } from 'src/app/domain/cliente.model';

@Component({
  selector: 'app-cliente-list',
  templateUrl: './cliente-list.component.html',
})
export class ClienteListComponent implements OnInit {
  clientes: Cliente[] = [];

  constructor(private clienteUC: ClienteUseCase) {}

  ngOnInit() {
    this.clienteUC.listar().subscribe(data => this.clientes = data);
  }
}
```

---

## 🧪 Paso 8: Template básico

`cliente-list.component.html`:

```html
<h2>Lista de Clientes</h2>
<ul>
  <li *ngFor="let cliente of clientes">
    {{ cliente.nombre }} - {{ cliente.email }}
  </li>
</ul>
```

---

## 🔄 Paso 9: Rutas

Edita `app-routing.module.ts`:

```ts
const routes: Routes = [
  { path: 'clientes', component: ClienteListComponent },
  { path: 'formulario', component: ClienteFormComponent },
  { path: '', redirectTo: '/clientes', pathMatch: 'full' },
];
```

---

## 🔗 Paso 10: CORS en el backend

Asegúrate de tener en el backend:

```java
@CrossOrigin(origins = "http://localhost:4200")
@RestController
@RequestMapping("/api/clientes")
public class ClienteController {
   ...
}
```

---

## ✅ Paso 11: Probar y dejar que Copilot te sugiera mejoras

Ejecuta:

```bash
ng serve
```

Y abre [http://localhost:4200](http://localhost:4200) para verificar que carga correctamente la lista de clientes.

---

## 🤖 Tips para usar Copilot en Angular

- [Angular Official Docs](https://angular.io/docs)
- [Arquitectura Hexagonal en Frontend](https://blog.octo.com/en/hexagonal-architecture-in-frontend-projects/)
- [Spring Boot REST Docs](https://spring.io/guides/gs/rest-service/)
- [RxJS](https://rxjs.dev/)
- [VS Code: Angular Snippets, ESLint, Prettier]

---

## 🚀 Mejoras futuras

- Validaciones en formularios reactivos
- Autenticación JWT
- Paginación y ordenamiento
- Interceptores para manejo de errores
- Pruebas unitarias y end-to-end

---

---

## 🤖 Tips para usar Copilot en Angular

- Usa comentarios como `// Create Angular service for API REST` y presiona `Tab` para que Copilot sugiera automáticamente el código.
- Copilot también puede completar tus templates HTML si detecta un `*ngFor` o `[(ngModel)]`.
- Utiliza comentarios en inglés para obtener mejores resultados: `// Create reactive form for Cliente`.
- Valida siempre la salida de Copilot, sobre todo en seguridad y errores.
- Puedes invocar Copilot manualmente con `Ctrl+I` en VS Code o escribiendo `// Copilot:` seguido del objetivo.

---

