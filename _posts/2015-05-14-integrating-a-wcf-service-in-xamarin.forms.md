---
layout: single
title: "Integrating a WCF service in Xamarin.Forms"
title: Integrating a WCF service in Xamarin.Forms
date: 2015-05-14
tags: ["Xamarin.Forms", "Windows Phone", "iOS", "Android", "Mobile"]
slug: "integrating-a-wcf-service-in-xamarin.forms"
---

[![WCFXamarinTitleLogo]({{ site.url }}{{ site.baseurl }}/images/db8355f4-f027-43cb-a945-f4c5607e753a.png "WCFXamarinTitleLogo")]({{ site.url }}{{ site.baseurl }}/images/5e993793-b8b7-4b95-92d5-ddd6de345fa4.png)

Windows Communication Foundation (WCF) used to be the way how web services were created in .Net. Allegedly it comes at no surprise that many backend services are implemented with WCF and therefore if you are in the business of writing mobile clients may face the task of integrating such a service. In this post we will look at how a WCF service can be integrated into a Xamarin.Forms app and how Begin/End async methods can be wrapped into task based async/await methods which are the defacto standard since C# 5/.Net 4.5.

In this blog post we will look at the following:

1. Looking WCF web service that can run on IIS or Azure
2. Creating the Xamarin.Forms App
    1. Add UI for interaction
    2. Add service reference in the Portable Class Library (PCL)
3. Mapping to the new async/await model


# The basic setup

Lets first have a look at how the app and the web service are setup before we do the final wiring up of the service.

## Looking at the WCF web service

