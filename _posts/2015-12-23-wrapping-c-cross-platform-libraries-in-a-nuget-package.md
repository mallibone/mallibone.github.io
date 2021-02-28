---
layout: single
title: "Wrapping C# cross platform libraries in a NuGet package"
title: Wrapping C# cross platform libraries in a NuGet package
date: 2015-12-23
tags: ["UWP", "iOS", "Xamarin", "Xamarin.Forms", "Android", "Windows Phone"]
slug: "wrapping-c-cross-platform-libraries-in-a-nuget-package"
---

[![nuget]({{ site.url }}{{ site.baseurl }}/images/fc2b63ed-0934-4d7d-aae0-be5fba0a2475.png "nuget logo")]({{ site.url }}{{ site.baseurl }}/images/a5ef11ae-a5be-41d9-b40d-492bf5417ae4.png)
 
See how you can easily create NuGet packages to publish cross platform C# libraries that allow to integrate platform specific code in your Portable Class Library (PCL) based Projects. In a [former post](https://mallibone.com/post/implement-corss-platform-apis-with-c "Former Post") you can read about how to create a cross platform library and see how the library has to be consumed by the e.g. app consuming the library. This post builds up on where that last post left off and shows how we can use the library providing the OS version number in a NuGet package. So the package we want to create has the following structure:
 

 
- <font face="Consolas">OSVersion.Core</font>
- <font face="Consolas">OSVersion.UWP</font>
- <font face="Consolas">OSVersion.Droid</font>
- <font face="Consolas">OSVersion.iOS</font>

 

 
Our goal is to generate a single NuGet package which can be added to any of the above named platforms.
 

> Note: Though this post focuses on the platforms Universal Windows Platform (UWP), Android and iOS the approach can be easily extended to support additional platforms such as .Net, Silverlight et al.

 
# Creating the package definition
 
