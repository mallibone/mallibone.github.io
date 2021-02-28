---
layout: single
title: "Accessing platform specific features in your Portable Class Library (PCL) through Dependency Injection (DI)"
title: Accessing platform specific features in your Portable Class Library (PCL) through Dependency Injection (DI)
date: 2015-11-19
tags: ["Android", "MVVM Light", "UWP", "Windows", "Xamarin", "Windows Phone", "Xamarin.Forms", "iOS"]
slug: "accessing-platform-specific-features-in-your-portable-class-library-pcl-through-dependency-injection-di"
---

[![3856456237_f05ebd2602_o]({{ site.url }}{{ site.baseurl }}/assets/images/623a47e3-57b5-4c30-904f-062c5c09e625.jpg "3856456237_f05ebd2602_o")]({{ site.url }}{{ site.baseurl }}/assets/images/bb33281a-154e-456c-a77c-00b9df76790d.jpg)
 
The Portable Class Library (PCL) allows developers to share C#, F# and VB code across multiple devices, platforms and runtime environments such as .Net, Xamarin or Windows Store Apps. In this previous post you can see how you can write portable business logic and integrate it with all the various platforms. But what if you actually want to use something the other way round say the GPS position, file system etc.. What then? This is exactly his post we will look at the following topics:
 
- Using Interface Segregation and Dependency Inversion to our favor
    - Interface in the PCL
    - Implementation on the Platform
- Injecting the implementation at runtime into the PCL
- Making the resolution of dependencies a breeze with MVVM Light

 
So lets get to it!
 
# Using Dependency Injection to our favor
 
