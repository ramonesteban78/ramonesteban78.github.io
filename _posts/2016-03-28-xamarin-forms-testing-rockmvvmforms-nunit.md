---
layout: post
title: Testing Xamarin.Forms apps with RockMvvmForms and NUnit
---

![RockMvvmForms logo]({{ site.url }}/images/rockmvvmforms_logo.png)

This is my second post about [RockMvvmForms]({{ site.url }}/xamarin-forms-rockmvvmforms.html), a new and simple framework I did for my Xamarin.Forms projects.

The next logic move in the evolution of the framework is Testing. How do I test my Xamarin.Forms projects using `RockMvvmForms`?

![NUnit logo]({{ site.url }}/images/nunit.png)

If you have read [my previous post]({{ site.url }}/xamarin-forms-and-mvvm.html), one of the principles was to use `DependencyService` as a Service Locator in order to leave the framework as simple as possible. Well, after trying to create a `NUnit` Library Project to test one of my projects, I just realized that `DependencyService` requires Forms to be initialized in order to use it, which means that we need to call the method `Forms.Init()` from a Xamarin.Android or a Xamarin.iOS project. 

Xamarin provides you with Test project templates for Xamarin.iOS or Xamarin.Android, but it results in having to execute an app to run your unit tests. This could be a solution, but I prefer a NUnit Library Project to be able to execute the tests from the Unit Test runner in Xamarin Studio:

![NUnit test runner Xamarin Studio]({{ site.url }}/images/nunit2.png)

So the best option is avoid the use of the `DependencyService` to allocate my services and the `ViewFactory`. Instead we could use an IoC like `Autofac`, `Unity`, etc. or my own Service Locator.

## Add RockServiceLocator

I decided to add a new Service Locator just to leave the framewrok opened to add any other types of IoC containers, if someone preffers that option.

I needed to do a couple of changes to the [RockMvvmForms](https://www.nuget.org/packages/RockMvvmForms/){:target="_blank"} framework in order to simplify even more the initial setup:

Not allocate the `ViewFactory` in `DependencyService` or any other container option. So I changed the `ViewFactory` to a static class.
Not to use `DependencyService` to register my own services. Instead I will use the new `RockServiceLocator`.
This is the implementation of the `RockServiceLocator`:

```c#
namespace RockMvvmForms
{
    public class RockServiceLocator
    {
        private static RockServiceLocator instance;

        private RockServiceLocator ()
        {
            services = new Dictionary<Type, object> ();
        }

        public static RockServiceLocator Current
        {
            get { return instance ?? (instance = new RockServiceLocator()); }
        }

        /// <summary>
        /// List of services
        /// </summary>
        private readonly Dictionary<Type, object> services;

        /// <summary>
        /// Gets the first available service
        /// </summary>
        /// <typeparam name="T">Type of service to get</typeparam>
        /// <returns>First available service if there are any, otherwise null</returns>
        public T Get<T>() where T : class
        {
            object service = null;
            if (services.TryGetValue (typeof(T), out service))
                return service as T;
            return default(T);
        }

        /// <summary>
        /// Registers a service provider
        /// </summary>
        /// <typeparam name="T">Type of the service</typeparam>
        /// <param name="service">Service provider</param>
        public RockServiceLocator Register<T>(T service) where T : class
        {
            var type = typeof(T);
            if (this.services.ContainsKey(type))
            {
                this.services.Remove (type);
            }

            this.services.Add (type, service);

            return this;
        }

        public RockServiceLocator Register<T, TImpl>()
            where T : class
            where TImpl : class, T
        {
            var service = Activator.CreateInstance (typeof(TImpl)) as T;
            return Register<T>(service);
        }

    }
}
```

And this is how the initialization has changed in `RockMvvmForms`:

```c#
public partial class App : Application
{
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
}
```

Now that we have these changes to the framework, we will be able to create our Unit Tests.

## Add Test project

We will create then a NUnit Library Project:

![NUnit Xamarin Studio project template]({{ site.url }}/images/nunit3.png)

## Testing a Service

To test a Service we have to create our own mock with fake data or use the populr [Moq library](https://www.nuget.org/packages/Moq/){:target="_blank"} available in Nuget. And this is the resulting Test class:

```c#
namespace MarvelRockSample.Test
{
    [TestFixture ()]
    public class MarvelApiServiceTest
    {
        IMarvelApiService mock;

        [SetUp()]
        public void Init()
        {
            mock = new MockMarvelApiService ();        
        }

        [Test()]
        public async void Check_Marvel_APi_when_returns_data ()
        {
            var results = await mock.GetCharacters (string.Empty, 0, 0);

            Assert.IsNotNull (results);
            Assert.IsNotNull (results.Results);
            Assert.IsTrue (results.Results.Any());
        }

        [Test()]
        public async void Check_Marvel_APi_when_filter_the_data ()
        {
            var results = await mock.GetCharacters ("thor", 0, 0);

            Assert.IsNotNull (results);
            Assert.IsNotNull (results.Results);
            Assert.IsTrue (results.Results.Any());
        }

        [Test()]
        public async void Check_Marvel_APi_when_no_data_is_returned ()
        {
            var results = await mock.GetCharacters ("spiderman", 0, 0);

            Assert.IsFalse (results.Results.Any());
        }
    }
}
```

## Testing the ViewModel

I will put here just the testing of the `InitAsync` call of one of my ViewModels. This is the Test class using the Mock service already created

```c#
namespace MarvelRockSample.Test
{
    [TestFixture ()]
    public class FirstViewModelTest
    {
        FirstViewModel vm;

        [SetUp()]
        public void Init()
        {
            RegisterServices ();
            vm = new FirstViewModel ();
        }


        [Test()]
        public async void FirstViewModel_init_async_test ()
        {
            await vm.InitAsync ();

            Assert.IsTrue (string.IsNullOrEmpty(vm.SearchText));
            Assert.IsNotNull (vm.CharacterList);
            Assert.IsTrue (vm.CharacterList.Any ());
        }


        // Register services in the DependencyService Service Locator
        private void RegisterServices()
        {
            RockServiceLocator.Current.Register<IMarvelApiService, MockMarvelApiService>();
        }
    }
}
```

As you can see our day by day as developers is to identify possible issues and refactor our code in order to continue improving. `RockMvvmForms` is my own tool that I use to learn and improve and I hope you find these series of Posts useful.

I have updated [my previous post]({{ site.url }}/xamarin-forms-and-mvvm.html) of how to use `RockMvvmForms` taking into account the refactoring I've done to make the framework testeable.

You can find the whole implementation in my [Github account](https://github.com/ramonesteban78/RockMvvmForms){:target="_blank"}.

Thanks for visiting and enjoy!




