---
layout: single
title: "C# code everywhere thanks to the Portable Class Library"
title: C# code everywhere thanks to the Portable Class Library
date: 2015-10-26
tags: ["Android", "UWP", "Windows", "Windows Phone", "Xamarin", "iOS", "MVVM Light"]
slug: "c-code-everywhere-thanks-to-the-portable-class-library"
---

These days if you can write code with C# you can write code for Windows, Linux and OSX, your code can run on Desktop, Server, Mobile or Embedded Devices. So C# runs pretty much everywhere which if you are a C# developer is a great thing! ![Smile]({{ site.url }}{{ site.baseurl }}/assets/images/96cb8970-8084-4ab4-ac6b-0e01dbabf7f3.png) Only little hiccup, even though C# is a standardised language, there are multiple runtimes existing today on which C# runs. This may lead to the assumption of having to rewrite code for every runtime. As the standard .Net C# library can not be reused on a Silverlight runtime, or the new Windows RT runtime, or I think you get the message. But fear not – thanks to the Portable Class Library (PCL) one is able to write code that can be reused over multiple platforms.

# Portable Class Libraries

When adding a Portable Class Library (PCL) you will have to choose which platforms you want to use the platform on. The supported platforms have an impact in the functionality the PCL provides without adding any additional libraries (usually via NuGet). You can find a list  of libraries supported by each platform on [MSDN](https://msdn.microsoft.com/en-us/library/gg597391%28v=vs.110%29.aspx).

For this post we will want to reuse our code on the following platforms:

- Universal Windows App (Windows 10)
- Classic desktop WPF app
- iOS (with Xamarin)
- Android (with Xamarin)


To keep things simple the logic in the app will be creating a list of people. We will even be able to share our view models from the PCL so the only overhead we will be having when creating the app is the actual UI code and wiring up the view models to the actual views. Now in this setup the timesaving's might seem marginal but think about your standard business app and the savings will start to up add up quickly in a major way.

# Creating a PCL

Adding the platform specific projects is pretty straight forward. Simply add the corresponding project template to the solution when adding a new project, so I’ll leave it at that and go over to adding a PCL. When adding the PCL a dialog will be opened offering to choose the platforms that shall be supported i.e. can consume the PCL.

[![Showing Add Portable Class Library Visual Studio dialog with .Net Framework 4.6, Windows Universal 10.0, Xamarin.Android and Xamarin.iOS selected]({{ site.url }}{{ site.baseurl }}/assets/images/d883359b-3484-4e89-a41b-57500d4b5462.png "Showing Add Portable Class Library Visual Studio dialog with .Net Framework 4.6, Windows Universal 10.0, Xamarin.Android and Xamarin.iOS selected")]({{ site.url }}{{ site.baseurl }}/assets/images/c6637561-1e3f-4139-8bc6-9326dda1ad79.png)


> The iOS and Android option will show up after installing the Xamarin tool chain.


With the platforms chosen we can now start implementing the “business logic” which in our case is a simple generator that we invoke, called <font face="Consolas">PersonService.cs</font>:


    public class PersonService
    {
        public async Task<IEnumerable<Person>> GetPeople(int count = 42)
        {
            var people = new List<Person>(count);
            // In case of large counts lets be safe and not run this on the UI thread
            await Task.Run(() =>
            {
                for (int i = 0; i < count; ++i)
                {
                    people.Add(new Person{FirstName = NameGenerator.GenRandomFirstName(), LastName = NameGenerator.GenRandomLastName()});
                }
            });
    
            return people;
        }
    }


To present our data to the UI we will use the MVVM pattern.

## Adding MVVM Light

When using MVVM in a project (which is like always for me ![Winking smile]({{ site.url }}{{ site.baseurl }}/assets/images/e9a9d73c-7bab-4c6b-80b6-05f750bd80da.png)) I prefer using MVVM Light. MVVM Light can be added by simply installing a [NuGet](https://www.nuget.org/ "link to nuget page") package:


    PM> Install-Package MvvmLightLibs



> As we will access the view models from the platforms make sure to add the MVVM Light libraries also to the platform specific code bases.


Now we can add the view model <font face="Consolas">MainViewModel.cs</font> that will hold the list of people we will display.


    public class MainViewModel:ViewModelBase
    {
        private readonly IPersonService _personService;
    
        public MainViewModel(IPersonService personService)
        {
            if (personService == null) throw new ArgumentNullException(nameof(personService));
            _personService = personService;
            if (IsInDesignMode)
            {
                People = new ObservableCollection<Person> {new Person{FirstName = "Doctor Who"}, new Person {FirstName = "Rose", LastName = "Tyler"}, new Person {FirstName = "River", LastName = "Song"} };
            }
            else
            {
                People = new ObservableCollection<Person>();
            }
        }
    
        public ObservableCollection<Person> People { get; set; }
    
        public async Task InitAsync()
        {
            var people = await _personService.GetPeople();
            People.Clear();
            foreach (var person in people)
            {
                People.Add(person);
            }
        }
    }


Next we still need to configure the <font face="Consolas">ViewModelLocator.cs</font>, which allows us to easily manage dependencies.


    public class ViewModelLocator
    {
        public ViewModelLocator()
        {
            ServiceLocator.SetLocatorProvider(() => SimpleIoc.Default);
    
            SimpleIoc.Default.Register<IPersonService, PersonService>();
            SimpleIoc.Default.Register<MainViewModel>();
        }
    
        public MainViewModel MainViewModel => SimpleIoc.Default.GetInstance<MainViewModel>();
    }


So now we have all of our cross platform code setup. Let’s get it running on our desired targets.

# The platforms

On the platforms we can simple reference the PCL like any other library we might use and access the features. On our platforms we will have to do the following steps:

1. Create the UI
2. Hook up the<font face="Consolas">ViewModelLocator.cs</font>
3. Wire up the view model to the view


Let’s do these steps for the Universal Windows App.

## Universal Windows App (UWA)

For the view we will simply add a <font face="Consolas">ListView</font> to the <font face="Consolas">MainPage.xaml</font>:


    <Page
        x:Class="PclSample.UWA.MainPage"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="using:PclSample.UWA"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        DataContext="..."
        mc:Ignorable="d">
    
        <ListView ItemsSource="{Binding People}">
            <ListView.ItemTemplate>
                <DataTemplate>
                    <StackPanel Orientation="Horizontal">
                    <TextBlock Text="{Binding FirstName}"/>
                    <TextBlock Text="{Binding LastName}"/>
                    </StackPanel>
                </DataTemplate>
            </ListView.ItemTemplate>
        </ListView>
    </Page>


Under Windows (this is also true for WPF) MVVM Light really integrates really nicely. The Locator we can simply add to the <font face="Consolas">App.xaml</font>:


    <Application
        x:Class="PclSample.UWA.App"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="using:PclSample.UWA"
        RequestedTheme="Light">
        <Application.Resources>
            <vm:ViewModelLocator x:Key="Locator" xmlns:vm="using:PclSample.Core" />
            <ResourceDictionary />
        </Application.Resources>
    </Application>


This allows us to access properties from the <font face="Consolas">ViewModelLocator.cs</font> class within our view(s) i.e. <font face="Consolas">MainPage.xaml</font>. So the next step would be wiring up the view model to the views <font face="Consolas">DataContext</font>, which will populate the view:


    <Page
        x:Class="PclSample.UWA.MainPage"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="using:PclSample.UWA"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        DataContext="{Binding Source={StaticResource Locator}, Path=MainViewModel}"
        mc:Ignorable="d">
    
        ...
    </Page>


Now all is set and we can enjoy the view of our light weight UWA.

[![Screenshot of the UWA running in the Windows 10 Mobile Emulator]({{ site.url }}{{ site.baseurl }}/assets/images/3a33da57-4c43-46c9-8b07-b0acaca834bb.png "Screenshot of the UWA running in the Windows 10 Mobile Emulator")]({{ site.url }}{{ site.baseurl }}/assets/images/502e66e6-7b21-40c9-85ef-7602852b6093.png)

## Windows Presentation Foundation (WPF)

Following the same steps as in the UWA in a WPF app will lead to the same result.

[![Shows sample app running in as a WPF desktop application]({{ site.url }}{{ site.baseurl }}/assets/images/c7a64078-ad28-496b-b40b-ca006407121b.png "Shows sample app running in as a WPF desktop application")]({{ site.url }}{{ site.baseurl }}/assets/images/026d1274-3a5b-4e95-ae6b-e383c305df77.png)

So we can reuse all of our business logic from one Microsoft client development model to an other one. Isn’t that just great? ![Smile]({{ site.url }}{{ site.baseurl }}/assets/images/96cb8970-8084-4ab4-ac6b-0e01dbabf7f3.png) But wait there is more. We can take the C# code outside of the Microsoft Ecosystem. With for example Xamarin we can reuse our C# code on iOS and Android.

## iOS

Under iOS the UI is usually created in a designer (similar to Windows Forms) so there is not much code to show. We add the <font face="Consolas">ListView</font> equivalent from iOS a <font face="Consolas">UITableView</font> to a page and give it a name, in this sample *PeopleTableView*.

[![Showing iOS Designer and where the name of the UITableView is set.]({{ site.url }}{{ site.baseurl }}/assets/images/f4df40a1-1994-4848-a1f6-a41959d91096.png "Showing iOS Designer and where the name of the UITableView is set.")]({{ site.url }}{{ site.baseurl }}/assets/images/eca8f591-ead3-4d23-aa45-b81ae6158ca5.png)

Setting up the <font face="Consolas">ViewModelLocator.cs</font> is not quite as elegant when we leave the Microsoft platforms but then again it is done just as easily. In the <font face="Consolas">AppDelegate.cs</font> which is the start up point of every iOS app we simply create an Instance of the <font face="Consolas">ViewModelLocator.cs</font> and store it in a property. The property then can be accessed throughout the iOS app. The Wiring up of the view model is done in the <font face="Consolas">ViewController.cs</font> which is linked to the view that we setup earlier:


    public partial class ViewController : UIViewController
    {
        ObservableTableViewController<Person> _tableViewController;
    
        public ViewController (IntPtr handle) : base (handle)
        {
        }
    
        private MainViewModel Vm => Application.Locator.MainViewModel;
    
        public override void ViewDidLoad ()
        {
            base.ViewDidLoad ();
            // Perform any additional setup after loading the view, typically from a nib.
    
            _tableViewController = Vm.People.GetController(CreatePersonCell, BindPersonCell);
            _tableViewController.TableView = PeopleTableView;
        }
    
        public override async void ViewWillAppear (bool animated)
        {
            base.ViewWillAppear (animated);
            await Vm.InitAsync();
        }
    
        public override void DidReceiveMemoryWarning ()
        {
            base.DidReceiveMemoryWarning ();
            // Release any cached data, images, etc that aren't in use.
        }
    
        UITableViewCell CreatePersonCell (Foundation.NSString reuseId)
        {
            var cell = new UITableViewCell(UITableViewCellStyle.Default, null);
            return cell;
        }
    
        void BindPersonCell (UITableViewCell cell, Person person, Foundation.NSIndexPath path)
        {
            cell.TextLabel.Text = person.FullName;
        }
    }


Now all that is left to do for us is again fire up the iOS Simulator to verify all is correct and there we have it, the same C# code we used for our WPF and UWA apps is now running under iOS:

[![iphone]({{ site.url }}{{ site.baseurl }}/assets/images/38b3c252-301f-430f-a935-a746e7065bc1.png "iphone")]({{ site.url }}{{ site.baseurl }}/assets/images/9196fc2c-f978-4d18-a62b-5a8bcf36a992.png)

## Android

What we did under iOS we can also replicate under Android. Though not part of the Microsoft eco system the model should be more familiar if you have written WPF, Modern Apps (and all the other fancy names they had i.e. Universal Apps). Under Resources/Layouts there is the <font face="Consolas">main.axml</font> which is a (Application) XML file we can edit to tell the Android OS how to render the UI. Sounds a bit like XAML right? And the concepts do overlap in some parts too. Adding a <font face="Consolas">ListView</font> to the XML and set the id of the element which will allow us to access it later on:


    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="vertical"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent">
        <ListView
            android:layout_width="fill_parent"
            android:layout_height="fill_parent"
            android:id="@+id/PeopleListView" />
    </LinearLayout>


Again we will have to setup the locator. Under Android there is no fixed setup point, so we will use a static class <font face="Consolas">Locator.cs</font> and property which we will access from the Android app:


    internal static class Locator
    {
        private static readonly Lazy<ViewModelLocator> _locator = new Lazy<ViewModelLocator>(() => new ViewModelLocator());
        public static ViewModelLocator Instance => _locator.Value;
    }


Wiring up the view model is done in the <font face="Consolas">MainActivity.cs</font>, which you can think of as the code behind/ViewController equivalent. I wrote about this binding in more detail in a former blog post which you can find here.


    [Activity(Label = "PclSample.Droid", MainLauncher = true, Icon = "@drawable/icon")]
    internal class MainActivity : Activity
    {
        protected override async void OnCreate(Bundle bundle)
        {
            base.OnCreate(bundle);
    
            // Set our view from the "main" layout resource
            SetContentView(Resource.Layout.Main);
    
            await Vm.InitAsync();
    
            // Get our button from the layout resource,
            // and attach an event to it
            PeopleListView.Adapter = Vm.People.GetAdapter(GetPersonView);
        }
    
        public ListView PeopleListView => FindViewById<ListView>(Resource.Id.PeopleListView);
    
        private MainViewModel Vm => Locator.Instance.MainViewModel;
    
        private View GetPersonView(int position, Person person, View convertView)
        {
            View view = convertView ?? LayoutInflater.Inflate(Android.Resource.Layout.SimpleListItem1, null);
    
            view.FindViewById<TextView>(Android.Resource.Id.Text1).Text = person.FullName;
    
            return view;
        }
    }



> Note that the id we previously set allows us to retrieve the UI control.


At this point we can start up the Android Emulator and just to have mentioned it Visual Studio 2015 provides a great Android Emulator – way better then the stock emulator! And once again we can see the app is running reusing the C# code we tucked away in our PCL.

[![Shows app running in the Android Emulator]({{ site.url }}{{ site.baseurl }}/assets/images/6a0d8429-7bba-4454-9be2-0236b20206a6.png "Shows app running in the Android Emulator")]({{ site.url }}{{ site.baseurl }}/assets/images/4d7d7498-875e-4301-b7df-51ef274c86a2.png)

# Conclusions

In this blog post we saw how one can share C# code with a Portable Class Library (PCL). The PCL can be included in all major C# stacks .Net, UWP, Silverlight, Xamarin etc. and though the capabilities in the PCL are more limited then a standard .Net library it does allow to share code from a legacy WPF/Win Forms app to a mobile application.

PCLs can be extended by installing further NuGet packages such as storage, network access and many more. The PCL has been around for a couple of years now and has proven itself to be a valuable component when writing cross platform applications.

The entire sample can be found on [GitHub](https://github.com/mallibone/PclSample.git "Link to sample code on github").
