---
layout: single
title: "Using AutoMapper in your Xamarin Forms apps"
title: Using AutoMapper in your Xamarin Forms apps
date: 2020-02-26
tags: ["Xamarin.Forms", "AutoMapper"]
slug: "xamarin-automapper"
---

[![Icons of Xamarin &amp; Automapper in a mobile phone]({{ site.url }}{{ site.baseurl }}/assets/images/d5bd01c1-11cf-47ee-b1c2-4f17bd587ff7.png "Icons of Xamarin &amp; Automapper in a mobile phone")]({{ site.url }}{{ site.baseurl }}/assets/images/0ba0e9a8-7c08-4191-9516-2c7e01e877fd.png)

You might have already heard of [AutoMapper](https://automapper.org/), the library that helps you to copy Properties form Object to another written by [Jimmy Bogard](https://twitter.com/jbogard). Whenever you are creating a larger Xamarin Forms application, you usually end up with different models representing similar data but for different areas of your app. For example, you will get a minimalist Data Transfer Object (DTO) from your backend, which you might copy into another app-internal model or directly to the View Model representing the data displayed on your view. And this is where AutoMapper will help you out and prevent you from writing all that copy code.

I must confess - while it always felt a bit like overkill at the beginning. At the end of the project, I was still happy to have made the decision at the beginning. So let's see how we get started.

### Getting Setup

The setup is quite straight forward actually add the [NuGet package](https://www.nuget.org/packages/AutoMapper/) of AutoMapper to your Xamarin Forms project. That is until you start compiling for iOS, then it will all blow up due to some reflection issue. Since iOS is compiled Ahead Of Time (AOT), you can't do any runtime operations such as reflections. Now AutoMapper does not use reflection when running on iOS. Still, due to some weird compilation thingy issue - I haven't understood the point in detail - the compiler ends up trying to add reflection which will not work on iOS. So to make things run under iOS, we have to add the following line to our iOS `csproj` file:


    <?xml version="1.0" encoding="utf-8"?>
    <Project ToolsVersion="4.0" DefaultTargets="Build" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
      ...
      <ItemGroup>
        ...
        <PackageReference Include="System.Reflection.Emit" Version="4.7.0">
          <ExcludeAssets>all</ExcludeAssets>
          <IncludeAssets>none</IncludeAssets>
        </PackageReference>
      </ItemGroup>
      ...
    </Project>


Then there is still the Linker under iOS that tries to remove the `System.Convert` assembly. Which is required by AutoMapper. But luckily we can help the linker out here by adding an XML file:


    <linker>
      <assembly fullname="mscorlib">
        <type fullname="System.Convert" preserve="All" />
      </assembly>
    </linker>


And setting the build property to `LinkDescription`:

[![Showing Visual Studio Properties Pane]({{ site.url }}{{ site.baseurl }}/assets/images/d6ffc41e-bd07-44f5-b087-0525df99eb0a.png "Showing Visual Studio Properties Pane")]({{ site.url }}{{ site.baseurl }}/assets/images/fffafeef-db25-4a63-b700-6d26cc57374b.png)

### Configuration and usage

Okay so everything is set up now we still have to tell AutoMapper what which objects we would like to copy from A to B and back again. In the small app I prepared for this blogpost we will copy the note DTO object to it's View Model counterpart. So to get a configured AutoMapper instance, we would write something like this during the start-up of the app:


    public static IMapper CreateMapper()
    {
        var mapperConfiguration = new MapperConfiguration(cfg =>
        {
            cfg.CreateMap<Note, NoteViewModel>();
            cfg.CreateMap<NoteViewModel, Note>();
        });
    
        return mapperConfiguration.CreateMapper();
    }jjjjjjj


Now we could convert the DTO to a View Model by invoking the AutoMapper instance as follows:


    var viewModel = _mapper.Map<MainViewModel>(note);


But what I often end up doing is populating the View Model after creation or when the Page/View it is being used in is getting created. For example, an `Init` method on the View Model might be invoked during the `OnAppearing` of the View. Then we could call a service in the View Model which would return a DTO of that object. In this scenario, we will want to tell AutoMapper to map the DTO directly on the View Model itself:


    public async void Init()
    {
        var note = (await _noteService.GetNotes()).First();
        _mapper.Map(note, this);
    }


If you had a List of View Models, i.e. a [CollectionView](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/user-interface/collectionview/) or [ListView](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/user-interface/listview/), we would use [LINQ](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/) in combination with AutoMapper to quickly convert from DTO to View Model:


    Notes = (await _noteService.GetNotes())
                .Select(_mapper.Map<MainViewModel>)
                .ToList();


Having AutoMapper in place you whenever a new DTO has to be presented in your app, you would add a new configuration and then be able to use the mapping. So the more mapping you require, the quicker using AutoMapper will pay itself off.

But there is one more thing I really like when using AutoMapper in my projects, and that is testing.

### Testing your mappings

"Testing? Are you serious?" - well yes, I actually am. You can check your configuration with a simple test case:


    [Fact]
    public void AppBoostrapper_ValidateMapping_AssertCorrectness()
    {
        var mapper = AppBootstrapper.CreateMapper();
        mapper.ConfigurationProvider.AssertConfigurationIsValid();
    }


This test will tell you if AutoMapper has all the information necessary to copy your data from one object to the other. So if we start adding Commands to the View Model, the mapping will fail with the information that it can not map the command into our DTO. And we obviously do not want to send an `ICommand` data field back to the server, so we ignore it:


    cfg.CreateMap<Note, NoteViewModel>()
        .ForMember(n => n.ExecuteReset, opt => opt.Ignore())
        .ForMember(n => n.ExecuteStore, opt => opt.Ignore());
    cfg.CreateMap<NoteViewModel, Note>();


But what when we add a data field `WriterMood` to the DTO and forget to add it to the View Model? Correct, the test will fail and inform us that we have forgotten to add the field.

[![Screenshot of failed AutoMapper config test]({{ site.url }}{{ site.baseurl }}/assets/images/32821658-e8fe-4afd-ab5f-0bf60554165e.png "Screenshot of failed AutoMapper config test")]({{ site.url }}{{ site.baseurl }}/assets/images/93dd03ba-dcf8-4247-9589-426c80fc939a.png)

And that test has saved me from so many forgotten data fields - ahem what I meant to say it saved a friend of mine... 😅

### Looking back

I will surely use AutoMapper in my future Xamarin Forms project that have their share of DTOs, internal Models and View Models since it is not only convenient to copy the properties. But it also saves me from forgetting to add properties to objects.

Be sure to check out the official [documentation](https://automapper.readthedocs.io/en/latest/) of AutoMapper since this post barely scratches the surface on what you can configure with AutoMapper.

As always, you can find a complete little sample application on [GitHub](https://github.com/mallibone/XamarinAutomapper).

HTH
