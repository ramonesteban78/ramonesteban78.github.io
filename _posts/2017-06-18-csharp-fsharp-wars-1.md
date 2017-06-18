---
layout: post
title: CSharp FSharp wars. Kata - Are they same?
categories: es tutorials 
---

Este año he decidido aprender un nuevo lenguaje: `F#`. La razón es intentar comprender un poco más el mundo de la programación funcional que está tanto al alza, y tengo que decir que conforme lo voy conociendo, más me sorprende la sencillez del lenguaje.

Una de las mejores fuentes para aprender F#, es sin duda la web [fsharpforfunandprofit.com](http://fsharpforfunandprofit.com/){:target="_blank"} del gran [Scott Wlaschin](https://twitter.com/ScottWlaschin){:target="_blank"}. Y adicionalmente, me gusta mucho prácticar lo que voy aprendiendo con unas cuantas Katas de la web [CodeWars](https://www.codewars.com/){:target="_blank"} (muy recomendable).

Debido a la poca documentación que existe de F# en español, he decidido crear una serie posts conforme voy aprediendo el lenguaje. Esto quiere decir que el código que muestre en la serie de posts, no tiene por que ser la mejor de las soluciones, por lo que se admiten sugerencías y mejoras. 

La idea de esta serie de posts es enfrentar C# y F# en distintas Katas de código y a partir de ahí ir explicando los conceptos de F# que usamos en cada una de ellas para ir aprendiendo el lengaje con ejemplos prácticos. 

## Kata - Are they Same?

La Kata `Are they same?` consiste en comparar dos arrays de enteros `a` y `b`, de forma que los elementos de `b`, deben de ser el cuadrado de los elementos de `a`. Los elementos de ambas listas no tienen porque tener el mismo orden dentro los arrays. 

Primero voy a poner la soción al problema implementandola con `C#` debido a que es el lenguaje con el que trabajo diariamente, y después explicaremos de forma detallada la misma solución pero con `F#`.

## Solución C\#

### Código

```c#
using System;
using System.Collections.Generic;
using System.Linq;

namespace CodeWarsCSharp
{
    public static class Ensure
    {
        public static bool AreNotNullAndSameLength(IEnumerable<int> list1, IEnumerable<int> list2)
        {
            if (list1 == null || list2 == null)
                return false;
            return list1.Count() == list2.Count(); 
        }
    }

    public class AreTheySame
    {
        public static bool comp(int[] a, int[] b)
        {
            if (Ensure.AreNotNullAndSameLength(a, b))
            {
                var aOrdered = a.Select(x => x*x).OrderBy(x => x);
                var bOrdered = b.OrderBy(x => x);
                return aOrdered.SequenceEqual(bOrdered);
            }
            return false;
        }
    }
}
```

Como véis he utilizado `Linq` para dar lugar a un código más entendible. La idea es transformar el array `a` calculando el cuadrado de cada uno de sus elementos y después ordenando ambas listas para poder compararlas con `SequenceEqual`.

He creado además una clase estática `Ensure` y un método que comprueba que las listas no sean nulas y que contengan la misma cantidad de elementos.


## Solución F\#

### Código

```c#
module AreTheySame = 
    let comp a b = 
     let newa = a |> List.map (fun x -> x*x) |> List.sort
     let newb = b |> List.sort
     newa = newb
```

En un principio, ambas soluciones son muy parecidas, pero a primera vista cabe destacar la simplicidad del código de `F#`. Voy a ir explicando en detalle esta implementación.

### Definición de una función en F\#

En F# no es necesario utilizar paréntesis como en C# para enumerar los parámetros de una función separados por comas, esta sintaxis se utliza en F# para referirnos a los `Tuples`. De forma resumida, está es la definición general de una función en F#:

```c#
let nombreFuncion param1 param2 ...
```

Pero además F# te permite pasar funciones como parámetros de otras funciones, de forma similar a C# cuando utilizamos `delegates`, `Actions` o `Func` como parámetros, pero sin esa complejidad de sintaxis. Es algo natural y embedido en el lenguaje.

### Type inference

Aunque en F# no indiquemos el tipo de los parámetros en la definición de la función, se trata de un lenguaje altamente tipado y el compilador nos evaluará los tipos de las variables según el uso que hagamos de ellas dentro de nuestras funciones. Esto es lo que se conoce como `type inference`.

Para saber como evaulua el compilador nuestra función, haremos uso del `F# Interactive`.

### F\# interactive y definicón de funciones

