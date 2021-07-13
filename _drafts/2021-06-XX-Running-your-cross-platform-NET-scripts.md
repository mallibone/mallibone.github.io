---
layout: single
title: "Running your cross platform .NET scripts"
date: 2021-05-08 10:03:00
tags: ["F#", "Dev", "FSI", "Scripting"]
slug: "run-scripts"
---



Since playing around more with F# scripts to ease my daily chores on the computer. I felt like my previous post LLLINK left out on the part of how you can execute those scripts. So let's focus less on the developing/tinkering but more on the execution of those F# scripts. Running scripts can be done by using the F# Interactive LLLINK tooling that is shipped with the .NET SDK LLLINK. We can execute a script as follows:

```bash
dotnet fsi the-name-of-your-script.fsx
```

So if we would have a simple script that greets you according to the time of day:

CCCCODE

You see that we are stopping the execution time for this script by using `#time`. When we execute this script I get the following result on my console:

CCCODE

But that is only half of the truth. It takes some time to spin up FSI. The spin up time is not included in the previous stats. But there  we can use the command line to help us out here. For Mac/Linux we can use the `time` tool:

```bash
time dotnet fsi greetings.fsx
```

And now we get the actual execution time:

CCCODE

Now we could put on our science hat and put this in a loop and then do some averaging etc. But nothing changes the fact that it seems to take over one second to execute this simple script. And most of the time seems to be due to FSI spinning up.

> FSI is a great tool. While it does allow you to use it for running scripts it is geared towards supporting developers as a Read Eval Print Loop (REPL) tool. And as a REPL the startup costs only hit you once, think of it as your Integrated Development Environment (IDE) starting up - it takes some time. But once the IDE is up and running you get a great productivity boost. And same goes for FSI, it is optimised for a different use case.

Now XXXX

For demonstration purpose I will reuse the script LLLINK I in my previous post that cleans up the `bin` and `obj` folders within a the subdirectories of a given path.



PATH



Alias



Slow startup times -> Use fake cli (compare startup times)



Looking back



## Less typing

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