---
layout: post
title: Xamarin.Forms and ReactiveUI adventures
categories: tutorials
---

## Intro

Taking advantage that I have new blog, I decided to create a new post series of study and investigation of how to
create an app using Xamarin.Forms and ReactiveUI.

ReactiveUI is a MVVM framework based on Reactive Extensions that will change your point of view of how to implement
a MVVM application.

I recently did a [talk](http://slides.com/ramonesteban/xamarinreactiveui#/){:target="_blank"}, (here in Málaga - Spain) of an introduction to ReactiveUI, but I was wondering how much effort will take to create a real app using ReactiveUI.

### Why Xamarin.Forms

I choose Xamarin.Forms in order to reduce time designing the UI and
put all the effort on understand how ReactiveUI works and how to use it correctly when we develop an application.

Antother important reason is that ReactiveUI with Forms allows ViewModel First navigation. 
If you want to know a little bit about it have a look to the following stack-overflow [thread](https://stackoverflow.com/questions/26898381/reactiveui-view-viewmodel-injection-and-di-in-general){:target="_blank"}. The idea in general is to be able to control the navigation of the app in the ViewModels.

## Initial Setup

The idea of the setups is expense the less time possible preparing them before start executing code. These are the things we need to take into account

- **Splat**: ReactiveUI uses the Service Locator of this library to associate ViewModels with Views.
- AppBootstraper class that implements IScreen.
- Router
- RouteViewHost

## Using Observable instead of Task

We know that the Task based world is huge, but if you want to use the whole power of Reactive Extensions
you have to embrace Observables. Move from one world to another is hard, but it would open your mind
to another approach of how to program.

For example. Suppose we want to display a simple message to the user. In ReactiveUI, alerts or user interactions are exposed in the ViewModel using `Interactions`, you can find more info about it [here](https://docs.reactiveui.net/en/user-guide/interactions/){:target="_blank"}

```c#
// Tasks
// Wait user interaction
var result = await AccessCamaraNotAllowed.Handle(Unit.Default).ToTask();
// Execute an action after the interaction
if (result)
    CrossPermissions.Current.OpenAppSettings();


// Obserbables
// Same code than the previous one but using Observables
AccessCamaraNotAllowed
    .Handle(Unit.Default)
    .ObserveOn(RxApp.MainThreadScheduler)
    .Subscribe((result) => { if (result) { CrossPermissions.Current.OpenAppSettings(); } });
```
## Load data in the constructor through ReactiveCommand

To load data when a ViewModel is loaded, we will use a ReactiveCommand. The exeuction of the command will be made in the view like this:

```c#
this.WhenAnyValue(x => x.ViewModel.LoadItems)
    .SelectMany(x => x.ExecuteAsync())
    .Subscribe();
```

More info [here](http://codereview.stackexchange.com/questions/74642/a-viewmodel-using-reactiveui-6-that-loads-and-sends-data){:target="_blank"}
