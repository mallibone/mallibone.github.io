---
layout: single
title: "Creating a Web API service with F# and hosting it on Azure"
title: Creating a Web API service with F# and hosting it on Azure
date: 2015-07-19
tags: ["Azure", "F#"]
slug: "creating-a-web-api-service-with-f-and-hosting-it-on-azure"
---

In this post I want to look at how to create a Web API project with F# and host it on Azure. This will part of a blog series on how to create a mobile application (incl. the backend) based on F#. The blog post assumes that you the reader has a basic understanding of how to create a Web API web service.
 

> F# is a functional programming language based on the .Net Framework which contrary to many other functional programming languages also makes it a general purpose language. In my endeavor to explore a functional programming language I chose F# as it will allow me to create not only .Net applications but also Xamarin based cross platform apps.

 
To deploy to Azure you will need a valid Azure account. Alternatively you can run the App on your local machine.
 
# Setting up the project
 
I’ll be using Visual Studio 2015 RC for this setup but you can follow along on VS 2013 just as well. Create a new project then go to ***Online*** projects, Select ***Visual F#*** and then select ***F# MVC 5***. This only has to be done during the first setup. From now on you will find the project template under the ***Installed***project templates.
 
[![NewProjectFromOnlineTemplate](http://mallibone.com/posts/files/01a6225b-0f9f-4c8a-98f5-77af21c2f351.png "NewProjectFromOnlineTemplate")](http://mallibone.com/posts/files/76307dbb-7756-4509-acbb-a0384356b95c.png)
 
## Having a look at the solution
 
If you are familiar with a Web API solution you will find yourself right at home.
 
[![WebApiSolution](http://mallibone.com/posts/files/d83af68e-fa02-4354-b5a1-0d27bb5b0cb9.png "WebApiSolution")](http://mallibone.com/posts/files/e84b12c0-2846-4c3f-b5fa-4966eb1339f0.png)
 
Under Controllers you find the expected definitions, as well as under models. The ***Global.asax.fs*** also has a familiar ring to it:


    namespace FSharpWebSample
    
    open System
    open System.Net.Http
    open System.Web
    open System.Web.Http
    open System.Web.Routing
    
    type HttpRoute = {
        controller : string
        id : RouteParameter }
    
    type Global() =
        inherit System.Web.HttpApplication() 
    
        static member RegisterWebApi(config: HttpConfiguration) =
            // Configure routing
            config.MapHttpAttributeRoutes()
            config.Routes.MapHttpRoute(
                "DefaultApi", // Route name
                "api/{controller}/{id}", // URL with parameters
                { controller = "{controller}"; id = RouteParameter.Optional } // Parameter defaults
            ) |> ignore
    
            // Configure serialization
            config.Formatters.XmlFormatter.UseXmlSerializer <- true
            config.Formatters.JsonFormatter.SerializerSettings.ContractResolver <- Newtonsoft.Json.Serialization.CamelCasePropertyNamesContractResolver()
    
            // Additional Web API settings
    
        member x.Application_Start() =
            GlobalConfiguration.Configure(Action<_> Global.RegisterWebApi)


# Extending the Controller

When we look at the controller we see that an array of the model is created and there is on method that will return the list. Let’s extend the controller to return a value requested by it’s Id. So first we have to extend the model **Car.fs**:


    namespace FSharpWebSample.Models
    
    open Newtonsoft.Json
    
    [<CLIMutable>]
    type Car = {
        Make : string
        Model : string
        Id : int
    }


Now we can extend the **CarsController.fs** so that we can filter the list and get the according value:


    namespace FSharpWebSample.Controllers
    open System
    open System.Collections.Generic
    open System.Linq
    open System.Net.Http
    open System.Web.Http
    open FSharpWebSample.Models
    
    /// Retrieves values.
    type CarsController() =
        inherit ApiController()
        
        let values = [ { Make = "Ford"; Model = "Mustang"; Id = 1 }; { Make = "Nissan"; Model = "Titan"; Id = 2 }; { Make = "Audi"; Model = "R8"; Id = 3 } ]
        
        /// Gets all values.
        member x.Get() = values
        
        member x.Get(id:int) = values |> List.filter(fun v -> v.Id = id)


# Deploying to Azure

Deploying to azure luckily is luckely the same as it always has been for C#, right click onto the Web API project, select ***Publish…, Microsoft Azure Web Apps*** then create a new web app enter a name and finally click on ***Pubish***. Now all that is left is to test the web service so lets do this the manual way.

## Testing the controller

Opening the prefered browser of your choice e.g. the new Edge browser enter the path to the azure website and be executing the following URI:


    http://fsharpwebsample.azurewebsites.net/api/cars


We receive the entire list:


    [{"make":"Ford","model":"Mustang","id":1},{"make":"Nissan","model":"Titan","id":2},{"make":"Audi","model":"R8","id":3}]


When we add a Id parameter:


    http://fsharpwebsample.azurewebsites.net/api/cars/3


We only receive the requested record:


    [{"make":"Audi","model":"R8","id":3}]


# Conclusion

Creating a Web API service with F# is not much different than creating it with C# thanks to [Ryan Riley](https://wizardsofsmart.wordpress.com/) and [Daniel Mohl](http://blog.danielmohl.com/) who provide the F# MVC 5 template. Also uploading it to Azure and sending requests are the way one would expect the service to run. Looking forward to go on further down the F# rabbit hole.

You can find the sample on [GitHub](https://github.com/mallibone/FSharpSamples.git).
