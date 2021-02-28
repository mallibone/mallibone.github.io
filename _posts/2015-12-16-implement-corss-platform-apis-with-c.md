---
layout: single
title: "Implement Cross Platform APIs with C#"
title: Implement Cross Platform APIs with C#
date: 2015-12-16
tags: ["Mobile", "UWP", "Windows", "Xamarin", "Android", "iOS"]
slug: "implement-corss-platform-apis-with-c"
---

See how you can offer cross platform APIs based on C# that you can consume in Universal Windows Platform Apps (UWP), Xamarin.iOS, Xamarin.Android, Xamarin.Forms, ASP.Net and even your classic .Net WPF app. In my last [post](https://mallibone.com/post/accessing-platform-specific-features-in-your-portable-class-library-pcl-through-dependency-injection-di) I showed how one can use platform specific libraries and code in a Portable Class Library with dependency injection. This approach is great when you are developing your own app but it gets somewhat quirky, when you want to offer a library to fellow co-workers/developers, as you force them to setup your own library through dependency injection. But there is of course another way that allows you to encapsulate your implementation details neatly while still providing platform specific features to your PCL code.

The library in this blog post will provide the OS version for a UWP, Xamarin.iOS and Xamarin.Android app.

# Setting up the library

The library consists of four projects:

- <font face="Consolas">OSVersion.Core</font>
- <font face="Consolas">OSVersion.UWP</font>
- <font face="Consolas">OSVersion.Droid</font>
- <font face="Consolas">OSVersion.iOS</font>


One thing which is a bit special about all the projects is that all of their Default Namespace and the Assembly name all have the same name.

