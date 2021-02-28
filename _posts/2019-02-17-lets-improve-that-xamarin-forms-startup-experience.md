---
layout: single
title: "Let us improve that Xamarin Forms startup experience"
title: Let us improve that Xamarin Forms startup experience
date: 2019-02-17
tags: ["Xamarin.Forms", "Xamarin.Android", "Xamarin.iOS", "UWP"]
slug: "lets-improve-that-xamarin-forms-startup-experience"
---

[![Image of an escalator going towards the light](https://mallibone.com/posts/files/a98a96fa-d1e3-4ff4-92f2-817831409f1e.jpg "Image of an escalator going towards the light")](https://mallibone.com/posts/files/580b36a8-b509-4a28-a540-c0be421bd2ca.jpg)

This post is part of the Xamarin Month, which is about community and love. Looking after a nice UI and User Experience is one way how a product team or developer can show love to its user. So let focus on a small detail which always makes me smile when done right 🙂

Xamarin Forms apps have a reputation for taking their time to load. While quicker loading times are always preferred and are an excellent place to start. Sometimes there is no way around letting the user wait, while a background process is doing it's best to do its task. However, there is an alternative to speed: Distraction. Distraction is what Airplanes do with their onboard entertainment, and it is what some apps like Twitter do on startup with an animated logo. Since Xamarin Apps fall into the latter category, let's see how we can improve our startup experience with some fancy animated Xamarin Hexagon.

However, before we get started with the animation part, I'm afraid we have to take a quick look into one of our platform projects - into the Android project that is.

## The empty feeling when starting Xamarin.Forms on Android

Have you ever wondered why the startup screen experience of your Xamarin app on Android differs from iOS or UWP? While we are greeted instantly with a logo when starting up our Xamarin.iOS app, when starting the same app on Android, a blank screen stares at us. Why is that so?

[![Screenshot_1550416214](https://mallibone.com/posts/files/0992ae5a-a7f4-4e0d-a191-7fcfc375afba.png "Screenshot_1550416214")](https://mallibone.com/posts/files/fc01e7a6-558b-46b6-b57f-84a5fac54f0b.png)

Just point it out: this is not the fault of Xamarin Forms, it is more a difference in the two platforms. While iOS forces you to provide a startup storyboard (a single view), there is no such thing under Android. At least that may seem so at first. However, from where is this blank screen? You probably already know that the starting point of a Xamarin.Forms app on Android is the `MainActivity.cs` or to be more precise that one activity which has the following attribute set:


    [Activity( ... Theme = "@style/MainTheme", MainLauncher = true, ... ]
    public class MainActivity : global::Xamarin.Forms.Platform.Android.FormsAppCompatActivity
    {// ...
    }


One attribute that is getting set is the theme. This theme is where Android "draws it's inspiration" for the splash screen. We can find it defined under `Resources\values\styles.xml`. Now to replicate the startup image, we first have to define a layout in `Resources\drawables\splash_screen.xml` along the following lines:


    <?xml version="1.0" encoding="utf-8"?>
    <layer-list xmlns:android="http://schemas.android.com/apk/res/android">
      <item>
        <color android:color="@color/colorPrimary"/>
      </item>
      <item>
        <bitmap
            android:src="@drawable/SplashScreen"
            android:tileMode="disabled"
            android:gravity="center" />
      </item>
    </layer-list>


Now we can modify `styles.xml` by adding new style with the following lines:


    <?xml version="1.0" encoding="utf-8" ?>
    <resources>
    
      <!-- ... -->
    
      <style name="SplashTheme" parent ="Theme.AppCompat.Light.NoActionBar">
        <item name="android:windowBackground">@drawable/splash_screen</item>
        <item name="android:windowNoTitle">true</item>
        <item name="android:windowFullscreen">true</item>
      </style>
    </resources>


Starting the app and we see the Xamarin logo while starting up. Unfortunately, it does not go away when we get to our Hello World page in Xamarin Forms. The reason being that we have overwritten the default style which is also used by our Xamarin.Forms app. However, we can fix this by adding an activity solely to display this new style, once the new `SplashActivity.cs` is rendered we switch over to the current `MainActivity.cs`. The `MainActivity.cs` uses the original style and starts the Xamarin.Forms part of our app.

[![Screenshot_1550416896](https://mallibone.com/posts/files/93993bbc-37b2-4f8e-ae88-97ef5b1a45ef.png "Screenshot_1550416896")](https://mallibone.com/posts/files/f6975711-66c1-46ea-b0c9-0debc0b2a8f8.png)

If we let the app run the app now. We do see a splash screen which disappears after starting up the app. So now that we have Android on par with iOS and UWP let's shift gears and implement that bouncy startup animation.

## Bouncy startup animation

Drawing some inspiration from the Twitter app, let's let our logo bounce similarly. We implement the animation of the hexagon in Xamarin.Forms. The animation could - in a real app - buy us some time while we are starting up. So what we need is again a splash screen but this time a Xamarin.Forms view. The XAML displays an image in the centre:


    <?xml version="1.0" encoding="utf-8" ?>
    <ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
                 xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
                 x:Class="CustomSplash.SplashPage"
                 BackgroundColor="#2196F3">
        <ContentPage.Content>
            <Grid>
                <Image x:Name="SplashIcon"
                       HorizontalOptions="Center"
                       VerticalOptions="Center"
                       Source="SplashScreen.png" />
            </Grid>
        </ContentPage.Content>
    </ContentPage>


The XAML is ready. However, this would solely extend the static native splash screens. The animation takes place in the code behind. We can override the `OnAppearing` method and add some animation logic to it:


    await SplashIcon.ScaleTo(0.5, 500, Easing.CubicInOut);
    var animationTasks = new[]{
        SplashIcon.ScaleTo(100.0, 1000, Easing.CubicInOut),
        SplashIcon.FadeTo(0, 700, Easing.CubicInOut)
    };
    await Task.WhenAll(animationTasks);


First, we shrink the image, then we expand and simultaneously let it go transparent. Combining the animations gives our app a nice fluid effect. While we could now put this puppy in a loop and endeavour it forever and ever and ever and... well most probably we only want to show it once and then move on to our main page. The following lines achieve this:


    Navigation.InsertPageBefore(new MainPage(), Navigation.NavigationStack[0]);
    await Navigation.PopToRootAsync(false);


The above lines insert the main page as the first page in the navigation stack. In other words, we insert the main page before the splash screen. Then we `PopToRoot` so the splash screen is no longer present on the navigation. So while the lines might look a bit odd at first. They prevent the user from navigating back to the splash page. Further, it allows the splash page to be garbage collected. Bottom line all the things we want to do with a splash screen once it has served its purpose.

The resulting app looks something like this:

[![Animation splash screen on iOS](https://mallibone.com/posts/files/533ff281-d9d5-48d7-ba9f-063fd426756b.gif "Animation splash screen on iOS")](https://mallibone.com/posts/files/bdd46516-0544-4b9e-aad6-5bcce1e7b76f.gif)

I am a firm believer that these little things can go a long way and show your user right from the get-go that you care about your app. While the native splash screen is a good start. The animated load screen can buy you a bit of extra time to start up your app while distracting the user. You can find the entire demo app on [GitHub](https://github.com/mallibone/LaunchExperience).

Be sure to check out the other blog posts in the [Xamarin Universe](https://luismts.com/blog/xamarin/xamarin-month-february-2019/) and happy coding!
