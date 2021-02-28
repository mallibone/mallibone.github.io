---
layout: single
title: "PowerShell for the C# developer"
title: PowerShell for the C# developer
date: 2016-12-21
tags: ["PowerShell"]
slug: "powershell-for-the-c-developer"
---

![Screenshot of a Windows Powershell from WikiPedia](https://upload.wikimedia.org/wikipedia/commons/d/d5/Windows_PowerShell_1.0_PD.png "Screenshot of a Windows Powershell from WikiPedia")

Lately I have been doing some work with PowerShell. PowerShell is a well known in the Windows IT Pro space when it comes to the automation of tasks. In todays fast paced world the idea of automating mundane tasks is an attractive idea. PowerShell brings forward the basic toolset to automate, not just mundane tasks, but set up entire environments. Since I do most of my day work in C# I wanted to share some of my insights into developing with PowerShell. Which should just be enough to get into trouble ![Winking smile]({{ site.url }}{{ site.baseurl }}/images/c44ff76a-bbc3-4057-a670-40ee02c3d2d3.png)



# The development environments

PowerShell comes right out of the box on Windows 7 or higher machines and even comes with it’s own editor the PowerShell Integrated Scripting Environment (PowerShell ISE). PowerShell ISE allows you to edit PowerShell files which includes code highlighting and IntelliSense and run them in Debug mode (F5) or running the selected code parts (F8). It is even possible to set breakpoints (F9) which should make C# developers feel right at home.

[![image]({{ site.url }}{{ site.baseurl }}/images/3d393440-70ab-48a5-a990-233249f0b65a.png "image")]({{ site.url }}{{ site.baseurl }}/images/231a0e18-ad5d-45a0-80c2-a5378982433b.png)

An interactive PowerShell displays the output of the running script. The shell also comes in handy while debugging your code. At a given breakpoint you can inspect the variable by typing it's name into the console. There only rant I have about the PowerShell ISE is that the console not be reset. The only way this can be achieved is by restarting the editor and having to open (all) the files again.

## Visual Studio

As a C# developer the daily driver usually is Visual Studio. If you can’t bear the thought of using a different Editor. Well you are covered. The experience is much like with the PowerShell ISE. So if you have any Visual Studio Plugins such as [VsVim](https://github.com/jaredpar/VsVim "Link to the VsVim Git repository") you do not want to miss this may be your preferred route.



If you haven’t installed PowerShell during the initial installation of Visual Studio. You can add PowerShell by modifying your Visual Studio installation. Open *Programs and Features*, choose Visual Studio and then select *Change*. In the dialog select *Modify* and choose *PowerShell Tools for Visual Studio*. Finally select *Next.*You might want to get a cup of your favourite beverage while the installation is taking place.


> Visual Studio will create it’s usual Solution and Project files. This will result in some extra clutter that your average PowerShell developer might not recognise. Keep this in mind when you check those files into your repo.


## 

## Beyond Windows

PowerShell is usually run on top of the .Net Framework. As of lately PowerShell also supports running on .Net Core. This means your PowerShell scripts are not limited to Windows. You can write them also for Apple and Linux machines. One of the PowerShell editors of choice for Linux and Mac, would be [Visual Studio Code](https://code.visualstudio.com/) and the [PowerShell Plugin](https://github.com/PowerShell/PowerShell/blob/master/docs/learning-powershell/using-vscode.md "Link to VS Code PowerShell plugin project site") for it.

[![image]({{ site.url }}{{ site.baseurl }}/images/3aa76444-c904-4385-a224-86c9ea17c57c.png "image")]({{ site.url }}{{ site.baseurl }}/images/fed40103-37e2-4732-bea1-5f06466ab8ec.png)

You can see an extensive description on how to use PowerShell with Visual Studio Code on the [msdn](https://blogs.msdn.microsoft.com/powershell/2015/11/16/announcing-powershell-language-support-for-visual-studio-code-and-more/ "Link to MSDN article &quot;Announcing PowerShell language support for Visual Studio Code and more!&quot;") website.

# Why choose PowerShell over C# or [CScript](http://scriptcs.net/)

![reactions why ryan reynolds but why](https://media.giphy.com/media/1M9fmo1WAFVK0/giphy.gif)

Since PowerShell runs on the .Net Framework, why should we use PowerShell in the first place? Wouldn't it be easier to just write the code with C# in the first place? One of the best arguments in my opinion comes from the reason why PowerShell came to be. It is designed from it's roots up to automate IT tasks. There are many hooks in the OS or other major Services. Active Directory, IIS, Exchange and many more provide interfaces that can be consumed with PowerShell. So if your task is to automate a Windows environment, PowerShell will be able to consume and interact with many existing APIs. Making the task easier and less time consuming.


> A fun fact is that the UI for configuring IIS actually performs PowerShell commands in the background. The identical commands are used when invoking them through a PowerShell script.


Since we are writing infrastructure code there is a good chance that an Ops person will end up maintaining it. There is a good chance that this person has an interest in putting in the effort of learning PowerShell. Since it is their day job and PowerShell is sought out to enable them during their daily tasks.

And third, it is always fun to dive into a new programming language. And finally PowerShell has a way to interact with C#. So if the need ever arises to get stuff done the good old C# way there is nothing stopping you ![Smile]({{ site.url }}{{ site.baseurl }}/images/8cc19e6c-e6df-452a-b1f7-35e4628c91e0.png)

# 

Hello PowerShell

Before we end this post, let’s see the Hello World Example:


    echo "Hello PowerShell"


Jup that is it, one line and we are done.

In the [next post](https://mallibone.com/post/powershell-for-the-c-developer–part-2) we will dive into the basics of writing PowerShell code.

# References

There is more in this blog post series:

  


- [PowerShell setup](https://mallibone.com/post/powershell-for-the-c-developer)
- [PowerShell basics](https://mallibone.com/post/powershell-for-the-c-developer–part-2)

