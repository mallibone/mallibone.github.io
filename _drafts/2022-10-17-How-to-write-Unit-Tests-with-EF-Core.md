---
layout: single
title: "Getting started writing automated tests when using EF Core"
date: 2022-11-01 15:03
tags: ["EF Core", "Automated Testing", "Unit Testing"]
slug: "efcore-unittesting-pattern"

---

The soothing feeling after making changing or adding a few lines of code to your app and all tests are green is something I have come to greatly appreciate in my life as a developer. It goes without saying I am a huge fan of writing automated tests to verify that I have not done a mistake or worse, broken some aspect of an app that was perfectly fine before my change. This holds true when writing code that persists data to a database using EF Core LLLLINK. In this post I want to dive into a few aspects that have helped me along the way of writing unit tests with parts of code that use EF Core.

<!-- more -->

But before we get to actually writing tests, there are a few bases to cover first.

## Managing the database dependency

When writing automated tests it is often required to stub or mock a certain dependency. This allows to ensure the test will always get or receive the same data and will make it easier to test logic within the class under test. To ensure your code is not tightly coupled to an implementation one often injects the dependency into the class via the constructor. If you are  writing ASP.NET Core or .NET MAUI apps this has become the default if you use the dependency service during the startup of the app. Same is true for EF Core dependencies, if we inject them via the constructor we can control in our tests how the database context has been setup. So the first step to enabling your code to become testable is to inject the EF Core database context:

CCCCODE

## Speeding up your tests

Another consideration we have when testing with EF Core is: Do we really want to hit a real database? While there are arguments for writing tests that actually do invoke a real database there is quite an impact on the execution time of your tests. Personally I write almost all of my tests using an in-memory database provider. The [In-Memory database](https://learn.microsoft.com/en-us/ef/core/providers/in-memory/?tabs=dotnet-core-cli&WT.mc_id=AZ-MVP-5003494) provided by the EF Core team provides an easy way to create such an in-memory database for testing.

> Generally speaking it is a fact that Input / Output operations or IO for short take more time the accessing something stored into the computers RAM. So if we write our test accessing a real database we will end up having tests that take a lot longer due to the fact that the data will be served from another service that most probably will have to read the information from disk. While it is true, that many databases have excellent caching capabilities automated tests tend to benefit less from them since one paradigm of writing automated tests is that they should be executed in isolation which usually means resetting the entire database before every test run.
>
> If you are interested into the scales of what the differences are [Jeff Atwood aka Coding Horror](https://blog.codinghorror.com/about-me/) has a nice [blog](https://blog.codinghorror.com/the-infinite-space-between-words/) post that sums this topic up nicely.

We can create a DB context as follows:

CCCCODE

Note that given the same name we can get multiple "fresh" `DBContext` which all use the same in-memory database. This is a great feature as we will see later. But furthermore it allows for easily parallelizing tests without having them interfere with eachother. For example if we use a GUID in the test setup we can ensure that every test run will use it's own DB:

CCCCODE

The code above is built using [XUnit](https://xunit.net/). But this can also be achieved in the setup methods of other unit test frameworks such as MSTest, NUnit, etc..

## Writing tests

So with the setup out of the way let's look at writing actual tests. Let's look at a little example app which has is made for managing the reference of blogs to people. If we would look at the database diagram it would look like this:

IIIIMAGE

A controller is used for retrieving all the people and their blogs, with a little bug to make a point later on:

CCCCODE

With the dependency injection resolved and equipped with the knowledge of using an in-memory database. The only thing left to do is writing an actual automated test. So in the classical Arrange, Act, Assert setup let's dive into the test:

CCCCODE

Using a different context for the setup and the execution is key. EF Core will cache information to be more efficient. If the context from setup is reused for the test, the forgotten line to correctly request the blog information would slip through the test. 

## Conclusion

Writing automated tests can be great help and using EF Core should not put a stop to that. By using dependency injection we make our app code testable. Using an in-memory database can greatly speed up the execution time of tests, though it may not be suitable for you, if you intend to test aspects that are not available in the in-memory database and require the real database engine used. Last but not least it is important to seperate the contexts used for setting up the tests and the actual execution or else bugs will not be caught by the tests. I hope you could see in this post how you can setup your apps and tests to write successful tests. You can find the entire sample app on [GitHub](https://github.com/mallibone/EfCoreTesting).

HTH

