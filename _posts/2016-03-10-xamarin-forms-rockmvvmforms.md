---
layout: post
title: RockMvvmForms, I know.. another mvvm framework for Xamarin.Forms but...
---

## Intro

- <a href="itimekeep://home">Open the app</a>
- <a href="https://itimekeep.app.link/addtime">https://itimekeep.app.link/addtime</a>
- <a href="https://itimekeep.app.link/mytime">https://itimekeep.app.link/mytime</a>
- <a href="https://itimekeep.app.link/timecard?id=I-30-15718-155715">https://itimekeep.app.link/timecard?id=I-30-15718-155715</a>
- <a href="https://itimekeep.app.link/timecard?id=1111">https://itimekeep.app.link/timecard?id=1111</a>
- <a href="https://itimekeep.app.link/settings">https://itimekeep.app.link/settings</a>
- <a href="https://itimekeep.app.link/settings">https://itimekeep.app.link/dashboard?periodmode=weekly&period=current</a>
- <a href="https://itimekeep.app.link/settings">https://itimekeep.app.link/dashboard?periodmode=weekly&period=previous</a>
- <a href="https://itimekeep.app.link/settings">https://itimekeep.app.link/dashboard?periodmode=monthly&period=previous</a>
- <a href="https://itimekeep.app.link/settings">https://itimekeep.app.link/dashboard?periodmode=monthly&period=current</a>
- <a href="https://itimekeep.app.link/settings">https://itimekeep.app.link/dashboard?periodmode=yearly&period=current</a>

I want to make an introduction to RockMvvmForms, a very simple framework I've created to develop our Xamarin.Forms projects.

![RockMvvmForms logo]({{ site.url }}/images/rockmvvmforms.png)

The general idea of this framework is to use Xamarin.Forms as it is designed and with the minimum over-engineering as possible. We can summarize the framework with the following three principles:

Not IoC is implemented. I'm using my own Service Locator implementation: RockServiceLocator.
Separate completaly the Views from the ViewModels.
Navigation happens in the ViewModel.

## Initial setup

