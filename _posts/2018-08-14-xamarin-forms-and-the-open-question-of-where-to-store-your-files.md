---
layout: single
title: "Xamarin Forms and the open Question of where to store your files"
title: Xamarin Forms and the open Question of where to store your files
date: 2018-08-14
tags: ["Xamarin", "DotNetStandard", "Xamarin.Forms"]
slug: "xamarin-forms-and-the-open-question-of-where-to-store-your-files"
---

[![Title Image showing a library]({{ site.url }}{{ site.baseurl }}/assets/images/925e8e48-a5ab-4b0a-bf2b-167458053a4d.png "Title Image showing a library")]({{ site.url }}{{ site.baseurl }}/assets/images/da62308c-40f7-49b0-a0dc-e075607b93a6.png)

When it comes to file handling and Xamarin Forms you can find all you need in the official Documentation. However, when it comes to where the data should be stored the documentation leaves some points open. Moreover, might even lead to, dear I say it, your rejection in the App Store...

Also when writing Cross-Platform Code, with .Net Standard, it does depend on the Operating System (OS) that the app is being executed on, where to store your data. So let's dive into the platforms which are primarily supported by Xamarin Forms.

# Xamarin.Essentials

The most comfortable way is using Xamarin.Essentials. Currently still in preview and requires Android 7.1 or higher. However, if that does not make you blink, these are the options:

- Local Storage which is also backed up.
- Cache Storage which is, well for caching files but are more on the non-permanent side.
- Files bundled with the app, i.e. read-only files.


You can access the folder paths as follows:


    var rootDirectory = FileSystem.AppDataDirectory;


Currently Xamarin.Essentials supports Android, iOS and UWP. So if all the platforms your app requires are named, and you do not need any other location. Be sure to check out [Xamarin.Essentials](https://www.nuget.org/packages/Xamarin.Essentials) and check the [documentation](https://docs.microsoft.com/en-us/xamarin/essentials/file-system-helpers?tabs=android) for more details.

# Do it yourself

In most cases the folders offered by Xamarin.Essentials will suffice, but if you require a different folder, i.e. the document folder under iOS, you can always set the path to the location on your own. So let's have a look at how you can achieve this.

## Android

Now when it comes to Android, you can follow the documentation from Microsoft and use the following path for storing your files:


    _rootDirectory = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);


Looking to cache some files, then we can take the path and with `Path.Combine` set the path to the `cache` folder:


    _cacheDirectory = Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments), "..", "cache");


If you want to store files to the SD-Card, i.e. external storage under Android. You have to pass in the path from a platform Project to the .Net Standard project or use multitargeting to achieve this. You can get the path as follows:


    var sdCardPath = Environment.ExternalStorageDirectory.AbsolutePath;


Note that the Environment here is `Android.OS.Environment` and not `System.Environment`. So if Android is this easy how hard can iOS be?

## iOS

When storing files under iOS, there is a bit more documentation to read. The reason being that Apple uses multiple subfolders within the Sandbox container. The ones that are important for file storage are the following:

- **Documents:** In this folder, only user-created files should be stored. No application data, which includes that JSON file of your app, should be stored here. This folder is backed up automatically.
- **Library:** The ideal spot for any application data you do not want the user should have direct access to. This folder is backed up automatically.

    - **Library/Preferences:** A subdirectory which you should not directly access. Better use the [Xamarin.Essentials](https://docs.microsoft.com/en-us/xamarin/essentials/preferences?context=xamarin%2Fios&amp;tabs=android) library for storing any key/value data. This data is backed up automatically.
    - **Library/Caches:** This is an excellent place to store data which can easily be re-created. This data is not backed up.
- **tmp:** Good for temporary files, which you should delete when no longer used.


For a more detailed listing check out the [docs](https://docs.microsoft.com/en-us/xamarin/ios/app-fundamentals/file-system#application-directories). So when storing files, you are under iOS this code will point to the Documents folder:


    _rootDirectory = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);


If you intend to store information into the Library folder you can change the directory as follows:


    _rootDirectory = Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments), "..", "Library");


Equally, you can change your path to one of the other locations described above.

## UWP

UWP apps usually also store their data in a sandbox. Only if in the app's metadata the permission is set and a good reason was given, which Microsofts validates on submission to the store, can the app access other file locations outside of the sandbox. UWP apps also live in a sandbox. The `ApplicationData` offers to store data locally, roaming or in a temporary location. The local folder can be accessed as follows:


    _rootDirectory = Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData);


Similar the other locations can be accessed for example the roaming folder (which is synced automatically across all of your different devices):


    _rootDirectory = Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData), "..", "RoamingState");


The following directories are present in a UWP apps sandbox:

[![UWP Container folders: AC, AppData, LocalCache, LocalState, RoamingState, Settings, SystemAppData, TempState]({{ site.url }}{{ site.baseurl }}/assets/images/0ad0e054-a156-4dcb-828e-f77403e9ee21.png "UWP Container folders: AC, AppData, LocalCache, LocalState, RoamingState, Settings, SystemAppData, TempState")]({{ site.url }}{{ site.baseurl }}/assets/images/0522aa3a-386f-41b5-b7bd-87d384d07e46.png)

# However, what if I am targeting other platforms?

Since Xamarin Forms is no longer limited to Android, iOS and UWP, you might find yourself wanting to write files on another system such as Tizen. The best solution is to check the documentation of the given platform where data should be stored and then run the following code on the platform:


    class Gnabber 
    { 
        // ... 
        private IEnumerable<DirectoryDesc> DirectoryDescriptions() 
        { 
            var specialFolders = Enum.GetValues(typeof(Environment.SpecialFolder)).Cast<Environment.SpecialFolder>(); 
            return specialFolders.Select(s => new DirectoryDesc(s.ToString(), Environment.GetFolderPath(s))).Where(d => !string.IsNullOrEmpty(d.Path)); 
        } 
        // ... 
    } 
     
    class DirectoryDesc 
    { 
        public DirectoryDesc(string key, string path) 
        { 
            Key = key; 
     
            Path = path == null 
                ? "" 
                : string.Join(System.IO.Path.DirectorySeparatorChar.ToString(), path.Split(System.IO.Path.DirectorySeparatorChar).Select(s => s.Length > 18 ? s.Substring(0, 5) + "..." + s.Substring(s.Length - 8, 8) : s)); 
        } 
     
        public string Key { get; set; } 
        public string Path { get; set; } 
    }


The code above lists all used folders from the `System.Environment.SpecialFolder` for the given environment, and also provides you with the absolute path. If none of the special folders is the target location, you desired. Try using `Path.Combine` and a path nearby to get to your desired location.

# Conclusion

Storing files is not tricky but put some thought into where to store your applications data can go a long way. Xamarin.Essentials may provide all the functionality you need, but if not you usually use the `System.Environment.SpecialFolders` and the `System.Environment.GetFolderPath` to get access to different folders offered by the platform you are on.