![F# Interactive]({{ site.url }}/images/fsharpinteractive.png)

`F# interactive` es una herramienta nos permite evaluar nuestro código conforme lo vamos implementando. Está disponible en cualquier IDE que permita desarrollo con F#. En el caso de nuestra función, esta sería la firma de la misma:

```c#
val comp : a:int list -> b:int list -> bool
```

Esta sería la traducción de lo que nos muestra F# interactive: La función `comp` recibe como parámetro una lista de `int` **a** y otra lista de `int` **b** y devuelve un `bool`. Como truco rápido, cuando veamos la definición de una función y queramos saber el número de parámetros que contiene, contad el número de veces que aparece el símbolo `->`, es un truco cutre, pero no falla ;).

Es muy importante entender las firmas de las funciones de F# para poder trabajar de forma adecuada con el lenguaje. Si queréis conocer más al respecto echad un ojo a este post: [F# - Functions signatures](https://fsharpforfunandprofit.com/posts/function-signatures/){:target="_blank"}

Por último, si utilizáis Visual Studio Code con la extensión Ionide para F#, te mostrará como cabecera la firma de tu función sin necesidad de utilizar F# Interactive. Super útil.

![VSCode Ionide]({{ site.url }}/images/vscodeionide.png)

### Pipeline operator |>

Se trata de una función en `F#` que es a la vez muy simple, pero muy potente. Esta es su definición:

```c#
let (|>) x f = f x
```

Si os dáis cuenta lo que hace es alterar el orden de una función con su parámetro. Es decir, partimos de un parámetro y aplicamos una determinada función al mismo. 

![Pipeline operator]({{ site.url }}/images/pipe.png)

Veamos como lo hemos utilizado en el ejemplo de AreTheySame:

```c#
let newa = a |> List.map (fun x -> x*x) |> List.sort
```
El nuevo valor `newa` es el resultado de partir de la lista `a` aplicarle el cuadrado a cada uno de sus elementos con `List.map (fun x -> x*x)` y después la ordenamos con `List.sort`.

Como véis el operador `pipe |>` podemos crear estructuras complejas de alteración de datos a partir de composición de funciones de forma simple.

### List

Otro de los cambios radicales entre programación imperativa y programación funcional son los flujos que siguen nuestro código.

Si queremos aplicar conceptos de programación funcional de forma correcta con F#, empezad aplicando estos dos conceptos sencillos de entender:

- Utilizad `Pattern Matching` en vez de `if-then-else`
- Utilizad los metodos de extensión de `List` en vez de `loops`

Nosotros estamos utilizando la segunda opción para evitar `loops`. Como véis en la solución de `C#` implementamos algo similar con `Linq`, una clara aportación de la programación funcional a C#. De echo, hemos aplicado programación funcional a la solución de C#. Solo imaginaros como sería hacerlo con `loops` sino tuvieramos `Linq`.

## Tests

Estos son los tests unitarios para ambas soluciones:

### C\#

```c#
[Fact]
public void AreTheySameTest()
{
    int[] a = new int[] { 121, 144, 19, 161, 19, 144, 19, 11 };
    int[] b = new int[] { 11 * 11, 121 * 121, 144 * 144, 19 * 19, 161 * 161, 19 * 19, 144 * 144, 19 * 19 };
    bool r = AreTheySame.comp(a, b); // True
    Assert.True(r);
}
```

### F\#

```c#
module Tests =
 [<Fact>]
 let ``AreTheySameTest``() = 
     let a1 = [121; 144; 19; 161; 19; 144; 19; 11]
     let a2 = [11*11; 121*121; 144*144; 19*19; 161*161; 19*19; 144*144; 19*19]
     Assert.True(AreTheySame.comp(a1,a2))
     let a3 = [121; 144; 19; 161; 19; 144; 19; 11]
     let a4 = [11*21; 121*121; 144*144; 19*19; 161*161; 19*19; 144*144; 19*19]
     Assert.False(AreTheySame.comp(a3,a4))
```

## Conclusión

En esta primera Kata hemos revisado tres conceptos fundamentales de `F#` que son la definición de funciones, el `Pipeline operator |>` y como utilizar `List` para evitar `loops`. Espero que os haya despertado el gusanillo por la programación funcional y F# (aunque sea un poquito).

Intentaré seguir creando esta serie de posts (si el tiempo me lo permite) y si véis que existen mejores formas de conseguir lo mismo (tanto C# como F#), o si tenéis dudas, o si creéis que alguno de los conceptos que he intentado explicar no están del todo claros o he utlizado una forma incorrecta de definirlos (just learning man), no dudéis en dejar vuestros comentarios. 

Por último os dejo aquí el proyecto de ejemplo en mi cuenta de [GitHub](https://github.com/ramonesteban78/CodeWarsCSharp){:target="_blank"}

## Recursos

Os dejo aquí una seríe de artículos que podéis ver donde se profundiza en los conceptos de F# que hemos visto en este ejemplo:

- [F# - Defining functions](https://fsharpforfunandprofit.com/posts/defining-functions/){:target="_blank"}
- [F# - Functions signatures](https://fsharpforfunandprofit.com/posts/function-signatures/){:target="_blank"}
- [F# - Understanding type inference](https://fsharpforfunandprofit.com/posts/type-inference/){:target="_blank"}
- [F# - Function associativity and composition](https://fsharpforfunandprofit.com/posts/function-composition/){:target="_blank"}
- [F# - Control flow expressions And how to avoid using them](https://fsharpforfunandprofit.com/posts/control-flow-expressions/){:target="_blank"}