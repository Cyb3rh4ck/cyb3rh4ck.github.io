+++
title = "La Relevancia de SOLID (Traducción)"
date = "2026-03-28T21:42:00-06:00"
draft = false
description = "Una traducción fiel del artículo 'Solid Relevance' de Uncle Bob sobre la vigencia de los principios SOLID."
tags = ["SOLID", "Diseño de Software", "Traducción", "Uncle Bob"]
categories = ["Programación"]
+++

> **Nota:** Esta es una traducción fiel al español (latino) del artículo original ["Solid Relevance"](https://blog.cleancoder.com/uncle-bob/2020/10/18/Solid-Relevance.html) escrito por Robert C. Martin (Uncle Bob).

Recientemente recibí una carta de alguien con una inquietud. Decía lo siguiente:

> Durante años, el conocimiento de los principios SOLID ha sido una parte estándar de nuestro procedimiento de contratación. Se esperaba que los candidatos tuvieran un buen conocimiento práctico de estos principios. Sin embargo, últimamente, uno de nuestros gerentes, que ya no programa mucho, ha cuestionado si eso es sensato. Sus puntos fueron que el principio Abierto-Cerrado (Open-Closed) ya no es muy importante porque la mayor parte del código que escribimos no está contenido en grandes monolitos y hacer cambios en pequeños microservicios es seguro y fácil. El Principio de Sustitución de Liskov está muy desactualizado porque ya no nos enfocamos tanto en la herencia como lo hacíamos hace 20 años. Creo que deberíamos considerar [la postura de Dan North sobre SOLID](https://speakerdeck.com/tastapod/why-every-element-of-solid-is-wrong): "Simplemente escribe código simple".

Le escribí la siguiente carta como respuesta:

Los principios SOLID siguen siendo tan relevantes hoy como lo eran en los años 90 (y de hecho, desde antes). Esto se debe a que el software no ha cambiado tanto en todos estos años, y eso es porque el software no ha cambiado mucho desde 1945, cuando Turing escribió las primeras líneas de código para una computadora electrónica. El software sigue siendo declaraciones `if`, bucles `while` y sentencias de asignación: Secuencia, Selección e Iteración.

Cada nueva generación asume que su mundo es enormemente diferente al de la generación anterior. Cada nueva generación se equivoca al respecto; lo cual es algo que cada nueva generación aprende una vez que llega la siguiente para decirles cuánto ha cambiado todo. `<grin>`

Así que analicemos los principios, uno por uno.

### SRP) El Principio de Responsabilidad Única (Single Responsibility Principle)
*Reúne las cosas que cambian por las mismas razones. Separa las cosas que cambian por razones diferentes.*

Es difícil imaginar que este principio no sea relevante en el software. No mezclamos reglas de negocio con código de la interfaz gráfica (GUI). No mezclamos consultas SQL con protocolos de comunicaciones. Mantenemos separado el código que cambia por diferentes razones para que los cambios en una parte no rompan otras partes. Nos aseguramos de que los módulos que cambian por diferentes razones no tengan dependencias que los enreden.

Los microservicios no resuelven este problema. Puedes crear un microservicio enredado, o un conjunto enredado de microservicios si mezclas código que cambia por diferentes razones.

Las diapositivas de Dan North no entienden en absoluto el punto sobre esto, y me convencen de que no entendió el principio en lo más mínimo (o que estaba siendo irónico, lo cual, conociendo a Dan, es mucho más probable). Su respuesta al SRP es "Escribe Código Simple". Estoy de acuerdo. El SRP es una de las formas en que mantenemos el código simple.

### OCP) El Principio Abierto-Cerrado (Open-Closed Principle)
*Un módulo debe estar abierto para su extensión pero cerrado para su modificación.*

De todos los principios, la idea de que alguien cuestione este me llena de temor por el futuro de nuestra industria. Por supuesto que queremos crear módulos que puedan extenderse sin modificarlos. ¿Te imaginas trabajar en un sistema que no tuviera independencia de dispositivos, donde escribir en un archivo de disco fuera fundamentalmente diferente a escribir en una impresora, una pantalla o un pipe? ¿Queremos ver declaraciones `if` esparcidas por todo nuestro código para lidiar con todos los pequeños detalles?

O... ¿Queremos separar los conceptos abstractos de los conceptos detallados? ¿Queremos mantener las reglas de negocio aisladas de los pequeños y desagradables detalles de la interfaz gráfica, los protocolos de comunicación de los microservicios y los comportamientos arbitrarios de la base de datos? ¡Por supuesto que sí!

De nuevo, la diapositiva de Dan se equivoca completamente en esto. Cuando los requerimientos cambian, solo una parte del código existente está "mal". Gran parte del código existente sigue estando "bien". Y queremos asegurarnos de no tener que cambiar el código que está bien solo para hacer que el código que está mal vuelva a funcionar. La respuesta de Dan es "escribe código simple". De nuevo, estoy de acuerdo. E, irónicamente, él tiene razón. El código simple es tanto abierto como cerrado.

### LSP) El Principio de Sustitución de Liskov (Liskov Substitution Principle)
*Un programa que usa una interfaz no debe confundirse por una implementación de esa interfaz.*

La gente (incluyéndome) ha cometido el error de pensar que esto trata sobre la herencia. No es así. Trata sobre los subtipos (sub-typing). Todas las implementaciones de interfaces son subtipos de una interfaz. Todos los duck-types son subtipos de una interfaz implícita. Y todo usuario de la interfaz base, ya sea declarada o implícita, debe estar de acuerdo con el significado de esa interfaz. Si una implementación confunde al usuario del tipo base, entonces las declaraciones `if`/`switch` proliferarán.

Este principio trata de mantener las abstracciones nítidas y bien definidas. Es imposible creer que este sea un concepto anticuado.

Las diapositivas de Dan son completamente correctas en este tema; simplemente no captó el objetivo del principio. El código simple es el código que mantiene relaciones de subtipos nítidas.

### ISP) El Principio de Segregación de Interfaces (Interface Segregation Principle)
*Mantén las interfaces pequeñas para que los usuarios no terminen dependiendo de cosas que no necesitan.*

Todavía trabajamos con lenguajes compilados. Todavía dependemos de las fechas de modificación para determinar qué módulos deben ser recompilados y redesplegados. Mientras esto siga siendo cierto, tendremos que enfrentar el problema de que cuando el módulo A depende del módulo B en tiempo de compilación, pero no en tiempo de ejecución, los cambios en el módulo B forzarán la recompilación y el redespliegue del módulo A.

Este problema es especialmente agudo en lenguajes de tipado estático como Java, C#, C++, Go, Swift, etc. Los lenguajes de tipado dinámico se ven mucho menos afectados, pero tampoco son inmunes. La existencia de Maven y Leiningen es prueba de ello.

La diapositiva de Dan sobre este tema es demostrablemente falsa. Los clientes sí dependen de métodos que no llaman, si tienen que ser recompilados y redesplegados cuando uno de esos métodos es modificado. El punto final de Dan sobre este principio está bien, hasta cierto punto. Sí, si puedes separar una clase con dos interfaces en dos clases separadas, entonces es una buena idea hacerlo (SRP). Pero dicha separación a menudo no es factible, ni siquiera deseable.

### DIP) El Principio de Inversión de Dependencias (Dependency Inversion Principle)
*Depende en la dirección de la abstracción. Los módulos de alto nivel no deben depender de los detalles de bajo nivel.*

Es difícil imaginar una arquitectura que no haga un uso significativo de este principio. No queremos que nuestras reglas de negocio de alto nivel dependan de detalles de bajo nivel. Espero que eso sea perfectamente obvio. No queremos que los cálculos que nos hacen ganar dinero se contaminen con SQL, o validaciones de bajo nivel, o problemas de formato. Queremos aislamiento entre las abstracciones de alto nivel y los detalles de bajo nivel. Esa separación se logra manejando cuidadosamente las dependencias dentro del sistema para que todas las dependencias del código fuente, especialmente aquellas que cruzan fronteras arquitectónicas, apunten hacia abstracciones de alto nivel, no hacia detalles de bajo nivel.

En todos los casos, las diapositivas de Dan terminan con: *simplemente escribe código simple*. Este es un buen consejo. Sin embargo, si los años nos han enseñado algo es que la simplicidad requiere disciplinas guiadas por principios. Son esos principios los que definen la simplicidad. Son esas disciplinas las que obligan a los programadores a producir código que se incline hacia la simplicidad.

La mejor manera de hacer un desastre complicado es decirle a todos que "simplemente sean simples" y no darles ninguna otra guía.
