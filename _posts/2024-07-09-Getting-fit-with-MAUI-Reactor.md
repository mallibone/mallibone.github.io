---
layout: single
title: "Getting fit with MAUI Reactor"
date: 2024-07-09 00:03:00
tags: [".NET", ".NET MAUI", "MAUI Reactor"]
slug: "maui-reactor-fit"
---

MAUI Reactor is a code first, MVU style framework based on .NET MAUI that promises less ceremony when writing your apps. And I like the sound of that and since this post will be part of [MAUI UI July](https://goforgoldman.com/posts/mauiuijuly-24/) I wanted to tackle a long hedged goal of mine which is to write a app that will keep me motivated during my home workouts.

<!-- expand -->

> This app has been heavily inspired by the [WODTimer](https://www.smartwod.app/wod-timer) app. If you are looking for a sound and complete app (for free) be sure to check it out.
>

I have been working out at home for quite some time now. And while I have started to join again in some group training, I often find myself in my home/hotel/outdoor gym by myself. Finding motivation to workout can be tough, but time boxing the training is a great helper on my end. There are multiple forms on workout timers out there and I want to add a few of them into my app. These are for example:

* EMOM
* AMRAP
* For Time
* TABATA

I will dive in later what these modes mean but for now, I want to have a general overview which will allow me to select the workout mode. Once on the page I will want to configure the workout timer. Depending on the option chosen I will need to set different options. Then I want to start the workout. In this first post we will focus on the start page and the AMRAP workout since it is the simplest and will let me show how you can work with setting up the pages, navigation (incl. parameters) and share styling accross pages.

## Setting things up

MAUI Reactor builds on top of .NET MAUI, so first we will want to have .NET MAUI [installed](https://learn.microsoft.com/en-us/dotnet/maui/get-started/installation?view=net-maui-8.0&tabs=visual-studio-code&wt.mc_id=DT-MVP-5002881) and properly configured. Next you can [install MAUI Reactor](https://adospace.gitbook.io/mauireactor/getting-started), don't forget to also [install the tooling](https://adospace.gitbook.io/mauireactor/getting-started#install-mauireactor-hot-reload-console) that enables hot reload, since it will greatly improve your productivity, when developing an app. As for the editor - you can choose whatever you want. I have been using it with Visual Studio on Windows; Rider and VS Code on macOS.

I decided to use the Shell for navigation - especially since MAUI Reactor comes with a nice template.

```bash
dotnet new maui-reactor-appshell -n AlohaFit
```

## The start page

With everything setup, let's have a look at the main page.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/2024-07-09-MauiReactorMain.png" alt="Main page of the MAUI Reactor fitness app" style="zoom:40%;" />

The main thing I want to do is have a list of buttons - this works well for the limited amount of workout types in the app. The general setup of the page looks as follows:

```c#
using AlohaFit.Types;
using MauiReactor;

namespace AlohaFit.Pages;

class MainPage : Component
{
    private async void NavigateToAmrap() =>
        await Microsoft.Maui.Controls.Shell.Current.GoToAsync<WorkoutParameters>(NavigationRoutes.WorkoutPage,
            param => param.SelectedWorkoutMode = WorkoutModes.AMRAP);

    // Other workout navigation methods

    public override VisualNode Render()
        => ContentPage(
            ScrollView(
                VStack(
                        Label("TIMER").Style("Headline").HCenter().Margin(0,0,0,32),
                        
                        Components.PrimaryButton("AMRAP", Colors.DarkOrange, NavigateToAmrap),
                        // Other workout timer buttons
                    )
                    .VCenter()
                    .Spacing(25)
                    .Padding(30, 0)
            )
        );

}
```

Since the buttons all look the same we can easily extract a component that will give us the functionality and consistent style.

```c#
using System;
using MauiReactor;

namespace AlohaFit.Pages;

public static class Components
{
    public static Button PrimaryButton(string title, Color color, Action handleOnClicked) =>
        new Button(title)
            .BackgroundColor(color)
            .CornerRadius(24)
            .HeightRequest(50)
            .WidthRequest(200)
            .OnClicked(handleOnClicked)
            .HCenter();
}
```

Extracting components and reusing them is nothing new. It can be done in [.NET MAUI](https://learn.microsoft.com/en-us/dotnet/maui/user-interface/controls/contentview?view=net-maui-8.0&wt.mc_id=DT-MVP-5002881), but using C# makes this approach a lot more terse. But let's have a look at our options when it comes to creating reusable views and styling in general.

### Working with styles

Especially since the properties we have are simply function parameters for smaller things. If we wanted to create an entire View - we can do that too, and we can then even provide properties that we can reuse - you can read more about that in the MAUI Reactor [docs](https://adospace.gitbook.io/mauireactor/components/component-properties).

When working on apps it is common to extract styles. When working with MAUI Reactor we can do exactly that too. If we look under Resources/Styles, we will find xaml files for the colour and the styles for our components. We can change the styles and the components used in MAUI Reactor will actually use those styles. Which is super neat, because this means you can reuse any existing mobile styles you might already use. But let's get back to our workout timer.

## The workout timer

Generally speaking all workout timers require configuring the timer. Once configured the user can start the timer. When running the timer will complete or the user can abort. For now I will focus on the AMRAP timer, but in the near future I will want to add the other timers. Since they all seem very similar in function I will try to create a workout page which will render itself based on the mode and the features each timer will provide. So when choosing a timer I will want to open the workout page and pass in as a parameter which workout has been chosen.

### The navigation

Since we are using the Shell we can configure the navigation as follows:

```c#
class AppShell : Component
{

    protected override void OnMounted()
    {
        MauiControls.Application.Current.UserAppTheme = AppTheme.Dark;
        
        Routing.RegisterRoute<WorkoutPage>(nameof(WorkoutPage));
        base.OnMounted();
    }

    public override VisualNode Render() => Shell(ShellContent().Title("").RenderContent(() => new MainPage()));
}

```

And invoke the navigation by using the Shell in the MainPage.

```c#
await Microsoft.Maui.Controls.Shell.Current.GoToAsync<WorkoutParameters>(NavigationRoutes.WorkoutPage,
            param => param.SelectedWorkoutMode = WorkoutModes.AMRAP);
```

Note we are also passing a navigation parameter. MAUI Reactor uses a class as navigation parameter. I kind of like this, because it extends the Shell navigation but also provides type safety. The workout page defines the parameter as additional generic argument `WorkoutParameters` - which is the name of the class:

```c#
class WorkoutPage : Component<WorkoutState, WorkoutParameters>
{
    // ...
}
```

And we can access the parameters via the `Props` attribute in a `Component` in the code:

```c#
Props.SelectedWorkoutMode
```

I really like this approach. It gives me the typesafety of a language like C#, but also requires very little fuss when doing so. Be sure to check the MAUI Reactor docs to get the full picture of the [Shell navigation](https://adospace.gitbook.io/mauireactor/components/navigation/shell) in MAUI Reactor.

### Dealing with state

With the navigation and the parameters out of the way. Let's implement the timer. For this we will need some state. The standard MAUI app usually relies on the MVVM pattern for dealing with seperation of View ViewModel and Models. But in MAUI Reactor we use a Model View Update (MVU) based approach. The state of the page is stored in a seperate class (the Model of MVU). But before we dive into that I feel like owing you an explanation of what AMRAP stands for...

> **AMRAP (As Many Rounds/Reps As Possible)**: If you have a given workout for example 5 Pullups, 10 Pushups and 20 Air Squats and your AMRAP is 15 minutes long. You will try to do as many rounds as possible. You should generally try to be able to keep on working out without any larger breaks. So your pace will become a factor once you have built the muscular endurance.

With that out of the way - let's look at what information we need when setting our timer:

```c#
public class AmrapState
{
    public DateTimeOffset StartTime { get; set; }
    public TimeSpan Duration { get; set; }
    public TimeSpan RoundDuration { get; set; }
    public int Rounds { get; set; }
    public bool IsRunning { get; set; }
    public IReadOnlyCollection<DurationOption> DurationOptions { get; set; }
    public int SelectedDurationIndex { get; set; }
}

```

The wiring of the view to the state is done with like with the navigation parameters via a generic attribute.



### Starting the workout

With all this in place - we are left with starting the workout. So once the user hits the button the timer starts:

```c#
  private void StartWorkout()
  {
      SetState(s => s.IsRunning = true);
      SetState(s => s.StartTime = DateTimeOffset.Now);
      // ...
      
      _timer.Start();
  }
```



The wiring up is very straight forward. You can also see that choosing between the configuration or running sub-view is done by observing a boolean in the state. Further note that we do not have to implement any events, raise property changed or the like. It's simply calling SetState and assigning the property. So less hassle, same effect.. I would call that neat. 😃

And that is my basic AMRAP timer using MAUI Reactor.

## Looking forward

I hope this post could give you an overview of what Reactor MAUI is and how it extends .NET MAUI to provide a different approach in writing native apps. In future posts I will expand this app to provide different workout modes, expand the design. So stay tuned for future updates. Also be sure to check out Reactor MAUIs GitHub page and the docs.

You can find the app to this post on [here](https://github.com/mallibone/AlohaFit).

HTH