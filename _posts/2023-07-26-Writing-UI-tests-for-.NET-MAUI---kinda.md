---
layout: single
title: "Writing UI tests for .NET MAUI - POC"
date: 2023-07-26 00:03:00
tags: [".NET MAUI", "UI Test"]
slug: "uitest-maui-101"
---

This blog post is part of the .NET MAUI UI July - be sure to check out the [other blog posts](https://goforgoldman.com/posts/maui-ui-july-23/). 🙂

Automated UI tests allow you to write tests that execute your app as a user would. This can be a significant selling point compared to a unit, integration, and other non-UI automated tests. Non-technical users are no longer left with a chart of how many passed, failed, skipped, etc. - they actually get to see what the tests performed. At first glance, automated UI tests seem like the ultimate automated tests, but more later.

If you have written Xamarin apps in the past, you might have heard about Xamarin UI Tests. The testing framework based on [Calabash](https://github.com/calabash) and [NUnit](https://nunit.org/) allows you to write automated UI tests. Sadly for users of said framework, there never has been a .NET MAUI UI Test successor - or has there? This post will examine how to use the Xamarin UI Test framework to write .NET MAUI UI tests.

## Foreword

Before we start, I want to be honest with you. A lot of the following steps may seem odd. While you will see how to write UI tests using the [Xamarin UI Test](https://www.nuget.org/packages/Xamarin.UITest) framework, a few parts remain left to be migrated to certain foundations that .NET MAUI is built, such as using .NET. So if a few of the following details feel "hacky" - perhaps one could even say "yucky". I would concur but also add that only if there is actual usage of a product will any further development be put behind it. So, in the hopes of UI tests living onwards, let's look at how we can cobble together - erh, write UI tests for .NET MAUI.

## Writing Tests

UI tests are currently supported for Android, iOS and MacCatalyst. Given the "Create new .NET MAUI Project" starting point, we first want to add the test project using your favourite .NET IDE. Therefore, we add a .NET Framework 4.8 library project. That is not a typo; we add a .Net Framework 4.8 library project. Next, open the newly added csproj and change it to the following lines:

```xml
<Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <TargetFramework>net481</TargetFramework>
        <IsPackable>false</IsPackable>
        <IsTestProject>true</IsTestProject>
    </PropertyGroup>

    <ItemGroup>
        <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.5.0" />
        <PackageReference Include="NUnit" Version="3.13.3" />
        <PackageReference Include="NUnit3TestAdapter" Version="4.5.0" />
        <PackageReference Include="NUnit.Analyzers" Version="3.6.1" />
        <PackageReference Include="coverlet.collector" Version="3.2.0" />
        <PackageReference Include="SharpZipLib" Version="1.4.2" />
        <PackageReference Include="Xamarin.UITest" Version="4.2.0" />
    </ItemGroup>

</Project>
```

With all this in place, we are ready to write our first UI test for Android. We will get back to the Apple platforms in a minute. UI Tests require more setup. We generally have to pass in the platform, which allows the test framework to translate your test code into actual UI actions. Since you will do this setup for every test class, it is often moved to a base class:

```c#
using System;
using NUnit.Framework;
using Xamarin.UITest;

namespace MauiTesting101.UiTests
{
    [TestFixture(Platform.Android)]
    [TestFixture(Platform.iOS)]
    public class BaseTest
    {
        IApp _app;
        protected readonly Platform Platform;

        protected BaseTest(Platform platform) => Platform = platform;

        protected IApp App => _app ?? throw new NullReferenceException();

        [SetUp]
        public virtual void BeforeEachTest()
        {
            _app = AppInitializer.StartApp(Platform);

            App.Screenshot("App Initialized");
        }
    }
}
```

We can write our first test with the UI test setup out of the way. Since we are doing this for the .NET MAUI "Hello World" app, let's check if the button changes its text accordingly after it gets selected:

```c#
[Test]
public void WhenCounterClicked_IncrementCount()
{
    // Arrange: Wait until counter Button is present
    const string countButton = "CounterBtn";
    App.WaitForElement(countButton);

    // Act: Tap the button
    App.Tap(countButton);


    // Assert: check label updated
    var buttonLabelValue = "Clicked 1 time";
    Assert.DoesNotThrow(() => App.WaitForElement(x => x.Marked(buttonLabelValue)), "Button was not clicked");

    // Take a screenshot
    App.Screenshot("Tapped 1 time");
}
```

Now, let's look at some of the things going on. The arrange, act, assert style of writing the test is optional, but I like the structure it gives the tests. One thing that you will quickly notice when writing UI tests is that everything takes more time. The test code must wait for the app to start before we tap the button. And this is what is happening on this line:

```c#
[Test]
public void WhenCounterClicked_IncrementCount()
{
    // Arrange: Wait until counter Button is present
    ...
    App.WaitForElement(countButton);

    // Act
    ...
}
```

This line will either timeout (UI element not found) or complete without exception. The second one means the UI element is present. Once that is the case, we invoke the actual tapping of the button:

```c#
[Test]
public void WhenCounterClicked_IncrementCount()
{
    // Arrange: Wait until counter Button is present
    ...

    // Act: Tap the button
    App.Tap(countButton);


    // Assert: check label updated
    ...
}
```

Then we wait again for the UI to update and verify the value has changed according to our expectations. Even with this relatively simple test, you notice a crucial point when writing UI tests. Before you invoke an action, you ensure that the expected UI is loaded by waiting for a UI element.

You may have noticed that we invoke the button using an identifier string CounterBtn. This tag is set on in the XAML by setting the *AutomationId* property:

```c#
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage ...>

    <ScrollView>
        <VerticalStackLayout
            ...>

            ...

            <Button
                x:Name="CounterBtn"
                AutomationId="CounterBtn"
                ... />

        </VerticalStackLayout>
    </ScrollView>

</ContentPage>
```

The *AutomationId* property is not mandatory but significantly reduces the time to write a UI test. It also significantly improves the quality of your tests since you will not have to rewrite code if, for example, the UI hierarchy changes. But it does require that during development (or when writing the UI test), the *AutomationId* Property is set.

If we execute the test, we will see the following:

![Android UI test recording on device emulator]({{ site.url }}{{ site.baseurl }}/assets/images/2023-07-AndroidUITest.gif)

Next up is iOS. For iOS, we must install the [Xamarin.TestCloud.Agent](https://www.nuget.org/packages/Xamarin.TestCloud.Agent) NuGet package. This is required to invoke the UI commands. Since .NET MAUI uses a single project file, we must ensure that this NuGet package is only added when compiling against iOS. The same applies to macCatalyst. When adding the NuGet to the solution, this is automagically done for us:

```xml
<Project>
  ...
  
  <ItemGroup Condition="'$(TargetFramework)' == 'net7.0-ios'">
	  <PackageReference Include="Xamarin.TestCloud.Agent" Version="0.23.2" />
	</ItemGroup>

	<ItemGroup Condition="'$(TargetFramework)' == 'net7.0-maccatalyst'">
	  <PackageReference Include="Xamarin.TestCloud.Agent" Version="0.23.2" />
	</ItemGroup>

</Project>

```

I had some issues with this approach, so I changed the lines to the following:

```xml
<ItemGroup Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'ios'">
  <PackageReference ... />
</ItemGroup>
```

Generally speaking, this ensures the NuGet package is only added when compiling against iOS or macCatalayst. We could further check that the package only gets used for Debug builds. This would ensure we don't submit our app with this NuGet package to the store. It has been heard that the package can lead to rejection during the store evaluation process. But then again, sometimes you want to test against the release build, which could require dedicated build configurations for UI tests. And down the rabbit hole we go. But if you are in the situation, this would be an option.

After installing the NuGet package, we must enable the testing agent by adding `Xamarin.Calabash.Start()` to the *AppDelegate.cs* file:

```c#
using Foundation;
namespace MauiTesting101;

[Register("AppDelegate")]
public class AppDelegate : MauiUIApplicationDelegate
{

    protected override MauiApp CreateMauiApp() 
    {
        #if ENABLE_TEST_CLOUD
                Xamarin.Calabash.Start();
        #endif
        
        return MauiProgram.CreateMauiApp();
    }
}
```

You can define the constant as follows in your .NET MAUI app project file:

```xml
<Project Sdk="Microsoft.NET.Sdk">

	<PropertyGroup>
		...

	<PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Debug|AnyCPU' ">
		<DefineConstants Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'ios' OR $([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'maccatalyst' ">TRACE;DEBUG;NET;NET7_0;NETCOREAPP;ENABLE_TEST_CLOUD</DefineConstants>
		<DefineConstants Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) != 'ios' AND $([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) != 'maccatalyst' ">TRACE;DEBUG;NET;NET7_0;NETCOREAPP</DefineConstants>
		...
	</PropertyGroup>

	...

</Project>

```

I had again some issues setting this up. Due to the single project file structure, I added two definitions depending on the platform. There might be a nighter way; let me know what I'm missing out on. 😆

With all this in place, we can now execute the same test for iOS.

![iOS UI test recording on device simulator]({{ site.url }}{{ site.baseurl }}/assets/images/2023-07-iOSUITest.gif)

## When should you write UI Tests?

As I said in the beginning, UI tests are great at first glance. The end user sees what is being tested, and you write tests very closely to how a real user would interact with the application. However, there are some caveats when writing UI tests, so many do not even try to write them. So, let's start with the three concerns I hear most:

- UI tests are slow
- UI tests are brittle
- UI tests require high maintenance

### Performance

Right out of the gate, yes, UI tests are slow. If we compare the execution time of writing the above test as a unit test, we compare these two times to each other (run on a MacBook Pro M2Max):

UI Test: 9086 ms

Unit test: 7 ms

There is one significant takeaway from the times you see above. Do not replace your unit tests using UI tests. Also, if all you are checking could be done in a unit test, then go with the unit test. UI tests are not meant to replace a unit test. They are intended to allow you to test the UI layer without having to go over a test script on every release (you do have a test script, right? 😉)

One strategy to improve performance on your UI tests, apart from ensuring they add value by testing the UI layer, is to parallelize the execution of your tests. This will reduce the time it takes until you get the feedback from an automated build that includes UI tests.

### Flaky UI tests

Most UI test setups test the entire code base. They are used to test the UI and whatever service is invoked by UI actions. It will execute said code chain from the UI, ViewModel, Service and sometimes even the backend. So, if anything goes wrong, it could be at any stage. A test that appears to be randomly failing or succeeding is terrible since you will lose confidence when it is read that there is a problem you should actually be investigating.

So, have different tests to catch errors in separate layers. The UI test can be a great canary, but it often will not be able to pinpoint the source of the issue.

Another reason UI tests can fail is that some action takes too long. There can be many reasons, from slow devices, slow network, overloaded test server, etc. - long story short, this can be the most unnerving because everything is correct, and it is really tough to figure out why the app is running so slow. One obvious solution is cranking up the default timeout. Still, it can also be a great starting place to think about improving your app's performance to ensure the user experience does not take a hit due to a bad performance.

### UI tests require high maintenance

Writing a good UI test takes more thought than your average unit/integration test. There are more moving parts, and anyone could fail. It is now up to you if that failure should indicate the app has failed or if there was some glitch that should be caught by the user/test code.

Another reason UI Tests take longer is whenever the UI has some considerable change (redesign). It might require you to rewrite the UI tests. How the UI tests interact with your app also means you must execute the test to determine if it is still working.

### Is it worth the trouble

As you can see, UI tests are no silver bullet. But they provide a unique opportunity to validate that your app is running as it should. I like the idea of an automated smoke test of an app. This test checks if the app starts up as expected. The next series of tests validate that every screen opens without issue in the app. These tests usually take little to write and maintain.

The next step would be to test an app's most common scenarios. This usually eliminates the check every screen test. Writing these tests is more complex and will require additional thought when creating them. Generally speaking, I would strive to test the happy path. The maintenance will be higher and should be considered when choosing which tests to perform using UI tests.

## Conclusion

You can actually write UI tests for your .NET MAUI app. Yes, there are still some rough edges here and there to get it running. But it does run.

![Ugliest tests meme, but they do run]({{ site.url }}{{ site.baseurl }}/assets/images/202307_SparrowMeme.jpeg)

The nice thing about the UI tests is that they follow the spirit of writing once and then having it run on multiple platforms. You do not have to implement tests for every platform your app runs on. Currently, the tests are limited to iOS, Mac Catatalyst and Android. As stated above, it is to be seen how this goes forward. When used appropriately, UI tests can be a great addition to your other tests and be a real time and money saver.

The only alternative to automated UI tests is manual or exploratory field testing, usually a combination of bug reports by your users and (sound) log files to find the error and have an update ready as quickly as possible once a problematic bug has been discovered.

Kudos to Gerald for creating the [YouTube video](https://youtu.be/0c2U-TzmTnQ) that inspired this post. 🙂

HTH