The WCF service I’m using for this sample code, which you can find as usual on [GitHub](https://github.com/), is a WCF Application. A WCF Application can run on IIS and be easily deployed to Azure which makes it perfect for getting up and running in no time without having to mess with IIS Express Firewall settings.

The service we will be integrating has a single method which will take an *int* and return a *string*, here is the interface definition:


    [ServiceContract]public interface IWcfService{    [OperationContract]    string GetData(int value);}


And the implementation is equally simple:


    public class WcfService : IWcfService{    public string GetData(int value)    {        return string.Format("You entered: {0}", value);    }}


So as you can see the service method **GetData** takes an *int* wraps it in a *string* and will send it back to the caller. Now lets have a look at the client and the UI.

## Xamarin.Forms project setup

Now on the client we will setup a XAML based Xamarin.Forms application with a PCL as core library. The app will be a single page app that displays a **Picker**, **Button** and **Label**control which will look like this on XAML:


    <?xml version="1.0" encoding="utf-8" ?><ContentPage xmlns="http://xamarin.com/schemas/2014/forms"             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"             x:Class="WcfIntegrationXamarinForms.View.MainPage">  <StackLayout VerticalOptions="Center" HorizontalOptions="Center">    <Picker x:Name="NumberPicker" SelectedIndex="{Binding SelectedNumberIndex}">      <Picker.Items>        <x:String>1</x:String>        <x:String>2</x:String>        <x:String>3</x:String>        <x:String>4</x:String>        <x:String>5</x:String>        <x:String>6</x:String>        <x:String>7</x:String>        <x:String>8</x:String>        <x:String>9</x:String>        <x:String>10</x:String>      </Picker.Items>    </Picker>    <Button Text="Submit" Command="{Binding SubmitCommand}"></Button>    <ActivityIndicator IsRunning="{Binding IsCallingWebservice}"></ActivityIndicator>    <Label Text="{Binding Response}"  />  </StackLayout></ContentPage>


Notice that the **Picker** does not allow to set the Items programmatically and only provides the **SelectedIndex** to get information on the selected item. Now at the back of the view we will be using a ViewModel based on [MVVM Light](http://www.mvvmlight.net/), to which you can find a basic introduction [here](http://www.mallibone.com/post/xamarin.forms-and-mvvm). The view model manages the inputs from the view calls the **IWcfServiceWrapper** and pushes the result back to the UI. Here is the implementation of the View Model:


    public class MainViewModel : ViewModelBase{    private readonly IWcfServiceWrapper _wcfServiceWrapper;    private string _response;    private readonly IList<string> _numbers;    private bool _isCallingWebservice;    /// <summary>    /// Initializes a new instance of the MainViewModel class.    /// </summary>    public MainViewModel(IWcfServiceWrapper wcfServiceWrapper)    {        if (wcfServiceWrapper == null) throw new ArgumentNullException("wcfServiceWrapper");        _wcfServiceWrapper = wcfServiceWrapper;        _numbers = new List<string> {"1", "2", "3", "4", "5", "6", "7", "8", "9", "10"};        Response = "Pick a number and click submit";        SubmitCommand = new RelayCommand(GetResponseAsync, () => CanGetResponse);        CanGetResponse = true;    }    private async void GetResponseAsync()    {        CanGetResponse = false;        IsCallingWebservice = true;        Response = await _wcfServiceWrapper.GetDataAsync(Int32.Parse(_numbers[SelectedNumberIndex]));        CanGetResponse = true;        IsCallingWebservice = true;    }    public int SelectedNumberIndex { get; set; }    public bool CanGetResponse { get; set; }    public ICommand SubmitCommand { get; set; }    public bool IsCallingWebservice    {        get { return _isCallingWebservice; }        set        {            if (value == _isCallingWebservice) return;            _isCallingWebservice = value;            RaisePropertyChanged(() => IsCallingWebservice);        }    }    public string Response    {        get { return _response; }        set        {            if (_response == value) return;            _response = value;            RaisePropertyChanged(() => Response);        }    }}


We simply implement the command which enables the **ActivityIndicator** to show a progress bar while we are waiting for our response.

# Integrating the service

We will integrate the WCF service in the PCL which will allow us to use the interface for each platform. This approach is also valid when you develop for one single platform such as Windows (Phone), Android or iOS, so I really recommend you choose to implement your service in a PCL even if for now you are only targeting one platform. Adding WCF services i.e. creating the proxy classes and calls has always been a very trivial process and hasn’t changed in the modern world. Expand the PCL project, right click on **References** and select **Add Service Reference…**enter the URL of your web service and click **OK**.

[![AddServiceReference]({{ site.url }}{{ site.baseurl }}/images/e428c6ac-770d-4d35-8ff8-9895ce99f194.png "AddServiceReference")]({{ site.url }}{{ site.baseurl }}/images/7c61bfd8-2584-4a99-91bb-60994fc2b815.png)

This will generate all the necessary proxy classes and definitions so you can call the Webservice. Unfortunately though all the asynchronous are implemented with the Begin/End asynchronous pattern. Luckily we can easily wrap those calls and use the newer async/await model of asynchronous programming with C#.

# Mapping to the new async/await model

With the Task library we get the **TaskFactory**class which has a **[FromAsync](http://msdn.microsoft.com/query/dev12.query?appId=Dev12IDEF1&amp;l=EN-US&amp;k=k%28&quot;System.Threading.Tasks.TaskFactory.FromAsync``2&quot;%29;k%28TargetFrameworkMoniker-.NETPortable,Version%3Dv4.5%29;k%28DevLang-csharp%29&amp;rd=true)** method which lets us wrap Begin/End async methods into a single awaitable task based asynchronous call. So we can add a **WcfServiceWrapper.cs** to our portable project which wraps the call as follows:


    public class WcfServiceWrapper:IWcfServiceWrapper{    private IWcfService _wcfService;    public WcfServiceWrapper(IWcfService wcfService)    {        if (wcfService == null) throw new ArgumentNullException("wcfService");        _wcfService = wcfService;    }    public async Task<string> GetDataAsync(int number)    {        Task<string> getDataTask = new TaskFactory().FromAsync<int,string>(_wcfService.BeginGetData, _wcfService.EndGetData, number, null, TaskCreationOptions.None);        return await getDataTask;    }}


# Conclusion

We looked at how a simple WCF service can be integrated in a Xamarin.Forms mobile app. Further the post shows you how you can wrap the Begin/End asynchronous pattern into todays async/await task based async pattern.

You can find the entire project on [GitHub](https://github.com/mallibone/WcfIntegrationXamarinForms.git).

This post was originally posted on the [Noser Blog](http://blog.noser.com/integrating-a-wcf-service-in-xamarin-forms).



Technorati Tags: [Xamarin.Forms](http://technorati.com/tags/Xamarin.Forms),[WCF](http://technorati.com/tags/WCF),[Xamarin.iOS](http://technorati.com/tags/Xamarin.iOS),[Xamarin.Android](http://technorati.com/tags/Xamarin.Android),[Windows Phone](http://technorati.com/tags/Windows+Phone),[Windows Phone Silverlight](http://technorati.com/tags/Windows+Phone+Silverlight),[WP8 silverlight](http://technorati.com/tags/WP8+silverlight)
