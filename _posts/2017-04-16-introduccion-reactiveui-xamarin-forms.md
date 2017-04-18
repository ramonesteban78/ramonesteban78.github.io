---
layout: post
title: Introducción a ReactiveUI con Xamarin.Forms - Parte 1
categories: es tutorials
---

## Intro

Después de mi último post sobre [Xamarin.Forms y Mvvm]({{ site.url }}/es/tutorials/mvvm/xamarin-forms-and-mvvm.html) he decido crear uno nuevo sobre uno de los frameworks de Mvvm que más me gustan: [ReactiveUI](http://reactiveui.net/){:target="_blank"}.

El desarrollo frontend es bastante complejo porque tenemos que lidiar con eventos, acciones de un usuario que no podemos prever de antemano.

Por mucho que intentes hacer una buena arquitectura desde el principio, si las necesidades del proyecto no te permiten hacer un refactoring apropiado, tus ViewModels pueden terminar siendo dificiles de gestionar.

[ReactiveUI](http://reactiveui.net/){:target="_blank"} permite mantener un código limpio y testeable al mismo tiempo que implementamos funcionalidades complejas.

## Functional Reactive Programming

[ReactiveUI](http://reactiveui.net/){:target="_blank"} está basado en los principios de **Functional Reactive Programming**. Los principios fundamentales de <abbr title="Functional Reactive Programming">FRP</abbr> son los siguientes:

- Indicamos a nuestro código **que hacer** en vez de **como hacerlo**, que es lo que solemos estar acostumbrados en una programación imperativa.
- **Functional**: Una función no tiene efectos secundarios. No existen estados mutables en funciones.
- **Reactive Programming**: Los cambios se propagan en nuestro sitema de forma automática. 

Para que todo esto sea posible FRP necesita **Signals** a los cuales nos podremos subscribir y reaccionar a estos dependiendo de las condiciones que nos aportan.

Para terminar de entender estos conceptos básicos de FRP es mejor utilizar un ejemplo: ![FRP example]({{ site.url }}/images/frp-excel.gif)
Como véis, lo primero que hacemos es indicar al sistema **que hacer** cuando se produzcan cambios. Definimos nuestra regla a través de una **función** que indica que C es la suma de A y B. Como véis una vez definidas las reglas dejamos al sistema que reaccione a las acciones del usuario.

## Reactive Extensions

![Reactive Extensions]({{ site.url }}/images/reactivex.png)

Se trata de otro de los pilares en los que sustenta [ReactiveUI](http://reactiveui.net/){:target="_blank"}. Tal y como hemos indicado <abbr title="Functional Reactive Programming">FRP</abbr> necesita de **Signals** para poder reaccionar a los cambios. 

[Reactive Extensions](http://reactivex.io/){:target="_blank"} proporciona herramientas para construir programas asíncronos y basados en eventos, y la clave es la interfaz `IObservable<T>`. Con **ReactiveX** podemos observar los eventos que va generando un usuario en nuestra intefaz, podemos incluso filtrarlos, y reaccionar a una determinada combinación de los mismos.

No es la intención de este post explicar exactamente como funciona **ReactiveX**. Si queréis una buena introducción a los mismos, podéis echar un ojo a la siguiente serie de [posts](http://rehansaeed.com/reactive-extensions-part1-replacing-events/){:target="_blank"}.

## ¿Qué es ReactiveUI?

![ReactiveUI]({{ site.url }}/images/reactiveui.png)

Se trata de un framework MVVM que utiliza **ReactiveX** dando lugar a un código elegante y testeable. Su objetivo final es hacer código entendible por humanos que da la casualidad que una máquina puede compilar.

Es compatible con otros frameworks como **Xamarin.Forms**, **MvvmCross**, etc y está disponible en todas las tecnologías de UI de .Net: Xamarin, UWP, WPF, etc.

Como sabéis un framework MVVM tiene que implementar la interfaz `INotifyPropertyChanged`. Lo que crea esta interfaz no es más que un evento entre nuestro `ViewModel` y la `Vista`, y tal y como hemos dicho anteriormente los eventos se pueden observar a través de `Observables`que proporcionan **ReactiveX**.

Esa es la clave de **ReactiveUI**. ¿Porqué no observar los eventos que lanzan nuestros `ViewModels` a la `Vista` y viceversa y reaccionar a ellos con una seríe de reglas definidas?

## ViewModels

Para poder empezar a utilizar **ReactiveUI** nuestros `ViewModels` deben de heredar de `ReactiveObject` el cual implementa por nosotros la interfaz `INotifyPropertyChanged`.

Otra de las características de **ReactiveUI** es que la inicialización de nuestras propiedades y `Commands`se realiza en el constructor de los `ViewModels`.

### WhenAny / WhenAnyValue

Permiten observar cambios en las propiedades de nuestro `ViewModel`, es decir, crearemos un `Observable` que capture los eventos de cambio de una propiedad de nuestro `ViewModel`. Lo mejor para entederlos en ver como se utilizan:

```c#
// in the viewmodel
this.WhenAnyValue(x => x.SearchText)
    .Where(x => !String.IsNullOrWhiteSpace(x))
    .Throttle(TimeSpan.FromSeconds(.25))
    .InvokeCommand(SearchCommand)
```

Vamos a ir viendo sentencia a sentencia para entender que estamos definiendo en este ejemplo:

- `this.WhenAnyValue(x => x.SearchText)`: observa cualquier cambio en la propiedad `SearchText`.
- `.Where(x => !String.IsNullOrWhiteSpace(x))`: asegurate que el valor de `SearchText` no es nulo ni espacio vacio.
- `.Throttle(TimeSpan.FromSeconds(.25))`: si ves que no existe ningún cambio durante un periodo de 0.25 segundos.
- `.InvokeCommand(SearchCommand)`: invoca el `Command` `SearchCommand`.

Como véis con 4 lineas de código hemos definido el comportamiento por ejemplo del serach bar de Google, que nos va mostrando sugerencias conforme escribimos. ¿Os imagináis como sería representar la misma lógica con el patrón tradicional de MVVM? Pues serían bastantes más lineas de código, eso seguro.

### Propiedades

Existen tres tipos de propiedades que podemos utilizar en nuestros `ViewModels` en **ReactiveUI**. Veamos cuales son y cuando en combeniente utilizarlas en cada caso:

#### Read-Write properties

Se trata de una propiedad que lanza un evento cada vez que es actualizada con un valor nuevo:

```c#
string name;
public string Name {
    get { return name; }
    set { this.RaiseAndSetIfChanged(ref name, value); }
}
```

#### Read-Only properties

Como los `Commands` debemos de inicializarlos siempre en el constructor de nuestro `ViewModel`, es un buen candidato para este tipo de propiedad:

```c#
public ReactiveCommand<Object> PostTweet 
{ 
    get; 
    protected set; 
}
```

#### Output properties

Este es un nuevo tipo de propiedad que no existe en otros frameworks de MVVM. Se trata de una propiedad que solo puede ser informada por un `Observable`. Veamos como se define y se utiliza:

```c#
protected ObservableAsPropertyHelper<bool> _IsLoading;
public bool IsLoading
{
    get { return _IsLoading.Value; }
}

this.SearchCharacters.IsExecuting
	.ToProperty(this, x => x.IsLoading, out _IsLoading);
```

Como véis un flag que indica que se está ejecutando una búsqueda es perfecto para este tipo de propiedades. Fijaos que sencillo es indicar a nuestra vista que tiene que mostrar un `ActivityIndicator`.

### ReactiveCommand

Es el tipo de `Command` que utilizaremos en **ReactiveUI**. Son el candidato perfecto para ser declarados como `Read-Only`properties y se utilizan de la siguiente forma:

```c#
public ReactiveCommand SearchCharacters { get; protected set; }

this.SearchCharacters = ReactiveCommand
    .CreateFromTask<string>((text) => LoadData(text));
```

#### ReactiveCommand Handle Exceptions

El manejo de excepciones en `ReactiveCommand` lo realizaremos a través de `ThrownExceptions`. Así se utiliza:

```c#
// Handle erros
this.SearchCharacters.ThrownExceptions
    .Subscribe ((obj) => this.LogException(obj, "Error while getting Marvel characters"));
```

#### ReactiveCommand IsExecuting

Otra de las características que oferce `ReactiveCommand` es poder definir un comportamiento mientras un `Command` se encuentra en ejecución. Para esto utlizamos `IsExecuting`:

```c#
// Loading flag
this.SearchCharacters.IsExecuting
	.ToProperty(this, x => x.IsLoading, out _IsLoading);
```

Hemos cubierto el comportamiento básico que deben tener nuestros `ViewModels` con **ReactiveUI**. Veamos ahora los Bindings con nuestras vistas.

## Binding

El `Binding` en **ReactiveUI** se realiza en el code-behind de nuestra Vista. Se que esto no les gustará demasiado a los amantes de XAML, pero tiene ciertas ventajas hacerlo así:

- El `Binidng` es el mismo para todas las plataformas que implementemos.
- Utilizamos expresiones para escribir los `Bindings`, y significa que si por casualidad cambiamos el nombre de una propiedad en nuestro `ViewModel` nuestro código no compilará y dará error, cosa que no ocurre con Bindings en archivos XAML hoy en día.

Los tipos de Bindings en **ReactiveUI** son los siguientes:

### OneWayBinding

```c#
var disp = this.OneWayBind(ViewModel, x => x.Name, x => x.Name.Text);
```

### TwoWay-Binding

```c#
this.Bind(ViewModel, x => x.Name, x => x.Name.Text);
```

### BindCommand

```c#
this.BindCommand(ViewModel, x => x.OkCommand, x => x.OkButton);
```

Una de las cosas de este tipo de Binding a través de expresiones, es que no hace falta que creemos `Converters`, sino que podemos indicar la conversión de nuestro Binding, con una acción adicional:

```c#
// Binding with a converter
this.OneWayBind(ViewModel,
                vm => vm.Thumbnail,
                cell => cell.imageThumbnail.Source,
                (string arg) => ImageSource.FromUri(new Uri(arg)));
```
En este ejemplo estamos transformando la url de una imagen en un `ImageSource` de una imagen en **Xamarin.Forms**, con una simple expresión sin necesidad de crear una nueva clase, crear una recurso estático en nuestro archivo XAML e indicar su uso cuando asignamos el Binding.

## WhenActivated

Como ya sabemos una de las principales causas de **Memmry leaks** durante el desarrollo de una app, son los eventos que quedan vivos entre dos objetos. **ReactiveUI** implementa una variante del patrón `IDisposable` que nos ayudará a que nuestros Bindings no generen **Memory leaks** haciendo uso del método `WhenActivated`:

```c#
this.WhenActivated((CompositeDisposable disposables) =>
{
    // Binding with a converter
    this.OneWayBind(ViewModel,
                    vm => vm.Thumbnail,
                    cell => cell.imageThumbnail.Source,
                    (string arg) => ImageSource.FromUri(new Uri(arg)))
        .DisposeWith(disposables);
    
    this.OneWayBind(ViewModel, 
                    vm => vm.Name, 
                    cell => cell.textName.Text)
        .DisposeWith(disposables);
});
```
Como véis cada uno de los Bindings es incluido en una colección `CompositeDisposable`. Eso nos asegura que cuando el <abbr title="Garbage Collector">GC</abbr> se ejecute sepa de antemano que los Bindings pueden ser liberados en memoría.

## Conclusión Parte 1

Hasta aquí los conceptos básicos de **ReactiveUI** necesarios para afrontar el desarrollo de una app con **Xamarin.Forms**. En el siguiente post veremos como integrar **ReactiveUI** con nuestra app revisando conceptos como el `Routing`, `ReactiveList`, etc.

Os dejo a continuación una seríe de recursos por si queréis seguir apliando conocimientos:

- [Reactive Extensions (RX)](http://rehansaeed.com/reactive-extensions-part1-replacing-events/){:target="_blank"}
- [Reactive UI Documentation](https://docs.reactiveui.net/en/){:target="_blank"}
- [Ejemplo App utilizando ReactiveUI y Xamarin.Forms](https://github.com/kentcb/WorkoutWotch){:target="_blank"}
- [Video: Why You Should Be Building Better Mobile Apps with Reactive Programming – Michael Stonis](https://www.youtube.com/watch?v=DYEbUF4xs1Q){:target="_blank"}
- [ReactiveUI v7.0.0 released](https://ghuntley.com/archive/2016/11/12/reactiveui-version-7-0-0-released/){:target="_blank"}

