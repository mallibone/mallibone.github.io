---
layout: single
title: "Developing .NET on a macOS"
date: 2025-03-01 00:03:00
tags: [".NET", ".NET MAUI"]
slug: "dotnet-on-macos"
---

When developing on a Mac what are your options for developing those .NET applications? Especially since .NET offers a wide variaty of possible applications you can create. From a basic console application, to a desktop apps for Windows/Mac/Linux, mobile apps LLLINK, web apps LLLINK, cloud native apps, games and IoT Applications running on embedded Hardware. And that list is not even complete. .NET just seems to run about anywhere these days. But at the beginning everything starts with a few lines of code. And while theoretically we could just simply use some run-of-the-mill text editor. In todays world we want a good code highlighting as a base minimum and AI assisted Programming as a norm. So let's look at options we have as .NET developers using a Mac to realise our imaginations.

<!-- expand -->

Before we dive into the editors let's have a little look into the past. .NET has it's origins with the .Net Framework which was solely based on Windows. With Mono LLLINK the first port to Linux and Mac was here. With that came Mono Develop an IDE that allowed you to write C# without having to use Visual Studio. Mono Develop then became Xamarin Studio which then was renamed to Visual Studio for Mac after Xamarin got acquired by Mircosoft. Visual Studio for Mac aka VS 4 Mac got retired by Microsoft on the [31. August 2024](https://learn.microsoft.com/en-us/visualstudio/mac/what-happened-to-vs-for-mac?view=vsmac-2022&viewFallbackFrom=vsmac-2022&wt.mc_id=DT-MVP-5002881) - and that marked an end of an era for some developers. Further Microsoft no longer offers a IDE for non-Windows systems. But Microsoft does however offer a Code Editor that is being used by many developers outside of the .NET ecosystem.  And that is nothing less than Visual Studio Code. LLINK

Let's look if Visual Studio Code up to the job of developing large .NET applications? Also let's look at the alternatives if you want an IDE experience on your Mac.

## Visual Studio Code

Chances are high you already know Visual Studio Code (VS Code). A lightweight code editor that supports most languages under the sun. It comes with the options of installing extension. Extensions allow VS Code to incorporate new functionality. Such as syntax highlighting, AI Assitant chats, cloud integrations. It is safe to say you could loose entire weeks combing through all the extensions and what they make possible at this point in time.

IIIIMAGE

So let's have look at what you will need to get started with VS Code. First let's download the [VS Code](https://code.visualstudio.com). Next - and since these steps sometimes get changed or improved - check out the  [official docs for .NET devs](https://code.visualstudio.com/docs/languages/dotnet?wt.mc_id=DT-MVP-5002881) and if mobile is a goal the [official docs for .NET MAUI devs](https://learn.microsoft.com/en-us/dotnet/maui/get-started/installation?view=net-maui-8.0&tabs=visual-studio-code&wt.mc_id=DT-MVP-5002881). Here you will get all the information/plugins to setup you up for success. Further your license questions should also be answered here.

One feature of Visual Studio Code that stands out is the extensive plugin ecosystem. The plugins extend VS Code abbility to be used for various tasks. For instance you will find plugins for managing Azure, Docker and even virtual Pets to keep you company. LLLINK When following the links above you will install plugins that will enable VS Code for .NET and .NET MAUI development.

For your AI powered "pairing-sessions" you can further install the [GitHub Copilot](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot) plugin. Paired with the [GitHub Copilot chat](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot-chat) you will be able to chat with the CoPilot and ask it inline or in a side pannel for help, insights and code. You can use it for free or pay a fee if you start to use it more extensivly for your development.

Once you have your essential plugins installed you can start using VS Code. You will find it is quite capable. You can Develop, Debug, Test, Deploy you projects from VS Code. One main difference between VS Code and other IDEs like Visual Studio or Rider is the keyboard centric interaction. While IDEs come with many keyboard shortcuts - in VS Code the onw you will to remember is `Shift+Cmd+P`. This opens the command palette. The command palette opens a search like entry which allows you search and execute commands you want to. Need to open a terminal window to do some command line shenanigans. Simply start typing "New Term". And the option will be presented to you. Plugins will hook into the command palette and extend it with additional commands. Now you can still use your mouse/trackpad - but searching for that menu item just tends to be so much quicker. Another VS Code shortcut that can come in handy is `Cmd+P` which will allow you to search files in the opened directroy and sub-directories.

I would be amiss that one of the great features of VS Code is how well GitHub Copilot is integrated. Copilot will autocomplete code you are writing. And you can even write comments in line that Copilot will pick up on. But by using the inline chat `Cmd+I` you can write/ask copilot directly at the line/position your cursour currently is. If you want some more extensive edits, that might involve multiple files you can open the chat window `Shift+Cmd+I`. In the chat you can reference files, terminal, compile error window or even the entire codebase and give Copilot the context it requires to answer your instruction/question best. The ease of integration and that fact that new models/features are introduced first in VS Code has lead me to use VS Code for more and more developement tasks.

Generally VS Code with the .NET extensions, Github Copilot and some personal preferences (like using VIM keybinding) makes it a very snappy editing experience. Are there features missing? Yes - for instance there is no integrated memory analyzer. Generally many of the visual interactions are not 

Another feature I have become quite fond of is the integration with GitHub Copilot. XXXXX		

If you come from Visual Studio or VS4Mac this might feel a bit rough at the beginning. However keep in mind that many developers actually prefer developing this way. A command line often gives you options that are difficult to pack into a visual way without bombing the user experience. This is one of the reasons I never bothered to learn how to use Git via Visual Studio/Rider/VS4Mac UI interfaces. Once you know the commands in the terminal you tend to get more options and quicker than the peek and click UI option.

!!!! If you are a .NET MAUI developer I would be amiss to  [Meteor](https://marketplace.visualstudio.com/items?itemName=nromanov.dotnet-meteor) extension by [Nikita Romanov](https://www.linkedin.com/in/nikita-romanov-75b837281/). Which provides not only  Intellisense. It will also enable hot reload. Be sure to check out the GitHub [repo](https://github.com/JaneySprings/DotNet.Meteor).

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

Developing .NET on Mac is not only possible you get first party support for many of the tools from Microsoft. Gone are the days where .NET development was a Windows only thing. At work some of my team prefer a Windows machine while others (including myself) work on a Mac. We work on the same git repositories and everything just works. And since .NET also supports ARM it does not if you are using an X64 or ARM based CPU. As a .NET developer it feels great to know that my code will run on so many platforms. Having multiple options when it comes to the tool I want to use to write my code with is icing on the cake.

Please note that as a Microsoft MVP I get a complementary Visual Studio, Visual Studio Code and Jetbrains Rider license. Neither Microsoft nor Jetbrains have asked me to write this blog, nor have they received advance notice or have reviewed this post before it has been published.



Links VS Code setup for .NET devs: https://code.visualstudio.com/docs/languages/dotnet