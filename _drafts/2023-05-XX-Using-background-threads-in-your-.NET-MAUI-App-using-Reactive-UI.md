---
layout: single
title: "Mastering threads in your .NET MAUI App using Reactive UI"
date: 2023-05-23 00:03:00
tags: ["ReactiveUI", "Rx", ".NET MAUI"]
slug: "rxui-schedulers-101"
---

Reactive UI provides features that keep the .NET MAUI UI thread responsive. Frozen UIs do not only look ugly and may give you a bad rap with your users. It might also lead to the OS forcefully quitting the app. Therefore, every developer must know how to write code that does not block the UI.

<!-- expand -->

When a .NET MAUI app starts, it invokes all code on one thread. Since it not only runs the logic but also renders the UI. The thread is called the UI or main thread. There are two main reasons why a thread can get blocked. One is waiting for a response, aka busy-wait. This is usually some Input/Output (IO) operation like waiting for a web request, reading/writing a file etc. In .NET, asynchronous operations have become much easier with the task-based asynchronous/parallel pattern. Then again, Reactive Extensions ([Rx.NET](https://github.com/dotnet/reactive)) have been around longer and offer a way to do said tasks. They even allow us to easily mix and mingle with Task-based flows. Let's see how Rx keeps your apps nice and fluid while avoiding common gotchas.

## The scenario

We are writing an application that retrieves JSON serialized data from a web service. The JSON must then be deserialized and processed before we can use it further in the app. The web request is a classical I/O operation typical in many mobile applications. In our code, we can write the web request as follows:

CCCCODE

One thing to always keep in mind is that Rx.NET allows to change between `Task` and `IObservable`. Depending on your preferences you can rewrite the code above as follows:

CCCCODE

There is no wrong or right here. But Working with `IObservable` allows you to chain calls in a way that many C# devs know from LINQ. If we want to deserialize the result we can write it by mapping the input string to the deserialized Plain-Old-C#-Object (POCO):

CCCCODE

So far you have seen how we can handle asychnronous operations. But what if our deserialization is something more fancy. What if it would take multiple seconds until we had our data converted?



## Running in the background

Whenever there is a compute intesive task that blocks a thread, we have an issue when writing a UI application. Since the thread that every user interaction originates from. Having the thread running a 100% on a non-UI thread results in a unsresponsive UI. This is bad. To work around set issue, we will want to hand off the compute intensive work to another thread. With Observables we have helpers to instruct on which thread operations shall be performed. By using XXX we can rewrite the code as follows:

CCCODE

Reactive UI provides a the helper XXX which allows to choose between XXX and YYY. By choosing XXX we can instruct the following operations to run on a different thread then the UI thread. By using YYY we can switch back to the UI thread. Switching back is important, trying to change the UI from another thread than the UI thread leads to an app stopping exception. The exception will inform you: "You shall not update the UI from another thread."

When looking at Rx.NET there is also the ZZZ. At first this can be a bit confusing. Like XXX the ZZZ statement call be placed anywhere in the Observable composition. But it only affects how the `Subscribe` part will be invoked. Depending on implementation of your Observable composition, the UI will be updated somewhere in the middle. Or at the end in the `Subscribe` part. With this knowledge it is up to you the developer to choose the right function.

## Async, parallel and MVVM

A nice thing to note is that whenever you run something in parallel or while waiting for a response from a backend, you seldom want the user to invoke this action again while it is still being executed. If you are using Reactive Commands i.e. ReactiveCommand.XXX or ReactiveComand.YYY the command will automatically be XXXXXX

## Conclusion

Reactive UI comes with nice helpers for solving asynchornous and parallel challanges that can arise in your .NET MAUI application. Knowing how to use the helpers is key. When writing asychronous i.e. non-blocking code it is good to know that switching between a `Task` and an `IObservable` is done with ease. While composing the observables you can choose to run certain actions on a different thread. Running work on a different thread allows the UI thread to stay responsive. Last but not least, using ReactiveCommands will ensure that the user will not "try-to-speed-things-up" aka push a button multiple times while waiting for a result, since the command will be disabled until the IObservable or Task completes.

HTH
