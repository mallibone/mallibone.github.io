---
layout: single
title: "Accessing a private NuGet feed from Azure DevOps"
title: Accessing a private NuGet feed from Azure DevOps
date: 2019-11-06
tags: ["Azure DevOps"]
slug: "private-nuget-feed-azure-devops"
---

![Black Metal Padlock](https://images.pexels.com/photos/706500/pexels-photo-706500.jpeg?auto=compress&amp;cs=tinysrgb&amp;h=750&amp;w=1260)


> **Update:** So after posting this my colleague and friend [Daniel](https://twitter.com/damukles_)approached me and showed me the [Azure Artifacts Credentials Provider](https://github.com/Microsoft/artifacts-credprovider) by Microsoft which automates the steps bellow. Be sure to check it out. Thanks, Daniel for showing me this and making my life easier 😃


So lately I was playing around with one of Azure DevOps many features. Namely pushing freshly created NuGet packages to your [private feed](https://docs.microsoft.com/en-us/azure/devops/artifacts/get-started-nuget?view=azure-devops). Bringing up the question how can I access the feed and authenticate during a NuGet restore process via `dotnet restore`?


> While this blog post shows steps to be taken for Azure DevOps - the same actions are required in the `NuGet.config` for other sources.


While I knew how to click my way around Visual Studio to do this. Under my Ubuntu Shell, this was not an option. Luckily adding a NuGet feed is quite common knowledge, while the paths differ under Windows and Unix systems you will find it in your home directory under:

`~/.nuget/NuGet/NuGet.config`

Or for Windows that would be:

`%appdata%\NuGet\`

You can add the feed to your `NuGet.config` file:


    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
        <packageSources>
            <add key="nuget.org" value="https://api.nuget.org/v3/index.json" protocolVersion="3" />
            <add key="NameOfYourFeed" value="path to your nuget/index.json" />
        </packageSources>
    </configuration>


Now for accessing a private NuGet feed, you will have to provide a username and password. You can add them to the config file:


    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
        <!-- packageSources -->
        <packageSourceCredentials>
            <NameOfYourFeed>
                <add key="Username" value="gnabber"/>
                <add key="ClearTextPassword" value="YourPassword"/>
            </NameOfYourFeed>
        </packageSourceCredentials>
    </configuration>


The only issue being you probably do not want to store your Azure DevOps password in plain text on a computer. And you shouldn't do that either. So let's head back over to Azure DevOps, click on your profile picture and select "Security". Now generate a new token. Be sure to select "Show all scopes" then under Packaging choose the "Read" permissions.

[![Image showing the dialog to generate a token for readonly access to your package feed]({{ site.url }}{{ site.baseurl }}/assets/images/f6f77fca-f0cc-48a2-902e-14a35b753f67.png "GenerateToken")]({{ site.url }}{{ site.baseurl }}/assets/images/b93fda55-7f79-452d-8d6f-527a83cc3a74.png)

Copy the generated token and store it in the NuGet.config within the `PlainTextPassword` field. You can now `dotnet restore` your packages from the private Package feed.

HTH
