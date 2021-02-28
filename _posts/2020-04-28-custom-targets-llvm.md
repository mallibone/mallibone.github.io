---
layout: single
title: "Enabling LLVM for your Xamarin.iOS custom build targets"
title: Enabling LLVM for your Xamarin.iOS custom build targets
date: 2020-04-28
tags: ["Xamarin.iOS", "Xamarin.Forms"]
slug: "custom-targets-llvm"
---

[![Title image showing a rev counter gauge]({{ site.url }}{{ site.baseurl }}/assets/images/73503f52-0a3a-49f9-a6cf-3ee47a31c3a0.jpg "Title image showing a rev counter gauge")]({{ site.url }}{{ site.baseurl }}/assets/images/b282fc7c-de5c-42e4-9e21-09107926d7ac.jpg)

Build targets are not only great to differentiate between a Debug and Release builds. You can also use them for targeting different environments or configurations of your app. Now I always like the idea of getting the best performance for apps that I put into my user's hands - in other words; I fancy to enable [LLVM](https://en.wikipedia.org/wiki/LLVM)

Unfortunately when creating a new Target with [Visual Studio 2019](https://visualstudio.microsoft.com/) (as of writing 16.5.4) the option to enable LLVM is disabled.

[![Showing disabled LLVM option in Visual Studio - and a screaming emoji]({{ site.url }}{{ site.baseurl }}/assets/images/6ca080fb-e56a-4e1e-a598-577855555ac0.png "Showing disabled LLVM option in Visual Studio - and a screaming emoji")]({{ site.url }}{{ site.baseurl }}/assets/images/614a5719-17a7-4950-b570-3d7de89842ba.png)

The [issue](https://developercommunity.visualstudio.com/content/problem/1001497/xamarinios-llvm-cant-be-enabled-for-custom-build-t.html) is under consideration by the team, and for the time being, there is no way to enable LLVM via the UI Wizard in Visual Studio. Now one way to solve this is to clone your solution on to a machine running macOS and then enabling it in Visual Studio for Mac. But under Windows, the only option is to open up the `csproj` file and enable LLVM manually:


    <?xml version="1.0" encoding="utf-8"?>
    <Project ToolsVersion="4.0" DefaultTargets="Build" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
      <!-- Stuff -->
      <PropertyGroup Condition="'$(Configuration)|$(Platform)' == 'Gnabber|iPhone'">
        <OutputPath>bin\iPhone\Gnabber\</OutputPath>
        <DefineConstants>__IOS__;__MOBILE__;__UNIFIED__;</DefineConstants>
        <Optimize>true</Optimize>
        <PlatformTarget>AnyCPU</PlatformTarget>
        <UseVSHostingProcess>false</UseVSHostingProcess>
        <LangVersion>7.3</LangVersion>
        <ErrorReport>prompt</ErrorReport>
        <CodeAnalysisRuleSet>MinimumRecommendedRules.ruleset</CodeAnalysisRuleSet>
        <!-- Add the line bellow -->
        <MtouchUseLlvm>true</MtouchUseLlvm>
      </PropertyGroup>
      <!-- More Stuff -->
    </Project>


Thanks, [Victor Garcia Aprea](https://twitter.com/vga) for pointing this out to me and I hope this can be of some help to anyone out there stumbling over the same problem. If you want to check

You can find a small sample Project with a custom build target `Gnabber` up on [GitHub](https://github.com/mallibone/CustomTargetLLVM).

HTH
