---
layout: single
title: "Deploying your ASP.NET CORE app to NGNIX on Linux"
date: 2021-04-01 15:03
tags: ["ASPNETCORE", "NGNIX", "AZURE DEVOPS"]
slug: "aspnet-core-classic-linux"
---

One great thing about ASP.NET Core web apps is their capability to develop and run on Windows, Linux and macOS. Linux is often the choice when it comes to choosing an operating system (OS) for the server.  But how do you set yourself up to have a sustainable, trustworthy way to deploy your ASP.NET Core and operate it on a Linux server? Let's find out.

We will be touching up on the following points during this blogpost:

* Setting up the build
* Setting up the server

* Installing and configuring the build agent
* Configuring the deployment pipeline
* Configuring updates



The build pipeline







The Setup

I will assume you already have a Linux installed on your computer. For a playground you can choose a Linux VM on Azure LLLINK. First we will want to install the .NET runtime LLLINK. As of writing .NET 6 is the latest version and also a Long Term Supported (LTS) version. Next we will install the NGNIX web server LLLINK.

> Why do we need NGNIX when ASP.NET Core comes with Kestrel? While Kestrel is a more then capable web server to run your website, it does lack certain functionalities that NGNIX provides. For example it does not provide the functionality to configure a reverse proxy.







Intro

The setup (classic)

Setup the build

Install the agent

Setup the deployment pipeline

Allow for restarting

sudo visudo

```
<username> ALL = NOPASSWD: /bin/systemctl restart your-ervice.service

```

