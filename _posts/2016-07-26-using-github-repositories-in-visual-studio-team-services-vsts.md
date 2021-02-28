---
layout: single
title: "Using GitHub repositories in Visual Studio Team Services (VSTS)"
title: Using GitHub repositories in Visual Studio Team Services (VSTS)
date: 2016-07-26
tags: ["git", "GitHub", "Visual Studio Team Services"]
slug: "using-github-repositories-in-visual-studio-team-services-vsts"
---

[![Blog title image showing github octocat and Visual Studio Logo]({{ site.url }}{{ site.baseurl }}/images/3ccc7f8d-1a8d-42b4-b038-1c8b20a2abfc.png "Blog title image showing github octocat and Visual Studio Logo")]({{ site.url }}{{ site.baseurl }}/images/339b4438-b71f-413a-961a-b11b6bd9a48a.png)
 
During this post I assume you are already familiar with GitHub. Visual Studio Team Services (VSTS) is a cloud hosted service which focuses on DevOps centric teams. You can use the integrated Git repository if you are looking for a private repository. But in case the project is already hosted on GitHub or it is an open source project. VSTS can act as a build server which supports 3rd party hosted repository. The path for integrating GitHub is one of the best predefined paths a thereby is one of those services. But it is that comes with a great support for integration.
 
VSTS is not a tool which is focused on a single task. It is rather a tool trying to deliver a common platform to support Teams that are living a DevOps mentality. Per default VSTS provides teams with a default tool set for source code management, build and testing. Plugins allow to bind in different tools for a certain task at hand e.g. versioning control. These plugins or external service endpoints allow to integrate GitHub as VCS with ease. I'll assume you meet the following preconditions:
 

> - Have a VSTS account
>     - You can create your free account here.
> - Have a VSTS Project
> - Have a GitHub account
> - (Have the code repository pushed to GitHub)

 
Open the projects dashboard and then open the settings menu. In the *Control panel* of your project open the tab *Services*:
 
[![Tabs in the VSTS project control panel with services selected.]({{ site.url }}{{ site.baseurl }}/images/e8e44a20-a022-4ec8-aaa7-69b60a470666.png "Tabs in the VSTS project control panel with services selected.")]({{ site.url }}{{ site.baseurl }}/images/d1397547-25d9-43c0-977a-db3882119dcb.png)
 
Then add a new service endpoint and select GitHub:
 
[![Adding a new service endpoint which includes a GitHub option.]({{ site.url }}{{ site.baseurl }}/images/b104355b-8be6-43f3-9121-6a0666ac86a3.png "Adding a new service endpoint which includes a GitHub option.")]({{ site.url }}{{ site.baseurl }}/images/9444fa5e-f6b9-4260-8bd6-048df8670bbb.png)
 
In the following dialog give the connection a name I.e. GitHub. Note that if you are already logged in to GitHub with an account, this account will be used to make the connection. If you want to use another account. Try using the private mode of your browser while performing these steps.
 
[![Dialog for configuring the external GitHub service, simply select Authorize to proceed.]({{ site.url }}{{ site.baseurl }}/images/a8375146-5957-4d4e-9689-a9dc9c8f135c.png "Dialog for configuring the external GitHub service, simply select Authorize to proceed.")]({{ site.url }}{{ site.baseurl }}/images/7895aeba-1093-48b7-b3c9-ff07a1b7dd41.png)
 
After tapping on Authorize your VSTS will be connected to your GitHub account. When configuring the build configuration for a project. It now is possible to select GitHub as the repository type I.e. select a repository hosted on GitHub.
 
[![Screen shot showing the repository tab in a build config which allows selecting GitHub as repository type.]({{ site.url }}{{ site.baseurl }}/images/8ce9af8d-6d99-49ff-ad67-97414c395c5a.png "Screen shot showing the repository tab in a build config which allows selecting GitHub as repository type.")]({{ site.url }}{{ site.baseurl }}/images/448dcf20-1dbf-4d51-9f88-870d79799015.png)

# Conclusion
 
In this post we saw how we can add GitHub as an external service to a VSTS project. After adding GitHub as a service. It is possible to select a GitHub repository within a build configuration. So you can now build source code from GitHub via VSTS. The integration at the time of writing is still limited to only the build process. It currently is not possible to reference code from GitHub in the planning part. Or see the repository in the code part of the VSTS web dashboard.
