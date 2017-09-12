---
layout: post
title: Test de integración con xUnit y Xamarin.iOS
categories: es tutorials
---

Después de un mes de vacaciones (todos las merecemos) vuelvo con un pequeño post de como crear un proyecto de Unit Test con Xamarin.iOS y [xUnit](http://xunit.github.io/){:target="_blank"}.

No se si os ha ocurrido, que cuando inicias un proyecto, normalmente te pasan los endpoints de la Api, pero no puedes ir probando las llamadas a los mismos hasta que no llegues a implementar la pantalla en concreto en donde se utilizan. 

En estos casos, yo suelo hacerme un proyecto de Test de Integración con Xamarin.iOS, para ir probando los modelos o posibles excepciones que pueda devolver el Web Api. Pero antes de avanzar en como montar tu proyecto de Tests de integración paso a paso, me guataría exoplicaros el porque de elegir **xUnit** y el porque de un proyecto de Unit Test de Xamarin.iOS:

## ¿Por qué un proyecto de Test de integración con Xamarin.iOS?

¿Por qué no utilizar una PCL para probar los end points? Bien, es una decisión provocada por la librería que estoy usando para realizar mis llamadas a los endpoints: [Refit](https://github.com/paulcbetts/refit){:target="_blank"}.

**Refit** no puede probarse en una PCL porque está implementado de forma nativa en cada plataforma en el caso de **Xamarin**, por lo que es necesario un proyecto de *Xamarin.iOS* o *Xamarin.Android*.

## ¿Por qué xUnit?

¿Por qué no utilizar el proyecto de Unit Test de Xamarin.iOS que viene como template en el Visual Studio for Mac el cual utiliza NUnit en vez de xUnit?

La razón es muy sencilla, la librería NUnitLite que utiliza Xamarin para estos proyectos no soporta *async/await*, podéis encontrar má info aquí al respecto: [Deadlock in NUnit test](https://forums.xamarin.com/discussion/14175/deadlock-in-nunit-test){:target="_blank"}.

## Como montar el proyecto de Tests de integración Xamarin iOS con xUnit

Es muy simple, tan solo tenemos que crear un proyecto **Xamarin.iOS** vacio e instalar los siguientes paquetes de Nuget:

- [xUnit](https://www.nuget.org/packages/xunit){:target="_blank"}.
- [xUnit.Runner.Devices](https://www.nuget.org/packages/xunit.runner.devices){:target="_blank"}.

Una vez instalados, el runner de **xUnit** te crea un *AppDelegate.cs.txt*. Copiamos el contenido del mismo y lo pegamos en nuestro *AppDelegate.cs*. 

Ya solo queda crear nuestros Test de integración. Aquí os dejo un pequeño ejemplo de un Test con **xUnit** de una llamada de Login:

```c#
public class ApiTests
{
    private readonly IApiService _ApiService;

    public ApiTests()
    {
        _ApiService = new ApiService();
    }

    [Fact]
    public async Task login_ok()
    {
        var result = await _ApiService.Login("testuserok", "testpasswordok");
        Assert.NotNull(result);
    }

    [Fact]
    public async Task login_error()
    {
        await AssertEx.ThrowsAsync<InvalidUserApiException>(() => _ApiService.Login("wronguser", "wrongpassword"));
    }
}
```

## Último paso

Muchas veces podemos probar nuestro Web API para que devuelva una excepción de un cierto tipo. Es lo que estoy haciendo en el método `login_error`, lo que ocurre es que la forma de probar excepciones en **xUnit** es con `Assert.Throws` la cual no acepta llamadas *async/await*. Para ello me he creado una clase estática con el método `ThrowsAsync`. Os dejo la implementación de la misma:

```c#
public static class AssertEx
{
    public static async Task ThrowsAsync<TException>(Func<Task> func)
    {
        var expected = typeof(TException);
        Type actual = null;
        try
        {
            await func();
        }
        catch (Exception e)
        {
            actual = e.GetType();
        }
        Assert.Equal(expected, actual);
    }
}
```

Eso es todo, en unos pocos pasos ya estamos listos para montar nuestros Test de integración con **xUnit** y **Xamarin.iOS**. Os dejo aquí unas pantallitas de los resultados para que no diagais que no pongo imágenes en los Posts :). 

Ejecutamos nuestro proyecto y a probar. Good Testing!

![IntegrationTest1]({{ site.url }}/images/integrationtest1.png)
![IntegrationTest2]({{ site.url }}/images/integrationtest2.png)
![IntegrationTest3]({{ site.url }}/images/integrationtest3.png)

Saludos