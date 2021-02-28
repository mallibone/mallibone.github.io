---
layout: single
title: "Thoroughly testing your mobile app"
title: Thoroughly testing your mobile app
date: 2016-03-24
tags: ["Xamarin", "Mobile", "testing", "Xamarin Test Cloud"]
slug: "thoroughly-testing-your-mobile-app"
---

[![testcloud_devices]({{ site.url }}{{ site.baseurl }}/images/8f22c8bc-a8da-424d-b7a0-b8fd5704fa83.png "testcloud_devices")]({{ site.url }}{{ site.baseurl }}/images/1de4c855-d59f-4afd-a8ab-1bad7317bbce.png)
 
When developing an app e.g. a mobile app for iOS, Android and Windows 10 or a web app what initially stands in the spotlight is a core idea that brings a core value to it’s users. As soon as the scope is better known one ideally starts developing a Minimal Viable Product which demonstrates the core ideas and let’s the creators get first feedback from potential users. With the initial ideas and the feedback the team starts to add more and more functionality to the app and all is well. Until users start reporting some odd behaviour and the well intended changes start having side effects on other features. So every release gets thoroughly tested before it is sent out. The time it takes to check on all features starts to increase, but holding off of going through all the tests may lead to a new bug that snuck into the app. So what to do?
 
# Bring The Fairy Dust - Automated UI tests
 
This is often the point where one person e.g. a non-technical manager hears about a UI testing framework – which solves exactly the problem of automating manual tests. Plus one great thing about a UI testing framework usually is that there are no changes required in the app. The coverage of a test is very high since one works against the entire stack.
 
Automated UI tests simulate user interaction on the UI layer of an app. Depending on what kind of app there are different frameworks that one can choose from. For mobile apps on iOS and Android the solution we will be looking at is Xamarin Test Cloud (XTC) which is based on the Calabash testing framework. XTC allows to create UI test that can be used across platforms. They can be run locally i.e. read “phones attached to the Computer of the tester”. Far more interesting though is the opportunity of running the same tests in Xamarins Test Cloud which offers hundreds of devices (with different OS versions) on which the app and tests can be run on.
 
But as so many things in life - UI tests come at a cost. Compared to unit- or integration tests they are rather slow and like integration tests they tend to be more on the brittle side i.e. a test fails due to changes to the app. This is especially true if the UI is still under a lot of change. That being said they can be a great tool to bring down manual testing time. But solely relying on UI tests for your mobile tests will most probably not result in the result you were hoping for. So what are the sweet spots where automated UI tests will bring you the largest benefit?
 
Well lets start by looking at what kind of tests you can actually implement when considering mobile applications:
 
[![TestDistribution]({{ site.url }}{{ site.baseurl }}/images/68134717-0591-488b-b7eb-f32c6f8d2ff5.png "TestDistribution")]({{ site.url }}{{ site.baseurl }}/images/f3e3749e-49e0-47a2-8760-6ce06590ca38.png)
 
As the graphic indicates the wider the pyramid the more tests you will probably want to have in your project. So lets dive into what the main benefits are for the different areas and let’s also look at how they could be implemented.
 
# Unit Tests
 
Testing your basic logic of a class is where Unit Tests come into play. A Unit Test should not rely on any dependencies a class may have and therefore requires Stubs and Mocks to evaluate the correct working of business logic. Unit Tests should always pass without an error since they check the basic logical implementations of an app.
 

> UI Tests should always pass i.e. be green since they only test logic without any dependencies.

 
Further Unit Tests are fast. In fact a Unit Tests is considered slow if it runs longer than100 milliseconds. For larger projects it is not uncommon to have a large number of these tests checking that all the small building blocks are working according to specification/user story. The additional work required for stubbing out dependencies may result in tedious work for certain parts of an app that are mainly coordinating workflows. Further since they only test logic in isolation the Unit Tests do not give any feedback how the components work together. But when all Unit Tests are green the confidence in the logic of an is very high and allows to shift the focus on search for an issue to the interaction of the building blocks which is where Integration Tests come into play.
 
## Characteristics of a Unit Test
 

  |  What is tested |  Logic Blocks, single Classes, Parts of Classes |
| --- | --- |
 |  Execution Time / Test |  &lt; 100 ms |
 |  Reliability |  Very High |
 |  Impact on Architecture |  High |

 
# Integration Tests
 
