---
layout: single
title: "Migrate your Xamarin App from PCL to .Net Standard"
title: Migrate your Xamarin App from PCL to .Net Standard
date: 2017-11-27
tags: ["Xamarin.Forms", "Xamarin.iOS", "Xamarin", "UWP", "DotNetStandard", "Xamarin.Android"]
slug: "migrate-from-pcl-to-net-standard"
---

[![pexels-photo-64774](https://mallibone.com/posts/files/1aca268d-f5c9-44da-80d5-654fbe59e21c.jpg "pexels-photo-64774")](https://mallibone.com/posts/files/af5d98f9-7978-4675-8cd7-3b16ee5a368e.jpg)


> **Update:** Hey there thank you for reading my blog, since I wrote this post I have learnt a lot and have found a better way to migrate your Xamarin Apps so please check out my latest [blogpost](https://mallibone.com/post/migrating-your-xamarin-projects-to-use-nuget-references-ie-the-full-odyssey-of-migrating-to-net-standard) on this matter.


If you haven’t heard or dived into .Net Standard you are in for a treat. In short it provides a way to share code across platforms but in contrast to the PCL it gives you so many more platform specific features. For an in depth overview check out the [official docs](https://docs.microsoft.com/en-us/dotnet/standard/net-standard "Net Standard Documentation").


> **Note:** If you are starting a new Project today with Visual Studio (VS) 2015.4, you will not be able to select .Net Standard by default. But the [new templates for .Net Standard](https://blog.xamarin.com/net-standard-comes-xamarin-forms-project-templates/ "Xamarin bringing .Net Standard Template as Default in VS 2015.5 Blogpost") will be included in Visual Studio 15.5.


## <font style="font-weight:normal">Creating the .Net Standard Class Library</font>

Migrating an existing app to .Net Standard is pretty straight forward. Step one add a .Net Standard Library to replace your PCL project.

[![The VS Add New Project dialog, under Visual C# select Class Library (.Net Standard)](https://mallibone.com/posts/files/e1fb5a61-7d2d-4a75-be14-f58cc48cec7a.png "The VS Add New Project dialog, under Visual C# select Class Library (.Net Standard)")](https://mallibone.com/posts/files/91844268-f632-4170-8928-0dcc9b74a976.png)

## <font style="font-weight:normal">Migrating your source code</font>

Then drag and drop all of your existing files from the PCL project to your .Net Standard library. Note that you don’t want to copy the *packages.config* or any of the files under *properties*.

[![CopyFiles](https://mallibone.com/posts/files/f8ba4038-8054-4f45-874d-1080ba0a67c4.png "CopyFiles")](https://mallibone.com/posts/files/ae173402-ee79-4413-8647-50ceef4d9722.png)


> Now you can delete your PCL project that you have migrated. If you do so from Visual Studio note that the Project is still available in the file system. Which means you still have it if you forgot something, but also means that you will have to delete it later on if you want to remove it from the source control workspace.


Next step is to add all the NuGet packages you have used in the PCL project. If you are having trouble adding some of the packages look out for NuGet packages that are no longer needed due to .Net Standard support such as file system access. In other cases it could be because the NuGet package has not (hopefully yet) migrated to .Net Standard. In that case check out this [post](https://mallibone.com/post/using-pcl-only-libraries-with-net-standard "post to update your csproj file for backwards compatibility") and don’t forget to ask the maintainers of the project when the project will be available for .Net Standard ![Winking smile](https://mallibone.com/posts/files/22ffcfdf-77ed-43a5-9495-78b1e37790bf.png)

## <font style="font-weight:normal">Hooking up the projects</font>

You can now add the reference to the .Net Standard in your Android and iOS project. If you created a new namespace I strongly recommend you refactor them after adding them to your projects. Or else your refactoring tool of choice will only do half the magic and you will still have some work left to do.

If you are using a UWP project please read the section bellow as you will need to make some additional steps to make it work.

## <font style="font-weight:normal">Fixing the csproj for Xamarin Forms</font>

Unfortunately when moving a Xamarin Forms app over to .Net Standard you will get weird compilation errors. The cause of this is that the XAML files are referenced in the csproj file:

[![02_1_RemoveEmbeddedResources](https://mallibone.com/posts/files/5603a768-8298-4cfa-a1d8-5d92de0e2248.png "02_1_RemoveEmbeddedResources")](https://mallibone.com/posts/files/08f2b642-7b59-4d94-a8ec-767270992e2d.png)

Simply remove them as they are not needed and the compile errors should be history.

## <font style="font-weight:normal">When using UWP</font>

If you are using UWP as a target (I.e. using the default Project provided up to VS 2015.4). You will have to remove and add the project anew:


> If you are unsure if you really have to update your UWP project. Check if you have a project.json file in your UWP project. If the answer is yes, I’m afraid you will have to follow the following steps.


1. Remove the UWP project from the solution in Visual Studio
2. Rename the UWP project folder in the file explorer
3. Add a new UWP project in Visual Studio (with the same name as the one just removed)
4. Set the minimal supported Windows 10 version to the Fall creators update  
[!\[VS Dialog Window with Target and Minimum Version set to Fall Creators Update\](https://mallibone.com/posts/files/bb6690d8-a16a-49e6-b76e-ba1f2ac0fb59.png "VS Dialog Window with Target and Minimum Version set to Fall Creators Update")](https://mallibone.com/posts/files/d80eab22-7e8a-499c-855d-9d4a258f7f51.png)
5. Add any NuGet references the just removed project had (you can peek into the project.json file in the renamed location if you are unsure which packages to add)
6. Copy and paste all your UWP files (except the project.json)
7. Add a reference to the Standard Library project


If you know an easier way to upgrade a UWP project, please let me know in the comments bellow ![Smile](https://mallibone.com/posts/files/32f86b30-7eb3-434b-ab17-fa6f37a0de5a.png)

# Conclusion

In this post we went over the steps required to migrate an existing PCL project to .Net Standard. All the steps were done with Visual Studio 15.4.
