---
layout: post
title: Introducción a ReactiveUI con Xamarin.Forms - Parte 2
categories: es tutorials
---

Esta es la segunda parte de la Serie [Introducción a ReactiveUI con Xamarin.Forms]({{ site.url }}/es/tutorials/introduccion-reactiveui-xamarin-forms.html). Si te lo perdiste, te recomiendo que le eches un vistazo, ya que aclara conceptos que utilizaremos en este post para seguir profundizando en el uso de [ReactiveUI](http://reactiveui.net/){:target="_blank"} con **Xamarin.Forms**.

Podéis encontrar la app de ejemplo de este post en el siguiente repositorio de [GitHub](https://github.com/ramonesteban78/MarvelFormsReactUI){:target="_blank"}.

Una de las cosas que no me gusta demasiado cuando estás utilizando un framework nuevo, es el setup que tienes que hacer inicialmente para empezar a trabajar con él, y da la casualidad que en la mayoría de los casos, no comprendes exactamente que estás haciendo durante la implementación de dicho setup. Este es el propósito de este segundo post, seguir aclarando conceptos de **ReactiveUI** cuando nos enfrentemos a una nueva implementación desde cero.

Utilizar **ReactiveUI** como framework MVVM de nuestra app, es prácticamente inmediato, pero si quieres aplicar **ViewModel first navigation**, es decir, ser capaz de realizar acciones de navegación desde nuestros ViewModels, es necesario realizar una serie de pasos extra para poder ponerla en marcha.

Antes de ver como empezar con el setup a realizar en una app en **Xamarin.Forms**, veamos algunas de las herramientas que vamos a utilizar.

## Splat

![Splat]({{ site.url }}/images/splat.png)

[Splat](https://github.com/paulcbetts/splat){:target="_blank"} es una librería (creada por [Paul Betts](https://twitter.com/paulcbetts)) que es utilizada bastante por la comunidad de **ReactiveUI** y **Xamarin** en general. La parte que nos interesa de esta librería es el `Service Locator` simple y flexible que ofrece.

Pero espera un momento, ¿no es `Service Locator` un anti-patrón? Si queréis ver una opinión distinta al respecto recomiendo mucho esta respuesta de [StackOverflow](http://stackoverflow.com/a/22795888/2188509){:target="_blank"}. Os dejo aquí el primer parrafo de la misma:

> If you define patterns as anti-patterns just because there are some situations where it does not fit, then YES it's an anti pattern. But with that reasoning all patterns would also be anti patterns.

En el desarrollo móvil multiplataforma, el tiempo de arranque de una app es fundamental, así como el consumo de memoria, algo que no tienen en cuenta muchos frameworks de **IoC/DI**, ya que en sus origenes fueron creados para tecnologías web. 

El **Service Locator** de **Splat** es rápido y no requiere de ningún setup inicial, como la mayoría de los **IoC**. Podéis ver la respuesta del propio **Paul Betts** [aquí](http://stackoverflow.com/a/26924067/2188509){:target="_blank"}, no tiene desperdicio.

### Recomendación de como usar Splat Service Locator

El registro de un servicio con **Splat** es bastante sencillo y como hemos dicho no requiere de ningún setup inicial:

```c#
Locator.CurrentMutable.RegisterLazySingleton(() => new MarvelApiService(), 
                                                typeof(IMarvelApiService));
```

La mejor forma de utilizar el **Service Locator** de **Splat** es resolviendo las dependencias en el constructor de nuestras clases de la siguiente forma:

```c#
public SearchViewModel (IMarvelApiService marvelService = null)
{
    _marvelService = marvelService ?? 
        Locator.Current.GetService<IMarvelApiService>();
}
```

De esta forma, haremos nuestras clases Test-Friendly, y no resolveremos dependencias donde no debemos.

## AppBootstrapper

`AppBootstrapper` es una clase que definimos en el directorio raiz de nuestro proyecto **Xamarin.Forms**. Para hacernos una idea, es donde realizaremos todo el setup inicial de nuestra app como registro de servicios que vamos a utilizar, registro de nuestras Vistas y ViewModels asociados, así como nuestro `Router` para poder realizar navación entre vistas desde nuestros ViewModels (ViewModel First navigation).

Veamos como se implementa e iremos analizando detalle a detalle:

```c#
public class AppBootstrapper : ReactiveObject, IScreen
{
    public AppBootstrapper()
    {
        // IScreen 
        Locator.CurrentMutable.RegisterConstant(this, typeof(IScreen));
        // Services
        Locator.CurrentMutable.RegisterLazySingleton(() => new MarvelApiService(), 
                                                        typeof(IMarvelApiService));
        // Views and ViewModels
        Locator.CurrentMutable.RegisterLazySingleton(() => new SearchView(), 
                                                        typeof(IViewFor<SearchViewModel>));
        Locator.CurrentMutable.RegisterLazySingleton(() => new DetailView(), 
                                                        typeof(IViewFor<DetailViewModel>));
        Locator.CurrentMutable.RegisterLazySingleton(() => new CharacterItemViewModel(), 
                                                        typeof(IViewFor<CharacterCell>));
        // Routing
        Router = new RoutingState();
        Router.NavigationStack.Add(new SearchViewModel());
    }

    public RoutingState Router { get; protected set; }

    public Page CreateMainPage()
    {
        return new RoutedViewHost();   
    }
}
```

- `IScreen`: Nuestra app debe de registrar la clase que se encargará de realizar las acciones de navegación. 

- `Router`: Gestiona el Stack de ViewModels y Views.

- `RoutedViewHost`: Observa el stack de ViewModels y devuelve la vista a cargar cuando realizamos una navegación. Se trata de la clase que ejecuta la navegación entre Vistas.

**Nota:** Esta forma de definir la navegación está teniendo en cuenta que solo existirá un stack de navegación global para toda la app. Normalmente este tipo de navegación no encaja en la mayoría de apps que realizaremos, pero es un buen uso inicial para entender los conceptos de routing mencionados antes. En el caso de que tuvieramos que introducir tipos de Navegación más complejos como `TabbedPage` o `MasterDetailPage`, esta solución no nos valdría. En futuros posts abordaré como implementar una navegación más compleja.

La inicialización de esta clase se realiza en la clase `App` de **Xamarin.Forms**:

```c#
public partial class App : Application
{
    public App()
    {
        InitializeComponent();

        var appBootstrapper = new AppBootstrapper();
        var navpage = (NavigationPage)appBootstrapper.CreateMainPage();
        navpage.BarBackgroundColor = Color.FromHex("#D32F2F");
        navpage.BarTextColor = Color.White;

        MainPage = navpage;
    }

    protected override void OnStart()
    {
        // Handle when your app starts
    }

    protected override void OnSleep()
    {
        // Handle when your app sleeps
    }

    protected override void OnResume()
    {
        // Handle when your app resumes
    }
}
```

## Views

Si os habéis fijado en la creación de la clase `AppBootstrapper`, estamos registrando nuestras vistas con los ViewModels de la siguiente forma:

```c#
// Views and ViewModels
Locator.CurrentMutable.RegisterLazySingleton(
    () => new SearchView(), 
    typeof(IViewFor<SearchViewModel>));
```

Lo que significa que nuestras vistas deben de implemenatar `IViewFor<T>` donde `T` es el tipo del ViewModel asociado a la vista.

Existe un paquete de Nuget de ReactiveUI específico de forms que podemos utilizar para facilitarnos esta tarea:  [reactiveui-xamforms](https://www.nuget.org/packages/reactiveui-xamforms/){:target="_blank"}. Este nuget ofrece una vista base que implementa `IViewFor<T>` para Pages, Views o Cells que tenemos disponibles en **Xamarin.Forms**. Este es el listado de implementaciones que ofrece:

![reactiveui-xamforms views]({{ site.url }}/images/reactiveui-xamfors-views.png)

Si optamos por esta opción hay que tener en cuenta el tipo de argumento que necesitan nuestros  XAML para hacerlo funcionar. Por ejemplo, en el caso de un `ContentPage` podríamos hacer que nuestra vista herede de `ReactiveContentPage`. Este sería la definición de nuestra vista entonces:

```c#
public partial class SearchView : ReactiveContentPage<SearchViewModel>
```
Y en el caso de nuestro XAML tendríamos que indicar el tipo de ViewModel que necesita nuestra vista para ser definida:

```xml
<rxui:ReactiveContentPage xmlns="http://xamarin.com/schemas/2014/forms" 
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml" 
    xmlns:rxui="clr-namespace:ReactiveUI.XamForms;assembly=ReactiveUI.XamForms"
    xmlns:vms="clr-namespace:MarvelFormsReactUI.ViewModels;assembly=MarvelFormsReactUI"
    x:Class="MarvelFormsReactUI.SearchView"
    x:TypeArguments="vms:SearchViewModel">
```

En el caso de que optemos por implementar `IViewFor` esta es la implemntación de la misma:

```c#
private MyViewModel _viewModel;
public MyViewModel ViewModel
{
    get { return _viewModel; }
    set
    {
        // Informamos el BindingContext de nuestra View
        this.BindingContext = value;
        _viewModel = value;
    }
}

object IViewFor.ViewModel
{
    get { return ViewModel; }
    set { ViewModel = (MyBotiquinViewModel)value; }
}
```

Está en vuestras manos decidir si queréis implementar vosotros mismos la interfaz `IViewFor` o bien optar por la opción de instalar el Nuget **reactiveui-xamforms**. 

## BaseViewModel

Por último, y antes de comenzar con la implementación, veremos la implementación base de la que heredarán todos nuestros ViewModels. Esta es la implementación:

```c#
public class ViewModelBase : ReactiveObject, 
                                IDisposable, 
                                IRoutableViewModel
{
    protected readonly CompositeDisposable disposables;

    public ViewModelBase(IScreen hostScreen = null)
    {
        disposables = new CompositeDisposable();

        HostScreen = hostScreen ?? Locator.Current.GetService<IScreen>();

    }

    protected void LogException(Exception ex, string errorMessage)
    {
        this.Log().Write(ex.Message, LogLevel.Error);
        this.Log().Write(ex.StackTrace, LogLevel.Error);
    }

    public void Dispose()
    {
        disposables?.Dispose();
    }

    public string UrlPathSegment
    {
        get;
        protected set;
    }

    public IScreen HostScreen
    {
        get;
        protected set;
    }
}
```

### ReactiveObject 

Como ya mencioné en el post anterior todos nuestros ViewModels deben de heredar de **ReactiveObject**.

### IRoutableViewModel

Es necesario implementar está interfaz para poder acceder a `IScreen` desde nuestros ViewModels. Recordemos que la clase `AppBootstrapper` es la que implementa la interfaz `IScreen` y por tanto la que controla la navegación de toda la aplicación. En el constructor de nuestro ViewModel base resolveremos `IScreen`:

```c#
public ViewModelBase(IScreen hostScreen = null)
{
    HostScreen = hostScreen ?? Locator.Current.GetService<IScreen>();
}
```
Y navegaremos desde nuestros ViewModels de la siguiente forma:
```c#
HostScreen.Router.Navigate.Execute(new DetailViewModel(item));
```

### CompositeDisposable

Otra de las acciones comunes en **ReactiveUI** es implementar la variante del patrón `IDisposable` en nuestro `ViewModelBase`. Esto nos permite liberar de memoria cada una de las reglas o comportamientos que definimos en nuestros ViewModels. Veamos la implementación paso a paso:

Primero definimos la colección `CompositeDisposable`:
```c#
protected readonly CompositeDisposable disposables;
```

Seguidamente implementamos `IDisposable` y en el método `Dispose` liberamos la colección completa:
```c#
public void Dispose()
{
    disposables?.Dispose();
}
```

De forma que cuando hacemos uso de **ReactiveUI** para definir reglas en nuestros ViewModels, solo tenemos que utilizar el método de extensión `DisposeWith` para añadirlas a la colección de Disposables:
```c#
// Selected Item and navigation
this.WhenAnyValue(x => x.SelectedItem)
    .Where(x => x != null)
    .Subscribe (x => NavigateToDetailPage(x))
    .DisposeWith(this.disposables);
```

### Log 

Por último, vamos a ver el sistema de **Log** elegido en esta app. Para ello haremos uso de la interfaz `ILogger` que nos proporciona **Splat**. 

Lo primero es implementar esta interfaz en cada una de las plataformas y registrarlas. Este es el ejemplo el caso de iOS:

```c#
public class Logger : ILogger
{
    public Logger()
    {
    }

    public LogLevel Level { get; set; }

    public void Write([Localizable(false)] string message, LogLevel logLevel)
    {
        Console.WriteLine($"[{DateTime.Now.ToLongTimeString()}]: {logLevel.ToString()}");
        Console.WriteLine($"{message}");
    }
}
```
Y la registramos en el `AppDelegate` de la siguiente forma:
```c#
Locator.CurrentMutable.RegisterLazySingleton(() => new Logger(), typeof(ILogger));
```
Una vez registrada la implementación de `ILogger` podemos hacer uso del método de extensión `Log` gracias a que la clase `RecativeObject` implementa la interfaz `IEnableLogger`.

```c#
this.Log().Write(ex.Message, LogLevel.Error);
```

## Conclusión

Hasta aquí llega la segunda parte de Introducción a ReactiveUI con Xamarin.Forms. Creo con este segundo post en combinación con el [primero]({{ site.url }}/es/tutorials/introduccion-reactiveui-xamarin-forms.html) son suficientes para afrontar una app de **Xamarin.Forms** con **ReactiveUI** entendiendo bien los conceptos en los que se basa.

No dudeis en preguntar cualquier duda que pudieráis tener.

Saludos