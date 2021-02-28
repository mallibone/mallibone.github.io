---
layout: single
title: "Looking at Xamarin.Forms from Enterprise view"
title: Looking at Xamarin.Forms from Enterprise view
date: 2015-06-17
tags: ["Mobile", "Xamarin", "Xamarin.Forms"]
slug: "looking-at-xamarin.forms-from-enterprise-view"
---

[![Xamarin Platform](http://www.mallibone.com/posts/files/91258526-3740-4689-8ef2-9a4bc60eef66.png "Xamarin Platform")](http://www.mallibone.com/posts/files/54276cf1-1dee-49c5-a520-528f79c5a4e0.png)
 
Xamarin founded in 2011 (former Mono) has been enabling C# developers to write mobile apps for Android and iOS. Xamarin provides full access to the entire SDKs of the targeted platforms and therefore allows to build native apps that perform like apps written in the platform default language e.g. Java, Swift or Objective-C, look like native apps and provide a user experience like native apps. This makes it an ideal candidate to write high quality apps which provide a great user experience without having to rewrite the entire application for each targeted platform. See Christoph Rehmann’s [post](http://blog.noser.com/cross-platform-development-mit-xamarin/) on an overview of Xamarin and a comparison to Hybrid and web applications in general.
 
# Anatomy of a Xamarin application
 
The standard approach for writing Xamarin applications is to write The common approach of writing Xamarin apps allows to share all the platform independent code.
 
***[![XamarinStack](http://www.mallibone.com/posts/files/c6feacd5-2ddf-4967-95c2-27cd66ccee0f.png "XamarinStack")](http://www.mallibone.com/posts/files/d98cdc14-9b07-494e-940e-50a58c1cc7ef.png)***
 
But as soon as it comes down to writing the UI for the application or integrating native features such as a *Widget* or *Push Notifications* the developer will integrate those using the platform specific SDKs which usually are different for each platform. This is a logical coincidence for getting the full native SDKs and therefore also means that developers must embrace the platform designs and principals or else frustrated users will follow. For certain apps this approach makes a lot of sense as it allows to share the logic and present the user a native user interface that acts the way she knows and is accustomed to. Plus it let’s you design those fancy and unique UIs that really can set you apart from the competition. But what about the following two examples?
 
- But in many apps there are screens that look pretty much the same across multiple platforms, this can be ranging from an about page, manual, checkout or login screen. With the standard Xamarin approach these views have to be written for each platform, though one might argue that apart from some small tweaks the UI stays greatly the same.
- Other apps just want to allow the user aka employee to get stuff done while he is on the go. These apps are often form based applications that have the demand of getting out of the users way, work offline and be available on different form factors and in case of a Bring Your Own Device (BYOD) policy platforms.

 
For these kind of cases a lot of people start looking towards a HTML based or hybrid approach. Making a decision between native i.e. Xamarin and Hybrid is a discussion on it’s own. The gist is that Hybrid applications in theory allow you to not only share your business logic but also your UI, on the UI part you might end up writing code specific for each platform but as stated this shall not be the focus of this post. That being said wouldn’t it be great if for simple UIs we had the possibility to share the UI code across platforms? And this is exactly where Xamarin.Forms comes in.
 
# 
 
# Meet Xamarin.Forms
 
Given the requirements stated above, paired with the desire of relying on native applications with native user controls that not only looks like a native app but also performs the way the user expects an app to work is exactly the reason why Xamarin.Forms was created. Xamarin.Forms allows you to write your UI once and share it across multiple platforms resulting in a maximum of code being shared:
 
***[![Xamarin.Forms](http://www.mallibone.com/posts/files/09f428ba-eafe-4e87-9b89-05cf8496e665.png "Xamarin.Forms")](http://www.mallibone.com/posts/files/4c2d7d4d-321e-4d2f-856a-b33f2cb098a8.png)***
 
The UI is still fully native and allows the developer to create a native app experience. But instead of writing the UI over and over for each platform the UI is only created once and is then matched to the native controls and layouts that exist on Android, iOS and Windows (Phone). And the best part is that one can mix it with the standard Xamarin approach. So even if one is going for a unique design experience the simpler screens can still be written in Xamarin.Forms and shared across the platforms.
 
## 
 
## The UI
 
The UI is usually written in the Portable Class Library (PCL) which is a special library that allows code to be shared across C# library stacks such as Phone/Tablet Store Apps, .Net, ASP.Net, X-Box and with Xamarin iOS and Android. So writing your code in the PCL allows you to share the UI code to all platforms which are supported by the Xamarin.Forms framework which are:

- iOS
    - iPhone/iPod
    - iPad & iPad Mini
- Android
    - Phone
    - Tablet
- Windows
    - Widnows Phone 8 & 8.1
    - Windows 8.1

 
The UI usually needs some tweaking when moving from one form factor to another but again some controls could be shared and with Xamarin.Forms they can be shared not only across factors but platforms. From a UI stand point Xamarin.Forms comes with limitations. The controls are a limited set and though they theoretically do allow to create a custom design it is currently not advised as the rendering will perform not as well than it’s Windows, Xamarin.iOS or Xamarin.Android UI counterparts.
 

> Generally speaking as long as you are creating an app that can be created with standard components with minimal modifications Xamarin.Forms should serve you just fine.

 
That being said it is possible to write a part of the UI with Xamarin.Forms and integrate it with native screens. So one could start out with a Xamarin.Forms app and when you start bumping into performance issues on certain screens you can rewrite it for each platform and benefit from creating customized UIs and integrate to standard components such as login forms with Xamarin.Forms.
 
## 
 
# Integrating the platform
 
So Xamarin.Forms is an abstraction layer that allows developers to create UI’s once and then translates the UI to native UI controls on the specific platform. But what if you want to access the SDK of a given platform e.g. push notifications. With other cross platform approaches this often means you have to use a plugin that provides you the functionality for the platform. But as Xamarin.Forms builds on the full Xamarin stack you have full access to the SDK of the platform and are not limited by available plugins or SDK restrictions.
 
# 
 
# Limitations
 
So apart from not being great for designing highly customized apps are there any further limitations to Xamarin.Forms? I guess one of the biggest limitations to Xamarin.Forms at the moment is it’s age. It came out on 28. May 2014 and still is a 1.0 product, which are known for having some rough edges and Xamarin.Forms is no exception. So there are some issues that one may be bound to run into e.g. no UI designer, performance on data heavy views and the usual bugs that accompany V1.0 releases. That being said there is a huge interest in the Framework and it is already being used by a wide variety of apps i.e. fields. And given the recent efforts and releases of the team I believe Xamarin.Forms is a great option for certain types of applications.
 
# Conclusion
 
We saw that Xamarin.Forms allows to develop native mobile apps for multiple platforms and thereby share the business logic and UI. Xamarin.Forms allows to write apps with XAML and is a great starting point for apps or writing prototypes. Because it integrates fully into the existing Xamarin stack there is no downside and switching between native UIs and Xamarin.Forms is a possibility for developers.
 
If your interested in the basic setup of a Xamarin.Forms app, [here is a Hello World](https://github.com/mallibone/HelloXamarinFormsXaml) app that will run on Windows Phone, iOS and Android and shares all the XAML UI code.
 
This post was originally posted on the [blog of Noser Engineering](http://blog.noser.com/looking-at-xamarin-forms-from-enterprise-view).
