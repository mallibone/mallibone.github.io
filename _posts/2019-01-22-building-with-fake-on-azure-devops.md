---
layout: single
title: "Building with FAKE on Azure DevOps"
title: Building with FAKE on Azure DevOps
date: 2019-01-22
tags: ["Azure DevOps", "Mobile", "F#", "Fabulous"]
slug: "building-with-fake-on-azure-devops"
---

[![Showing Wall-E infront of a yellow VW bus with taxi stripes]({{ site.url }}{{ site.baseurl }}/assets/images/cbd8f3a6-3222-454b-9123-889428497f30.jpg "Showing Wall-E infront of a yellow VW bus with taxi stripes")]({{ site.url }}{{ site.baseurl }}/assets/images/291cd57e-ac98-4fdc-9519-ab37855338a6.jpg)

I have taken quite a liking into [Fabulous](https://github.com/fsprojects/Fabulous) - a wrapper around Xamarin.Forms allowing you to write functional UIs with F#. When first looking at the project I noticed that is was being built on [AppVeyor](https://www.appveyor.com/) and [Travis](https://travis-ci.org/). I asked myself: Why use two CI Systems for compiling one project? After some further digging I found out that there are no hosted macOS Agent on AppVeyor. Travis on the other hand did come with agents for Windows and macOS but did not have the Xamarin Toolchain installed on the agents. Installing the Xamarin Toolchain on every run lead to a build time of over 30 minutes. Since [Azure DevOps](https://azure.microsoft.com/en-us/solutions/devops/) supports building on Windows and macOS I thought I would give it a go and setup a Pipeline to build Fabulous - I mean how hard can it be? Well hard enough to write a blog post to sum up the steps to get over the pitfalls


> TLDR: How to run your FAKE scripts on Azure DevOps


Fabulous uses [FAKE](https://fake.build/) to execute the build, tests and create the NuGet packages. FAKE is a power full tool for writing build scripts. FAKE is also a [.Net Core CLI tool](https://docs.microsoft.com/en-us/dotnet/core/tools/?tabs=netcore2x) which is designed for being installed and executed from the command line, so it should be a great fit for running on any build server.

# Installing FAKE

Azure DevOps build agents do not come with FAKE preinstalled. Since FAKE is a .Net Core CLI tool this is no problem. The following command should solve this issue:


    dotnet tool install fake-cli -g


Unfortunately executing FAKE after installation fails. This is because the installation directory on the Azure DevOps build agents differs from the standard installation location of .Net Core - Why? you ask, well the answer given is security. On Windows we can circumvent this fact by installing FAKE into the Workspace directory:


    dotnet tool install fake-cli --tool-path .


Under macOS (and Linux) this approach still fails. The [suggested solution](https://github.com/Microsoft/azure-pipelines-image-generation/issues/531) is to set `DOTNET_ROOT`. I ended up with the following lines to be executed on the macOS agent:


    export DOTNET_ROOT=$HOME/.dotnet/
    export PATH=$PATH:$HOME/.dotnet/tools:/Library/Frameworks/Mono.framework/Versions/Current/Commands
    dotnet tool install fake-cli -g


On Linux the approach had to adopted again - go figure. I ended up with these lines:


    export PATH=$PATH:$HOME/.dotnet/tools:/Library/Frameworks/Mono.framework/Versions/Current/Commands
    dotnet tool install fake-cli -g


Now you should be able to run your FAKE script on Azure DevOps

# Using NuGetFallbackDirectory

This part is not directly related to FAKE but is something I stumbled over while running on Azure DevOps. One test script was referencing the NuGet packages via the global `NuGetFallbackDirectory` and was looking for them under the default location. Under macOS the location is in the users home directory, so adopting the path as follows did the trick:


    let tfsEnvironment = Environment.GetEnvironmentVariable("TF_BUILD")
    if (String.IsNullOrEmpty(tfsEnvironment)) then
        "/usr/local/share/dotnet/sdk/NuGetFallbackFolder"
    else
        let homepath = Environment.GetEnvironmentVariable("HOME")
        Path.Combine(homepath, ".dotnet/sdk/NuGetFallbackFolder")


Note that the variable `TF_BUILD` is expected to only be set on TFS/VSTS/Azure DevOps. This will allow the script to fall back to the default location should it be executed on a developers machine.

# But why even bother?

What is the motivation of migrating from a working CI to another? Are you doing because you are a Microsoft MVP?

These were questions I got when talking with colleagues about my endeavors to build Fabulous on Azure DevOps. I think AppVeyor and Travis are great tools and they have shown that they are up to the task building and testing Fabulous. Other than because I was curious how hard it could be, there were two aspects why I wanted to try to migrate the build to Azure DevOps:

1. Merging the builds, having two places doing one thing always comes with overhead.
2. The other one was seeing how much the build time would be reduced by not having to install Xamarin.


So here is a comparison between the build times before and after:
<figure><table>
<thead>
<tr><th>CI Platform</th><th>Agent OS</th><th>Build</th><th>Test</th><th>Time (minutes)</th></tr></thead>
<tbody><tr><td><a href="https://travis-ci.org/fsprojects/Fabulous/builds">Travis</a></td><td>macOS</td><td>✅</td><td>✅</td><td>~30-32</td></tr><tr><td>Azure DevOps</td><td>macOS</td><td>✅</td><td>✅</td><td>~13-14</td></tr><tr><td><a href="https://ci.appveyor.com/project/dsyme/elmish-xamarinforms/history">AppVeyor</a></td><td>Windows</td><td>✅</td><td>❌</td><td>~4</td></tr><tr><td>Azure DevOps</td><td>Windows</td><td>✅</td><td>❌</td><td>~6</td></tr></tbody>
</table></figure>
Now to keep in mind, the build on macOS and Windows are ran in parallel. So in case of AppVeyor and Travis the resulting build time would be 30-32 minutes. With Azure DevOps this can be brought down to 13-14 minutes.

I would argue that merging two build scripts into one and cutting build time roughly in half are good arguments for why Azure DevOps seems to be a better fit for Fabulous. Then again there was some pain on getting the .Net CLI tools running, which I hope the Azure DevOps team will solve in the future - being products from the same company and all *cough*

Another aspect was having a build on Linux in the future, since Fabulous supports GTK since 0.30 it would be nice to also compile it on Linux. At the time of writing there were still a few kinks in the build process of Fabulous, but nothing that can't be solved in the future.

# Thanks

Thank you [Timothé Larivière](https://twitter.com/Tim_Lariviere) and [Stuart Lang](https://twitter.com/stuartblang) for all the tips and hints along the way
