---
layout: single
title: "PureLayout for Xamarin with PureLayout.Net"
title: PureLayout for Xamarin with PureLayout.Net
date: 2017-08-15
tags: ["Xamarin", "Xamarin.iOS"]
slug: "purelayout-for-xamarin-with-purelayoutnet"
---

![](https://github.com/mallibone/PureLayout.Net/blob/master/Images/LogoLarge.png?raw=true)

[PureLayout](https://github.com/PureLayout/PureLayout "PureLayout Github Page") is a library for iOS developers that allows to create UI views in code behind with ease. With PureLayout.Net this option is now available for Xamarin.iOS.

Since PureLayout.Net is based on the idea of defining your UI in code you might be tempted, especially as a C# developer, to dismiss it as an utterly bad idea. Because defining UIs in code is such a no-no right? Well actually no, not under iOS. The standard approach under iOS is to use the Storyboard to define your UIs and your screen flow. This is great at first since it is very much like Windows Forms with all it’s drag and drop goodness. The problem arises in two areas:

1. Having more then one person editing the Storyboard
2. Sharing standard values such as Margins over the entire UI



> **TL;DR; **Define your UIs in a reusable way that allows to share design standards and reduce duplication within UI work. Enable collaboration on UIs without the fear of merge conflicts by using [PureLayout.Net](https://www.nuget.org/packages/PureLayout.Net/ "PureLayout.Net NuGet repository"). Which you will be able to [use in no-time](https://github.com/mallibone/PureLayout.Net/blob/master/README.md "PureLayout.Net GitHub readme, with a how to get started guide").


Since the storyboard is one large (generated) XML file. Any change to the UI is stored in a single location. In a larger project this will mean that whenever two or more people will want to make changes to the UI a merge conflict could arise. Should a merge conflict arise… Well to put it short you are out of luck. Since the XML gets generated it is not human friendly to merge. An error while merging will result in the entire UI of the app being in tatters. Now one could mitigate this problem to a certain degree by using XIBs. But those come with some extra glue code to get it running. Plus there is still this second point.

If you are working on a larger app, you will (want to) have some sort of a style guide. In it you will define a set of defined constants for your colours, margins, text sizes, fonts etc.. Going with the Storyboard or XIB designer will not provide you with a central coded style file. The only way to define and distribute the style is in a written text document such as a Word-, PDF-File or a Wiki page. This approach comes down to every one, who works on the UI, needing to know this document. The current version may I add and apply the styles. This can be a challenge to say the least.

All this can easily be avoided by using PureLayout.Net. - Which I hope does not come as surprise at this moment ![Winking smile]({{ site.url }}{{ site.baseurl }}/assets/images/c56015e2-6edd-427e-a304-0ac4a4a1d989.png)

## Where to get it

Simply add the [NuGet package](https://www.nuget.org/packages/PureLayout.Net/ "PureLayout.Net NuGet page") to your iOS project and you will be all set:

`Install-Package PureLayout.Net`

## How to get started

A getting started guide can be found on the [projects GitHub page](https://github.com/mallibone/PureLayout.Net/blob/master/README.md).

## Thank you

I would like to point out that PureLayout.Net is only a wrapper around the existing PureLayout library written Objective-C. Therefore a big thank you to [Tyler Fox](https://github.com/smileyborg "Tyler Fox Github Profile") the creator and [Mickey Reiss](https://github.com/mickeyreiss "Link to Mickey Reiss GitHub Profile") for currently maintaining the library.

Further I would like to thank [Samuel Debruyn](https://www.chipsncookies.com) for helping me a lot on the topics of wrappers with his [blog post](https://www.chipsncookies.com/2016/creating-a-xamarin.ios-binding-project-for-dummies/ "Xamarin iOS Wrappers").
