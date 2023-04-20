---
layout: single
title: "Using Reactive UI in your .NET MAUI app"
date: 2023-04-25 00:03:00
tags: ["ReactiveUI", "Rx", ".NET MAUI"]
slug: "rxui-maui-101"
---

One of my favourite MVVM framework libraries for C# is Reactive UI. But is it ready for prime time when developing a .NET MAUI app? Spoiler: It is. So let's look at how to get Reactive UI setup in a .NET MAUI app and how the basics are implemented in Reactive UI. We are talking bindings, commands and wiring up the dependency injection.

<!-- expand -->

We will rewrite the Hello World .NET MAUI project using Reactive UI in this post. Even add a little timer, aka scheduler sample. I assume you are familiar with the basic MVVM concept. If not, there is an introduction by Microsoft [here](https://learn.microsoft.com/en-us/dotnet/architecture/maui/mvvm?WT.mc_id=AZ-MVP-5003494).

## Setup

After creating a new .NET MAUI project. Add the following NuGet packages:

- Reactive UI .NET MAUI 
- Reactive UI Fody

Now we can start pushing the logic from the View into the ViewModel.

## Writing a ViewModel

When using Reactive UI the View Model inherits from `ReactiveObject`. To move the basic functionality of the Hello World application, we will implement a property that we will bind to the button text. A counter value to keep track of the count. And a Command property to handle when the button is selected. Let's start with the properties which afterwards can be used in the View via binding:

```c#
public class MainViewModel : ReactiveObject, IActivatableViewModel
{
    public MainViewModel()
    {
        // Configuration of CounterButtonText and ButtonClickedCommand
    }
    
    [Reactive] public int Count { get; set; }
    [ObservableAsProperty] public string CounterButtonText { get; }
    public ReactiveCommand<Unit,Unit> ButtonClickedCommand { get; }
  
    // ...
}
```

Reactive UI Fody provides the attributes `[Reactive]` and `[ObservableAsProperty]`. Which reduces the amount of typing required to define bindable properties. You can read more on the [official docs](https://www.reactiveui.net/docs/handbook/view-models/boilerplate-code). The `Count` is a property with a getter and setter and can be used like a normal Property in C#. The `CounterButtonText` is a bit special. When we increment the counter we want to change the button text. Since our `Counter` property raises an event everytime it gets changed. Which means we can write a [Observable as Property](https://www.reactiveui.net/docs/handbook/view-models/#read-only-properties) and implement the button text changes:

Reactive UI Fody provides the attributes `[Reactive]` and `[ObservableAsProperty]`. The attributes reduce the amount of typing required to define bindable properties. More info can be found in the [official docs](https://www.reactiveui.net/docs/handbook/view-models/boilerplate-code). The `Count` is a property with a getter and setter and can be used like a typical Property in C#. The `CounterButtonText` is special. Since it will never be set by the UI. A Read Only Property can be used. When the `Count` has a new value. The button text is changed. The Counter property raises an event every time it gets changed. This event can be observed by using [Observable as Property](https://www.reactiveui.net/docs/handbook/view-models/#read-only-properties); the text can be updated:

```c#
this.WhenAnyValue(vm => vm.Count)
	.Select(c => c switch
	{
		0 => "Click me",
		1 => "Clicked 1 time",
		_ => $"Clicked {c} times"
	})
	.ToPropertyEx(this, vm => vm.CounterButtonText);
```

Implementing the (synchronous) Command is done with the [ReactiveCommand](https://www.reactiveui.net/docs/handbook/commands/) helpers from ReactiveUI:

```c#
ButtonClickedCommand = ReactiveCommand.Create(ButtonClicked);
```

With all these pieces in place, we are feature complete with the original hello world sample using the MVVM architectural pattern. MVVM is a proven way to write extensible and maintainable front-end code. But using a library like Reactive UI provides additional features. Features are helpful in common front-end scenarios. For instance, having a timer that periodically checks/changes a value. The standard "Hello World" message could use some improvement. Let's have it display a reactish Hello World message.

![Changing Text sample]({{ site.url }}{{ site.baseurl }}/assets/images/20230425_ReactiveMaui101.gif "Chaning Text sample")

Let's start by adding a new property to hold the displayed text:

```c#
[Reactive] public string Greeting { get; set; }
```

With the property in place - the timer and the logic to change the text every second can be implemented:

```c#
public class MainViewModel : ReactiveObject, IActivatableViewModel
{
    public MainViewModel()
    {
        this.WhenActivated(disposables =>
        {
          // ...
          
          Observable
            .Timer(
              TimeSpan.FromMilliseconds(100), // give the view time to activate
              TimeSpan.FromMilliseconds(1000),
              RxApp.MainThreadScheduler)
            .Do(
              t => {
                var newGreeting = $"Hello, {Traits[t % Traits.Length]} world !";
                Console.WriteLine(
                  $"[vm {Thread.CurrentThread.ManagedThreadId}]: " +
                  $"Timer Observable -> " +
                  $"Setting greeting to: \"{newGreeting}\"");
                Greeting = newGreeting;
              },
              () => 
                Console.WriteLine(
                  "Those are all the greetings, folks! " +
                  "Feel free to close the window now...\n"))
            .Subscribe()
            .DisposeWith(disposables);

          // ...
    }
    
    [Reactive] public string Greeting { get; set; }
    // More Properties

    public ViewModelActivator Activator { get; } = new();
}
```

The timer is placed in the `WhenActivated` extension method. This helper allows starting and disposing of Observables. The default usage is bound to the lifecycle of the View. When the View is being rendered `WhenActivated` is invoked. When the user navigates away from the View, all the items are disposed of by a [CompositeDisposable](https://learn.microsoft.com/en-us/previous-versions/dotnet/reactive-extensions/hh228980(v=vs.103)?WT.mc_id=AZ-MVP-5003494). As per [documentation](https://www.reactiveui.net/docs/handbook/when-activated/), the ViewModel inherits from `IActivatableViewModel`. The View invokes the method by inheriting `ReactiveContentPage<T>`. For more details, be sure to read the docs.

## Dependency Injection

With all the functionality in place, there is one last thing left to do. Wiring up the ViewModel to the View. While it is possible to do this all by hand, aka `BindingContext = new FunkyViewModel()`, this can get rather tedious. With .NET MAUI, there is a standard way to register and resolve dependencies automagically.

In the *MauiProgram.cs* file, use the builder to register all our classes, including Views and ViewModels:

```c#
public static class MauiProgram
{
	public static MauiApp CreateMauiApp()
	{
		var builder = MauiApp.CreateBuilder();
		
    // Maui registration stuff

		builder.Services.AddTransient<MainPage>();
		builder.Services.AddScoped<MainViewModel>();

		// ...

		return builder.Build();
	}
}
```

With this in place, we can change the constructor of our View to get the ViewModel injected:

```c#
public partial class MainPage : ReactiveContentPage<MainViewModel>
{
	public MainPage(MainViewModel viewModel)
	{
		ViewModel = viewModel;
		InitializeComponent();
		this.WhenActivated(_ => { });
	}
}
```

Check out the official [docs](https://learn.microsoft.com/en-us/dotnet/architecture/maui/dependency-injection?WT.mc_id=AZ-MVP-5003494) to learn more about how Dependency Injection works in .NET MAUI.

## Conclusion

.NET MAUI and Reactive UI allow you to write native applications for desktop and mobile using the well-known and proven patterns of reactive programming using C#. This post looked at many topics concerning integrating Reactive UI into your .NET MAUI application.

You can find the complete sample on [GitHub](https://github.com/mallibone/HelloReactiveMaui) using the fundamental functions of Reactive UI, such as Bindings, Commands and timers.

HTH