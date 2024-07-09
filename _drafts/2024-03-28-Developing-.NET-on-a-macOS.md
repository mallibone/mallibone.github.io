---
layout: single
title: "Developing .NET on a macOS"
date: 2024-05-21 00:03:00
tags: [".NET", ".NET MAUI"]
slug: "dotnet-on-macos"
---

When developing on a Mac what are your options for developing those .NET applications? Especially since .NET offers a wide variaty of possible applications you can create. From a basic console application, to a native Mac Catalyst app, mobile apps LLLINK, web apps LLLINK, cloud native apps and IoT Applications running on embedded Hardware. There are many options. But it usually all starts with a few lines of code. And while yes theoretically we could just simply use some run-of-the-mill text editor. In todays world we want a good code highlighting as a base minimum and AI assisted Programming as a norm. So let's look at options we have as .NET (MAUI) developers using a Mac to realise our imaginations.

<!-- expand -->

## Visual Studio for Mac

When on Mac we do have to mention [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/) (VS4Mac for short). Having it's origins back as the IDE (Integrated Development Environment) for developing C# outside of the Microsoft sanction Windows Visual Studio to develop Mono and later Xamarin .Net mobile Applications.

IIIMAGE

VS4 Mac provides an IDE experience with many features like Testrunner, Project-, NuGetmanagement and intellisense. But VS4Mac always lagged behind it's name bretheren Visual Studio (for Windows). And Microsoft has announced that they will discontinue to maintain Visual Studo for Mac on the [31. August 2024](https://learn.microsoft.com/en-us/visualstudio/mac/what-happened-to-vs-for-mac?view=vsmac-2022&viewFallbackFrom=vsmac-2022&wt.mc_id=DT-MVP-5002881).

With that you should probably look elsewhere if you are looking for a .NET coding solution on MacOS.

## Visual Studio Code

Chances are high you already know Visual Studio Code (VS Code). A lightweight code editor that supports most languages under the sun. It comes with the options of installing extension. Extensions allow VS Code to incorporate new functionality. Such as syntax highlighting, AI Assitant chats, cloud integrations. It is safe to say you could loose entire weeks combing through all the extensions and what they make possible at this point in time.

IIIIMAGE

So let's have look at what you will need to get started with VS Code. First let's download the [VS Code](https://code.visualstudio.com). If you have not installed .NET and/or .NET MAUI yet using Visual Studio for Mac, you can check out the [official docs for .NET devs](https://code.visualstudio.com/docs/languages/dotnet?wt.mc_id=DT-MVP-5002881) or the [official docs for .NET MAUI devs](https://learn.microsoft.com/en-us/dotnet/maui/get-started/installation?view=net-maui-8.0&tabs=visual-studio-code&wt.mc_id=DT-MVP-5002881) for .NET MAUI setup - it will explain how you can setup your machine and which extensions/licenses are required.

At the time of writing if you are devleoping .NET MAUI apps using XAML, you will find a few things missing. For one Hot Reload is missing and XAML intellisense just made it into the preview LLLINK. So be sure to checkout the [Meteor](https://marketplace.visualstudio.com/items?itemName=nromanov.dotnet-meteor) extension by [Nikita Romanov](https://www.linkedin.com/in/nikita-romanov-75b837281/). Which provides not only  Intellisense. It will also enable hot reload. Be sure to check out the GitHub [repo](https://github.com/JaneySprings/DotNet.Meteor).

For your AI powered "pairing-sessions" you can further install the [GitHub Copilot](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot) extension. By further installing the [GitHub Copilot chat](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot-chat) you will be able to chat with the CoPilot and ask it inline or in a side pannel for help, insights and code. Please note that at time of writign GitHub Copilot requires you to have a payed license after the evaluation period expires.

When using VS Code you will notice that it requires you doing things in the Terminal (the default command line app for macOS). While you can have the terminal window embedded in VS Code. If you come from Visual Studio or VS4Mac this might feel a bit rough at the beginning. However keep in mind that many developers actually prefer developing this way. A command line often gives you options that are difficult to pack into a visual way without bombing the user experience. This is one of the reasons I never bothered to learn how to use Git via Visual Studio/Rider/VS4Mac UI interfaces. Once you know the commands in the terminal you tend to get more options and quicker than the peek and click UI option.

But what if you are not looking to become the next terminal warrior? Well we still have options. 😉

## Jetbrains Rider

Rider is a crossplatform .NET IDE from Jetbrains. LLLINK It comes packed with features that you come to expect from an IDE. If you have been developing .NET for some time you might know Resharper. A tool that also is developed by Jetbrains as a plugin for Visual Studio (Windows). Those features are part of the Rider IDE. There are many features you would expect from IDE such as sourcecontrol integration, NuGet package management, test runners, DB accessors, Docker DevContainer integrations, and [many more](https://www.jetbrains.com/rider/features/). You even get Profiling (CPU and Memory) for .NET applications - though not for .NET MAUI apps. But you also get many cool plugins - for example my much beloved [VIM plugin](https://github.com/JetBrains/ideavim). Not telling you this is a must, but if you know VIM keybindings, be sure to check it out. XXXX

IIMAGE

Rider provides all those refactoring features and many more. To name a few more you will get sourcecontrol (git) Integration, DB management tools, NuGet package management, Profiling (CPU and Memory), Testrunners and many more. Rider can be extended with Plugins LLLINK. And also your AI code buddy is there for you. Jetbrains provides the option to use their own AI XXXX tool. Or you can get access to GitHub Co-Pilot via Plugin.

If you are looking for a .NET IDE experience on macOS and have been using Visual Studio or Visual Studio for Mac so far. You should give Jetbrains Rider a try. There are some differences, but all in all it will provide you with the IDE experience you have become accustomed to. Out of personal experience it is a nice, snappy and powerful coding tool. And I use it as my daily driver for developing code on my mac. Note that Rider does usually require you to purchase a license.

## Visual Studio

Another option is using Visual Studio for Windows. For this you will have to run Windows in a VM. Prallels LLLINK or VM Ware Fusion LLLINK are two of the more popular choices that many use. Then install Visual Studio. If you are using a Apple Silicon device (with 16+ GB of RAM), this experience is absolutely doable. The performance is not laggy and you it does not feel like you are waiting for the VM to catch up to your typping. And you will get to use Visual Studio with all of it's many features. LLLINK. I do use this approach for projects that either require Windows (Desktop and/or .Net Framework projects). At the time of writing you will also get the best hot-reload experience using Visual Studio on a VM.

But I fully understand this approach is not for everyone. Especially since you probably would be using a Windows machine if you would want to develop on Windows. But it is an option, and it works really well if your machine has the necessary horse power to run a VM.

## Closing thoughts

Developing .NET on Mac is not only possible you get first party support for many of the tools from Microsoft. Gone are the days where .NET development was a Windows only thing. At work some of my team prefer a Windows machine while others (including myself) work on a Mac. We work on the same git repositories and everything just works. As a .NET developer it feels great to know that I have multiple options to write my code in.

Please note that as a Microsoft MVP I get a complementary Visual Studio, Visual Studio Code and Jetbrains Rider license. Neither Microsoft nor Jetbrains have asked me to write this blog, nor have they received advance notice or have reviewed this post before it has been published.



Links VS Code setup for .NET devs: https://code.visualstudio.com/docs/languages/dotnet