First of all we will need to install the NuGet command line tool. Make sure that you [add it to your PATH](http://www.howtogeek.com/118594/how-to-edit-your-system-path-for-easy-command-line-access/ "instruction how add an applicaiton to your path"). After it is added to your path you can execute the NuGet commands with PowerShell. To create a NuGet package you need a <font face="Consolas">nuspec</font> file which you can generate by executing the command):


    nuget spec


The location of the <font face="Consolas">nuspec</font> file does not really matter – I tend to have it in my root directory i.e. where the Solution file is. If you open the file in your [lightweight editor of choice](https://code.visualstudio.com/ "Link to Visual Studio Code Editor page"), you can edit the package specification to your gusto or for our sample somewhat like this:


    <?xml version="1.0"?>
    <package >
      <metadata>
        <id>OSVersion</id>
        <version>1.0.0.3</version>
        <title>OS Version</title>
        <authors>Mark Allibone</authors>
        <owners>Mark Allibone</owners>
        <!--<licenseUrl>http://LICENSE_URL_HERE_OR_DELETE_THIS_LINE</licenseUrl>
        <projectUrl>http://PROJECT_URL_HERE_OR_DELETE_THIS_LINE</projectUrl>
        <iconUrl>http://ICON_URL_HERE_OR_DELETE_THIS_LINE</iconUrl>-->
        <requireLicenseAcceptance>false</requireLicenseAcceptance>
        <description>Get the OS version for your UWP, Android or iOS app.</description>
        <releaseNotes>This is the initial release.</releaseNotes>
        <copyright>Copyright 2015</copyright>
        <tags>OSVersion</tags>
      </metadata>
      
      <files>
        <file src="OSVersionAPI\bin\Release\OsVersionAPI.Core.dll" target="lib\portable-net45+wp8+wpa81+netcore45+monoandroid1+xamarin.ios10+UAP10\OsVersionAPI.Core.dll" />
        <file src="OSVersionAPI\bin\Release\OsVersionAPI.Core.pdb" target="lib\portable-net45+wp8+wpa81+netcore45+monoandroid1+xamarin.ios10+UAP10\OsVersionAPI.Core.pdb" />
        <file src="OSVersionAPI\bin\Release\OsVersionAPI.Core.xml" target="lib\portable-net45+wp8+wpa81+netcore45+monoandroid1+xamarin.ios10+UAP10\OsVersionAPI.Core.xml" />
        
        <file src="OSVersionAPI.UWP\bin\Release\OsVersionAPI.Core.dll" target="lib\UAP10\OsVersionAPI.Core.dll" />
        <file src="OSVersionAPI.UWP\bin\Release\OsVersionAPI.Core.pdb" target="lib\UAP10\OsVersionAPI.Core.pdb" />
        <file src="OSVersionAPI.UWP\bin\Release\OsVersionAPI.Core.xml" target="lib\UAP10\OsVersionAPI.Core.xml" />
        
        <file src="OSVersionAPI.Droid\bin\Release\OsVersionAPI.Core.dll" target="lib\monoandroid1\OsVersionAPI.Core.dll" />
        <file src="OSVersionAPI.Droid\bin\Release\OsVersionAPI.Core.pdb" target="lib\monoandroid1\OsVersionAPI.Core.pdb" />
        <file src="OSVersionAPI.Droid\bin\Release\OsVersionAPI.Core.xml" target="lib\monoandroid1\OsVersionAPI.Core.xml" />
        
        <file src="OSVersionAPI.iOS\bin\iPhone\Release\OsVersionAPI.Core.dll" target="lib\xamarin.ios10\OsVersionAPI.Core.dll" />
        <file src="OSVersionAPI.iOS\bin\iPhone\Release\OsVersionAPI.Core.pdb" target="lib\xamarin.ios10\OsVersionAPI.Core.pdb" />
        <file src="OSVersionAPI.iOS\bin\iPhone\Release\OsVersionAPI.Core.xml" target="lib\xamarin.ios10\OsVersionAPI.Core.xml" />
      </files>
    </package>


Beware of the target folder structure, these names may seem a bit strange (I’m looking at you Xamarin.Android…) but these actually have to align with a pattern or else you will run into problems when installing the NuGet package. You can find the target folder names on the [NuGet website](https://docs.nuget.org/Create/TargetFrameworks "Link to NuGet listing possible target folders").

## Integrating some Metadata in the NuGet package


> You are not required to add the pdb (used for Debugging) or the XML file (Code Documentation for Intellisense) but it is considered best practice and makes the life of the developer using the library easier so I would suggest you follow along with this.


To enable the XML metadata file, right click on a project under Properties, Build you can set the checkmark for *XML documentation file*. Now you should have an XML file in your build output.

Per default the pdb file should be generated during the build, except for iOS (it’s always you iOS isn’t it…) - anyhow, right click on your project then under *Properties, Build, Advanced…* set *Debug Info* to ***pdb-only***.


> Beware that these settings are per build Configuration so make sure you set it for ***Release*** builds as you usually ship the ***Release*** build of a library.


# Packing it up

Now all that is left to do is creating the package which we can do with the following command:


    nuget pack


After the package is created the next thing to do is make it available for consumption. This means either pushing the package to NuGet or creating your own NuGet repository. Due to the fact, that this here is only a demo I will go with the second option.

# Integrating the package in an app

Now we can simply add the OS Version library over the package manager. Notice that we get the same result as in the former blog post but this time the developer using the library will not be required to add the library multiple times to the project i.e. this potential error source no longer exists.

So when we now run the app under UWP we get the following output.

[![Win10VersionNumber]({{ site.url }}{{ site.baseurl }}/images/e16f8be0-b087-4d4b-b72d-9614ac4a6628.png "Windows 10 App showing Version Number")]({{ site.url }}{{ site.baseurl }}/images/cc7adb78-38cf-4ac6-8e18-f3f362cad4da.png)

Note if we would only add the NuGet package to the <font face="Consolas">NativePcl.Core</font> project the output would simply show the stub implementation.

[![StubVersionNumber]({{ site.url }}{{ site.baseurl }}/images/a40c6aed-89b4-4b41-8340-a4688f2592e7.png "App running showing Stub Version Number")]({{ site.url }}{{ site.baseurl }}/images/5dd4e2fe-ca8d-427c-8841-8d922b59644f.png)

So all the principals that were described in the [last post](https://mallibone.com/post/implement-corss-platform-apis-with-c "Post showing how to create cross plattform APIs") still apply. But adding the library has been greatly simplified.

# Conclusion

In this blog post you saw how a cross platform NuGet package can be created which lends itself very well to create cross platform libraries that implement native features which have to implemented differently for each targeted platform. There are many great libraries already out in the wild that use this approach e.g. MVVM Light, SQLite, PCL Storage and many many more.

You can find the sample project on [GitHub](https://github.com/mallibone/NativePclSample "Link to Sample Code on GitHub").
