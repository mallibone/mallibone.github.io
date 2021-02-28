---
layout: single
title: "Hello Universal Windows Platform (UWP)/Windows 10"
title: Hello Universal Windows Platform (UWP)/Windows 10
date: 2015-09-15
tags: ["Windows", "MVVM Light", "Mobile", "F#", "Windows Phone", "UWP"]
slug: "hello-universal-windows-platform-uwpwindows-10"
---

For my recent blog post about how to consume a web service in the Portable Class Library (PCL) I decided to write the client for Windows 10 which allows to create apps for every platform that runs Windows 10. In this post I will focus on the platforms Desktop, Tablet and Mobile showing how to create a basic application based on a MVVM architecture. To create a Windows 10 app you will need a computer running Windows 10 and Visual Studio 2015.
 
# Creating a basic app
 
First lets create a list of people that we can navigate to in the detail view. Here fore we will create two pages. On the first page ***MainPage.xaml*** we will simply show the list:


    <Page
        x:Class="HttpClientSample.MainPage"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="using:HttpClientSample"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        DataContext="{Binding Main, Source={StaticResource Locator}}"
        mc:Ignorable="d">
    
        <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
            <GridView ItemsSource="{Binding People}" SelectedItem="{Binding SelectedPerson, Mode=TwoWay}"></GridView>
    
            <AppBar VerticalAlignment="Bottom" IsOpen="True" >
                <AppBarButton Icon="Add" Label="Add Person" Command="{Binding AddNewPerson}" />
            </AppBar>
        </Grid>
    </Page>


