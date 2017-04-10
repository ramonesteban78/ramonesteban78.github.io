---
layout: post
title: Xamarin.Forms y Mvvm
---

## Introducción

Debido al [webinar](https://info.microsoft.com/WE-EAD-WBNR-FY17-03Mar-31-Sesin2Xamarin.FormsyMvvm_305428_01Registration-ForminBody.html){:target="_blank"} que he dado recientemente, he decidido crear un post sobre el tema para todo aquel que le pueda interesar repasar MVVM utilizado en Xamarin.Forms.

La mayoría de las empresas dedicadas al desarrollo de aplicaciones de escritorio, Xamarin, Windows 10, etc, han adoptado el patrón MVVM como base del desarrollo sus aplicaciones. Por eso es muy importante adquirir los conceptos básicos del mismo.

## ¿Que es MVVM - Model View ViewModel?

Se trata de un patrón de desarrollo que nos permirte separar o desacoplar la interfaz de usuario del resto del código. Podemos dividir nuestra aplicación en tres capas:

![Flujo Mvvm]({{ site.url }}/images/flow-mvvm-2.png)

### Model

Representa la capa de datos y la lógica de negocio de nuestra app. También es denominado como objeto del dominio.

### View

Interfaz de usuario o Vistas de nuestra app. En Xamarin.Forms podemos crear interfaces de usuario con XAML o con C#. En nuestro caso utilizaremos XAML para diferenciar bien la interfaz de usuario de la lógica de presentación (ViewModel).

### ViewModel

Se trata de la lógica de presentación de nuestra la vista. Realiza la función de intermediario entre el Modelo y la Vista. El ViewModel contiene el estado de la vista y se comunica con ella a través de Data Binding, Commands y Notificaciones gracias a la interfaz `INotifyPropertyChanged`.

![Flujo Mvvm 2]({{ site.url }}/images/flow-mvvm-3.png)

Esta separación nos da ciertas ventajas:
- Separar el desarrollo de la interfaz de usuario del resto del código. Es decir podemos tener un equipo de diseño trabajando en la interfaz de usuario y a los programadores haciendo el resto de la aplicación.
- La lógica de presentación puede ser testeada con tests unitarios al estar desacoplada de nuestras vistas.
- Muy util cuando estamos desarrollando aplicaciones multiplataforma, ya que tanto la lógica de negocio, como la lógica de presentación es común a todas las plataformas.

## Bindings

Como hemos dicho la forma de comunicación entre el ViewModel y la Vista es a través de Data Binding y Commands. Esta comunicación es posible gracias a la interfaz `INotifyPropertyChanged`. Veamos un ejemplo de la implementación de esta interfaz:

```c#
public class MyViewModel : INotifyPropertyChanged
{
    public MyViewModel ()
    {
    }
    
    public event PropertyChangedEventHandler PropertyChanged;

    protected void RaisePropertyChanged([CallerMemberName] string propertyName = null)
    {
        if (PropertyChanged != null)
            PropertyChanged(this, 
                new PropertyChangedEventArgs(propertyName));
    }

    private string _Title;

    public string Title {
        get {
            return _Title;
        }
        set {
            _Title = value;
            RaisePropertyChanged();
        }
    }
}
```

Como podemos comprobar lo que realmente hace esta interfaz, es lanzar un evento cada vez que una propiedad cambia. Nuestra vista estará subscrita a estos eventos de cambio y eso es lo que llamamos Data Binding o Binding.

Un Binding relaciona 2 propiedades entre si de forma que se mantengan sincronizadas. Existen 2 tipos de Bindings. Para explicarlos y entenderlos mejor, imaginemos que queremos "Bindear" una propiedad `string Title` de nuestro ViewModel a la propiedad `Text` de un control [Entry](https://developer.xamarin.com/api/type/Xamarin.Forms.Entry/){:target="_blank"} de nuestra vista:

### One Way Binding 

Los cambios que realicemos en la propiedad `Title` de nuestro ViewModel se propagaran a la propiedad `Text` del control `Entry`, pero no se actualizará `Title` en el caso de que editemos la caja de texto.

```c#
// Código en nuestro ViewModel:
private string _Title;
public string Title {
    get {
        return _Title;
    }
    set {
        _Title = value;
        RaisePropertyChanged();
    }
}
```
```html
// Binding en nuestro archivo XAML
<Entry Text="{Binding Title}" />
```

### Two Way Binding

Las dos propiedades enlazadas tienen la capacidad de propagar los cambios. Es decir, si cambiamos la propiedad de nuestro ViewModel, se informará a la vista del cambio, y si editamos la caja de texto mandará el valor que escribamos a la propiedad `Title`.

```c#
// Código en nuestro ViewModel:
private string _Title;
public string Title {
    get {
        return _Title;
    }
    set {
        _Title = value;
        RaisePropertyChanged();
    }
}
```
```html
// Binding en nuestro archivo XAML
<Entry Text="{Binding Title, Mode=TwoWay}" />
```

![Data Binding]({{ site.url }}/images/data-binding.png)

## Commands

Ya que queremos que toda la lógica de presentación trasladarla a nuestros ViewModels, debemos incluso trasladar Eventos como por ejemplo sobre el click de un botón. Esta abstracción de eventos sobre controles en la vista es posible gracias a la interfaz `ICommand`.

La forma de enlazar un Command con el control de una vista es también a través de Binding:

```c#
// Definición de un Command en nuestro ViewModel
private ICommand _SearchByName;

public ICommand SearchByNameCommand {
    get {
        return _SearchByName ?? (_SearchByName = new Command (
            async () => await ExecuteSearchByNameCommand (),
            CanExecuteSearchByNameCommand)); 
    }
}

private async Task ExecuteSearchByNameCommand ()
{
    await LoadData (SearchText);
}

private bool CanExecuteSearchByNameCommand()
{
    return SearchText.Lenght > 0;
}
```
```html
// Command binding en nuestro archivo XAML
<SearchBar Text="{Binding SearchText}" 
SearchCommand="{Binding SearchByNameCommand}"></SearchBar> 
```

## Conceptos a revisar de MVVM para Xamarin.Forms

### Estructura básica de un proyecto MVVM-Xamarin.Forms

![Proyecto básico Xamarin.Forms]({{ site.url }}/images/mvvm-xamarinforms-solution.png)

En cuestión de proyectos podemos distinguir los siguientes:
- Core (XamFormsMarvel)
- Android (XamFormsMarvel.Droid)
- iOS (XamFormsMarvel.iOS)
- UWP (XamFormsMarvel.UWP)
- UITest (XamFormsMarvel.UITests) - Este es un proyecto opcional

Analizemos ahora en concreto el proyecto Core. Estas son las carpetas a las que tenemos dar importancia:
- Models - Nuestros modelos de datos
- ViewModels - Tendremos un ViewModel por Vista
- Views - Archivos XAML con la interfaz de usuario

### BaseViewModel

Una de las primeras acciones a tomar a la hora de empezar con el patrón Mvvm es crear nuestro clase base ViewModel. Esta clase será la encargada de implementar la interfaz `INotifyPropertyChanged`. Todos nuestros ViewModels heredaran de esta clase base.

Aunque la creación de una clase base no es obligatorio si que es recomendable ya que así evitamos implementar `INotifyPropertyChanged` en todos nuestros ViewModels.

```c#
public class BaseViewModel : INotifyPropertyChanged
{
    public BaseViewModel () { }
    public event PropertyChangedEventHandler PropertyChanged;
    protected void RaisePropertyChanged([CallerMemberName] string propertyName = null)
    {
        if (PropertyChanged != null)
            PropertyChanged(this, 
                new PropertyChangedEventArgs(propertyName));
    }
    private bool _IsBusy;
    public bool IsBusy {
        get {
            return _IsBusy;
        }
        set {
            _IsBusy = value;
            RaisePropertyChanged();
        }
    }
    private string _Title;
    public string Title {
        get {
            return _Title;
        }
        set {
            _Title = value;
            RaisePropertyChanged();
        }
    }
}
```

### Converters

Muchas veces el valor que recibimos de nuestro ViewModel no es suficiente para nuestra Vista y necesita de un tratamiento adicional. En este tipo de situaciones haremos uso de los Converters, que son clases auxiliares que efectuan una determinada acción sobre un Binding y que implementan la interfaz `IValueConverter`.

Veamos un ejemplo de implementación de un Converter y como se realiza el Binding en nuestra Vista:

```c#
public class DescriptionToImageValueConverter : IValueConverter
{
    public DescriptionToImageValueConverter()
    {
    }

    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        string strValue = value?.ToString();

        if (string.IsNullOrEmpty(strValue))
        {
            return ImageSource.FromFile("wrong");
        }
        else
        {
            return ImageSource.FromFile("ok");
        }
    }

    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        return null;
    }
}
```

En el caso de nuestra Vista XAML, los converters deben de ser declarados como recursos estáticos de nuestro ContentView o ContentPage, y después utilizarlos en el Binding:

```xml
<ContentView.Resources>
    <ResourceDictionary>
        <local:DescriptionToImageValueConverter x:Key="okWrongConverter"></local:DescriptionToImageValueConverter>
    </ResourceDictionary>
</ContentView.Resources>

<Image Source="{Binding Description, 
Converter={StaticResource okWrongConverter}}"></Image>
```

## Conclusión

Hemos revisado los conceptos básicos de MVVM y como aplicarlos en una aplicación Xamarin.Forms. Os dejo el enlace de [Github](https://github.com/ramonesteban78/XamFormsMarvel){:target="_blank"} con el código en el que está basado el post. Echad un vistazo para profundizar un poco más en los conceptos de MVVM.

## Framewroks existentes de Mvvm que podemos usar con Forms

- [FreshMvvm](http://www.michaelridland.com/xamarin/freshmvvm-mvvm-framework-designed-xamarin-forms/){:target="_blank"}
- [MvvmCross](https://github.com/MvvmCross/MvvmCross/tree/develop/MvvmCross-Forms){:target="_blank"}
- [MvvmLight](http://blog.galasoft.ch/posts/2014/07/using-xamarin-forms-with-mvvmlight/){:target="_blank"}
- [ReactiveUI](http://reactiveui.net/){:target="_blank"}
- Incluso yo tengo uno: [RockMvvmForms](https://github.com/ramonesteban78/RockMvvmForms){:target="_blank"}

## Últimos consejos y opiniones personales sobre MVVM

Aunque Mvvm es un patrón de desarrollo muy ventajoso, hay que tener cuidado en ciertas cosas. Lo que planteo aquí son mis propias conclusiones y opiniones depués de haber trabajado con MVVM durante años y en proyectos con cierta complejidad.

### Relación entre Vista y ViewModel

Es muy importante dejar claro que nuestro ViewModel no tenga conocimiento alguno de la Vista al que va a ser bindeado. Si tenemos claro este concepto durante el desarrollo, el mismo ViewModel puede servirnos para la vista de iPad, la de iPhone, la de Mac, la de UWP, etc. 

El contrario no aplica, la vista tiene la libertad de estar altamente acoplada a un ViewModel en concreto.

### Existe una tercera opción de Binding

La utilización de Mvvm muchas veces nos lleva al error de crear todas las propiedades de nuestro ViewModel ejecutando el `PropertyChanged` y lancen un evento informando a la vista. En muchas ocasiones nos encontraremos que no necesitamos que se propaguen cambios hacia la vista, sino que se tratan de propiedades que son unicamente `readonly`. Por ejemplo, imaginaos el título de una Vista el cual es solo informativo y no cambia, y por tanto no necesita cambios de ningún tipo. Lo trataríamos de la siguiente forma:

```c#
// Código en nuestro ViewModel:
public string Title { get; private set; }
```
```html
// Binding en nuestro archivo XAML
<Entry Text="Binding Title"/>
```

Reducir el número de Bindings o eventos entre propiedades de nuestro ViewModel y Vista, es bueno para el rendimiento de la app que estamos implementando.

### Control del estado de nuestra vista

Hay que procurar que el estado de nuestra vista siempre sea, en la medida de lo posible, a través de `Commands`, evitar el efecto rebote que puede provocar que el en `set` de una propiedad se actualice otra propiedad y asi sucesivamente. Esto hace que perdamos el control del estado de la vista.

### Messenger

Existe otra forma de comunicación entre la Vista y los ViewModels que es usando un patrón MessageBus. En Xamarin.Forms es `Messenger`. Aunque algunas veces no tenemos más remedio que hacerlo, debemos de procurar evitarlo.

### Code Behind or not Code Behind

La mayoría de los programadores que usan MVVM no les gusta nada tener código en la clase parcial de nuestro XAML (code-behind). Aunque esto es posible llevarlo a cabo, a través de `Triggers` o `Behaviours`, en mi opinión da lugar a un XAML que es dificil de mantener y entender en un primer vistazo. Yo personalmente prefiero utilizar el code-behind e implemntar en el acciones específicas de la Vista, como animaciones, posición de un scroll, etc. Esto da lugar a un XAML con prácticamente Binding y estilos más mantenible y conciso en un primer vistazo.