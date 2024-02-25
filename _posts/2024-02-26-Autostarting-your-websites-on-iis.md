---
layout: single
title: "Autostarting your (ASP.NET Core) websites on IIS"
date: 2024-02-26 00:03:00
tags: ["IIS", "ASP.NET Core"]
slug: "iis-autostart-website"
---

You might have an ASP.NET Core app that you deploy to IIS (Internet Information Server) that, for example, listens in the background to incoming IoT Signals from a Message Queue, but it only does so after you have visited the server in the browser. Having an app that should do background work but does not start automatically can be cumbersome at best and lead to outages, aka angry support calls, at worst. Let's look at how we can ensure an app is started automatically on startup. How can we change this to make your app appear automatically after deployment?

<!-- expand -->

You might ask yourself: "Okay, let's see how we can configure a site to come up automatically. But why do I even have to do this in the first place?" - The answer is quite simple. IIS does this to save resources. Reducing resource usage allows hosting a plethora of sites on your server, and IIS will ensure that sites that are not in use will spin down, aka only get started once they are in need. While this is great if you have many small sites to run that only run once in a while, it's a pain in the neck if you have a site that should always be up and running since it does more things than respond to HTTP requests.

## IIS Setup

Usually, hosting your site on IIS will run on a Windows Server once you hit production. IIS is installed using the *Server Manager* and added under *Roles and Features*.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/2024-02-24-IisRoleWindowsServer.jpg" alt="IIS Role installation under server manager, Roles and Features." style="zoom:40%;" />

The *Application Initialization* role service is not added by default when enabling IIS. Be sure it is enabled for auto-start to work.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/2024-02-24-IisAddAppRoleApplicationInitialization.jpg" alt="IIS Application initialization role." style="zoom:50%;" />

## IIS app configuration

Microsoft [docs](https://learn.microsoft.com/en-us/aspnet/core/tutorials/publish-to-iis?view=aspnetcore-8.0&tabs=visual-studio&wt.mc_id=DT-MVP-5002881) already cover deploying your app to IIS on a Windows server. After that, change the *Start Mode* to *Always Running* under *Advanced Settings* of the AppPool.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/2024-02-24-IisRoleWindowsServer.jpg" alt="IIS app pool always running configuration" style="zoom:50%;" />

Then go to the website and set *Always Running* to *true* in the *Advanced Settings*.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/2024-02-24-IisWebsiteAlwaysRunningTrue.jpg" alt="IIS web application always running configuration" style="zoom:50%;" />

Your app will automatically start running once the app pool starts up. Note that you will only want this for some apps you are developing. But whenever you have an app that also serves Requests over a different interface than HTTP - for instance, RabbitMQ, MQTT, SQL, file-based, etc., you will either have to remember always to visit the page or else it will not respond as you might intend.

HTH