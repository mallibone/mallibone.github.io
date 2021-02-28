---
layout: single
title: "PowerShell for the C# developer – Part 4 Testing"
title: PowerShell for the C# developer – Part 4 Testing
date: 2017-06-07
tags: [" PowerShell", "testing", "PowerShell"]
slug: "powershell-automated-testing-with-pester-for-the-csharp-developer"
---

[![Processed with VSCO with j2 preset](https://mallibone.com/posts/files/38c2ad07-b3dd-47c4-be02-7f828ed6841a.jpg "Processed with VSCO with j2 preset")](https://mallibone.com/posts/files/7b2028b2-a2f3-4eec-8a9f-fdcaaca7309c.jpg)
 
In the [previous post](https://mallibone.com/post/powershell-for-the-c-developer-&ndash;-part-3) we saw how methods i.e. functions can be implemented with PowerShell. Functions are not only a great way to structure code for reuse but also allow to create larger scripts. Whenever writing code that ends up in production it is always a must to ensure that the code runs as expected. When the Script is small or does not invoke any long running services we can do this quite simply by executing the script. However the more complex a script becomes or is integrated into long running processes the harder it becomes to ensure that the script is still running as intended after changes.
 
This is where automated Testing allows to ensure functional correctness without demanding any great user interaction to execute multiple test scenarios.
 
# Writing automated Tests
 
PowerShell comes with it’s own testing framework called [Pester](https://github.com/pester/Pester "Link to GitHub site of Pester"). Which is the de facto standard PowerShell testing framework and comes with every installation of Windows 10. Pester follows a Behavior Driven Development (BDD) test-development style. I’ll leave it up to you to follow up on the difference between Test Driven Development (TDD) and BDD, for a framework it usually shows in a more human friendly description of the tests.  
Let’s get started by having a look at what we want to do in a C# context.
 
For the C# samples we will be using the [xUnit.net](https://xunit.github.io/) framework which follows the [xUnit pattern](https://en.wikipedia.org/wiki/XUnit). Let’s assume we have the following class we would like to test:
 
<script src="https://gist.github.com/mallibone/185042ff5f74b66f8ece8f4b67ba6042.js"></script>
 
We could verify the functionality with the following test:
 
<script src="https://gist.github.com/mallibone/1b1644fcbc2f7e48ec1391118d678704.js"></script>
 
For PowerShell the same function could look like this:
 
<script src="https://gist.github.com/mallibone/40bd5b0419e38747f7ca7cb05a10670b.js"></script>
 
And the corresponding test function looks like this:
 
<script src="https://gist.github.com/mallibone/c392a5393a8d0554d79973594f637454.js"></script>
 
From the structure there is a lot in common.


> Note that Pester strongly nudges the developer to go along with a certain naming pattern for files. When a function is in a file called <font face="Consolas">Something.ps1</font>, the corresponding tests should be in the file <font face="Consolas">Something.Tests.ps1</font>.


When we execute the PowerShell test from the PowerShell IDE console pane we get the following result in the PowerShell Window.
 
[![Showing console output of the testrunner (executed in VS Code)](https://mallibone.com/posts/files/4bfd00f3-88dc-41e7-9c34-e9c6efe78274.png "Showing console output of the testrunner (executed in VS Code)")](https://mallibone.com/posts/files/4cb2e875-4ea3-454c-ae51-dfdf04c8b8ea.png)
 
There are multiple assertions you can use within pester as one can see in the following overview:

- <font face="Consolas">Should Be </font>
- <font face="Consolas">Should BeExactly </font>
- <font face="Consolas">Should BeNullOrEmpty </font>
- <font face="Consolas">Should Match </font>
- <font face="Consolas">Should MatchExactly </font>
- <font face="Consolas">Should Exist </font>
- <font face="Consolas">Should Contain </font>
- <font face="Consolas">Should ContainExactly </font>
- <font face="Consolas">Should Throw</font>

 

 
# Mocking external dependencies
 
When writing automated tests one topic quickly arises and that is how to manage dependencies to the outside world. When writing automated tests we can separate them into different categories:
 



  |  **Category** |  **Description** |
| --- | --- |
 |  Unit Tests |  Testing the core logic of code. Tests that logic has been implemented according to specs and ensures that the core building blocks are working correctly. No dependencies to other code e.g. Modules in PowerShell. Should make up the majority of automated tests in a project. |
 |  Integration Tests |  Test a part of an application i.e. Script. Ensuring that the tested parts work together which may involve calling services outside of the script itself.  <br>These tests are usually the easiest to implement but are more fragile than unit tests. An error might be caused by an other system or a logic error in the script. In large projects you should have more Unit Tests than Integration Tests. |
 |  System Tests |  Test entire use case scenarios end to end. These tests should run scenarios of the script e.g. “Setup new Web Server Instance”, “Setup new DB instance”. They usually take a long time to run and may require upfront work regarding infrastructure i.e. a staging/testing environment.  <br>In general you should only test your core scenarios with System Tests. They usually take a lot of upfront investment to create the environments and also require a lot of maintenance since they have to be adapted once a change to the system is made.  |

 
Since PowerShell was mainly created for automating infrastructure tasks, most of the code you would write is interacting with one or multiple systems in one kind or the other. As long as our scripts are only reading we might not feel the need for mocking our services i.e. as long as they are not overly slow. But when we start writing to production systems we surely do want to be certain that our scripts run as intended. And this is where mocking comes in. Under C# there are multiple libraries for mocking external dependencies of a class. A very popular one (and my personal favorite) is Moq.
 
So let’s assume we want to mock an external dependency which returns us all the entries in a directory. We would write it something like this in C# (a bit lenthy I know...):
 
<script src="https://gist.github.com/mallibone/207a202f9f169687c66804a86d93420d.js"></script>
 
Now in PowerShell we would implment the function as follows:

<script src="https://gist.github.com/mallibone/9f1fdd41b8df92a480ca57cf6355ea55.js"></script>

And Pester provides a handy mocking functionality:
 
<script src="https://gist.github.com/mallibone/edeb656f9081ea19b489a62c09e59206.js"></script>
 
Not only is this a great way to test our code without having to actually be on the filesystem but it also shows how we could mock other dependencies such as a system clock (midnight testing anyone?), network calls etc. which for one will ensure that the script does not actually invoke any external sources. Another benefit you will most probably see is a speed up in execution time. Since everything is running from in memory (aka fast ![Winking smile](https://mallibone.com/posts/files/e5961ae2-20b4-4903-a56d-651894da8c95.png)) there is no network or disk delay. For further information and options on mocking check out the [official documentation](https://github.com/pester/Pester/wiki/Mocking-with-Pester) of Pester.
 
# Executing the Tests
 
We can also execute the test from the command line with the following command:
 

> <font face="Consolas">Invoke-Pester</font>

 
The command can be extended with parameters described [here](https://github.com/pester/Pester/wiki/Invoke-Pester "Link to the Pester Project Page describing the execution."). While skimming through the parameters you may notice that they would provide the means to enable Continuous Integration (CI) for PowerShell Scripts. This is the case, we could add a Step to a Visual Studio Team Service (VSTS) or Team Foundation Server (TFS) build configuration by adding a PowerShell step with the following parameters:
 
If you already have a build server setup (you can get VSTS for free within minutes….), it is easy to setup a CI job for your scripts. And CI can be a real cure to sleep problems – so if I can have it this cheap the answer is yes please ![Smile](https://mallibone.com/posts/files/8fb07399-04da-44c0-96b8-155eca499d39.png)
 
# Summary
 
In this post we went through how to write tests with PowerShell and execute them. Further we saw how we can mock dependencies. We further looked at how tests can not only be executed during development but also how we can execute the tests in an automated build process i.e. Continuous Integration (CI) environment.
 
## References
 
Check out my previous posts on how to get started with PowerShell for the C# developer:

- [Setting up your Development Environment](https://mallibone.com/post/powershell-for-the-c-developer)
- [PowerShell basics](https://mallibone.com/post/powershell-for-the-c-developer&ndash;part-2)
- [PowerShell functions and files](https://mallibone.com/post/powershell-for-the-c-developer-&ndash;-part-3)

 
You can find more detailed information about Pester on their [Project Page](https://github.com/pester/Pester/wiki "Pester GitHub Page").

### External References

- [http://www.powershellmagazine.com/2014/03/27/testing-your-powershell-scripts-with-pester-assertions-and-more/](http://www.powershellmagazine.com/2014/03/27/testing-your-powershell-scripts-with-pester-assertions-and-more/ "http://www.powershellmagazine.com/2014/03/27/testing-your-powershell-scripts-with-pester-assertions-and-more/")
- [Mocking with Pester](https://github.com/pester/Pester/wiki/Mocking-with-Pester)

