---
layout: post
title: Xamarin Forms 3 Presentación
categories: es news
---

# Intro

Mi nombre es Ramón Esteban y soy Xamarin Freelance developer que actualmente colabora con Bravent como Xamarin Technical Lead

Apuntaos mi twitter para todos aquellos que me quieran seguir, y si me seguís, siempre estoy tuiteando todo aquello relacionado con Xamarin, un poquito de ReactiveUI y espero que mucho más de F#, poco a poco.

Empecé trabajando con Xamarin allá por 2012 cuando aun se llamaba MonoTouch, por lo que la he visto evolucionar desde cero prácticamente.

# Xamarin.Forms

¿Cuantos de vosotros habéis oido hablar de Xamarin.Forms?

¿Cuantos de vosotros venís al Xamarin Dev Days a conocer la plataforma porque habeis oido hablar de ella pero no sabéis como funciona?

Supongo que muchos de vosotros ya conoceran el Framework. Vamos a ver un poquito que es lo que ofrece, pero esta charla se centrará sobre todo en las novedades que vienen en la próxima versión Xamarin.Forms 3.0.

# Xamarin + Xamarin.Forms

Existen 2 formas de afrontar una aplicación de Xamarin. La forma tradicional o bien Xamarin.Forms.

- Xamarin Classic o Tradicional puedes compartir la lógica de negocio de la app usando C#, pero la interfaz de usuario se implementa de forma nativa en cada plataforma. 

- Xamarin.Forms llega un paso más allá. Abstrae la capa de UI y nos permite incluso compartir gran parte de la interfaz de usuario de la aplicación haciendo uso de C# o XAML (ese lenguaje de marcado que tanto gusta en la comunidad .Net). Pero Xamarin.Forms no pinta por ejemplo un botón de la misma forma en cada plataforma, sino que renderiza a controles nativos de las mismas. Y si, una aplicación de Xamarin.Forms es 100% nativa.

## ¿Cuando elegir una opción u otra?

- - Xamarin.Forms: Aunque prácticamente a dia de hoy, se podría iniciar cualquier proyecto con Xamarin.Forms debido a la cantidad de recursos nativos que puedes utilizar, encuentro los siguientes puntos determinantes:

1. Que tu background sea Windows y ya hayas tenido experiencia con Silverlight, WPF o XAML en general. Te va a facilitar mucho la vida.
2. Proyectos cuya interfaz de usuario no sea demasiado compleja llevarla a cabo, aunque os tengo que decir que siempre, siempre, siempre vais a tener que recurrir a algún renderer nativo, y más si afrontais varias plataformas como target.

- - Xamarin Tradicional o Classic:

1. Cuando tu background como desarrollador venga de Android o iOS, puede que el impacto inicial con la plataforma sea menos doloroso
2. La interfaz de usuario es demasiado compleja como para llevarla a cabo con Xamarin.Forms. Muchas veces demasiados renderers en un proyecto, nos pueden llevar más tiempo que implementar vistas nativas.
3. No existe limitación ninguna en tu desarrollo. Si se puede hacer en nativo, se puede hacer con Xamarin.

En cualquier caso, el paso a Xamarin lleva consigo una curva de aprendizaje, pero quien dijo miedo! La alternativa es desarrollar en nativo en cada una de las plataformas, asi que ya estáis tardando.

# Waht's Included in Xamarin.Forms

Leer la diapositiva.

# Pages y Layouts

Se tratan del tipo de vistas y contenedores que podemos utilizar en Xamarin.Forms
- Pages: Las vas describiendo de una en una.
- Layouts: Los describes de uno en uno.

# Controls

Como he mencionado, Xamarin.Forms dispone de esta gran cantidad de controles pero son renderizados a su correspondiente control nativo de cada plataforma. Vamos que no es HTML.

# Xamarin.Forms Ecosystems

Existen ya terceras empresas que están haciendo controles de Xamarin.Forms muy profesionales. Caben destacar Syncfusion, yo he trabajado con ella en un proyecto de Xamarin.Android y la verdad es que es una maravilla.

# Syncfusion

Esta es la que más me gusta ya que tiene un community license. Yo por ser freelance tengo licencia gratuita, y como es gratis pues me mola más que ninguna.

# Infragistics

No la he probado

# Telerik UI

Estos siempre están en el meollo y seguro que los conoceis si venís del mundo de .Net.

# Native UI from Shared Code

Este es el ejemplo de una página de Login para las tres plataformas a partir de un archivo XAML. Como véis en cada plataforma los controles son renderizados a su correspondiente en nativo.

# Open source

Xamarin.Forms lleva un año de Open Source y desde entonces el desarrollo del producto ha sido increible.

# Features and Enhacements

Cabe destacar sobre todo el trabajo en performance. La diferencia en performance con respecto a las versiones iniciales es bastante considerable, sobre todo en Android.

Más cosas a destacar:

- Embedded NAtive Controls. Puedes escribir controles nativos directamente en XAML sin necesidad de crear custom renderers.
- .Netstandard support. Se trata de la converjencia de las librerías portables de .Net. En un futuro próximo, no nos hará falta lidiar con las distintas versiones de PCL tal y como ocurre ahora y existirá una librería standard de .Net portable.
- Fast renderers. Esta es la clave de la mejora de performance de Xamarin.Forms. Se trata de reducir el numero de capas de Layouts nativos que que por defecto añadia Xamarin.Forms al renderizar controles, sobre todo en Android.
- Mejoras en el performance de los ListViews, etc, etc, etc.

# Xamarin Forms Embedding

Se trata de reutilizar tus vistas de XAML en proyectos de Xamarin.Android, Xamarin.iOS o UWP nativos. De forma que las vistas que no sean complejas como los Acerca de, settings, etc, podemos implementarlas con Xamarin.Forms y el resto de vistas complejas las implementeremos con Xamarin Tradicional.

# Embedding 

Ahora mismo, funciona solo para ContentPages, nos deja utilizar el DependecyService y el MessageCenter. Lo que no creo que porten en este caso es el servicio de Navegación de Xamarin.Forms. 

Pero se está abriendo una puerta para que podamos hacer embedding incluso a proyectos nativos de Java, Objective-C, Swift, y quien sabe que más puede ofrecer esta herramienta.

Otra de las cosas que me gustaría mencionar pero que no viene en las diapositivas es XAML Standrad. Xamarin.Forms está involucrado en el proyecto, porque ya sabéis aquellos

# FlexLayout

Se trata de un Layout nuevo en Forms, basado en Flexbox (ni idea), para mi es como hacer float: right... o es float: left, ni idea.

# Multiplataforma de verdad de la buena

Esta si es la parte que más me gusta de todas las nuevas características, y se acerca poco a poco al concepto de multiplataforma total.

Vamos a poder reutilizar nuestro código de Xamarin.Forms para implementar apps en:
- macOS
- Linux GTK#
- WPF (Windows Presentation Foundation)

Solo falta que renderice a web y ya lo cerramos 100%.

# Xamarin Live Player

Si no habéis visto esto, es lo más de lo más, pero aun esta verde. Se trata de ejecutar tu app y que cuando haces cambios de código o bien de UI no tengas que volver a desplegar, tu app se actualiza on the fly.

Aun está verde verde, y para ejecutarlo necesitas el VS4Mac del Alpha Channel y yo ahora no puedo tomar el riesgo de instalarlo.

Seguramente Javier podrá mostrarnos el Xamrin Live Player rulando.

# Demo

Vamos a ver una app de Xamarin.Forms sencillita y también vamos a ver como crear una app para macOS con el mismo código.