Just install the nuget package [RockMvvmForms](https://www.nuget.org/packages/RockMvvmForms) in your PCL project.

Create 2 private functions to register services and Views in your Application Forms class as follows:

```c#
public partial class App : Application
{
    private IViewFactory _viewFactory;

    public App ()
    {
        InitializeComponent (); 

        // Register the services in the Forms Service Locator
        RegisterServices ();

        // Register the ViewModels and asociate them to the corresponding Views
        RegisterViews ();

        // The root page of your application
        MainPage = new RockNavigationPageService<FirstViewModel>().Create();
    }

    private void RegisterServices()
    {
        RockServiceLocator.Current.Register<IMarvelApiService, MarvelApiService> ();
    }

    private void RegisterViews()
    {
        ViewFactory.Register<FirstViewModel, FirstView> ();
        ViewFactory.Register<DetailViewModel, DetailView> ();
    }

    protected override void OnStart ()
    {
        // Handle when your app starts
    }

    protected override void OnSleep ()
    {
        // Handle when your app sleeps
    }

    protected override void OnResume ()
    {
        // Handle when your app resumes
    }
        
}
```

## ViewFactory

The ViewFactory is the repository for ViewModels and Views. This ViewFactory is registered in the DependencyService as follows:

```c#
private void RegisterViews()
{
     ViewFactory.Register<FirstViewModel, FirstView> ();
     ViewFactory.Register<DetailViewModel, DetailView> ();
}
```

I have avoided the registration using reflexion and name convetions just to have the flexibility to asign, for example, the same ViewModel to diffrent Views depending on the device, size screen, OS, etc.

## ViewModels

The ViewModels have to inherit from `ViewModelBase` class in order to use them using `RockMvvmForms`. The `ViewModelBase` offers the following capabilities:

- `Navigation` - Navigation Service implementation of Xamarin.Forms.

- `InitAsync` - You can override the `InitAsync` method of the `ViewModelBase`. This method is executed after the ViewModel constructor and before the View is displayed.

- `View_Appearing` - You can override appearing the Event of View in your ViewModels.

- `View_Disappering` - You can override the disappering event of the View. 

This is an example of how to create your ViewModels:

```c#
using System.Collections.Generic;
using System.Linq;
using System.Windows.Input;
using System.Threading.Tasks;
using Xamarin.Forms;
using RockMvvmForms;

namespace MarvelRockSample
{
    public class FirstViewModel : ViewModelBase
    {
        private readonly IMarvelApiService _marvelService;
        private bool _IsFirstLoad = true;
        public FirstViewModel ()
        {
            _marvelService = RockServiceLocator.Current.Get<IMarvelApiService>();
        }

        public override async Task InitAsync ()
        {
            await LoadData ();
        }

        private string _SearchText;

        public string SearchText {
            get {
                return _SearchText;
            }
            set {
                _SearchText = value;
                OnPropertyChanged ("SearchText");
            }
        }

        private List<CharacterItemViewModel> _CharacterList;

        public List<CharacterItemViewModel> CharacterList {
            get {
                return _CharacterList;
            }
            set {
                _CharacterList = value;
                OnPropertyChanged ("CharacterList");
            }
        }

        private ICommand _SearchByName;

        public ICommand SearchByName {
            get {
                return _SearchByName ?? (_SearchByName = new Command (
                    async () => await ExecuteSearchByNameCommand (),
                    ValidateSearchByNameCommand)); 
            }
        }

        private async Task ExecuteSearchByNameCommand ()
        {
            await LoadData (SearchText);
        }

        private bool ValidateSearchByNameCommand ()
        {
            return true;
        }

        public async Task LoadData (string filter = null, int limit = 0, int offset = 0)
        {
            IsBusy = true;

            var result = await _marvelService.GetCharacters (filter, limit, offset);


            if (result != null) {
                CharacterList = (from p in result.Results
                                select new CharacterItemViewModel () {
                        Id = p.Id,
                        Name = p.Name,
                        Thumbnail = p.Thumbnail.Path + "." + p.Thumbnail.Extension,
                        Description = p.Description
                }).ToList ();
            }

            IsBusy = false;

        }
    }
}
```

## Navigation in ViewModels

These are the Navigation methods you can use in your ViewModels:

- `PopAsync`
- `PopModalAsync`
- `PopToRootAsync`
- `PushAsync`
- `PushModalAsync`
- `DisplayAlert`
- `DisplayActionSheet`

And this is how to use it:

```c#
private ICommand _CharacterSelection;
 
public ICommand CharacterSelection {
    get {
        return _CharacterSelection ?? (_CharacterSelection = new Command<CharacterItemViewModel> (
            ExecuteCharacterSelectionCommand,
            ValidateCharacterSelectionCommand)); 
    }
}

private void ExecuteCharacterSelectionCommand (CharacterItemViewModel character)
{
    this.Navigation.PushAsync<DetailViewModel> (new DetailViewModel (character));
}

private bool ValidateCharacterSelectionCommand (CharacterItemViewModel character)
{
    return true;
}
```

This is an example of how to navigate to another ViewModel passing a parameter in the constructor.

## Rock Pages Services

In order to init the `MainPage`, and avoid accessing the ViewFactory manually, I've created a few Page Services for `ContentPages`, `NavigationPages` and `MasterDetailPages`. These are: `RockNavigationPageService`, `RockContentPageService` and `RockMasterDetailPageService`. The idea is very similar to the FreshMvvm Navigation Containers.

My intention is to create new Page Services for Tab pages and Caruossel pages and will be available in future versions.

Before I finish I would like to thank you the authors of the following resources that helped me to create RockMvvmForms:

- [FreshMvvm](https://github.com/rid00z/FreshMvvm)
- [ViewModel first navigation](http://adventuresinxamarinforms.com/2014/11/21/creating-a-xamarin-forms-app-part-6-view-model-first-navigation/)

Well, that's all I have for now. This is the link to the [GitHub project](https://github.com/ramonesteban78/RockMvvmForms) and fell free to collaborate if you like the idea and want to add missing functionality.

I hope you enjoyed the post!