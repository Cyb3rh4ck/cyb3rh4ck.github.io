---
title: "Errors vs Exceptions"
description: "Qué es un Error, por qué no se captura ni se lanza."
weight: 70
toc: true
---

## ¿Qué es un `Error`?

Un `Error` indica un fallo grave donde la JVM está en estado **no recuperable**.

Ejemplos:
- `OutOfMemoryError`
- `StackOverflowError`

## Figura 4.9 — Jerarquía completa

```text
Throwable
├── Error              (NO se maneja)
└── Exception
    ├── RuntimeException (unchecked)
    └── Checked exceptions
```

## Reglas senior (muy claras)

- ❌ No lances `Error`
- ❌ No declares `Error` en firmas
- ❌ No intentes capturar `Error`

¿Por qué?  
Porque cuando ocurre, el proceso puede estar inestable y **ni siquiera es seguro que tu catch/log se ejecute**.

> Si ves `Error`, asume: “el programa está muriendo”.  
> La acción es corregir el bug (memoria, recursión, etc.).
