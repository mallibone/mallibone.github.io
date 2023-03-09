---

layout: single
title: "Writing Unit Tests for your .NET MAUI app"
date: 2023-03-14 00:03:00
tags: ["Automated Testing", "XUnit", ".NET MAUI"]
slug: "maui-unit-testing"
---

Writing mobile apps usually brings along its fair share of application logic run on the device. When writing code, it is paramount to test it. How else will you know if it is working as expected? The answer is testing. And since we are developers, we like to automate mundane tasks. So Unit Testing our logic can be a great way to test our logic.

<!-- expand -->

> If you know how to set up unit tests, skip to the **TL;DR;** marker. 🙃

Unit Tests allow us to test the logic parts of our app. This will enable us to try most of our apps except the views. The most popular testing frameworks to the date of writing this post are [XUnit](https://xunit.net/), [NUnit](https://nunit.org/) and [MS Test](https://github.com/microsoft/testfx). All three are solid choices and mostly have the same functionality. They mainly differ in how tests are written. For my Unit Tests, I usually like to go with XUnit. Ensure that the .NET MAUI and test projects use the same .NET version. In this example, it will be .NET 7.

Given an existing .NET MAUI application, we can add a new testing project to the solution and choose XUnit as the testing framework. To have some code, we can test, I have rewritten the .NET MAUI hello world app to use a ViewModel (using the [Community Toolkit MVVM](https://learn.microsoft.com/en-us/dotnet/communitytoolkit/mvvm/?WT.mc_id=AZ-MVP-5003494) framework). Doing so gives us the following ViewModel code.

```c#
public partial class MainViewModel : ObservableObject
{
	[ObservableProperty] public int _count;
	[ObservableProperty] public string _text = "Click me";

	[RelayCommand]
	public void CounterClicked()
	{
		Count++;

		if (Count == 1) Text = $"Clicked {Count} time";
		else Text = $"Clicked {Count} times";

	}
}
```

We can test this logic by writing a Test that calls the command method in the well-known Arrange/Act/Assert pattern:

```c#
public class MainViewModelShould
{
    [Fact]
    public void IncrementcountOnCounterClicked()
    {
        // Arrange
        var sut = new MainViewModel();
        // Act
        sut.CounterClickedCommand.Execute(null);
        // Assert
        Assert.Equal(1, sut.Count);
    }
}
```

This will require us to add a project reference from the test project to the .NET MAUI project.

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <!-- ... -->

  <ItemGroup>
    <ProjectReference Include="..\MauiTesting101\MauiTesting101.csproj" />
  </ItemGroup>
</Project>
```



**TL;DR;** Up to this point, we just followed the usual steps for setting up a Unit Test project. If you put your app logic in the .NET MAUI app project, you will get a weird error when compiling the test project. To make the compiler happy, we will have to change the .NET MAUI .csproj file as follows:

```xml
<OutputType Condition="'$(TargetFramework)' != 'net7.0'">Exe</OutputType>
```

Note that we have to add a condition to the `<OutputType>` or else you will get an error that there is no main method when executing the test. If you are using a different .NET version i.e. .NET 6 you would have to write the condition as follows `<OutputType Condition="'$(TargetFramework)' != 'net6.0'">`. And for .NET 8 we would have to replace number with an 8 and so forth.

Note that we have to add a condition to the `<OutputType>`, or else you will get an error that there is no primary method when executing the test. If you are using a different .NET version, i.e. .NET 6, you would have to write the condition as follows `<OutputType Condition="'$(TargetFramework)' != 'net6.0'">`. And for .NET 8, we would have to replace the number with an 8. And for .NET 9, we ... you get the point. 😉

With these changes, we can start running the tests from your IDE or via the command line using `dotnet test` . You can find the complete example on [GitHub](https://github.com/mallibone/MauiTesting101).

HTH