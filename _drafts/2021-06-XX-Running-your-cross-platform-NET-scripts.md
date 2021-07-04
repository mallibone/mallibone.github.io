---
layout: single
title: "Running your cross platform .NET scripts"
date: 2021-05-08 10:03:00
tags: ["F#", "Dev", "FSI", "Scripting"]
slug: "run-scripts"
---



Intro



PATH



Alias



Slow startup times -> Use fake cli (compare startup times)



Looking back



## Go

With a working script we can now save the file and invoke it from the command line. Only problem is, we kind of have to remember where the script is. One possible solution is putting it into a folder in our users root directory. Let's say we put all our scripts into the `scripts` folder we could then invoke your script like this:

```bash
dotnet fsi ~/scripts/cleanBuildDirs.fsx
```

And although this command looks rather bash-like it actually also works when using PowerShell. But it still feels quite long winded, what if you have some scripts you always want at the tip of your fingers. Perhaps the ones that get you through your day just that bit easier and quicker, well for those we want to go one step further.

So with your script at hand how about we simply had to type a few keys hit enter and have it run, where ever we are. While this is not a F# tip per say, it is always good to remember that the command line comes with a few powermoves of it's own. To be more precise we actually have two options. One is adding the script folder to the path variable. With this you can simply type `dotnet fsi cleanBuildDirs.fsx` from where ever you want. Configuring the path depends on your operating system: Windows, macOS and Linux.

But let's say you write a script that you want to execute multiple times a day. You probably want something even faster than that. And this is where setting up an alias comes into play. On macOS you can add the following line to the end of your XXX file:

CCCODE

For powershell you can add the following line to your XXX file:

CCCODE

You might have to restart your terminal/shell of choice for the changes to take effect. But then you can simply enter the alias and off your script will run.

Note: If the file does not exist, you can go forward and create it.