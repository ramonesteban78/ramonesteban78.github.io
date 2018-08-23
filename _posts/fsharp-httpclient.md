---
layout: post
title: F# async await
categories: es tutorials
---

# Introducción

Si vienes de C# y quieres empezar a trabajar con F# con programación asincrona como hacemos habitualmente en C#, estos los conceptos que hay que tener claros para empezar.

# Trabajando con HpptClient

Si no utilizáis alguna librería como [Refit]("http://www.google.es") u otra similar, seguramente os ha tocado utilizar `HttpClient` para realizar tus llamadas a Web APIs. Vamos a hacer lo mismo con F# para que al mismo tiempo veamos como se realizan llamadas asincronas.

# Download url

Vamos a empezar por una función muy simple que consiste en descargar el html de una url que indiquemos en nuestra funcción:

```c#
let searchWeb url =
    async {
        let client = new HttpClient()
        let! response = client.GetAsync(url) |> Async.AwaitTask
        response.EnsureSuccessStatusCode () |> ignore
        let! content = response.Content.ReadAsStringAsync() |> Async.AwaitTask
        return content   
    }
``` 


`|> Async.RunSynchronously` espera la ejecución de un Task

Si definimos una función de forma async esperar la llamada de otra función asincrona no necesita de acciones con el operador `Async`. Este es un ejemplo de una función que utiliza la anterior definida:

```c#
let findSomething name = 
    async {
        let search = "https://www.google.es/search?q=" + name
        let! result = downloadHtml search
        return result
    } 
``` 

Lo único que tenemos que indicar es que se trata de una llamada asincrona en la declaración de la variable que va a obtener el resultado de la forma `let!`.

Lo más importante es que la forma natural de tratar async en F# es `Async<T'>`, es decir, no utilizan `Task`. Para esperar un Task utilizamos `Async.AwaitTask`.

`Async.AwaitTask` es tan solo una función para esperar Tasks que es lo que devuelven casi todas las llamadas de las librerias de .Net