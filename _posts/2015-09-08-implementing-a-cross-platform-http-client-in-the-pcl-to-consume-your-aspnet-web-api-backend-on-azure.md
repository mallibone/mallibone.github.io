---
layout: single
title: "Implementing a cross platform HTTP Client in the PCL to consume your ASP.Net Web API backend on Azure"
title: Implementing a cross platform HTTP Client in the PCL to consume your ASP.Net Web API backend on Azure
date: 2015-09-08
tags: ["Azure", "Mobile", "Xamarin", "Windows", "Windows Phone", "MVVM Light"]
slug: "implementing-a-cross-platform-http-client-in-the-pcl-to-consume-your-aspnet-web-api-backend-on-azure"
---

When working with a mobile app you will often work with a backend, which if you are working with C# might very probably be a Web API backend. In this blog we will look at how we can consume and write to a Web API Controller that we can host on Azure. Why Azure? Because it is the easiest way of hosting a web service straight out of Visual Studio and if you happen to have a MSDN subscription you can do this all for free. For the client we will be using a Windows 10 Universal App which allows us to write a client for Desktop, Tablet and Phone.
 
The app we will be building will add the capability to read a list of people from a service and add a person to the existing. So we will see how we can read and write to a HTTP service.
 
# The client
 
As a client we will be targeting Windows Phone (soon to be Windows Mobile again), Android and iOS. Our implementation of integrating with the backend we will perform in the Portable Class Library (PCL) which provides us the unique advantage of only having to write our backend service integration once.
 
## PCL integration
 
In the PCL we will require the following NuGet packages:
 
