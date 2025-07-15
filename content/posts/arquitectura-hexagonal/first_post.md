+++
date = '2025-07-15T12:14:34-06:00'
draft = false
title = 'Arquitectura hexagonal'
+++


La arquitectura hexagonal, también conocida como "Ports and Adapters", es un patrón de diseño de software que busca aislar el núcleo de la aplicación de los detalles externos, como bases de datos, interfaces de usuario o servicios externos. El núcleo contiene la lógica de negocio y se comunica con el exterior a través de puertos (interfaces) y adaptadores, lo que facilita la prueba, el mantenimiento y la evolución del sistema sin afectar la lógica principal. Este enfoque promueve la independencia y la flexibilidad en el desarrollo de aplicaciones.