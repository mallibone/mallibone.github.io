---
layout: single
title: "DevOps for mobile with Visual Studio App Center"
title: DevOps for mobile with Visual Studio App Center
date: 2018-01-17
tags: ["Xamarin", "Xamarin Test Cloud"]
slug: "devops-for-mobile-with-visual-studio-app-center"
---

[![flexibility-and-choice-by-microsoft-app-center.png_2]({{ site.url }}{{ site.baseurl }}/assets/images/b657d679-f6e8-428f-b3e6-61fc294ad6ef.png "flexibility-and-choice-by-microsoft-app-center.png_2")]({{ site.url }}{{ site.baseurl }}/assets/images/e6b55e67-3a1c-4bf9-a537-f0a300a50cb4.png)

What does the beginning every new year have in common? One thing is the list of resolutions we make to improve our lives and try to become a better human being. DevOps is one of those areas where many teams and companies could improve upon. DevOps has gotten quite some traction in the development world, but the task of setting up a DevOps pipeline for mobile apps can seem a bit daunting at first. While the philosophy of DevOps is by far more then simply using a piece of software, it helps to use tools to accomplish the job of automating the process of building, testing and releasing the software. To get the information of the app while it is being used there are also many great frameworks that will reliably collect data without getting in your way.


> If you are looking for an intro to DevOps. I recommend reading [The Phoenix Project](https://itrevolution.com/book/the-phoenix-project/ "The Phoenix Project book link.") which gives the reader a general idea what DevOps is and what the effects in a company may look like.


In the following blog posts you will see how a DevOps pipeline based on Visual Studio App Center is created. While creating the pipeline we will cover the following topics:

**Build** mobile apps for Android, iOS and UWP.

**Test** mobile apps on the UI level to ensure the core cases of the software are working before we release the software to users.

**Deploy** an app to testers and users. This will include internal releases and also publishing to the stores.

**Analyse** how the app is performing in the wild. Enable the app to record crashes and automatically send them to a backend. This will give the team insights on what features are being used and what bugs which closes the cycle of dev ops and allows the team to act upon their users behalf.

You will see that with Visual Studio App Center it is easy to integrate these steps and you will not even have to buy any new hardware or pay a penny to get started. If you have not done so already register your free [App Center account](https://appcenter.ms/ "App Center website"). Now let’s dive into the first post of the series regarding the build process of the app.
