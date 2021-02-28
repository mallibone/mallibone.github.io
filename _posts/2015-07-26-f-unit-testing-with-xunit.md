---
layout: single
title: "F# unit testing with xUnit"
title: F# unit testing with xUnit
date: 2015-07-26
tags: ["F#"]
slug: "f-unit-testing-with-xunit"
---

[![titleImage]({{ site.url }}{{ site.baseurl }}/images/1f53a2f6-7dfb-4989-afee-c312828501c6.jpg "titleImage")]({{ site.url }}{{ site.baseurl }}/images/4e1bec01-63dc-42fb-b818-de2baa80c867.jpg)
 
In this post I want to look at how to get started with Unit Testing while developing F#. I’m usually writing my code in C# and prefer to write my code in a TDD fashion, just to be on the safe side and sleep sound during the nights ![Winking smile]({{ site.url }}{{ site.baseurl }}/images/cd9d138d-9268-4091-b2f6-51bca942d450.png) In  my last blog post I looked at how to create a [Web API service with F#](http://mallibone.com/post/creating-a-web-api-service-with-f-and-hosting-it-on-azure) and ended up testing the controller manually via a browser. So lets look how we can extend that solution to provide an automated way to test the controller.
 
# Adding a test project
 
I’ll be using xUnit to create my tests. To get started you will just need a standard F# .Net Library (do not choose a Portable Class Library i.e. PCL). Then via NuGet add the following package:


    PM> Install-Package xunit -Version 2.0.0


To be able to test the controller we must also add the **FSharpWebSample** project to the **References**. Now we can start writing tests as for example we can see in **CarsControllerTest.fs**:


    namespace FSharpWebSample.Test
    open Xunit
    open FSharpWebSample.Controllers
    
    module CarsControllerTest =
    
        let carsController = new CarsController()
    
        [<Fact>]
        let Get_WhenInvoked_ReturnsAListContainingMultipleElements() = 
            let cars = carsController.Get()
            Assert.False(cars.IsEmpty)
    
        [<Fact>]
        let GetWithIndex_WhenInvokedWithAValidIndex_ReturnsASingleItem() = 
            let car = carsController.Get(2)
            Assert.True(car.Id = 2)


Ensure that the xUnit and Controllers namespace is referenced in the test class:


    open Xunit
    open FSharpWebSample.Controllers


# Running the tests

We can use the Test Explorer from Visual Studio run our tests as the xUnit runner NuGet package will make the xUnit tests visible to it. Simply add the package to the test project:


    PM> Install-Package xunit.runner.visualstudio -Version 2.0.1

Then we can run the tests via ***Test***, ***Run***and finally ***All Tests***. After executing our tests we get the all green.  
[![Showing test runner after executing the tests.]({{ site.url }}{{ site.baseurl }}/images/7d39e321-62e7-4894-9610-67a34c2c339d.png "UnitTestsAllGreen")]({{ site.url }}{{ site.baseurl }}/images/a44bebbe-d27a-4e4d-9f6d-e6b594be6d6c.png)

# Conclusion

In this post we saw how Unit Tests can be created for and executed to ensure our code is running correctly. Which reduces the load on manual testing and ensures that single code blocks work within expected parameters.

You can find the entire project on [GitHub](https://github.com/mallibone/FSharpSamples.git).
