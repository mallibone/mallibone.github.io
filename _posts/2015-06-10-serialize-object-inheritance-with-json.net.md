---
layout: single
title: "Serialize object inheritance with JSON.Net"
title: Serialize object inheritance with JSON.Net
date: 2015-06-10
tags: ["Mobile", "Xamarin", "Windows Phone", "Xamarin.Forms", "iOS", "Windows"]
slug: "serialize-object-inheritance-with-json.net"
---

[![JSON Logo i.e. blog post logo](http://www.mallibone.com/posts/files/e03a1ee3-7017-4f0c-b46a-9e46acfeff24.png "Blog Post title logo")](http://www.mallibone.com/posts/files/173388cb-4329-42a5-ab05-b9ae6746fa2c.png)
 
Though frowned upon often due to overuse inheritance remains a powerful and (when used correctly) great feature of an object oriented programming language such as C#. But when it comes down to serializing inheritance object structures i.e. deserializing them on the other side things tend to get hairy. The post will show you how serialization and deserialization of inheritance related objects can be performed with [JSON.Net](http://www.newtonsoft.com/json).
 
So lets have a look at the project setup. The server will be your standard ASP.Net Web API restful web service that comes with JSON.Net out of the box. On the client side we will be using consuming the service from a Portable Class Library PCL that will be able to run with most C# stacks e.g. Windows Store, .Net, Xamarin.iOS, Xamarin.Android etc.. I’ll be using a Xamarin.Forms app but the code will run just fine in your ever day WPF app. The basic system setup is the client calls the server for data, the server returns the data serialized as JSON which will then be deserialized on the client and then used to display the information to the user.
 
[![System overview showing communication flow from the client to the server and back via JSON serialized data.](http://www.mallibone.com/posts/files/4e1abc8f-8f76-4682-a12a-365f8f725302.png "system overview")](http://www.mallibone.com/posts/files/a65ea89a-9622-4aa2-aa0c-6058e2e526fe.png)
 
So lets have a look how it works in more detail.
 
# Setting up the project
 
Lets start with looking at the server which mainly exists of a controller:


    public class InheritanceController : ApiController{    // GET: api/Inheritance    public IEnumerable<ParentClass> Get()    {        return new ParentClass[] {new ParentClass(), new ChildClass()};    }}


The controller creates a list of **ParentClass**es and **ChildClass**es before serializing them back to the caller.


    public class ParentClass{    public virtual string Title    {        get { return "Parent"; }    }    public virtual string Descirption    {        get { return "Hello from the parent."; }    }}public class ChildClass:ParentClass{    public override string Title    {        get { return "Child"; }    }    public override string Descirption    {        get { return "The child says hi..."; }    }    public string ChildSecret    {        get { return "42"; }    }}


So not a lot of magic going as you can see on the server side. So lets have a look at the client. In the PCL we will use the HTTP Client NuGet package to call the server and the JSON.Net package to deserialize the data from the server.

[![Nuget Package overview on the client showing JSON.NET, Microsoft BCL Build Components, MVVMLIght libraries only, Microsoft HTTP Client Libraries and Xamarin.Forms packages](http://www.mallibone.com/posts/files/eb9fca4c-4b65-4237-b08d-31f672d3617d.png "ClientNugetPackageOverview")](http://www.mallibone.com/posts/files/f586c78f-c620-4cdf-b8bb-13c8e64b3da0.png)

On the client we will display the data in a list, you can see the UI code in the **MainPage.xaml**:


    <?xml version="1.0" encoding="utf-8" ?><ContentPage xmlns="http://xamarin.com/schemas/2014/forms"             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"             x:Class="InheritanceJsonSerialization.Client.Views.MainPage">  <Grid>    <ListView ItemsSource="{Binding DataItems}" ItemSelected="ListView_OnItemSelected" />    <ActivityIndicator IsRunning="{Binding IsLoading}" IsEnabled="{Binding IsLoading}" IsVisible="{Binding IsLoading}"/>  </Grid></ContentPage>


In the **HttpHandler** class in the **GetData** method the data is requested from the server and then deserialized.


    using System.Collections.Generic;using System.Net.Http;using System.Threading.Tasks;using InheritanceJsonSerialization.Client.Models;using Newtonsoft.Json;namespace InheritanceJsonSerialization.Client.Services.Http.Impl{    public class HttpHandler:IHttpHandler    {        private readonly HttpClient _httpClient;        public HttpHandler()        {            _httpClient = new HttpClient();        }        public async Task<IEnumerable<ParentClass>> GetData()        {            // Ensure that uri matches the service backend you are calling            //const string uri = "http://localhost:52890/api/inheritance";            const string uri = "https://inheritancejsonserializationweb.azurewebsites.net/api/inheritance";            var httpResult = await _httpClient.GetAsync(uri);            var jsonContent = await httpResult.Content.ReadAsStringAsync();            var result = JsonConvert.DeserializeObject<IEnumerable<ParentClass>>(jsonContent);            return result;        }    }}


Now when we deserialize the information on the client we run into the problem that the type information is not passed on to the client and therefore all objects deserialized are of type **ParentClass**.

# Serializing with type information

Luckely JSON.Net allows us to easily include the type information when we serialize the type. On the server in the **Register** method within the **WebApiConfig** class we can setup how JSON.Net serializes objects:


    public static void Register(HttpConfiguration config){    // Web API configuration and services    config.Formatters.JsonFormatter.SerializerSettings.TypeNameHandling = TypeNameHandling.All;    // Web API routes    config.MapHttpAttributeRoutes();    config.Routes.MapHttpRoute(        name: "DefaultApi",        routeTemplate: "api/{controller}/{id}",        defaults: new { id = RouteParameter.Optional }    );}


Now the information will be serialized in the JSON string:


    {    "$type": "InheritanceJsonSerialization.Models.ParentClass[], InheritanceJsonSerialization",    "$values": [        {            "$type": "InheritanceJsonSerialization.Models.ParentClass, InheritanceJsonSerialization",            "Title": "Parent",            "Descirption": "Hello from the parent."        },        {            "$type": "InheritanceJsonSerialization.Models.ChildClass, InheritanceJsonSerialization",            "Title": "Child",            "Descirption": "The child says hi...",            "ChildSecret": "42"        }    ]}


On the client side we will have to do a bit more work to get deserialization working.

# Deserializing with Type information

For deserializing with type information you might have already found the following line of code that will trigger JSON.Net to check for the type information and try to deserialize it according to the information present in the JSON string.


    JsonConvert.DeserializeObject<IEnumerable<ParentClass>>(jsonContent, new JsonSerializerSettings{TypeNameHandling = TypeNameHandling.All});


The problem here is that all will go well when you serialize and deserialize from the same binary/namespace but in this setup (which I regard as the common setup) you will get an exception as the namespace will not exist i.e. be different on the client than on the server. To solve this issue we have to write a SerializationBinder that inherits from the **DefaultSerializationBinder** as follows:


    public class InheritanceSerializationBinder : DefaultSerializationBinder{    public override Type BindToType(string assemblyName, string typeName)    {        switch (typeName)        {            case "InheritanceJsonSerialization.Models.ParentClass[]": return typeof(ParentClass[]);            case "InheritanceJsonSerialization.Models.ParentClass": return typeof(ParentClass);            case "InheritanceJsonSerialization.Models.ChildClass[]": return typeof(ChildClass[]);            case "InheritanceJsonSerialization.Models.ChildClass": return typeof(ChildClass);            default: return base.BindToType(assemblyName, typeName);        }    }}


This will simply map the server namespace to local types. Finally we can update the **GetData** method in the **HttpHandler** class:


    public async Task<IEnumerable<ParentClass>> GetData(){    // ...    var result = JsonConvert.DeserializeObject<IEnumerable<ParentClass>>(jsonContent, new JsonSerializerSettings{TypeNameHandling = TypeNameHandling.All, Binder = new InheritanceSerializationBinder()});    return result;}


And now when we deserialize the list the **ChildClass** will be correctly identified i.e. the list will be preserved through deserialization.

# Conclusion

In this post you saw how inheritance hirarchies can be correctly serialized and deserialized on the client using JSON.Net. By doing so we had to configure our ASP.Net Web API and add handling mechanisms on the client side. I hope this post is of any help and please let me know if I may have mist anything ![Smile](http://www.mallibone.com/posts/files/1ae5990a-b0d7-499b-80ef-33dc67afa970.png)

You can find the code on [GitHub](https://github.com/mallibone/InheritanceJsonSerialization.git).
