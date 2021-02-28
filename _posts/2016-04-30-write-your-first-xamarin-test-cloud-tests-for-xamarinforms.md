---
layout: single
title: "Getting started with Xamarin Test Cloud and Xamarin.Forms"
title: Getting started with Xamarin Test Cloud and Xamarin.Forms
date: 2016-04-30
tags: ["Xamarin", "Xamarin Test Cloud", "Xamarin.Forms"]
slug: "write-your-first-xamarin-test-cloud-tests-for-xamarinforms"
---

[![Xamarin Test Cloud Logo](https://mallibone.com/posts/files/38f6c9ae-a727-4029-b0d1-3004fbc18542.png "Xamarin Test Cloud Logo")](https://mallibone.com/posts/files/ef307e4e-3798-40a7-8af0-623d919ef1cf.png)
 
Automatically testing apps is not only a huge time saver but it also ensures that bugs introduced into the system get quickly caught. This leads to a shorter timespan to fixing these bugs since the memory of what has changed is still fresh in the head of the developers so overall shorter time means less money spent on hunting down bugs and more time to implement the next juicy feature. When developing a Xamarin application one can choose to run the Unit- and Integration Tests on a device/simulator to get a feeling how the logic will behave and perform in the real world. But with Xamarin Test Cloud we can go one step further and test the UI i.e. the entire stack of the application automatically. If you are interested into a general introduction into the topic I recommend you read this [post](https://mallibone.com/post/thoroughly-testing-your-mobile-app "Blog post on automated (UI) testing") before diving into the technical details of the lines to follow.
 

> For writing a Xamarin Test Cloud (XTC) test the [Xamarin Toolchain](https://www.xamarin.com/download "Link to Xamarin Download website.") has to be installed. This post was written using Visual Studio 2015 Update 2.

 
# Having a look at the app
 
Since this post focuses on writing UI tests the app is rather simple but will show the basic steps required to write maintainable UI tests. The app under test is a basic app with an input field, a button and a button that displays the entry of the input field. The app is based on the MVVM pattern (using [MVVM Light](http://www.mvvmlight.net/ "Link to MVVM Light project website")) and is based on Xamarin.Forms.
 
[![Screenshot of app that is going to be tested.](https://mallibone.com/posts/files/df661261-510a-4768-99e2-94784e5e5ccf.png "Screenshot of app that is going to be tested.")](https://mallibone.com/posts/files/07236faa-24b7-4359-b267-e50fdcd85941.png)
 
When a user enters a message and submits it, it will appear on the label bellow. Now that the basic outline is defined, lets have a look at how we can implement a UI test for the app.
 
# Writing UI Tests
 
If you created your Xamarin.Android and iOS projects with UI tests from the beginning you can skip the next steps and just start writing the tests. If not do not despair a few easy steps will allow you start writing those UI tests in no-time.
 
## Setup
 
To start writing UI tests we first have to add a UI testing project to an existing project. Simply selecting the UI Test Project will create the Project in the solution. What is still left to do is associate the Project we want to test with the test project. Right click onto References, select the Android and/or iOS Project you want to test and simply add them:
 
[![Adding iOS And Android App Projects to a UITestProject](https://mallibone.com/posts/files/6f91fb67-32ab-4c78-8557-66b76c1360da.png "Adding iOS And Android App Projects to a UITestProject")](https://mallibone.com/posts/files/faec5ff5-24bd-4f59-bce9-5d89353d9c9b.png)
 
If you take the default UI testing project from Visual Studio you might want to check if you have the latest version for Xamarin Test Cloud. Update XTC to 1.1.0 or higher or else you will get an error when executing the tests which is asking for an API key.
 

> Starting with Xamarin Test Cloud 1.1.0 or higher you no longer need a subscription for Xamarin Test Cloud to execute the tests locally.

 
### iOS and XTC
 
To run UI tests on iOS there is an additional library required that is included in the app - only the debug version, this will not be deployed to the store. Add the <font face="Consolas">Xamarin Test Cloud Agent</font> package to your iOS Project, then edit the App Delegate to look the following:
 
<script src="https://gist.github.com/mallibone/03660f676551378e4b4c92537e63ce04.js"></script>
 
Next open the Properties of your iOS project and ensure that Debug build configuration under the *Build* tab defines the Symbol: ENABLE\_TEST\_CLOUD
 
[![iOSProjectConfiguration](https://mallibone.com/posts/files/cad3a7d3-c0cf-4cdd-be52-e8649ee52382.png "iOSProjectConfiguration")](https://mallibone.com/posts/files/2b00af5c-7dee-4d57-a48f-cd92cc2b7b27.png)
 

> Now this symbol must only be set for Debug builds. Trying to submit an app with the calabash package included will lead to a rejection when trying to submit the app to the store.

 
## Writing tests
 
Enough chit chat, lets start writing those tests already. In the UI test project the <font face="Consolas">Tests.cs</font> class file contains the basic setup for a test. Lets change the AppLaunches method to this:
 
<script src="https://gist.github.com/mallibone/136d97ea86f144be73cc4d7ac8461603.js"></script>
 
Now when we start the test for i.e. Android the App will be started and a Read Eval Print-Loop (Repl) console will open. One of the most used commands when using the Repl is <font face="Consolas">tree</font><font face="Century Gothic"> now if we execute the command the following output will be printed:</font>
 
[![Shows the XTC Repl tree command when a UI has no Ids set.](https://mallibone.com/posts/files/75920c43-1890-4963-ba96-2c890dda9a2b.png "Shows the XTC Repl tree command when a UI has no Ids set.")](https://mallibone.com/posts/files/01d36e82-448c-4bb7-8884-4db909443aa3.png)
 
Now we can identify some of the controls, but it would be a lot better if we could just identify the elements over Ids. So lets just do that. Lets start by adding an AutomationId to all of our controls:
 
<script src="https://gist.github.com/mallibone/e5ee269db50a73bc2ff1200149294798.js"></script>
 
Next we will have to add a couple of lines of code to the AppDelegate.cs in the iOS Project:
 
<script src="https://gist.github.com/mallibone/58099b163906e70538e4ed9f1de4e4cb.js"></script>
 
And the MainActivity.cs of the Android project:
 
<script src="https://gist.github.com/mallibone/4aa172ba473c0c7637921f966e5f7fb7.js"></script>
 
If we rerun the test again and execute the command <font face="Consolas">tree</font> we will now see the following output:
 
[![XTC Repl showing Ids after executing the tree command](https://mallibone.com/posts/files/02247f17-af25-4cf6-8ee0-9e580123a4c2.png "XTC Repl showing Ids after executing the tree command")](https://mallibone.com/posts/files/079488cc-92ba-45cb-8e38-be27364115a5.png)
 
Working against Ids is considered best practice. For instance if the app is required to be available in multiple languages the UI test will work when the device is set to a different locale and can be used to acquire screenshots
 
In the Repl, we can now type in the following commands which will write a text in the Entry control, then submit the message and read out the text value of the label displaying the last submitted message:
 
[![Repl with the entered commands.](https://mallibone.com/posts/files/ba4656c1-ceef-442f-aeee-16d1b16c83d2.png "Repl with the entered commands.")](https://mallibone.com/posts/files/fb368afa-b3d6-4e93-80ff-38c39633d054.png)
 
One can use the copy Repl command to copy all of the executed commands to the clipboard (the tree command will be discarded from copying). We can create a new test method and simply paste the copied commands from the Repl. Add an Assert for the Text input et voila, we have created our first basic UI test. ![Smile](https://mallibone.com/posts/files/cf0207a1-554c-4bdf-9759-72f89882467c.png)
 
<script src="https://gist.github.com/mallibone/087161f91e644874b92e5bb94bfcd626.js"></script>
 
Now the test has been added with some additional lines that capture a screenshot of the app. These screenshots can be of great value when not observing the test while it is running. And since automation is all about freeing up the human to do other tasks you should enable your tests to take a screen shot after any meaning full step.
 

> Please note that taking screenshots does increase the runtime of your tests slightly so you might want to enable a global constant which allows you to easily en- i.e. disable the screenshots.

 
Further when running your tests on the Xamarin Test Cloud the screen shot will allow you to get a lot more information about each step such as CPU & RAM usage at the time of the screenshot. So it also allows you to get more telemetry of the app when the screenshot is taken. The string you pass to the method will also appear as under the test.
 
# Executing Tests
 
After creating the test we can run it on either Android or iOS. Per default the IDE settings are taken for executing the test so it is possible to select different emulators e.g. in Visual Studio you can select different Android Emulators to execute the tests on. If one hooks a device to the computer it is even possible to execute the test on the device. Again for all this there are no licenses required. The tests are base on NUnit, so as long as a NUnit Testrunner is installed the tests should just simply execute.
 
[![Showing test run result with the app in the background. The resharper NUnit Testrunner was used to perform the testrun.](https://mallibone.com/posts/files/d447ca6a-b2e0-4f4b-8647-f404b02d2c95.png "Showing test run result with the app in the background. The resharper NUnit Testrunner was used to perform the testrun.")](https://mallibone.com/posts/files/83c89d7f-3080-4bbd-bbe7-b3a1b2425921.png)
 

> At the time of writing this blog post the iOS execution is only possible on a Mac.

 
A small side note regarding the execution time of the test. Due to the fact that the app has to be deployed and then executed on the device the test takes a couple of seconds to run through. If we would think about an Integration Test performing the same task we would have a much shorter runtime, but would not be able to see if the information is correctly displayed on the screen for the user. Just keep in mind that UI Tests in general take a while longer to perform the same test that a integration test would need and that it is worth to choose beforehand which tests to execute.
 
## Running in the cloud
 
As stated before it is possible to execute the tests locally. This could even be integrated into an automated build process. But there are some limitations when executing the tests locally. For instance if we decide to attach a device we have to maintain that device. Ever heard of bloated batteries, broken cables and the like? Well those things can/will happen, further it is not possible to parallelise the tests locally out of the box. Another aspect is easily adding devices to the test set, well all this is what is the great strength of running your tests on [Xamarin Test Cloud](https://www.xamarin.com/test-cloud "The Xamarin Testcloud website"). Running the tests on the Xamarin Test Cloud can easily be integrated into a build server and allows to easily run the app on multiple devices (in parallel) and even allows to parallelize the execution of tests on multiple devices of the same type. So in short Xamarin Test Cloud offers a wide collection of devices with different OS versions that do not have to maintained and choosing which devices to use during the tests can be easily configured and changed. The results are displayed on a dashboard:
 
[![Screenshot showing the dashboard of XTC results for a testrun.](https://mallibone.com/posts/files/c289fae0-16d1-4ef5-9e8a-7cf9b18547f1.png "Screenshot showing the dashboard of XTC results for a testrun.")](https://mallibone.com/posts/files/fe95e17e-7255-41c6-9620-a5aef248fabb.png)
 
There is even an API that allows to control the execution and collect the results of a test run.
 

> Currently there are still some details missing from being available for the extraction. Such as the snapshots taking during a test run. If you would like the team to add this or other features please feel free to rate it up or tell them [here](https://testcloud.ideas.aha.io/?sort=popular "Xamarin Test Cloud User Feedback Page").

 
# Conclusion
 
In this blog post we saw how a UI Test based on Xamarin Test Cloud is created. How the UI for Xamarin.Forms should be adapted to make writing of UI tests more robust. Further we saw how we can configure the tests to capture screenshots from steps performed during a test run. Capturing screenshots of a test run not only allows to see more easily where a functional error occurred but also allows to see how the screen looks for a given device and if the layout is behaving as expected.
 
You can find the sample project and the test code on [GitHub](https://github.com/mallibone/MvvmLightSamples/tree/master/MvvmLightBindings "Link to the code of the app and the UI test.").
 
HTH
 
## Further Reading
 
[Getting started with iOS.](https://mallibone.com/post/see-how-to-get-started-writing-automated-ui-tests-with-xamarin-test-cloud-for-a-xamarinios-application "Getting started with iOS.")
