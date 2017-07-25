---
layout: post
title: Xamarin Forms for Mac
categories: es news
---

Una de las novedades que trae **Xamarin.Forms** en la versión de pre-release, es poder crear una app para macOS. Como siempre partiré del ejemplo que utlizo en mis charlas o para investigar un poco las novedades de **Xamarin.Forms** [XamFormsMarvel](https://github.com/ramonesteban78/XamFormsMarvel){:target="_blank"}.

## Actualizando paquetes a la versión pre-release de Xamarin.Forms

Si queréis empezar a experimentar un poco con Xamarin.Forms y mac, lo primero que debemos de hacer es actualizar los paquetes de Nuget de **Xamarin.Forms** a la última pre-realease que haya disponible.

## Añadir a la solución un proyecto Cocoa App

Añadimos un proyecto a nuestra solución que sea **Cocoa.App**, y tendremos que adaptarlo para que Xamarin.Forms funcione teniendo en cuenta lo siguiente:

- Aseguraos de tener instalado el framework **Xamarin.Mac**. Podéis descargar la última versión desde aqui: [Xamarin Downloads](https://store.xamarin.com/account/my/subscription/downloads)

- Añadimos el paquete de **Xamarin.Forms** pre-release que hemos utilizado en el resto de proyectos.

- Añadimos como refrencia el proyecto Core de **Xamarin.Forms** de nuestra solución.

- En el archivo **info.plist** del proyecto mac, eliminamos la entrada `NSMainStoryBoard`. Esto lo hacemos para indicar que las vistas no serán manejadas por ningún StoryBoard, sino que será **Xamarin.Forms** el que realice la carga de las distintas vistas.

- Modificamos la clase `Main.cs` para inicializar nuestro `AppDelegate`:

```c#
static class MainClass
{
    static void Main(string[] args)
    {
        NSApplication.Init();
        NSApplication.SharedApplication.Delegate = new AppDelegate();
        NSApplication.Main(args);
    }
}
```

## Modificaciones en el archivo AppDelegate

Veamos en detalle de cada uno de los pasos que debemos realizar para arrancar **Xamarin.Forms** desde nuestro `AppDelegate`:

- Hay que heredar la clase `AppDelegate` de `FormsApplicationDelegate`.

```c#
using AppKit;
using Foundation;
using Xamarin.Forms.Platform.MacOS;

namespace XamFormsMarvel.Mac
{
    [Register("AppDelegate")]
    public class AppDelegate : FormsApplicationDelegate
    {
        public AppDelegate()
        {
        }

        public override void DidFinishLaunching(NSNotification notification)
        {
            // Insert code here to initialize your application
        }

        public override void WillTerminate(NSNotification notification)
        {
            // Insert code here to tear down your application
        }
    }
}
```
- Definimos el tamaño de la pantalla que vamos a mostrar por defecto. Esto lo realizaremos en el constructor del `AppDelegate`:

```c#
NSWindow _window;
public AppDelegate()
{
    var style = NSWindowStyle.Closable | NSWindowStyle.Resizable | NSWindowStyle.Titled;

    var rect = new CoreGraphics.CGRect(200, 1000, 1024, 768);
    _window = new NSWindow(rect, style, NSBackingStore.Buffered, false);
    _window.TitleVisibility = NSWindowTitleVisibility.Hidden;
}
```

- Sobrescribimos la propiedad `MainWindow`, para hacer saber a **Xamarin.Forms** cual es la ventana principal:

```c#
public override NSWindow MainWindow
{
    get { return _window; }
}
```

- Por último, inicializamos **Xamarin.Forms** en el método `DidFinishLaunching`:

```c#
public override void DidFinishLaunching(NSNotification notification)
{
    Forms.Init();
    LoadApplication(new App());
    base.DidFinishLaunching(notification);

}
```

## XAML On Platform para Mac

En las nuevas versión de **Xamarin.Forms** el método `Device.OnPlatform` ha sido deprecado (o como se diga en español). Ahora la clase `Device` proporciona la propiedad `RuntimePlatform`, y si quisieramos hacer código condicional dependiendo de plataforma, incluyendo macOS lo deberemos de implementar con un `swicth` de l siguiente forma:

```c#
switch(Device.RuntimePlatform)
{
    case Device.UWP:
        listCharacters.ItemTemplate = new DataTemplate(() =>
        {
            var nativeCell = new NativeCell();
            nativeCell.SetBinding(NativeCell.NameProperty, "Name");
            nativeCell.SetBinding(NativeCell.ThumbnailProperty, "Thumbnail");

            return nativeCell;
        });
        break;
    case Device.macOS:
        searchBar.FontSize = 14;
        break;
}
```

Y en el caso de como aplicarlo a XAML, esta sería la nueva forma de implementarlo:

```xml
<Grid.RowDefinitions>
    <RowDefinition>
        <RowDefinition.Height>
            <OnPlatform x:TypeArguments="GridLength">
                <On Platform="iOS">44</On>
                <On Platform="Android">44</On>
                <On Platform="UWP">44</On>
                <On Platform="macOS">88</On>
            </OnPlatform>
        </RowDefinition.Height>
    </RowDefinition>
    <RowDefinition Height="*"></RowDefinition>
</Grid.RowDefinitions>
```

## DependdencyService

La utilización de DependencyService es exactamente igual que cualquier otra plataforma. Este sería por ejemplo la implementación de un DependencyService que abre una web en el navegador:

```c#
[assembly: Xamarin.Forms.Dependency (typeof (OpenWebService))]
namespace XamFormsMarvel.Mac.Services
{
	public class OpenWebService : IOpenWebService
	{
		public OpenWebService ()
		{
		}

		#region IOpenWebService implementation

		public void OpenUrl (string url)
		{
            NSWorkspace.SharedWorkspace.OpenUrl(new NSUrl(url));
		}

		#endregion
	}
}
```

## Conclusión

![Mac and Forms]({{ site.url }}/images/marvel-forms-mac.gif)

Con tan solo unos cuantos hacks he podido adaptar mi proyecto de prueba de **Xamarin.Forms** a una aplicación de macOS.

Hay que tener en cuenta que aun está en preview y que no podremos hacer maravillas con él, como utilizarlo en una app de producción, pero tiempo al tiempo.

No dudeis en probarlo. Espero vuestro feedback.

Saludos