[![NamespaceBinaryName](https://mallibone.com/posts/files/307903e9-e59a-44ad-8b15-6d2a32744bb9.png "Shows the assembly name and default namespace set to OsVersionAPI.Core")](https://mallibone.com/posts/files/d8a1c0b1-6975-484d-b15f-113de2bc0e63.png)

In each project we implement the <font face="Consolas">SystemInformationHandler.cs</font> class, on the PCL we simply implement a stub:


    public class SystemInformationHandler
    {
        #region Implementation of ISystemInformationHandler
    
        public static string OSVersion => "Gnabber";
    
        #endregion
    }


For the other platforms such as UWP we implement the calls to the UWP specific APIs:


    public class SystemInformationHandler
    {
        public static string OSVersion
        {
            get { return OsVersionString(); }
        }
    
        private static string OsVersionString()
        {
            string sv = AnalyticsInfo.VersionInfo.DeviceFamilyVersion;
            ulong v = ulong.Parse(sv);
            ulong v1 = (v & 0xFFFF000000000000L) >> 48;
            ulong v2 = (v & 0x0000FFFF00000000L) >> 32;
            ulong v3 = (v & 0x00000000FFFF0000L) >> 16;
            ulong v4 = (v & 0x000000000000FFFFL);
    
            return $"Windows {v1}.{v2}.{v3}.{v4}";
        }
    }


So in the end we have implemented the generic class in the PCL and specific classes for each platform we target to support. Now lets see how we can consume this library in an app.

# Consuming the library

The app will exist of the usual cross platform mobile app setup which means we will have usual suspects covering Windows, Android and iOS with a Portable Class Library aka Core to share the reusable code.

[![NativePclProjectOverview](https://mallibone.com/posts/files/65f7e8f5-fba3-4fac-9ed3-7ed379232d61.png "NativePcl Project overview")](https://mallibone.com/posts/files/11ad1c26-d030-4412-928b-1e9f7a2f9ace.png)

In the core we have our business logic code and view models (I’m a huge fan of the MVVM Light library, to find out more just view my [mvvm light posts](https://mallibone.com/category/mvvm+light)). In the NativePcl.Core project we have a service <font face="Consolas">OsVersionService.cs</font> for returning the version number:


    public class OsVersionService : IOsVersionService
    {
        public string GetVersion()
        {
            return SystemInformationHandler.OSVersion;
        }
    }


Reference the OSVersion.Core library as binary – DO NOT reference it as project or you will be facing compile errors in one of the next steps. Don’t ask me how I know ![Winking smile](https://mallibone.com/posts/files/d308138c-2d39-4319-9975-7826ec2e971e.png)


> Because all our assemblies of the OSVersion library will be compiled to assemblies with the same name we can’t just simply reference the project file but have to add the binary i.e. the compiled version of the library. If you are looking at the sample from GitHub, make sure to compile the solution under release build as the OSVersion is referenced via binary in the release output folder of the projects.


The method invokes the <font face="Consolas">OSVersion</font> property in the OSVersion library - leaving out the specific project e.g. UWP intent fully here. On the clients for example the Android app we have a single screen showing the version number defined in the <font face="Consolas">MainView.xaml</font>:


    <Page
        x:Class="NativePcl.UWP.MainPage"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="using:NativePcl.UWP"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        DataContext="{Binding Source={StaticResource Locator}, Path=Main}"
        mc:Ignorable="d">
    
        <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
            <TextBlock Text="{Binding OSVersion}" VerticalAlignment="Center" HorizontalAlignment="Center" Style="{StaticResource TitleTextBlockStyle}"></TextBlock>
        </Grid>
    </Page>


And the corresponding code behind or as we are using view models <font face="Consolas">MainViewModel.cs</font>:


    public class MainViewModel : ViewModelBase
    {
        private readonly IOsVersionService _osVersionService;
    
        public MainViewModel(IOsVersionService osVersionService)
        {
            if (osVersionService == null) throw new ArgumentNullException(nameof(osVersionService));
            _osVersionService = osVersionService;
        }
    
        public string OSVersion => _osVersionService.GetVersion();
    }


If we now run the app we will get the follow output:

[![coreVersion](https://mallibone.com/posts/files/69028bb5-5621-4546-a3d1-24e94e12bfd3.png "coreVersion of the OS version")](https://mallibone.com/posts/files/614f7d6c-241e-4d47-a09a-dd19bb73a062.png)

Now this is the output we defined in the <font face="Consolas">OSVersion.Core</font>, if we now reference the binary of the <font face="Consolas">OSVersion.UWP</font> in the <font face="Consolas">NativePcl.UWP</font> project things get really interesting. Instead of receiving an error because we are adding two assemblies with the same name the compiler will prefer the platform specific library over the PCL implementation. And even though we never invoke the library directly from our <font face="Consolas">NativePcl.UWP</font> app the OS version returned now is returned from the platform specific i.e. <font face="Consolas">OSVersion.UWP</font> implementation of the <font face="Consolas">GetVersion()</font> method resulting in the following output:

[![uwpVersion](https://mallibone.com/posts/files/df98e477-49f3-4552-9e18-3e51b246b998.png "UWP version information screenshot")](https://mallibone.com/posts/files/50f060de-8b98-46d1-bd79-e51883be369a.png)

Now while this approach allows for writing platform independent logic in the PCL and calling platform specific implementations during runtime it is somewhat tedious as it requires that the developer will reference the correct version of the library. And all of them have the same name which is just calling for some frustrated minutes to happen due to referencing the wrong version of the library. Luckily there is a technique that allows to use the approach described above without burdening the developer consuming the library of referencing the binaries correctly. [By wrapping the library in a NuGet package](https://mallibone.com/post/wrapping-c-cross-platform-libraries-in-a-nuget-package) consuming the library is as simple as adding the package to all of your projects and the NuGet package manager will just like the build engine prefer to link the platform specific binary over the PCL library to the project. So instead of having to choose which binary to add the developer can now simply add the identical NuGet package to all of the projects and it will just work.

# Conclusion

In this post you saw how to create APIs that can be consumed in the PCL and still provide platform specific functionality. For all of this to work we had to set the namespace and assembly of the projects to identical names. On the consumer one can either simply add a NuGet package containing all of the implementations i.e. PCL, UWP, Android and iOS or add the binaries of the corresponding API compilation to the app implementation e.g. <font face="Consolas">OSVersion.Core</font>to <font face="Consolas">NativePcl.Core</font>.

This approach allows you to create APIs which can be reused across multiple platforms providing the same method definitions across the platforms.

You can find the sample on [GitHub](https://github.com/mallibone/NativePclSample). (make sure to compile it first as a release build)
