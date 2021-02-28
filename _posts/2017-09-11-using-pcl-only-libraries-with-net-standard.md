---
layout: single
title: "Using PCL only libraries with .Net Standard"
title: Using PCL only libraries with .Net Standard
date: 2017-09-11
tags: ["Xamarin", "DotNetStandard", "Reactive UI"]
slug: "using-pcl-only-libraries-with-net-standard"
---

[![NetStandardLogo]({{ site.url }}{{ site.baseurl }}/images/d4afd6c8-921f-4985-aa42-adf70bb64e16.png "NetStandardLogo")]({{ site.url }}{{ site.baseurl }}/images/517038e9-020d-4f57-81f1-6bad34f22a2f.png)

The wait will soon be over and we will all be able to use .Net Standards for sharing all our code across .Net run-times. But for the current time we are still in a transition state, not all libraries support .Net Standard out of the box. One option would to rely on good old Portable Class Libraries (PCLs) for now. But the PCL comes with many restrictions compared to a .Net Standard library. Plus .Net Standard is where all the action is (and will be) happening. So it seems like a no brainer to choose .Net Standard as our library container to share code.

But what when we are using a library that does not support .Net Standard out of the box? For instance let’s say we want to use [Reactive UI](https://reactiveui.net/ "Reactive UI project website") as our MVVM framework for a Xamarin.iOS and Xamarin.Android app. When trying to add Reactive UI version 7.4.0 (latest stable version as of writing) via [NuGet](https://www.nuget.org/packages/reactiveui "NuGet site of Reactive UI") to our .Net Standard Project. We get the following error message:

[![Error message that .Net Standard library is not among the supported .Net Frameworks of this NuGet package]({{ site.url }}{{ site.baseurl }}/images/84fee2c3-34b8-4b8b-bebb-fda21fc32765.png "Error message that .Net Standard library is not among the supported .Net Frameworks of this NuGet package")]({{ site.url }}{{ site.baseurl }}/images/39ec2279-c988-42b6-8d1e-eb321c6054e7.png)

The solution to our problem is to define the required targets in our .Net Standard Project. The targets which the Reactive UI NuGet package defines. We can edit a .Net Standard library by right clicking on it in Visual Studio and selecting *Edit Project name*:

[![Right-Click menu showing the edit option of a .Net Standard Project]({{ site.url }}{{ site.baseurl }}/images/152cdff3-cc3b-4920-8dbd-6671b5ad0105.png "Right-Click menu showing the edit option of a .Net Standard Project")]({{ site.url }}{{ site.baseurl }}/images/2e10df31-cfd0-43ce-8ab4-fa2da172724a.png)

Now we can define the targets by adding the <font face="Consolas">PackageTargetFallback</font> line to our <font face="Consolas">DotNetStandard.Core.csproj</font> file:

<script src="https://gist.github.com/mallibone/c2cff1ff7eb3ac7cb61f6aa2b47137ea.js"></script>

Providing the targets provides NuGet with the information needed to install the package. Rerunning the NuGet installation once more leads to a success ![Smile]({{ site.url }}{{ site.baseurl }}/images/6a8e2d45-f67b-44d3-bcce-5f44780d14f3.png)

[![Successfull installation of Reactive UI after changing the project]({{ site.url }}{{ site.baseurl }}/images/a014a387-e099-4b3c-bc1b-1cd4284bfab9.png "Successfull installation of Reactive UI after changing the project")]({{ site.url }}{{ site.baseurl }}/images/01e9fc9d-4e50-4903-8741-32cd3fdcc5df.png)

# Conclusion

.Net Standard is the future of writing .Net Libraries which runs on multiple .Net runtime Environments.

Happy coding and don’t let those not-yet ported libraries stop you from achieving your goals!