The user can select an element of the list on which we will navigate to the detail of the person allowing us to edit the person. This is done in the ***MainViewModel.cs*** which is based on the [MVVM Light](http://www.mvvmlight.net/) Framework for implementation:


    public class MainViewModel:ViewModelBase
    {
        private readonly IPersonService _personService;
        private ObservableCollection<Person> _people;
        private Person _selectedPerson;
    
        public MainViewModel(IPersonService personService)
        {
            if (personService == null) throw new ArgumentNullException(nameof(personService));
            _personService = personService;
            _people = new ObservableCollection<Person>();
            ShowPerson = person => { };
            AddNewPerson = new RelayCommand(() => ShowPerson(-1));
        }
    
        public ObservableCollection<Person> People => _people;
        public Action<int> ShowPerson { get; set; }
    
        public Person SelectedPerson
        {
            get { return _selectedPerson; }
            set
            {
                if (_selectedPerson == value) return;
                _selectedPerson = value;
                RaisePropertyChanged(nameof(SelectedPerson));
                if (_selectedPerson != null) ShowPerson(_people.IndexOf(_selectedPerson));
            }
        }
    
        public ICommand AddNewPerson { get; private set; }
    
        public async Task Init()
        {
            var people = await _personService.GetPeople();
    
            _people.Clear();
    
            foreach (var person in people)
            {
                _people.Add(person);
            }
        }
    }


The View Model also prepares the List of people that is displayed on the main page. The ***PersonDetailPage.xaml*** will allow to edit the first and surname of the person:


    <Page
        x:Class="HttpClientSample.PersonDetailPage"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="using:HttpClientSample"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        DataContext="{Binding Person, Source={StaticResource Locator}}"
        Loaded="PersonDetail_OnLoaded"
        mc:Ignorable="d">
    
        <StackPanel Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
            <TextBox Header="Firstname" Text="{Binding Firstname, Mode=TwoWay}"/>
            <TextBox Header="Lastname" Text="{Binding Lastname, Mode=TwoWay}"/>
            <Button Content="Save" Command="{Binding StoreCommand, Mode=TwoWay}" IsEnabled="{Binding HasPendingChanges}" HorizontalAlignment="Center" Margin="0,10"/>
        </StackPanel>
    </Page>


When selecting save the person will be updated, all the logic is implemented in the ***PersonDetailViewModel.cs***:


    public class PersonDetailViewModel:ViewModelBase
    {
        private readonly IPersonService _personService;
        private int _id;
        private string _firstname;
        private string _lastname;
        private bool _hasPendingChanges;
    
        public PersonDetailViewModel(IPersonService personService)
        {
            if (personService == null) throw new ArgumentNullException(nameof(personService));
            _personService = personService;
    
            StoreCommand = new RelayCommand(StorePerson, () => true);
            HasPendingChanges = false;
        }
    
        public bool HasPendingChanges
        {
            get { return _hasPendingChanges; }
            set
            {
                if (value == _hasPendingChanges) return;
                _hasPendingChanges = value;
                RaisePropertyChanged(nameof(HasPendingChanges));
            }
        }
    
        public string Firstname
        {
            get { return _firstname; }
            set
            {
                if (value == _firstname) return;
                _firstname = value;
                HasPendingChanges = true;
                RaisePropertyChanged(nameof(Firstname));
            }
        }
    
        public string Lastname
        {
            get { return _lastname; }
            set
            {
                if (_lastname == value) return;
                _lastname = value;
                HasPendingChanges = true;
                RaisePropertyChanged(nameof(Lastname));
            }
        }
    
        public ICommand StoreCommand { get; set; }
    
        public async Task Init(int id)
        {
            _id = id;
            var person = id > 0 ? (await _personService.GetPeople()).ToList()[id] : new Person("", "");
            Firstname = person.FirstName;
            Lastname = person.LastName;
            HasPendingChanges = false;
        }
    
        private async void StorePerson()
        {
            HasPendingChanges = false;
    
            var person = new Person(Firstname, Lastname);
    
            if (_id >= 0)
            {
                await _personService.UpdatePerson(_id, person);
            }
            else
            {
                await _personService.CreatePerson(person);
            }
        }
    }


To prevent the user from invoking the storing of a person multiple times the store button is only enabled when there are pending changes and you probably want to improve the error handling in production code as it is non-existent in the ***StorePerson*** method.

# Basic Navigation

From the main page we implement the navigation in ***MainPage.xaml.cs*** which allows the user to navigate to the detail page. We also pass along a navigation parameter which contains the ID or ahem in this basic example the array index of the person we want to see:


    private void ShowPerson(int personId)
    {
        Frame.Navigate(typeof (PersonDetailPage), personId);
        _viewModel.SelectedPerson = null;
    }


The navigation is done over Frame.Navigate, which takes the type of the target and a parameter as an argument. The **SelectedPerson** is then set to null, after invoking the navigation. This is done so that the user may reselect the person the second time around.

Additionally when the user adds a person over the **AppBar** it will invoke the same navigation method but pass in the –1 as parameter indicating that no Index is set.

Further the user has the possibility to navigate back to the main page. Here fore we have to enable the back navigation which we set in the Loaded event handler of the person detail page and add an event handler for the **BackRequest**:


    private void PersonDetail_OnLoaded(object sender, RoutedEventArgs e)
    {
        SystemNavigationManager.GetForCurrentView().AppViewBackButtonVisibility = AppViewBackButtonVisibility.Visible;
        SystemNavigationManager.GetForCurrentView().BackRequested += OnBackRequested;
    }


The event handler is implemented as follows:


    private void OnBackRequested(object sender, BackRequestedEventArgs backRequestedEventArgs)
    {
        if (Frame.CanGoBack)
        {
            Frame.GoBack();
            backRequestedEventArgs.Handled = true;
        }
    }



> While testing the app under Windows 10 mobile I noticed that the navigation was invoked twice when selecting the hardware back button. By setting the Handled property to true in the event arguments the framework is indicated that the navigation has been taken care of.


# Taking it to other form factors

A great thing about creating your apps as a Universal Windows Platform app is that the APIs remain the same across the devices. In regards to the upcoming Windows 10 Mobile release we can even reuse the UI code. So we can run this basic app on a phone without making any changes. The UI will be automatically adopted to the new form factor.

Now of course for more complex apps you will want to use different layouts according to the screen size, but even then you will be creating adaptive layouts (same as with responsive Websites) rather than developing multiple apps to target different platforms. So a lot less overhead and work to have a running app across multiple form factors.

# Conclusion

In this post we saw how to create a basic Windows 10 UWP app including basic navigation features and how to implement the MVVM pattern. If you are coming from a WPF, Silverlight, Windows 8.x or Windows Phone background you have seen that a lot of elements are very familiar.

I’ve been lately playing around with F# which unfortunately is currently not a supported language to write UWP apps. This also includes not being able to share Portable Class Libraries with UWP apps. If you feel the same way as me or want to throw the F# community a bone please let Microsoft know that they enable support for F# UWP apps [here](https://wpdev.uservoice.com/forums/110705-dev-platform/suggestions/9110134-f-support-in-net-native-for-uwp "F# support in .Net native for UWP").

You can find the entire sample code of this blog post on [GitHub](https://github.com/mallibone/HttpClientSample.git).
