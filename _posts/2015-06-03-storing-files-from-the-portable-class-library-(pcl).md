---
layout: single
title: "Storing files from the Portable Class Library (PCL)"
title: Storing files from the Portable Class Library (PCL)
date: 2015-06-03
tags: ["Mobile", "Xamarin.Forms", "Windows Phone", "iOS", "Android", "Windows"]
slug: "storing-files-from-the-portable-class-library-(pcl)"
---

[![StorageImage](http://www.mallibone.com/posts/files/836a85e3-7557-4f93-9e52-bee97e9c3d1a.jpg "StorageImage")](http://www.mallibone.com/posts/files/ac5553fc-54b9-4fcd-aecc-cc800411b865.jpg)
 
A lot of applications have to persist application data or state. When developing mobile applications with C# such as for Windows or with Xamarin for iOS and Android it is a common approach to share your business logic and backend implementations in the Portable Class Library (PCL). In this blog post we’ll look at how we can also implement storage within the PCL and therefore will only need to implement the storage services once. We will be focusing on the following topics:
 
1. Setting up the project
2. Storing and loading objects to files
3. Storing/using binary data such as images

 
So lets start off by setting up the project. I’ll be demonstrating the sample with a [Xamarin.Forms](http://xamarin.com/forms) project as it allows me to quickly develop, deploy and test an app on all three major mobile platforms. But as the code is in the PCL you can use the exact same approach to implement the system in Windows Store, WPF etc. applications.
 
# Setting up the project
 
Lets create an app that lists Companies by name, URL and displays the company logo. To do this we will create a view that will display the content as follows:


    <?xml version="1.0" encoding="utf-8" ?><ContentPage xmlns="http://xamarin.com/schemas/2014/forms"             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"             x:Class="XamarinFormsOfflineStorage.Views.MainPage">  <Grid>  <Grid.RowDefinitions>    <RowDefinition Height="Auto"/>    <RowDefinition Height="*"/>  </Grid.RowDefinitions>  <Button Text="Refresh" Command="{Binding UpdateCompaniesCommand}"></Button>    <ListView ItemsSource="{Binding Companies}" Grid.Row="1">      <ListView.ItemTemplate>        <DataTemplate>          <ImageCell Text="{Binding Name}" ImageSource="{Binding ImageUri}" Detail="{Binding ImageDescription}"></ImageCell>        </DataTemplate>      </ListView.ItemTemplate>    </ListView>    <ActivityIndicator Grid.Row="1" IsRunning="{Binding IsLoadingData}"></ActivityIndicator>  </Grid></ContentPage>


The data is provided by a View Model that is based upon the <font style="background-color: #ffffff"><a href="http://www.mvvmlight.net/" target="_blank">MVVM Light</a></font> framework by [<font style="background-color: #ffffff">Laurent Bugnion</font>](http://www.galasoft.ch/). I wrote a post a while ago on how to get started with MVVM Light and Xamarin.Forms.


    public class MainViewModel:ViewModelBase{    private readonly ICompanyService _companyService;    private bool _isLoadingData;    public MainViewModel(ICompanyService companyService)    {        if (companyService == null) throw new ArgumentNullException("companyService");        _companyService = companyService;        Companies = new ObservableCollection<Company>();        IsLoadingData = false;        UpdateCompaniesCommand = new RelayCommand(UpdateCompanies, () => !IsLoadingData);    }    public bool IsLoadingData    {        get { return _isLoadingData; }        set        {            if (value == _isLoadingData) return;            _isLoadingData = value;            RaisePropertyChanged(() => IsLoadingData);        }    }    public ObservableCollection<Company> Companies { get; set; }    public ICommand UpdateCompaniesCommand { get; set; }    public async Task Init()    {        await UpdateCompaniesList();    }    private async void UpdateCompanies()    {        IsLoadingData = true;        await _companyService.UpdateCompanies();        await UpdateCompaniesList();        IsLoadingData = false;    }    private async Task UpdateCompaniesList()    {        var companies = await _companyService.GetCompanies();        Companies.Clear();        foreach (var company in companies)        {            Companies.Add(company);        }    }}


Which calls a **CompanyService** that calls the backend:


    // ...public async Task UpdateCompanies(){    // URL should point to where your service is running    const string uri = "http://offlinestorageserver.azurewebsites.net/api/values";    var httpResult = await _httpClient.GetAsync(uri);    var jsonCompanies = await httpResult.Content.ReadAsStringAsync();    var companies = JsonConvert.DeserializeObject<ICollection<Models.Company>>(jsonCompanies);    _companies = companies;}// ...


And provides a list of companies:


    public IEnumerable<Models.Company> GetCompanies(){    return _companies;}


Now lets look on how we can store this list to the “disk” and display the information even if the app is started when there is no connection to the backend.

## Enabling storage access in the PCL

Thanks to [Daniel Plaisted](https://twitter.com/dsplaisted)<font style="background-color: #ffff00"></font> accessing the file storage from within the PCL of your applications is actually really easy. All that is left to do for us is installing the NuGet package **[PCL Storage](https://github.com/dsplaisted/PCLStorage)**. Ensure that you install it not only for the PCL but only for the platforms you are targeting with your app:

[![PclStorageNugetInstall](http://www.mallibone.com/posts/files/23e0cc04-0895-43ce-b625-5eef5d02a918.png "PclStorageNugetInstall")](http://www.mallibone.com/posts/files/235ea2c2-4239-4512-9ef0-d7651e86a319.png)

Before we store lets just still quickly have a look at where we actually want to store the files. The PCL storage provides a start location which can be accessed by calling the **FileSystem.Current.LocalStorage** property. This will return an **IFolder** object which can be allows navigating to further folders by invoking the **GetFolder** method, alternatively the **CreateFolder** method can also be used with the collision option **OpenIfExists**, I generally use the second approach as it allows to write less code. So if we want to store our data in a folder within the root location of the local storage it would be done as follows:


    private static async Task<IFolder> NavigateToFolder(string targetFolder){    IFolder rootFolder = FileSystem.Current.LocalStorage;    IFolder folder = await rootFolder.CreateFolderAsync(targetFolder,        CreationCollisionOption.OpenIfExists);    return folder;}


As you can see the PCL Storage embraces the Async/Await pattern nicely and therefore will not block your app i.e. UI when working against the file system. Now lets get down to storing some data.

# Persist an object

Now lets first store the list of objects we receive from our service into a file. As I’m already using JSON.Net for deserializing the data that comes from our service I’ll be using this library to serialize our objects to a JSON string and then store the now text data to a file in the **SerializeCompanies** method:


    private static async Task SerializeCompanies(IFolder folder, ICollection<Models.Company> companies){    IFile file = await folder.CreateFileAsync(CompaniesFileName, CreationCollisionOption.ReplaceExisting);    var companiesString = JsonConvert.SerializeObject(companies);    await file.WriteAllTextAsync(companiesString);}


As you can see files are created similarly to folders but differ in the collision option. It generally is a generally easier to handle merging of data in the C# memory world itself (given that the list is not to large for this) and then simply overwriting the preexisting cache.

## Loading the data object

We can now update the **GetCompanies** method to return the list when we call it.


    public async Task<IEnumerable<Models.Company>> GetCompanies(){    return _companies ?? (_companies = await ReadCompaniesFromFile());}


Once the data has been persisted by the website we do no longer require an internet connection to present the user with the information. And we can simply read them from a file as in **ReadCompaniesFromFile**:


    private async Task<IEnumerable<Models.Company>>  ReadCompaniesFromFile(){    var folder = await NavigateToFolder(CompaniesFolder);    if ((await folder.CheckExistsAsync(CompaniesFileName)) == ExistenceCheckResult.NotFound)    {        return new List<Models.Company>();    }    IFile file = await folder.GetFileAsync(CompaniesFileName);    var jsonCompanies = await file.ReadAllTextAsync();    if (string.IsNullOrEmpty(jsonCompanies)) return new List<Models.Company>();    var companies = JsonConvert.DeserializeObject<IEnumerable<Models.Company>>(jsonCompanies);    return companies;}


Now the object data is persisted but we haven’t yet stored the images which is not text data but binary data. So lets see how we can store images to the file system.

# Store binary data

To store the images we will have to get the images and store the binary data from the web to a file. This is done using the file stream of the opened file in the **StoreImagesLocallyAndUpdatePath **method:


    private async Task StoreImagesLocallyAndUpdatePath(IFolder folder, IEnumerable<Models.Company> companies){    foreach (var company in companies)    {        var file = await folder.CreateFileAsync(company.Name + ".jpg", CreationCollisionOption.ReplaceExisting);        using (var fileHandler = await file.OpenAsync(FileAccess.ReadAndWrite))        {            var httpResponse = await _httpClient.GetAsync(company.ImageUri);            byte[] imageBuffer = await httpResponse.Content.ReadAsByteArrayAsync();            await fileHandler.WriteAsync(imageBuffer, 0, imageBuffer.Length);            company.ImageUri = file.Path;        }    }}


Streams should always be disposed or else memory leaks will occur so the stream is wrapped in a using block. For storing the binary data we download the image via the URL provided and read the content into a byte array which allows us to easily pass it into the stream so it can get stored. Finally we replace the **ImageUri** data in the POCO (Plain Old CLR Object) before it gets serialized.


> I recommend that you use the file endings according to the data you are storing. Certain containers rely on the information so as long as the good practice is kept up we are not running the risk of having to debug a strange error of information not showing up on the screen.


## Loading images

Now we could load the images back into memory but as images are somewhat a special case the local path / **ImageUri** alone is enough to be set in the image control. So we actually do not have to change any code in the UI or add any additional handling as the image control will simply now load the image from the local storage.

# Conclusion

In this post we saw how we can setup our projects to store files directly out of the PCL. We stored a data object i.e. text data to a file and proceeded to store an image/binary data. I think a big thank you is due to Daniel Plaisted as his work really streamlines persisting data with from the PCL.

You can find the entire code on [GitHub](https://github.com/mallibone/PCLOfflineStorage.git).

## References

Title Image by [Dr. Hannes Grobe](http://commons.wikimedia.org/wiki/User:Hgrobe) under [Creative Commons V3](http://creativecommons.org/licenses/by/3.0/legalcode)
