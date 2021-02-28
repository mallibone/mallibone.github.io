---
layout: single
title: "Getting started with Xamarin Test Cloud on iOS"
title: Getting started with Xamarin Test Cloud on iOS
date: 2016-06-28
tags: ["Xamarin", "Xamarin Test Cloud", "iOS"]
slug: "see-how-to-get-started-writing-automated-ui-tests-with-xamarin-test-cloud-for-a-xamarinios-application"
---

[![Title image showing the xamarin test cloud and ios logo](https://mallibone.com/posts/files/4a15711b-9132-4a93-875b-dee32156e646.png "Title image showing the xamarin test cloud and ios logo")](https://mallibone.com/posts/files/ff4ad80e-69b1-40fa-9cb5-bcd13faab9ef.png)
 
In this post we will look at how we can test a Xamarin.iOS app which is based on a storyboard. The app which we are testing is a basic app based on the MVVM pattern. You can find a detailed blog post on the in and outs of the app under test [here](https://mallibone.com/post/mvvm-light-bindings-under-xamarin.ios "Link to MVVM Light blog post describing the app").
 
Assuming you already have a [Xamarin Test Cloud](https://www.xamarin.com/test-cloud "The Xamarin Testcloud website") (XTC) project added to your solution as described in this [former post](https://mallibone.com/post/write-your-first-xamarin-test-cloud-tests-for-xamarinforms "Link to blog post on getting started with Xamarin Test Cloud"). Lets get started by preparing the app to be tested.
 
# Preparing the iOS for the XTC tests
 
When writing a test with Xamarin Test Cloud the test ideally does not recognize a control by the content it is displaying but rather via an ID which is not visible to the end user but rather a invisible ID. Setting this ID is best done via the accessibility label attribute which can be set in the storyboard under properties:
 
[![Shows where under properties the label can be enabled.](https://mallibone.com/posts/files/d5a2a437-2838-4570-8cde-5a86286a98d4.png "Shows where under properties the label can be enabled.")](https://mallibone.com/posts/files/747c4ceb-d28c-41bc-b6eb-9751f4568d00.png)
 
This ID will not have to change if the locale changes or the UI element is moved to a different location and therefore will allow you to write more resilient test code.
 
# Writing Test(s)
 
When the UI is prepared accordingly we can start writing the test. One way is to use the [XTC recorder](https://www.xamarin.com/test-cloud/recorder "Link to official Xamarin Test Cloud recorder website"). While this tool brings a great benefit when starting of with or smaller apps (as it would be the case in this post), I usually prefer to use the [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop "Wikipedia site over Read-eval-print loop (REPL)") which is easy to use and can be inserted to be started at any point in your code.
 

> The REPL is especially useful once you have a substantial testing framework written because it allows you to execute it after executing some prearranged steps to get to a certain point which you would like to test.

 
[![Shows repl output of the app](https://mallibone.com/posts/files/5cb6c85d-d0e8-4da8-812a-64e8520cf47d.png "Shows repl output of the app")](https://mallibone.com/posts/files/d122d8c7-ed99-49f5-a2a3-0b153eb20676.png)
 
Starting the REPL is as easy as adding the following line to your test method:
 
<script src="https://gist.github.com/mallibone/b21e9816b0051fc079235e317fc57af0.js"></script>
 
You can see the entire apps visual tree nicely outlined.
 
Now we can define the steps we want to perform in our test:
 
<script src="https://gist.github.com/mallibone/087161f91e644874b92e5bb94bfcd626.js"></script>
 
Note how the test takes screenshots at every stage. This allows to easily identify the steps on the Xamarin test cloud and give them a label. While this might be a bit overkill for such a small sample on larger tests when named right the steps and screen shots will greatly help you narrowing down where an error is happening.  
One good thing about the tests is that they also run well on your local machine. So using Xamarin Studio on a Mac we can simply execute the tests over the NUnit Runner:
 
[![image showing the nunit test runner of xamarin studio.](https://mallibone.com/posts/files/a75c56b7-4f47-4835-ac7a-ade95651715d.png "image showing the nunit test runner of xamarin studio.")](https://mallibone.com/posts/files/42cc6a70-7b65-457c-b779-841d6b8addf2.png)
 
Or submit it to the test cloud i.e. [integrate it in our build process](https://mallibone.com/post/see-how-to-run-xamarin-test-cloud-runs-from-the-command-line "Blog post describing how to execute tests from within a script").
 
# Conclusion
 
This post described how to get started writing automated UI tests with Xamarin Test Cloud for a Xamarin.iOS application. After some minor adjustments to the UI a test can be written using the familiar NUnit testing framework. Keeping in mind that UI tests should not be the only testing you apply to your app since they are rather slow, they do bring a lot of value when used properly since one can easily run the tests on different devices and iOS versions with the Xamarin Test Cloud.
 
You can find the entire sample on [GitHub](https://github.com/mallibone/Xtc101 "Link to entire source code hosted on GitHub- HTH").
