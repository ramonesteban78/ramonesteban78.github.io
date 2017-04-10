---
layout: post
title: Como hacer un Backend rápido para tus apps con Easy Tables Azure
categories: tutorials
---

## Intro

Cuando estamos probando algún código o haciendo un prototipo de app, o cuando quieres entregar una primera versión de tu app cuando el backend aun no está montado (que suele ocurrir con bastante frecuencia), es necesario muchas veces un pequeño backend o datos Fake que usará tu app.

Una solución rápida que podemos hacer es utilizar Easy Tables de Mobile Services de Azure. Este post no se trata de una guia de como dar de alta este servicio de Azure, para eso ya existen muchos posts al respecto. Lo que quiero mostraros es que estas tablas son accesibles a través de `HttpClient` y tienen las operaciones CRUD básicas a realizar sobre ellas.

## Como acceder a un Easy Table usando HttpClient


