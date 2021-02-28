---
layout: single
title: "Running xUnit.net tests on Android and iOS devices with Xamarin"
title: Running xUnit.net tests on Android and iOS devices with Xamarin
date: 2015-09-24
tags: ["Xamarin", "Xamarin.Forms", "Windows Phone", "iOS", "Mobile", "Android", "xunit", "testing"]
slug: "run-xunit-tests-on-android-ios-and-windows"
---

[![TitleImage](http://mallibone.com/posts/files/046ec8c1-37e6-41a2-90fd-a613d88f8a55.png "TitleImage")](http://mallibone.com/posts/files/1fedfe9f-b93e-44bd-aee3-65d25dedf6d1.png)
 
Writing Unit, Component and Integration tests allows to test functionality of software modules. In other words letting developers sleep tightly without worries at night. These tests run against certain parts of the code verifying the logic and interaction between modules of the app. All this is usually performed on the developers machine powered by a beefy processor and often unheard amount of RAM when it comes to mobile devices. Wouldn’t it be great if we could get a bit closer to the real thing? Well we can…
 
In this blog post we will look at how we can run [xUnit.net](https://github.com/xunit/xunit "Link to xUnit project site.") tests on a device. The app will be a Xamarin(.Forms) app, which consists of three platform projects and a shared Portable Class Library which can be used to write logic that can be consumed on every platform.
 
# Setting up xUnit.net
 
The typical xUnit.net tests are created in a standard .Net library. All that is required is the [xUnit.net NuGet](http://www.nuget.org/packages/xunit "Link to xUnit.net NuGet website") package:


    PM> Install-Package xunit -Version 2.0.0


Now we can create start writing unit tests, for this blog post let’s stick to the basic calculator example in the <font face="Consolas">BasicMathServiceTest.cs</font>:


    public class BasicMathServiceTest
    {
        private BasicMathService _basicMathService;
    
        public BasicMathServiceTest()
        {
            _basicMathService = new BasicMathService();
        }
    
        [Fact]
        public void Add_GivenTwoNumbers_TheSumIsReturned()
        {
            var x = 35;
            var y = 7;
            var expectedResult = 42;
    
            var result = _basicMathService.Add(x, y);
    
            Assert.Equal(expectedResult, result);
        }
    
        [Fact]
        public void Subtract_GivenTwoNumbers_TheSubtractedResultIsReturned()
        {
            var x = 456;
            var y = 123;
            var expectedResult = 333;
    
            var result = _basicMathService.Subtract(x, y);
    
            Assert.Equal(expectedResult, result);
        }
    
        [Fact]
        public void Mulitply_GivenTwoNumbers_TheResultIsReturned()
        {
            var x = 2;
            var y = 2;
            var expectedResult = 4;
    
            var result = _basicMathService.Multiply(x, y);
    
            Assert.Equal(expectedResult, result);
        }
    
        [Fact]
        public void Divide_GivenTwoNumbers_TheSumIsReturned()
        {
            var x = 848;
            var y = 8;
            var expectedResult = 106;
    
            var result = _basicMathService.Divide(x, y);
    
            Assert.Equal(expectedResult, result);
        }
    
        [Fact]
        public void TheFailingTest()
        {
            var x = 4;
            var y = 2;
            var expectedResult = 8;
    
            var result = _basicMathService.Add(x, y);
    
            Assert.Equal(expectedResult, result);
        }
    }


The code that we test against lives in a PCL and performs the calculations. Now let’s take these existing tests and execute them on the simulator or even a real device.

# Running Tests on devices

For this first add the platforms on which the project will run. Now add the following xUnit.net NuGet package:


    PM> Install-Package xunit.runner.devices -Version 1.0.0


Written by [Oren Novotny](https://twitter.com/onovotny) whom we have to thank for enabling xUnit.net on Android, iOS and Windows Phone! This package will automatically add the xUnit.net and other dependencies required for running tests on your devices according to the platform to which it’s being added. Depending on the platform the xUnit.net for Devices package adds different template files as text files:

- Android
    - MainActivity.cs.txt
- iOS
    - AppDelegate.cs.txt
- Windows Phone
    - MainPage.xaml.txt
    - MainPage.xaml.cs.txt


## 

Simply replace the content of their default counterparts e.g. for iOS replace the <font face="Consolas">AppDelegate.cs</font> with the content of the <font face="Consolas">App.Delegate.cs.txt</font> (and thanks to some extra effort by [Oren Novotny](https://twitter.com/onovotny) even the namespaces will just simply match ![Smile](http://mallibone.com/posts/files/7f9d38b7-159d-4e1f-9d51-5be18da182f1.png)). Now you could simply start adding new class files and write out tests. The runner will automatically find them via reflection (even if you place them in subfolders).

# Using existing tests

What is probably a more useful approach is to simply reuse existing tests. This is why in a next step lets see how we can add the unit test file <font face="Consolas">BasicMathServiceTest.cs</font> without creating a duplicate dopy. Simply right click on the Project then Add/Existing Item… browse to the unit test file and add it as link.

[![Shows the dropdown on the Add button that can be used to add a file as a link.](http://mallibone.com/posts/files/c2e88d77-2ef2-4881-b207-71468a9cea4b.png "AddAsLink")](http://mallibone.com/posts/files/20ee62ef-48ea-4615-af74-789cbb376542.png)


> As a link to the file is simply a pointer to the original we only have to maintain one unit test file and all the links will automatically “update” as they simply reference the original.


Now the Project can be set as startup project and executed on the device. Bellow you see some sample screenshots from an iOS Emulator.

[![TestOverview](http://mallibone.com/posts/files/afa8e738-8977-44cd-852f-8c4232278112.png "TestOverview")](http://mallibone.com/posts/files/b6bc3b3b-7cb7-48a9-af91-4021b5bc12fc.png)

You can even dig into the error messages and see what went wrong during a test.

[![TestDetails](http://mallibone.com/posts/files/80ffcab8-0692-476d-bfbc-54891fcf352d.png "TestDetails")](http://mallibone.com/posts/files/49191a8d-098a-4da0-ba2a-cfd37421c890.png)

# Conclusion

In this post we saw how to create and reuse xUnit.net tests on devices for iOS, Android and Windows. This can be very powerful when we have to test certain features on a device for example does this work with AOT on iOS? Or simply running integration tests on a real device, giving valuable insights and further allowing to tests performance, latency and so on.

This post focuses on xUnit.net and also shows one of it’s many strengths on the capabilities that can be easily added. The same can be done with [NUnit](http://nunit.org/ "Link to NUnit project page") which may be your testing framework of choice and which Xamarin provides project templates to run the tests on a device.

You can find the sample code on [GitHub](https://github.com/mallibone/xunitdevicerunner.git "Link to sample code github repository").