When multiple parts e.g. classes of an app want to be tested an integration test is usually the way to go. Integration tests can range from testing a couple of classes that work closely together to system tests that require multiple parts of a system up and running. For external services that are not part of the developed app stubs and mocks may be used to ensure that tests do not fail due to services failing outside of the teams scope. Integration tests usually take more time to run since they run the real life scenarios. Due to the number of different parts and sub-systems an integration test may be running through the tests tend to be more brittle than a unit test. Further it can get harder to pin point errors to a specific area of a project, so unit tests are a perfect completion to integration tests as they allow to test complex logical parts in separation. If we look at a standard app built according to the MVVM pattern, the Integration tests are usually run from the View Model or lower in the stack. Integration Tests do not only allow to test for functional correctness of an app but also allow to get a feel for how performant an app is. If you are interested to run your tests on an actual device you can read more about this topic [here](https://mallibone.com/post/run-xunit-tests-on-android-ios-and-windows "Blog post on running xunit tests on devices").
 
One issue with integration tests is that the View is not being tested. Even though may argument that when the MVVM pattern is applied correctly there is only minimal logic remaining in the View, the user will still use the app via the view. This leaves the possibility that the app will still have issues when operating it through the view. Here (automated) UI tests come into play.
 
Generally speaking if a project has no automated tests at all, integration tests tend to give a bigger bang for the buck. Since the architectural requirements are not equally thorough.
 
## Characteristics of an Integration Test
 

  |  What is tested |  Interaction between componentens, Integration with other System parts, Network communications, Long running tests, memory consumption |
| --- | --- |
 |  Execution Time / Test |  Seconds to hours |
 |  Reliability |  Medium - High |
 |  Impact on Architecture |  Medium |

 
# UI Tests – Xamarin Test Cloud
 
Now it might be tempting to avoid testing your app altogether when having a sophisticated test suite of Unit- and Integration Tests, but in the end the user will interact via the UI with the app. And generally speaking they do not care if the logic beneath is sound, when the app crashes on their Device due to some incompatibility with a UI element that is used in the app. So the minimum UI testing every app project should be performing is the manual test. A person taps through the app and ensures that the UI responds as expected and follows the intended workflow. Now as stated before a manual test takes a lot of time and the resource performing the test I.e. a human might feel a bit underwhelmed. So generally speaking where can we utilise automated UI tests to improve test coverage:
 
- Smoke Testing
- Workflows
- Testing interaction
- Presence of Elements
- Localization
- Multi device tests

 
It might be tempting to think one can eliminate manual testing altogether, but there are some areas that UI tests just won’t be able to cover automatically and will always need a human:

- Layout (Elements not rendering correctly)
- System Interactions
    - Camera
    - Photo Album
    - Sharing
- UI “Snappiness”
- Exploratory Testing
- Offline Testing
- Bluetooth

 
Even though the Layout can be covered by an automated test insofar, that it takes pictures which can be verified by a user/MD5 Hash compared. A computer (at least at the time of writing) does not have the capability to analyse the layout of an app for its correctness. So a human eye and brain is still needed for some tests, this is something to keep in mind when automating all the things. Other limitations come either from the setup e.g. there must be a WiFi connection to the phone to run the automated UI tests. In general testing interconnectivity to other devices i.e. proprietary hardware or plainly some other phone that has to be paired, UI tests often become quite cumbersome to setup and the XTC does generally not support these kind of scenarios. So yes there are limitations to UI tests but using them properly will not only greatly enhance the coverage of automated tests but also free up valuable tester time that can then be used to put the human brain to more advanced testing tasks then numbly tapping through an app.
 
## Characteristics of a UI Test
 

  |  What is tested |  Tests entire app software stack, Smoke Testing, Workflow Testing |
| --- | --- |
 |  Execution Time / Test |  Minutes |
 |  Reliability |  Medium – Low |
 |  Impact on Architecture |  Low |

 
# Conclusion
 
There are multiple different ways how one can approach automated testing of a mobile app. On one end there are tests verifying the logic of the single building blocks an application is built upon. Integration tests allow to verify the correct interworking of parts of the app, this can extend over the border of a an app and also include calls to a backend for verifying correct integration. The highest level tests available to mobile applicaitons are automated UI tests. The Xamarin Test Cloud allows create UI tests that can be executed on many hundred of devices. Testing apps on so many devices before the app ships can lead to valuable insights and automating these tests allows to easily perform regression tests not only on logic and integration layers but also for the UI. Think about an upcoming OS release and you want to check if there any issues with your app, given automated UI tests you can easily run a test suite that will quickly inform you if there is any concern with the upcoming update.
