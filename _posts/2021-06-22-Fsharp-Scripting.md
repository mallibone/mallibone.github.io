---
layout: single
title: "Automate your dev life with F# scripts"
date: 2021-06-22 10:03:00
tags: ["F#", "Dev", "FSI", "Scripting"]
slug: "fsharp-scripting"
---

Using scripts allows you to automate repeating tasks. Scripting should save not only time but also reduce the risk of making mistakes. You can think of them as automated checklists. There are many ways to write scripts, but my favourite way is to use F# since it gives me many helpers dealing with data input. Further, I can rely on all of the trusted .NET features plus pull in some NuGet packages should the need arise. So let's see how we can write scripts, ensure they do what we want and use them in our everyday life.

<!--more-->

## Getting setup
To write and run F# scripts on your machine, you will need the [.NET SDK](https://dotnet.microsoft.com/download/visual-studio-sdks), an editor with F# sintax highlighting - I recommend [Visual Studio Code](https://code.visualstudio.com/) (VS Code) and the [Ionide plugin](https://marketplace.visualstudio.com/items?itemName=Ionide.Ionide-fsharp). And that is it; you are ready to create your first script. Open VS Code, open a new tab, hit save and create a file with the file ending `.fsx`. 

> Though I recommend VS Code here, I have written a lot of code using [Visual Studio](https://visualstudio.microsoft.com/), [Visual Studio for Mac](https://visualstudio.microsoft.com/) and [Rider](https://www.jetbrains.com/rider/) from JetBrains. There is nothing wrong with those options, and if you are already familiar with one of them. Please do not feel pressured to change. However, if you are just getting started, VS Code is a lightweight option to get started.



## Getting to know F# scripts

If you are writing an F# script, it will execute on [F# Interactive](https://docs.microsoft.com/en-us/dotnet/fsharp/tools/fsharp-interactive/) (FSI). Let's start with this minimal script below:

```ocaml
printfn "Hello dear blog reader"
```

We could store this one-liner script head over to a terminal and (in the same directory as the script file) run the following command:

```bash
dotnet fsi the-name-of-your-file.fsx
```

The script will print the message on the terminal. But when developing, there is a way more convenient way to work with FSI as a REPL (Read-Eval-Print-Loop). So in VS Code, we can highlight a piece of your code and hit Alt+Enter. The highlighted code will be executed in VS Code and display our greeting. So same result but a lot more convenient while developing.

## Writing scripts
While this is all cool and great, let's look at something every .NET developer at some point will want to do: Smash that keyboard against the moni... - err, I mean deleting those obj and bin folders. Aka the build outputs from your project because something (for example, a Git rebase) made stuff go wonky.

When writing a script, you can start with no methods. Start listing all directories in a given path:

```ocaml
Directory.EnumerateDirectories("~/TestDirectory/") |> Seq.toList
```

And from there on, step by step, build out the functionality of your script. You are constantly testing it by sending the highlighted parts to the REPL. At some point, you will want to structure your code into methods to get a better overview of what is going on. So to get all bin and obj directory paths for a given root folder - we could end up with something like this:

```ocaml
let EnumerateDirectories path =
    Directory.EnumerateDirectories(path) |> Seq.toList

let isObjOrBinFolder (folderName:string) =
    folderName.EndsWith("obj", true, CultureInfo.InvariantCulture) || folderName.EndsWith("bin", true, CultureInfo.InvariantCulture)

let rec getFoldersToDelete path =
    match EnumerateDirectories path with
    | [] -> []
    | subfolders  ->
        let targetFolders = subfolders 
                            |> List.filter isObjOrBinFolder
        let targets = subfolders 
                            |> List.filter (isObjOrBinFolder >> not) 
                            |> List.collect getFoldersToDelete
                            |> List.append targetFolders
        targets
```

The `EnumerateDirectories `returns a list of directories in a path. The `isObjOrBinFolder` method returns `true` if a given folder is either named `bin` or `obj`. Finally, the `getFoldersToDelete` recursively iterates through every subdirectory. Collecting all the folders, we intend to delete. To remove the folders, we add an additional method:

```ocaml
let deleteFoldersAndSubFolders path =
    getFoldersToDelete path
    |> List.iter (fun dir -> 
        printfn "Deleting: %s" dir
        Directory.Delete dir)
```



What we now still need is the directory we scan for the `bin` and `obj` folders. Let's say we want to give us the option to pass in the path as a parameter, and if we don't hand in a parameter, we want to take the directory the script is invoked from:

```ocaml
match fsi.CommandLineArgs |> Array.skip 1 with
| [||] -> 
    deleteFoldersAndSubFolders (Directory.GetCurrentDirectory())
| directoryPaths -> 
    directoryPaths
    |> Array.iter deleteFoldersAndSubFolders
```

The `Directory.GetCurrentDirectory()` is a handy helper to provide you with the path from which your script is running from. Speaking of .NET helpers, F# offers built-in values that provide you information of the directory `__SOURCE_DIRECTORY__`, filename `__SOURCE_FILE__` and line number `__LINE__`. Another great helper I like to use in my scripts is `#time`. Placing it, e.g. at the top of your script, will tell you all kinds of information regarding CPU time and garbage collection. Note it acts as a toggle, so running `#time` multiple times in FSI will turn it on/off/on/off/you-get-it-by-now-right-😉.

## Using the power of NuGet
While it is pretty straightforward to reference parts of the .NET SDK by using `open Something.Something`, it is pretty much the same when you want to reference a NuGet package. For example, let's say you want to add Newtonsoft.JSON to our script. We could do this with the following lines:

```ocaml
#r "nuget: Newtonsoft.Json"
open Newtonsoft.Json

// more codez

let movie = JsonConvert.DeserializeObject<Movie>(json)
```

Pretty straight forward ey?  😃 You can use the same syntax to reference local dlls. So if you have a dll locally lying around but not on NuGet, you could do something like this `#r path/to/dll/theDllName.dll` and then open the namespace you want to use.

## The aftermath

Scripting can be a fun way to automate some everyday tasks and ensure that no human error creeps up. F# provides you with an excellent .NET based scripting experience. And being able to use NuGet packages is a significant productivity boost. Just keep in mind that writing a script may take some time. So you might want to think about the benefit vs the cost - as this [XKCD](https://xkcd.com/1319/) nicely highlights. Then again, writing scripts can be great fun and a great way to learn the .NET SDK, NuGet packages and F# in general. You can find the complete script [here](https://gist.github.com/mallibone/cf3fab05314ff5196a7936e005064f8c).

But enough of the ad-real. Time to think about what scripts you could write to automate your everyday developer life.

HTH
