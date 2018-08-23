---
layout: post
title: Elmish Xamarin Forms - Parte 1
categories: es tutorials
---

## Introducción

Este último año he estado aprendiendo en mi tiempo libre F#. El lenguaje es una maravilla y si no habéis visto nada aun os recomiendo como primer vistazo el post [Aprendiendo fsharp - Parte 1. Kata - Are they same?]({{ site.url }}/es/tutorials/learning-fsharp-1.html).

A pesar de que he disfrutado enormemente, siempre estaba con la inquietud de como podría utlizar F# para desarrollar aplicaciones móviles. Sabía desde el principio que Xamarin soporta F#, pero no terminaba de gustarme, ya que al final terminaba haciendo una app como la hubiera hecho con C#, pero sustituyendo el lenguaje. *Not good...*

Aunque muchos consideran una ventaja que F# te permita aplicar programación orientada a objetos, para mi es algo que estropea el lenguaje, ya que F# brilla cuando aplicas principios de programación funcional. Al menos, es verdad, que F# te lo pone dificil, ya que es *Functional-First*, todo inmutable por defecto, es decir, si quieres una variable que guarde estado, tienes que indicarlo explicitamente en el código con `mutable`, y si encima empiezas a con herencias y demás... *good bye functional programming...*

Pues bien, en mi busqueda de como aplicar F# de forma funcional en Xamarin surgió [Elmish.XamarinForms](https://github.com/fsprojects/Elmish.XamarinForms){:target="_blank"}. 

## ¿Qué es Elmish?

