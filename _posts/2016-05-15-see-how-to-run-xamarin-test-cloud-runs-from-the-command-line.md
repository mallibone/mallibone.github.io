---
layout: single
title: "Running your Xamarin Test Cloud from the Command Line"
title: Running your Xamarin Test Cloud from the Command Line
date: 2016-05-15
tags: ["Xamarin Test Cloud", "Xamarin", "testing"]
slug: "see-how-to-run-xamarin-test-cloud-runs-from-the-command-line"
---

[![title image showing 1s and 0s in a green console style.]({{ site.url }}{{ site.baseurl }}/images/c0c61947-1ed2-4304-bdb6-5ec45e00a7fe.jpg "title image showing 1s and 0s in a green console style.")]({{ site.url }}{{ site.baseurl }}/images/22d78ac7-7894-4c29-a692-089fa9abe23f.jpg)
 
Running your tests on [Xamarin Test Cloud](https://www.xamarin.com/test-cloud "The Xamarin Testcloud website") (XTC) is as easy as right clicking onto tests and select run in XTC. As easy as this approach is – as limited it becomes when more sophisticated demands arise. Those might be running a group of tests, running on preconfigured device groups or integrating into an automated Continuous Integration (CI) and Continuous Deployment (CD) process. But there are also some additional useful features and information which we shall see in the follow up of this blog post.
 
Thereby the focus will be on the following:
 
- Creating and Running the basic Command
- Setting Categories and using them in the Command
- Recording your Android XTC test runs
- Using the XTC API to retrieve even more information

 
So lets get started ![Smile]({{ site.url }}{{ site.baseurl }}/images/54ed6407-2cfa-4866-8eea-e3a55f99072d.png)
 
# The basics Command
 
The easiest way to get a test script is if you login to the Xamarin Test Cloud dashboard [website](https://testcloud.xamarin.com). Click on New Test Run, now you can either select an existing app or just choose to create a new App, select a team, devices that should be included in the test run and some additional information such as series, device locale and parallelisation. In the last step a sample script is generated which is the basic recipe for running the tests from the command line.
 
<script src="https://gist.github.com/mallibone/1b7ee061ce5ba1b3ece92f8a3044e6b9.js"></script>
 
Now this script has to be adopted at the various points to reflect the setup of the app under test but from there on simply navigate to the root e.g. the solution folder and execute the script from the command line, power shell or the shell of your choosing. The tests will be uploaded and run on the selected devices. Once the tests are completed the results can be viewed in the XTC dashboard.
 
# Running specific categories of your test suite
 
XTC allows to set categories when declaring a test as follows:
 
<script src="https://gist.github.com/mallibone/b64e57bd12cc2be9deead804cf37318a.js"></script>
 
Categories allow to choose a subset of the tests to be run when executing them on the XTC. This can be a great way to create fast running smoke tests that give instant feedback after a build or running focused tests on a area of the app. The command can be extended by adding the following parameter:
 
<script src="https://gist.github.com/mallibone/ff60de5a6152a9b4df530ea347c888d5.js"></script>
 
If applicable one choose multiple categories for one run which makes mix and matching an easy undertaking. So a command using e.g. the categories <font face="Consolas">SmokeTest</font> and <font face="Consolas">Purchase</font> would be the following command:
 
<script src="https://gist.github.com/mallibone/c4155e0d26a6297b60f01614a5e59cee.js"></script>
 
Okay now to becoming more insights into tests, which is especially interesting when they are failing…
 
# Recording your test runs
 
The basic option to get a better sense of what is going on during a test run is taking screenshots. This can be done by adding the following line to your tests as follows:
 
<script src="https://gist.github.com/mallibone/464a799ca5a73dd1ee986e4d572da468.js"></script>
 

> Note taking screenshots might increase the duration of a test run, in case of large suites one might want to reduce the overall time by using a constant within the UI test project that enables/disables detailed reporting of the test runs.

 
Now screenshots are already a great help. But in case of Android you are able to record the test runs. How awesome is that?! Enabling recordings is as easy as adding an additional parameter to the command line:
 
<script src="https://gist.github.com/mallibone/e2fec05a8fdcb96865b8bf3a4374c690.js"></script>
 
The recording can be viewed on the test dashboard with the very usable feature of speeding up the playback so you wont be bored by watching how your test fights it’s way to the juicy part. And as of date of writing recordings are only available for Android and can be solely enabled by adding the parameter to the Command Line Interface (CLI). If you try to use the command on an iOS run there will be no exceptions but the recording will just simply be missing.
 

> Also don’t forget to add a good failure message to the Assert. Or you will be left with the reason of a failed test because an expected value was false instead of true (not helpful for me…).

 
# Accessing the XTC API
 
Invoking tests via the command line is a great way to integrate UI tests running on XTC to an automated DevOps build pipeline. By default adding the following parameter to the command will allow to receive an NUnit Test report, it gives a very high level report of the run i.e. will show which tests have failed but not on which device etc.. If you desire to have a richer interaction with the data you can start interacting with the [Xamarin Test Cloud API](https://testcloud.xamarin.com/api_docs/index.html?csharp#introduction). There is a ton of information you can retrieve about a test run when using the API. Unfortunately at the current time it is not possible to retrieve images, recordings and some other information. If you are interested in giving the team an initiative to retrieve this information you can vote for this feature here.
 
All in all if you are using an on premise or other third party dashboard for displaying your test results, make sure you check out the API as it may just provide the more detailed information you are looking for and do not find within the NUnit test-report.
 
# Conclusion
 
Starting test runs for Xamarin Test Cloud from the command line not only lets you integrate it into a DevOps toolchain but also allows you to run sub-sets of your tests, or record your test runs for Android. Last but not least if you are interested on feeding your test results into a dashboard the Xamarin Test Cloud API might just provide you with the information you are looking for.
 
The command line option is not only valuable for integration into automated toolchains but also for a more granular control when executing the tests locally. Be it from Windows or OSX the Xamarin Test Cloud toolchain has you covered.
