---
layout: single
title: "Migrating your Xamarin Projects to use NuGet references i.e. the full odyssey of migrating to .Net Standard"
title: Migrating your Xamarin Projects to use NuGet references i.e. the full odyssey of migrating to .Net Standard
date: 2018-08-07
tags: ["Xamarin", "DotNetStandard"]
slug: "migrating-your-xamarin-projects-to-use-nuget-references-ie-the-full-odyssey-of-migrating-to-net-standard"
---

[![Image showing a laptop with graphs on it -looking fancy that's all...]({{ site.url }}{{ site.baseurl }}/images/7ef2d663-6185-4287-9845-78482fb270fa.png "Image showing a laptop with graphs on it -looking fancy that's all...")]({{ site.url }}{{ site.baseurl }}/images/eeff83de-30aa-4877-84ae-1f3a7f954ad1.png)

Did you know that with Visual Studio 2017 there was an update in the target project files of your Xamarin Projects? They no longer contain a `packages.config` file but contain the NuGet references directly in the `csproj`. Using NuGet references instead of the `packages.config` file has numerous benefits ranging from performance improvements to only showing your top-level dependencies (no longer will you have a scajilion package references ).

They also fix a pesky bug I have experienced since partially migrating to .Net Standard. Migrating to .Net Standard is not something new in the Xamarin World. There are many blog posts out there which will guide you through how to migrate your Portable Class Libraries (PCLs) to .Net Standard. My personal favorite is the approach I first read on [James Montemagnos blog](https://montemagno.com/how-to-convert-a-pcl-library-to-net-standard-and-keep-git-history/), which is pretty straightforward and will allow you to keep your version history.

But depending on when you have created your project you will start running into compile errors after the migration telling you that a dll from a NuGet which you have only referenced in the .Net Standard project(s) can not be found in the output folder. As is the case in our sample app which I have created for this blog post:

[![Showing compile error that ReactiveUI dll was not found in app output folder]({{ site.url }}{{ site.baseurl }}/images/b90eeb4d-96c6-4ab4-9834-99bef38e8a08.png "Showing compile error that ReactiveUI dll was not found in app output folder")]({{ site.url }}{{ site.baseurl }}/images/210484fd-42ab-430b-8bfc-42f95e3d1bee.png)

In the sample project, we are referencing the [ReactiveUI](https://reactiveui.net/) NuGet only in our .Net Standard code and therefore do not have a direct NuGet reference in our platform projects. When looking closely at the platform projects we can see that the Android and iOS project are using a `packages.config` file to reference their NuGet packages.


> One solution you will find on the internet is to add these NuGet packages to your target projects. While this works it has got that yucky feel to it. Furthermore, this can lead to quite a bit of bloat since some projects come with sub-dependencies which will all show up in your NuGet package manager. But there is an easier way which is also less nauseous


Migrating to the newer Package Reference style can be done by simply right clicking on your `packages.config` and selecting `Migrate packages.config to PackageReference...`:

[![Visual Studio dialog showing Migrate To PackageReference]({{ site.url }}{{ site.baseurl }}/images/7b19ac8d-5f71-45c1-8149-c248378c6464.png "Visual Studio dialog showing Migrate To PackageReference")]({{ site.url }}{{ site.baseurl }}/images/f15c6b16-5dbc-4008-9315-a4bad95fb335.png)

This action will change the `csproj` from this:


    <?xml version="1.0" encoding="utf-8"?>
    <Project ToolsVersion="4.0" DefaultTargets="Build" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
      <Import Project="..\..\packages\Xamarin.Forms.3.1.0.637273\build\netstandard2.0\Xamarin.Forms.props" Condition="Exists('..\..\packages\Xamarin.Forms.3.1.0.637273\build\netstandard2.0\Xamarin.Forms.props')" />
      <!-- File includes and that stuff -->
      <ItemGroup>
        <Reference Include="System" />
        <Reference Include="System.Xml" />
        <Reference Include="System.Core" />
        <Reference Include="Xamarin.Forms.Core, Version=2.0.0.0, Culture=neutral, processorArchitecture=MSIL">
          <HintPath>..\..\packages\Xamarin.Forms.3.1.0.637273\lib\Xamarin.iOS10\Xamarin.Forms.Core.dll</HintPath>
        </Reference>
        <Reference Include="Xamarin.Forms.Platform, Version=1.0.0.0, Culture=neutral, processorArchitecture=MSIL">
          <HintPath>..\..\packages\Xamarin.Forms.3.1.0.637273\lib\Xamarin.iOS10\Xamarin.Forms.Platform.dll</HintPath>
        </Reference>
        <Reference Include="Xamarin.Forms.Platform.iOS, Version=2.0.0.0, Culture=neutral, processorArchitecture=MSIL">
          <HintPath>..\..\packages\Xamarin.Forms.3.1.0.637273\lib\Xamarin.iOS10\Xamarin.Forms.Platform.iOS.dll</HintPath>
        </Reference>
        <Reference Include="Xamarin.Forms.Xaml, Version=2.0.0.0, Culture=neutral, processorArchitecture=MSIL">
          <HintPath>..\..\packages\Xamarin.Forms.3.1.0.637273\lib\Xamarin.iOS10\Xamarin.Forms.Xaml.dll</HintPath>
        </Reference>
        <Reference Include="Xamarin.iOS" />
      </ItemGroup>
      <ItemGroup>
        <ProjectReference Include="..\HelloReactiveUI\HelloNetStandard.csproj">
          <Project>{984433BA-6DB2-4606-8AB1-E5E070C60D44}</Project>
          <Name>HelloNetStandard</Name>
        </ProjectReference>
      </ItemGroup>
      <Import Project="$(MSBuildExtensionsPath)\Xamarin\iOS\Xamarin.iOS.CSharp.targets" />
      <Target Name="EnsureNuGetPackageBuildImports" BeforeTargets="PrepareForBuild">
        <PropertyGroup>
          <ErrorText>This project references NuGet package(s) that are missing on this computer. Use NuGet Package Restore to download them.  For more information, see http://go.microsoft.com/fwlink/?LinkID=322105. The missing file is {0}.</ErrorText>
        </PropertyGroup>
        <Error Condition="!Exists('..\..\packages\Xamarin.Forms.3.1.0.637273\build\netstandard2.0\Xamarin.Forms.props')" Text="$([System.String]::Format('$(ErrorText)', '..\..\packages\Xamarin.Forms.3.1.0.637273\build\netstandard2.0\Xamarin.Forms.props'))" />
        <Error Condition="!Exists('..\..\packages\Xamarin.Forms.3.1.0.637273\build\netstandard2.0\Xamarin.Forms.targets')" Text="$([System.String]::Format('$(ErrorText)', '..\..\packages\Xamarin.Forms.3.1.0.637273\build\netstandard2.0\Xamarin.Forms.targets'))" />
      </Target>
      <Import Project="..\..\packages\Xamarin.Forms.3.1.0.637273\build\netstandard2.0\Xamarin.Forms.targets" Condition="Exists('..\..\packages\Xamarin.Forms.3.1.0.637273\build\netstandard2.0\Xamarin.Forms.targets')" />
    </Project>


To this:


    <?xml version="1.0" encoding="utf-8"?>
    <Project ToolsVersion="4.0" DefaultTargets="Build" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
      <!-- File includes and that stuff -->
      <ItemGroup>
        <Reference Include="System" />
        <Reference Include="System.Xml" />
        <Reference Include="System.Core" />
        <Reference Include="Xamarin.iOS" />
      </ItemGroup>
      <ItemGroup>
        <ProjectReference Include="..\HelloReactiveUI\HelloNetStandard.csproj">
          <Project>{984433BA-6DB2-4606-8AB1-E5E070C60D44}</Project>
          <Name>HelloNetStandard</Name>
        </ProjectReference>
      </ItemGroup>
      <ItemGroup>
        <PackageReference Include="Xamarin.Forms">
          <Version>3.1.0.637273</Version>
        </PackageReference>
      </ItemGroup>
      <Import Project="$(MSBuildExtensionsPath)\Xamarin\iOS\Xamarin.iOS.CSharp.targets" />
    </Project>


Which only references the top level package. Further the package.config file will be removed. Since this method is still in development you might run into some bumps. So be sure to check out the [limitations](https://docs.microsoft.com/en-us/nuget/reference/migrate-packages-config-to-package-reference#package-compatibility-issues) which might apply to the packages you are currently using. If that is the case make sure to [let the team know](https://github.com/NuGet/Home/issues/). But once you have migrated, you will be able to use your project without the compile issue and all the benefits from using [Nuget package references](https://docs.microsoft.com/en-us/nuget/reference/migrate-packages-config-to-package-reference#benefits-of-using-packagereference).

# Conclusion

In this post, you saw how to migrate your Xamarin Projects to use package references. Using package references is intended to be the new adding NuGet package references directly to your `csproj`. With this migration, an error introduced when migrating your PCL projects to .Net Standard will also be solved. So be sure to check out this option when updating your Xamarin apps that have been out in the wild for a while.

**Note:** that all new projects created with Visual Studio 2017 will automatically use the package reference approach.

To see the sample app in full you can check them out on GitHub [packages.config](https://github.com/mallibone/XamarinPackageReferenceMigration/tree/feature/PlatformProjectsUsingPackageConfig) and [package reference](https://github.com/mallibone/XamarinPackageReferenceMigration/tree/feature/MigratePlatformsToPackageReference).

Thank you to [Pierce Boggan](https://twitter.com/pierceboggan) who pointed me in the right direction to get this problem solved.<u></u><sub></sub><sup></sup><strike></strike>