Muchos de vosotros habréis oido hablar de [ELM](https://guide.elm-lang.org/){:target="_blank"} supongo. Se trata de un *lenguaje funcional que compila a Javascript*. Lo más interesante de **ELM** es su arquitectura, muy similar a **React.js**, pero con un lenguaje 100% funcional y fuertemente tipado al igual que F#.

Pues bien **Elmish** es la adaptación a **F#** de ELM, compartiendo sus propios principios de arquitectura. 

El objetivo de este Post inicial sobre Elmish, es explicar los conceptos básicos de la arquitectura, así como los recursos de F# que utilizamos para implementarla. Para ello, tengo un repo de ejemplo que podéis clonar y juguetear con él para afianzar conceptos: [MarvelFormsElmish](https://github.com/ramonesteban78/MarvelFormsElmish){:target="_blank"}.

## ELM arquitecture

Se trata de una arquitectura para construir nuestro front-end de una forma funcional. Todos nuestras *vistas o componentes* implementarán generalmente la siguiente estructura:

```c#
// Modelo que representa el estado de nuestro elemento visual
type Model = {
    property1 : string;
    ...
}

// Se tratan de los mensajes que entregue la vista
type Msg =
    | Action1
    | ...

// Estodo inicial de nuestro elemento visual
// unit -> Model * Cmd<Msg>
let init() =
    ...

// Función que cambia el estado a partir de un determinado mensaje recibido
// msg:Msg -> model:Model -> Model * Cmd<Msg>
let update (msg:Msg) (model:Model) =
    ...

// Función que devuelve el elemento visual 
// model:Model -> dispatch:(Msg -> unit) -> VisualElement
let view (model: Model) dispatch =
    ...

```

De forma visual, podríamos representar la arquitectura de la siguiente forma:

![Elmish diagram]({{ site.url }}/images/elmish-diagram.png)

Así de primeras, puede ser que no se entiendan los conceptos por defecto, pero veamos cada elemento de la misma en detalle, para intentar aclarar alguno de ellos.

Aunque no entendáis el código que iré mostrando aquí inicialmente, no os preocupeis, voy a preparar un segundo post explicándolo todo correctamente. Lo importante en esta primera parte, es asimilar de forma correcta los conceptos.

### Model 
Representa el estado de nuestra vista o componente. En el caso de F#, se representará como un *Record Type*:

```c#
type Model = {
    listOfHeroes : Character.Model list option;
    searchText : string;
    searchingForHeroes : bool;
}
```

Los *types* de F# podrían definirse como una estructura de datos inmutables. Existen varias formas de definir *types* en F#, y los *Record Types* es una de ellas. Podéis investigar un poco más sobre ellos en el siguiente enlace: [Record Types](https://fsharpforfunandprofit.com/posts/records/){:target="_blank"}.

### Messages
Representa las diferentes acciones que pueden hacer cambiar de estado a nuestra vista o componente. Los *mensajes* están representados en F# por otro tipo de *types* denominados *discriminated unions*:

```c#
type Msg =
    | CharactersLoaded of Character.Model list
    | ExecuteSearch of string
    | SelectedHero of Characters.Msg 
```

Se tratan de *types* cuyo valor es una de las opciones que ofrecemos. Este tipo de *types* son perfectos para diferenciar los posibles cambios de estado. Podéis encontrar más información sobre ellos aquí: [Discriminated Unions](https://fsharpforfunandprofit.com/posts/discriminated-unions/){:target="_blank"}.

### View 
Función, que a partir del estado actual (Model), devuelve siempre un nuevo *UI layout/content*. Es decir, convertimos nuestras vistas o componentes en elementos de interfaz gráfica inmutables. Este sería un ejemplo de una implementación de la función *view* relativa al modelo anterior:

```c#
// model:Model -> dispatch:(Msg -> unit) -> VisualElement
let view (model: Model) dispatch = 
    View.ContentPage(
        title="Marvel Heroes",
        content=View.Grid(
            rowdefs=["auto"; "*"],
            rowSpacing = 0.0,
            children=[
                (SearchBar.view dispatch ExecuteSearch).GridRow(0)
                (Characters.view model.listOfHeroes (SelectedHero >> dispatch)).GridRow(1)
                (LoadingControl.view model.searchingForHeroes).GridRow(0).GridRowSpan(2)
            ]))
```

XamarinForms.Elmish utiliza siempre los métodos de extensión de *View* para generar nuestros *VisualElements*. Esto es en realidad es como JSX con React.js por si buscais una comparación.

#### Dispatch function

A la hora de trabajar con F#, es muy importante tener clara la definición o la firma de nuestras funciones. Aquí tenéis más información al respecto: [Functions Signatures](https://fsharpforfunandprofit.com/posts/function-signatures/){:target="_blank"}.

```c#
model:Model -> dispatch:(Msg -> unit) -> VisualElement
```
Si nos fijamos en la definición de la función *view*, esta acepta como segundo parámetro, una función *dispatch*. Esta función, es la encargada de comunicar al sistema de que una acción ha sido ejecutada por parte del usuario. 

Por ejemplo, cuando el usuario seleccione un heroe de nuestra lista *dispatch* emitirá un mensaje de tipo `SelectedHero`. Por simplificarlo para que se entienda, los eventos de nuestros elementos visuales, llamarán a la función *dispatch* pasando como parámetro un determinado mensaje `Msg`.

### Update 
Se trata de la función que se ejecutará siempre que el estado de nuestra vista o componente deba de cambiar. Recibe como parámetros un *Message* que indica la acción a tener en cuenta para cambiar el estado y el estado actual (Model) de nuestra vista o componente:

```c#
// msg:Msg -> model:Model -> Model * Cmd<Msg>
let update msg model =
    match msg with
    | CharactersLoaded chars -> 
       { model with listOfHeroes = Some chars; searchingForHeroes = false }, Cmd.none
    | ExecuteSearch text -> 
       { model with listOfHeroes = None; searchText = text; searchingForHeroes = true }, Cmd.ofAsyncMsg(loadCharacters text)
    | SelectedHero msg ->
        model, Cmd.none
```

De nuevo haciendo énfasis la firma de funciones en F#, podemos ver que la función *update* devuelve un *tuple* `Model * Cmd<Msg>`, donde Model es el nuevo estado después de haber sido procesado un *Message*, y `Cmd<Msg>` se trata de un *Command* de tipo *Msg*.

#### Commands

Como simplificación, *Commands* los utilizaremos, por ejemplo, para propagar un nuevo mensaje a través de una llamada asincrona. Veamos el ejemplo del código anterior dentro de la función *update*:

```c#
| ExecuteSearch text -> 
       { model with listOfHeroes = None; searchText = text; searchingForHeroes = true }, Cmd.ofAsyncMsg(loadCharacters text)
```

Cuando el usuario realiza una búsqueda, se ejecuta la función *update* para modificar el estado de nuestro componente visual, el nuevo estado esta definido en esta sentencía:

```c#
{ model with listOfHeroes = None; searchText = text; searchingForHeroes = true }
```

En este punto, la función *view* se invoca con el nuevo estado proporcionado por la función *update*, y al mismo tiempo que actualizamos el nuevo estado, estamos devolviendo un *Command*: 

```c#
Cmd.ofAsyncMsg(loadCharacters text)
```

Este *Command* ejecutará la función `loadCharacters` con el nuevo término de búsqueda, y cuando la operación asíncrona finalice, devolverá un nuevo *Message* que contiene la nueva lista de heroes que mostraremnos:

```c#
let private loadCharacters text =
    async {
        let! response = MarvelApi.getCharacters text 0 0
        let marvelData = JsonConvert.DeserializeObject<MarvelTypes.MarvelApiResult<Character.Model>>(response)
        return CharactersLoaded marvelData.data.results 
    }
```

Cuando el nuevo mensaje `CharactersLoaded` es emitido por nuestro *Command*, la función *update* se vuelve a ejecutar para actualizar el estado y por tanto invocar de nuevo la función *view*. 

Como resumen podemos "definir" los *Commands* como la herramienta para generar *Messages* adicionales cuando se procesa un nuevo estado de nuestra vista. 

### Init

Por último, aunque es lo primero que se ejecuta, tenemos la función *init* que se encarga de definir el estado inicial de nuestra vista o componente. Si nos fijamos en la firma de la función vemos no recibe ningún parámetro de entrada, pero si devuelve lo mismo que la función *update* que hemos visto, dando lugar al incio del ciclo de vida de nuestra vista:

```c#
// unit -> Model * Cmd<Msg>
let init() = 
    { listOfHeroes = None; searchText = ""; searchingForHeroes = true; }, Cmd.ofAsyncMsg(loadCharacters "")
```

## Resumen

Después de haber visto en detalle los diferentes elementos de arquitectura de XamarinForms.Elmish, echad un ojo de nuevo al diagrama visual de la misma:

![Elmish diagram]({{ site.url }}/images/elmish-diagram.png)

Si la véis al estilo de Neo en Matrix: "Ya se Kung Foo", es que he triunfado transmitiendo los conceptos. Sino, podéis plantearme cualquier duda que tengáis que estaré encantado de contestarlas. También comentar que no soy tampoco, como quien dice, un experto en la matería, por lo que si véis que alguno de los conceptos no están definidos de forma apropiada, hacedmelo saber e iré actualizando el contenido sin problemas. Sigo en proceso de aprendizaje...

Os dejo por último algunos enlaces adicionales para seguir profundizando en la materia:

- [F# Functional App Development, using Xamarin.Forms](https://fsprojects.github.io/Elmish.XamarinForms/index.html#getting=started){:target="_blank"}
- [Fable applications following "model view update" architecture](https://elmish.github.io/){:target="_blank"}

Saludos