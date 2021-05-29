---
layout: single
title: "Automate your dev life with F# scripts"
date: 2021-05-08 10:03:00
tags: ["F#", "Dev", "FSI", "Scripting"]
slug: "fsharp-scripting"
---



Using scripts allows you to automate repeating tasks. This should not only save time but also reduce the risk of making mistakes. You can think of them as automated checklists. There are many ways how you can write scripts but my favourite way is using F# since it gives me a lot of helpers dealing with data input. Further I can relly on all of the trusted .NET features plus pull in some NuGet packages should the need arise. So let's see how we can write scripts, ensure they do what we want and use them in our everyday life.

## On your marks

To write and run F# scripts on your machine you will need the .NET SDK LLLINK, an editor with F# sintax highlighting - I recommend Visual Studio Code LLLINK (VS Code) and the Ionide plugin LLLINK. And that is it your are ready to create your first script. Open VS Code open a new tab, hit save and create a file with the fileending `.fsx`. 

## Get set

If you are writing a F# script it will be executed using F# Interactive LLLINK (FSI), let's start with this minimal script bellow:

```F#
printfn "Hello dear blog reader"
```

We could store this one liner script head over to a terminal and (in the same directory as the script file) run the following command:

```bash
dotnet fsi the-name-of-your-file.fsx
```

The script will be executed printing the message on the terminal. But when developing there is a way more convenient way to work with FSI as a REPL (Read-Eval-Print-Loop). So in VS Code we can highlight a piece of your code and hit Alt+Enter. The highlighted code will be executed in VS Code and display our greating. So same result but a lot more convenient while developing. 

While this is all cool and great let's look at something every .NET developer at some point will want to do: Smash that keyboard against your mo... - err I mean deleting those `obj` and `bin` folders aka the build outputs from your project. Because something (for example a Git rebase) made stuff go wonky.

When writing a script you can start out with no methods. Simply start listing all directories in a given path:

CCCODE

And from there on step by step build out your scripts functionality. Always testing by sending the highlighted parts to the REPL.

This script could look something like this in F#:

CCCODE

Now at the top of the file you can see that the required namespaces are opened for the script to use.

Go





Getting setup (.NET SDK & Code + Ionide)

Writing a script (REPL, reference NuGets)

Having the scripts at the tip of your fingers

Conclusion (automate every day chores, more than just a fun toy)

Other Editors include VS & Rider