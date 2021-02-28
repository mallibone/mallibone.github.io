---
layout: single
title: "Consuming an Azure hosted ASP.Net WebAPI service with Xamarin.Forms"
title: Consuming an Azure hosted ASP.Net WebAPI service with Xamarin.Forms
date: 2015-05-27
tags: ["Xamarin.Forms", "Mobile", "Azure"]
slug: "consuming-an-azure-hosted-asp.net-webapi-service-with-xamarin.forms"
---

[![RESTXamarintTitleLogo]({{ site.url }}{{ site.baseurl }}/images/85978cd3-e10e-4321-95b0-3c2fb7f79ce0.png "RESTXamarintTitleLogo")]({{ site.url }}{{ site.baseurl }}/images/c7cb845c-a8c6-4863-8fb2-96cd9dc0c824.png)
 
Lets have a look at how we can consume a REST based web service in a Xamarin.Forms application. In this blog post we will look at how an ASP.Net WebAPI service that is hosted on Azure and how it can be consumed in the Portable Class Library (PCL) of your Xamarin.Forms application and write a small UI to interact with the REST service. Please note that I used Visual Studio 2013 while creating the example for this project.
 

 
# Creating the backend
 
An ASP.Net WebAPI can be considered as a headless website, so many of the concepts of ASP.Net MVC apply but we will not be writing any HTML, JavaScript and CSS code for displaying the data as it will simply provide the data through the controller. Per default the controller can return the data serialized to JSON or XML. For this example I have created two controllers, one will just simply return a string and return it as the content of the HTTP package:


    public class BasicController : ApiController{    // GET: api/Basic/5    public string Get(int id)    {        return string.Format("You entered: {0}", id);    }}


The second one will return a serialized object to the caller. The default settings will request the data to be serialized with JSON but we will come to that later on. The second controller looks as follows:


    public class ObjectController : ApiController{    // GET: api/Object/5    public NumberWrapper Get(int id)    {        return new NumberWrapper{WrappedNumber =  string.Format("You entered: {0}", id)};    }}


And the Plain Old CLR Object (POCO) which is used you can see here:


    public class NumberWrapper{    public string WrappedNumber { get; set; }}


So all pretty straight forward, now lets look at how we can consume the service once it is running.


> Note: You can publish this webservice on Azure which makes it really easy to access it from phones and other Virtual Machines (VMs) as you will not be hassled with firewall settings and therefore can focus at the task at hand.




# Consuming a REST service in the PCL

For consuming the REST service we will need to install the WebHttpClient [NuGet](https://www.nuget.org/) package from Microsoft. So lets open up the package manager by right-clicking on the solution and then searching for WebHttpClient, add the NuGet package to the PCL, Android, iOS and Windows Phone Project. For deserializing the JSON we will use the [JSON.Net](http://james.newtonking.com/json) library from [James Newton-King](https://twitter.com/JamesNK), so search once more for JSON.Net and install it again for all the client projects.

The communication with the backend is created in **HttpWrapper.cs**, the **BasicController** simply the **Content** of the **result** is read and passed back:


    public async Task<string> GetBasicDataAsync(int i){    var uri = string.Format("https://xamarinwebapiservice.azurewebsites.net/api/basic/{0}", i);        var result = await _httpClient.GetAsync(uri);    return await result.Content.ReadAsStringAsync();}


When we call the **ObjectController** we receive the JSON in the **Content** of the result and have to deserialize it first:


    public async Task<NumberWrapper> GetNumberWrapperDataAsync(int i){    var uri = string.Format("{0}object/{1}", BaseUrl, i);    var result = await _httpClient.GetAsync(uri);    var jsonNumberWRapper = await result.Content.ReadAsStringAsync();    return JsonConvert.DeserializeObject<NumberWrapper>(jsonNumberWRapper);}


So far so good now lets create a small UI to Consume the service and its controllers.



# The client UI

I’ll be using [MVVM Light](http://www.mvvmlight.net/) in the UI for the data binding, on a former post you can find an introduction on how to get started with MVVM Light. The UI lets you pick a number and you can choose the call by selecting the according button:


    <?xml version="1.0" encoding="utf-8" ?><ContentPage xmlns="http://xamarin.com/schemas/2014/forms"             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"             x:Class="WebApiXamarinForms.View.MainPage">  <StackLayout>    <Picker SelectedIndex="{Binding SelectedNumberIndex}" HorizontalOptions="CenterAndExpand" Title="Numbers">      <Picker.Items>        <x:String>1</x:String>        <x:String>2</x:String>        <x:String>3</x:String>        <x:String>4</x:String>        <x:String>5</x:String>        <x:String>6</x:String>        <x:String>7</x:String>        <x:String>8</x:String>        <x:String>9</x:String>        <x:String>10</x:String>      </Picker.Items>    </Picker>    <Button Text="Basic Submit" Command="{Binding BasicSubmitCommand}" HorizontalOptions="CenterAndExpand"></Button>    <Button Text="Object Submit" Command="{Binding ObjectSubmitCommand}" HorizontalOptions="CenterAndExpand"></Button>    <ActivityIndicator IsRunning="{Binding CallingService}"></ActivityIndicator>    <Label Text="{Binding Response}" VerticalOptions="Center" HorizontalOptions="CenterAndExpand" />  </StackLayout></ContentPage>



> Note that the **Picker** control does not yet support data binding so this is currently the only way how you can set the values.


In the **MainViewModel.cs** the calls to the service are made during the command invokes and the result is written accordingly to the UI. Here the **BasicSubmitCommand**:


    private async void GetBasicResponseAsync(){    CanGetBasicResponse = false;    CallingService = true;    Response = await _webService.GetBasicDataAsync(_numbers[SelectedNumberIndex]);    CallingService = false;    CanGetBasicResponse = true;}


And the **ObjectSubmitCommand**:


    private async void GetObjectResponseAsync(){    CanGetBasicResponse = false;    CallingService = true;    var numberWrapper = await _webService.GetNumberWrapperDataAsync(_numbers[SelectedNumberIndex]);    Response = numberWrapper.WrappedNumber;    CallingService = false;    CanGetBasicResponse = true;}


# Conclusion

In this blog post we took a quick look at how a WebAPI controller is setup. Further we looked at how HTTP calls are created on the Xamarin.Forms client and how responses can be parsed and used within a program. We also saw how we can consume objects that are received as JSON strings from a web service.

You can find the entire source code on [GitHub](https://github.com/).

del.icio.us Tags: [Xamarin.Forms](http://del.icio.us/popular/Xamarin.Forms),[Xamarin](http://del.icio.us/popular/Xamarin),[ASP.Net WebAPI](http://del.icio.us/popular/ASP.Net+WebAPI),[Azure](http://del.icio.us/popular/Azure)
