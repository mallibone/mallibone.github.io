---
layout: single
title: "Mastering threads in your .NET MAUI App using Reactive UI"
date: 2023-05-23 00:03:00
tags: ["ReactiveUI", "Rx", ".NET MAUI"]
slug: "rxui-schedulers-101"
---

Reactive UI provides features that keeping the .NET MAUI UI thread responsive. Frozen UIs do not only look ugly and may give you a bad rap with your users. It might also lead to the OS forcefully quiting the app. Therefore it is paramount that every developer knows how to write code that does not block the UI.

<!-- expand -->

When a .NET MAUI app starts it invokes the code on one thread. Since it not only runs the logic but also renders the UI. The thread is called the UI or main thread. There are two main reasons why a thread can get blocked. One is waiting for a response. This is usually some Input/Output (IO) operation like waiting for a web request, reading/writing a file etc.. In .NET many of the 

Intro

Chaning threads when using observables

ReactiveUI threading options

Conclusion