Remember [SOLID](https://en.wikipedia.org/wiki/SOLID_%28object-oriented_design%29 "link to solid explenation on wikipedia")? Well it turns out the last letter stand exactly for the pattern we will be using to solve our “How will I be able to use a platform specific feature cross platform “. The PCL code is platform independent and therefore can be used across multiple platforms, so we want to put near to all of our business logic into a PCL. Further by accessing concrete class implementations via interfaces we can define an Interface in the PCL and create it’s concrete implementation in a platform specific project.
 
[![PCLDIOverview]({{ site.url }}{{ site.baseurl }}/assets/images/f9d40d73-7cff-430d-9ddb-5e54a4988567.png "PCLDIOverview")]({{ site.url }}{{ site.baseurl }}/assets/images/31b74c3f-c869-4dac-8da5-933f2136b3a1.png)
 
# Dependency Injection in action
 
For example lets say we want to display the current OS version for each platform, this is a very platform specific feature. To access such a information in the PLC we first define an Interface in the PCL that describes the contract the implementation has to implement:


    public interface ISystemInformationHandler
    {
        string OSVersion { get; }
    }


In the concrete platforms we can then inherit the interface and implement the handler which calls the platform specific API (bellow you see the Android implementation):


    internal class SystemInformationHandler:ISystemInformationHandler
    {
        #region Implementation of ISystemInformationHandler
    
        //public string OSVersion => System.Environment.OSVersion.ToString();
        public string OSVersion => $"Android {Android.OS.Build.VERSION.SdkInt} {Android.OS.Build.VERSION.Release}";
    
        #endregion
    }


Now in the PCL we can have a class for example a view model that requires the handler to display the information.


    public class MainViewModel : ViewModelBase
    {
        private readonly ISystemInformationHandler _systemInformationHandler;
    
        public MainViewModel(ISystemInformationHandler systemInformationHandler)
        {
            if (systemInformationHandler == null) throw new ArgumentNullException(nameof(systemInformationHandler));
            _systemInformationHandler = systemInformationHandler;
        }
    
        public string OSVersion => _systemInformationHandler.OSVersion;
    }


The view model requires as a parameter argument an instance of the <font face="Consolas">IOSVersionHandler</font> interface. So when we create an instance of the <font face="Consolas">MainViewModel</font> in the platform project, we can simply create an instance of interface and pass it into the view model when we create it.


    var mainViewModel = new MainViewModel(new SystemInformationHandler());


And receive the following output for e.g. Android:



# Cross platform Dependency resolution with MVVM Lights

Now resolving all of these dependencies by hand can get very tedious over time. You might have noticed that I’m using MVVM Light which comes with a lightweight Dependency Injection Container called *Simple IoC*. Lets see how we can use this component to further streamline our cross platform development efforts.


> **Note: **for an introduction to MVVM Light please refer to this blog post for <font style="background-color: #ffff00"></font><font style=""><a title="Android MVVMLight Introduction" href="https://mallibone.com/post/xamarinandroid-and-mvvm-light-bindings" target="_blank"><font style="">Android</font></a><font style=""> or </font><a title="iOS MVVM Light introduction" href="https://mallibone.com/post/mvvm-light-bindings-under-xamarin.ios" target="_blank"><font style="">iOS</font></a> </font><font style="background-color: #ffff00"></font>to get you started with the basics.


In the PCL we have the <font face="Consolas">ViewModelLocator.cs</font> which defines all of the instances used, but without the platform specific resolution.


    public class ViewModelLocator
    {
        public ViewModelLocator()
        {
            ServiceLocator.SetLocatorProvider(() => SimpleIoc.Default);
    
            if (ViewModelBase.IsInDesignModeStatic)
            {
                // Create design time view services and models
                SimpleIoc.Default.Register<ISystemInformationHandler, SystemInformationHandlerStub>();
            }
    
            SimpleIoc.Default.Register<MainViewModel>();
        }
    
        public MainViewModel Main
        {
            get
            {
                return ServiceLocator.Current.GetInstance<MainViewModel>();
            }
        }
    }


On each platform we now extend the Locator and register the platform specific classes i.e. resolution for interfaces.


    internal class Locator : ViewModelLocator
    {
        private static readonly Lazy<Locator> _locator = new Lazy<Locator>(() => new Locator(), LazyThreadSafetyMode.PublicationOnly);
        public static Locator Instance => _locator.Value;
    
        private Locator()
        {
            SimpleIoc.Default.Register<ISystemInformationHandler, SystemInformationHandler>();
        }
    }


Now we can simply request an instance of the view model (MVVM Light per default only creates one instance for each registered class) and use it. In Android this would look something like this in the <font face="Consolas">MainActivity.cs</font>:


    [Activity(Label = "PclDISample.Droid", MainLauncher = true, Icon = "@drawable/icon")]
    public class MainActivity : Activity
    {
        #region UI Controls
    
        private TextView OSVersion => FindViewById<TextView>(Resource.Id.OSVersion);
    
        #endregion
    
        private MainViewModel VM => Locator.Instance.Main;
    
        protected override void OnCreate(Bundle bundle)
        {
            base.OnCreate(bundle);
    
            // Set our view from the "main" layout resource
            SetContentView(Resource.Layout.Main);
    
            // Get our button from the layout resource,
            // and attach an event to it
            OSVersion.Text = VM.OSVersion;
    
            var mainViewModel = new MainViewModel(new SystemInformationHandler());
        }
    }


In Windows 10 we can simply set the <font face="Consolas">Datacontext</font> accordingly in XAML:


    <Page
        x:Class="PclDISample.UWP.MainPage"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="using:PclDISample"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
      DataContext="{Binding Source={StaticResource Locator}, Path=Main}"
        mc:Ignorable="d">
    
        <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
            <TextBlock Text="{Binding OSVersion}" VerticalAlignment="Center" HorizontalAlignment="Center" Style="{StaticResource TitleTextBlockStyle}"></TextBlock>
        </Grid>
    </Page>


As you can see once all is registered with the simple *Inversion of Control* Container (IoC) of MVVM Light all of the dependency resolution will be performed automatically. If a dependency is missing, an exception will be thrown at runtime (**not** during compilation), so make sure to test all your views, user controls et al that they load correctly.

# Conclusion

In this post you saw how you can use Dependency Injection to provide platform specific features to cross platform code located in the PCL without creating any circular dependency errors. By using interfaces in the PCL and injecting the concrete implementations during runtime we can create thin wrappers that provide platform specific features through a generic defined interface. Finally we saw how we can use MVVM Light to create a more manageable and more manageable dependency resolution with the Simple IoC Locator.

You can find the complete sample on [GitHub](https://github.com/mallibone/PclDISample.git "Link to sample code on github").

Big thanks to [Laurent Bugnion](http://www.galasoft.ch/) the creator the MVVM Light.
