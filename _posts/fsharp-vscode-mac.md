---
layout: post
title: Trabajar con FSharp en Visual Studio Code y Mac
categories: es tutorial
---

Con el paso del tiempo, estoy apreciando cada dia más macOX como entorno de trabajo y desarrollo debido a gran parte a su estabilidad como OS, y porque si eres desarrollador de aplicaciones multiplataforma, es el entorno ideal para trabajar, a no ser que tengas Windows Phone o Windows 10 como target en el desarrollo del proyecto, lo cual ocurre muy eventualmente para seros sinceros. Pero bueno, este no es un post para ver que OS es mejor, si macOX o Windows. El mejor es el que más te guste.

El objetivo de este post es como preparar Visual Studio Code en Mac para desarrollar con FSharp. 

Para comenzar a trabjar con FSharp con Visual Studio Code hay que instalar [Ionide](http://ionide.io/){:target="_blank"}. Hay tres paquetes que debemos instalar, **Ionide-fsharp**, **Ionide-Paket** y **Ionide-Fake**: 

![Mac and Forms]({{ site.url }}/images/vscode_ionide.png)

**Paket** es un gestor de los Nuget packages que vamos a instalar en nuestros proyectos de FSharp, y **Fake** es la herramienta de compilación de FSharp.

Lo primero es crear un nuevo proyecto de FSharp. Es muy facil crearlo a través de la paleta de comandos de Visual Studio Code (Command + Shift + P). Una vez en la paleta escribimos *F# New Project*:

![F# New Project]({{ site.url }}/images/fsharp_new_project.png)