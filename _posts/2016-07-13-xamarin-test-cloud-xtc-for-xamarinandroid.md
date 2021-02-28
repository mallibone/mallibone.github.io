---
layout: single
title: "Getting started with Xamarin Test Cloud for Android"
title: Getting started with Xamarin Test Cloud for Android
date: 2016-07-13
tags: ["Xamarin Test Cloud", "Xamarin", "Android"]
slug: "xamarin-test-cloud-xtc-for-xamarinandroid"
---

[![image]({{ site.url }}{{ site.baseurl }}/images/c2526010-57a9-4e1b-8ffc-120819ebbeee.png "image")]({{ site.url }}{{ site.baseurl }}/images/ee2e2ffd-ec1d-4d37-8ac8-a49e8b26b2e4.png)
 
In this post we will look at how we can write UI tests for a Xamarin.Android app. The app which we are testing is a basic app based on the MVVM pattern. You can find a detailed blog post on the in and outs of the app under test [here](https://mallibone.com/post/xamarinandroid-and-mvvm-light-bindings "Link to post describing the MVVM based app").
 
Assuming you already have a [Xamarin Test Cloud](https://www.xamarin.com/test-cloud "The Xamarin Testcloud website") (XTC) project added to your solution. As described in this [former post](https://mallibone.com/post/write-your-first-xamarin-test-cloud-tests-for-xamarinforms "Link to blog post on getting started with Xamarin Test Cloud"). Lets get started by preparing the app for UI testing.
 
# Enabling your Android app for the XTC tests
 
You do not have to do anything to enable your app to be compatible with XTC. That being said when writing a test with Xamarin Test Cloud. The test should not recognize a control by the content it is displaying. Rather it identifies a control via an ID which is not visible to the end user but rather a invisible ID. Setting this ID is best done via the accessibility label attribute. You can set this in the AXML code of your view i.e. control:
 
<script src="https://gist.github.com/mallibone/3559cb0d87eed8b1e74912281d611165.js"></script>
 
This ID will not have to change if the locale changes. Even if the location of the UI element changes it will still be found by the test. Thus IDs should be generally used to identify controls as they allow for higher resilient test code.
 
# Writing Test(s)
 
After tuning our the UI code we can start writing the test. One way is to use the [XTC recorder](https://www.xamarin.com/test-cloud/recorder). While this tool brings a great benefit when starting off with or smaller apps (as it would be the case in this post). I usually prefer to use the [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) which is easy to use and can be invoked at any point in the test code.
 
When the UI is prepared accordingly we can start writing the test. One way is to use the [XTC recorder](https://www.xamarin.com/test-cloud/recorder "Link to official Xamarin Test Cloud recorder website"). While this tool brings a great benefit when starting of with or smaller apps (as it would be the case in this post), I usually prefer to use the [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop "Wikipedia site over Read-eval-print loop (REPL)") which is easy to use and can be inserted to be started at any point in your code.
 

> The REPL is especially useful once you have a large testing framework. Having a framework i.e. helpers in place you can run those before firing up the REPL. So you can start exploring and interacting with the UI right were you need to.

 
[![XtcTreeWithIds_thumb]({{ site.url }}{{ site.baseurl }}/images/6e9ed995-95fe-4b94-a312-23d3d70634ef.png "XtcTreeWithIds_thumb")]({{ site.url }}{{ site.baseurl }}/images/5d547587-883e-440e-927c-edc999e8cc31.png)
 
Starting the REPL is as easy as adding the following line to your test method:
 
<script src="https://gist.github.com/mallibone/b21e9816b0051fc079235e317fc57af0.js"></script>
 
You can see the entire apps visual tree outlined.
 
Now we can define the steps we want to perform in our test:
 
<script src="https://gist.github.com/mallibone/087161f91e644874b92e5bb94bfcd626.js"></script>
 
Note how the test takes screenshots at every stage. This allows to easily identify the steps on the Xamarin test cloud and give them a label. While this might seem to be a bit of an overkill for such a small sample. When writing larger tests naming your steps i.e. screen shots will help you narrowing down where an error is happening.  
One good thing about the tests is that they also run well on your local machine. So using e.g. the al Studio we can execute the tests over the NUnit Runner:

[![shows resharper testrunner in visual studio after executing a test]({{ site.url }}{{ site.baseurl }}/images/43cad23d-c32c-44b9-885a-70e450ea286e.png "shows resharper testrunner in visual studio after executing a test")]({{ site.url }}{{ site.baseurl }}/images/2ec1e9fc-2cd7-42c7-aba2-9a8b95c5754f.png)
 
You can choose to run the tests on an emulator or device as you like. Just select the desired target as you would when starting the application. Or submit it to the test cloud i.e. [integrate it in our build process](https://mallibone.com/post/see-how-to-run-xamarin-test-cloud-runs-from-the-command-line).
 
# Conclusion
 
This post described how to get started writing automated UI tests with Xamarin Test Cloud for a Xamarin.Android. Showing you how to adopt your UI for writing resilient test. Based on NUnit the tests are running on a stable and battle proven foundation.

Keep in mind that UI tests should not be the only testing you rely on for testing your app. But in combination with Unit and Integration tests they can provide great value to your project. Running them on Xamarin Test Cloud allows you to run them on a sea of devices. Along with different versions of Android i.e. vendor flavours.

You can find the entire sample on [GitHub](https://github.com/mallibone/Xtc101).