- [HTTP Client](https://www.nuget.org/packages/Microsoft.Net.Http/) by Microsoft
- [JSON.Net](https://www.nuget.org/packages/Newtonsoft.Json) by [James Newton-King](http://james.newtonking.com/)

 
Armed with the capabilities of now being able to communicate to a HTTP backend and serializing i.e. deserializing objects to and from JSON we can start implementing our communication layer by creating a HTTP client in the constructor. We usually only need one instance of the HTTP client as it will handle multiple requests and only spawn as many active connections as are reasonable over HTTP:


    public class PersonService : IPersonService
    {
        private HttpClient _httpClient;
    
        public PersonService()
        {
            _httpClient = new HttpClient();
        }
    
        // ... Person Service implementation
    }


For reading over HTTP we should use the GET keyword which is provided by the HTTP client and is implemented as follows:


    public async Task<IEnumerable<Person>> GetPeople()
    {
        var result = await _httpClient.GetAsync(BASE_URI);
        var peopleJson = await result.Content.ReadAsStringAsync();
    
        return JsonConvert.DeserializeObject<IEnumerable<Person>>(peopleJson);
    }



> Note that if you are using a free website on Azure you can choose to run communications over SSL, the only thing you have to change for this is adding an ***s*** in the URI after http i.e. resulting in an ***https*** URI.


Writing to a HTTP service depends on the action we want to perform. Creating something new e.g. adding a new person to the existing person list, should be performed over a HTTP POST.  Implementing the post can be done as follows:


    public async Task<bool> CreatePerson(Person person)
    {
        var personJson = JsonConvert.SerializeObject(person);
        var content = new StringContent(personJson, Encoding.UTF8, "application/json");
        var result = await _httpClient.PostAsync(BASE_URI, content);
    
        return result.IsSuccessStatusCode;
    }


The <font face="Consolas">StringContent</font> method has additional parameters to indicate to the server how the content can be interpreted. This is required to inform the backend service that the data we are sending is serialized as JSON. Not doing so would end in false information in the header which would prevent Web API to automatically deserialize the body.

Updating an existing person usually is done via PUT, which is similar to the <font face="Consolas">CreatePerson</font> method but contains an ID in the URI which usually corresponds with the ID from the Person which is set on the backend.


    public async Task<bool> UpdatePerson(int id, Person person)
    {
        var uri = Path.Combine(BASE_URI, id.ToString());
    
        var personJson = JsonConvert.SerializeObject(person);
        var content = new StringContent(personJson, Encoding.UTF8, "application/json");
    
        var result = await _httpClient.PutAsync(uri, content);
    
        return result.IsSuccessStatusCode;
    }


As you can see implementing a Web service is quite straight forward with the HTTP Client and allows to easily create a backend integration for native clients that can be shared across platforms.

# The UI

On the client side we now can invoke our portable service and display a list of people.


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
    
            <AppBar VerticalAlignment="Bottom">
                <AppBarButton Icon="Add" Label="Add" Command="{Binding AddNewPerson}" />
            </AppBar>
        </Grid>
    </Page>


In the ViewModel we can invoke the backend service to provide the data.


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
            AddNewPerson = new RelayCommand(() => ShowPerson(0));
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


Adding a person means we have to add a further page to our client application that provides us with a form to enter the name and first name of a person. To keep things simple we will just use the same UI and identify via the navigation parameter if we have to create or update the person on the backend.


    <Page
        x:Class="HttpClientSample.PersonDetail"
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


The ViewModel is handling the integration with the <font face="Consolas">PersonService</font> from the PCL library:


    public class PersonViewModel:ViewModelBase
    {
        private readonly IPersonService _personService;
        private int _id;
        private string _firstname;
        private string _lastname;
        private bool _hasPendingChanges;
    
        public PersonViewModel(IPersonService personService)
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
    
            if (_id > 0)
            {
                await _personService.UpdatePerson(_id, person);
            }
            else
            {
                await _personService.CreatePerson(person);
            }
    
            HasPendingChanges = false;
        }
    }


Again integrating a PCL library is not difficult and requires no additional hoops to jump through. What we gain by putting our logic into the PCL is that we can reuse the code across multiple platforms.

# The Controller

We will focus on the client part during this post but for getting an understanding of the bigger picture here is the controller we will be communicating with.


    public class PersonController : ApiController
    {
        private readonly static Lazy<IList<Person>> _people = new Lazy<IList<Person>>(() => new PersonService().GeneratePeople(1000), LazyThreadSafetyMode.PublicationOnly);
        // GET: api/Person
        public IEnumerable<Person> Get()
        {
            return _people.Value;
        }
    
        // GET: api/Person/5
        public Person Get(int id)
        {
            if (id < 0 || _people.Value.Count < id) return null;
            return _people.Value[id];
        }
    
        // POST: api/Person
        [HttpPost]
        public IHttpActionResult Post(Person value)
        {
            if (value == null) return BadRequest();
    
            _people.Value.Add(value);
    
            return CreatedAtRoute("DefaultApi", new { id = _people.Value.IndexOf(value) }, _people.Value.Last());
        }
    
        // PUT: api/Person/5
        [HttpPut]
        public IHttpActionResult Put(int id, [FromBody]Person value)
        {
            if (id < 0 || id >= _people.Value.Count()) return BadRequest();
    
            _people.Value[id] = value;
    
            return Ok(value);
        }
    
        // DELETE: api/Person/5
        public IHttpActionResult Delete(int id)
        {
            if (id < 0 || id >= _people.Value.Count()) return BadRequest();
    
            _people.Value.RemoveAt(id);
    
            return StatusCode(System.Net.HttpStatusCode.NoContent);
        }
    }
    
    internal class PersonService
    {
        public IList<Person> GeneratePeople(int count)
        {
            var firstNames = new List<string>(count);
            var lastNames = new List<string>(count);
    
            for (int i = 0; i < count; ++i)
            {
                firstNames.Add(NameGenerator.GenRandomFirstName());
                lastNames.Add(NameGenerator.GenRandomLastName());
            }
    
            return firstNames.Zip(lastNames, (firstName, lastName) => new Person(firstName, lastName)).ToList();
        }
    }


As you can see it is rather basic and through the static variable we have some sort of poor mans database. Do not use this way of storing data in a production environment!

# Conclusion

In this post we saw how we can integrate a ASP.Net Web API server in a PCL. Thereby seeing how to read and write objects to a web service. Further we saw how we can display and enter the data on a Windows 10 client. The entire sample could have been written for Xamarin.iOS, Xamarin.Android or WPF which is exactly the sweet spot of writing your business logic in the PCL it does not matter what client you are intending to write now or in the future as long as the PCL allows you to integrate multiple client platforms.

You can find the entire sample project on [NuGet](https://github.com/mallibone/HttpClientSample.git).
