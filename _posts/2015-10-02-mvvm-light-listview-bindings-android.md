---
layout: single
title: "Xamarin.Android MVVM Light ListView bindings"
title: Xamarin.Android MVVM Light ListView bindings
date: 2015-10-02
tags: ["MVVM Light", "Mobile", "Xamarin", "Android"]
slug: "mvvm-light-listview-bindings-android"
---

[![Screenshot_2015-10-02-08-41-47]({{ site.url }}{{ site.baseurl }}/assets/images/bb6a944f-4067-48c0-a6c7-464be87be12c.png "Screenshot_2015-10-02-08-41-47")]({{ site.url }}{{ site.baseurl }}/assets/images/1b923374-d5ba-4e52-a6d7-fc5354d3d182.png)
 
This is an extension to a previous [post](https://mallibone.com/post/xamarinandroid-and-mvvm-light-bindings) that describes how to create bindings for controls. In this post we will look at how to bind a collection to a Android <font face="Consolas">ListView</font> and update the View every time an item is added or removed from the collection in the View Model.
 
# Project Setup
 
Add the MVVM Light libraries from [Laurent Bugnion](http://www.galasoft.ch/) via NuGet to a Xamarin.Android project:


    PM> Install-Package MvvmLight


Then in the View Model create an <font face="Consolas">ObservableCollection</font> which will represent the entire List.


    public ObservableCollection<Person> People { get; private set; }


To setup the list we can use an Initializer method, which is currently generated in a separate thread. Now this only makes sense when the list is large i.e. processor heavy which this creation can be if we start jacking up the number. So with the basics all set lets turn our attention to the Activity.

# Activity

The ideal place to setup bindings is in the <font face="Consolas">OnCreate</font> method, so by overriding it we can setup the binding for the list. The data container of an Android ListView is the Adapter which we usually have to create by hand. Thanks to MVVM Light we can do this in one line of code:


    protected override async void OnCreate(Bundle bundle)
    {
        base.OnCreate(bundle);
    
        SetContentView(Resource.Layout.Main);
    
        await Vm.InitAsync();
        PeopleListView.Adapter = Vm.People.GetAdapter(GetPersonView);
        // ...
    }


The <font face="Consolas">GetPersonView</font> parameter is a method that gets invoked when a displayed row is created. In this method the data of the row is filled accordingly into the UI fields:


    private View GetPersonView(int position, Person person, View convertView)
    {
        View view = convertView ?? LayoutInflater.Inflate(Resource.Layout.RowPerson, null);
    
        var firstName = view.FindViewById<TextView>(Resource.Id.FirstName);
        var lastName = view.FindViewById<TextView>(Resource.Id.LastName);
    
        firstName.Text = person.FirstName;
        lastName.Text = person.LastName;
    
        return view;
    }


Note that the YYY parameter might contain a row that can be reused. As only a certain amount of rows can be displayed at a time rows become obsolete after the user has scrolled them to far out of view. These rows are then passed in as the parameter YYY which is then not null and the fields can be updated with new data. Resulting in a smaller memory footprint which is always good on memory constrained mobile devices.

# View

The layouts are simple. For the Activity a simple <font face="Consolas">LinearLayout</font> is used with a ListView within it:


    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="vertical"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent">
        <ListView
            android:minWidth="25px"
            android:minHeight="25px"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:id="@+id/People" />
    </LinearLayout>


The row contains two <font face="Consolas">TextFields</font> that are used to display the persons first and last name.


    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="horizontal"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:padding="5dp">
        <TextView
            android:text="Michael"
            android:textAppearance="?android:attr/textAppearanceMedium"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:id="@+id/FirstName"
            android:layout_margin="5dp" />
        <TextView
            android:text="Westen"
            android:textAppearance="?android:attr/textAppearanceMedium"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:id="@+id/LastName"
            android:layout_margin="5dp" />
    </LinearLayout>


The row layout is then inflated in the <font face="Consolas">GetPersonView</font> method.

# Adding and removing people from the ListView

Adding and removing people from the collection is now really simple. So all we need to extend our view by are two buttons:


    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="vertical"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent">
        <LinearLayout
            android:orientation="horizontal"
            android:layout_width="fill_parent"
            android:layout_height="wrap_content">
            <Button
                android:id="@+id/AddButton"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/AddPerson" />
            <Button
                android:id="@+id/RemoveButton"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/RemovePerson" />
        </LinearLayout>
        <!-- ... -->
    </LinearLayout>


Then wiring them up in the Activity to Commands in the View Model.


    protected override async void OnCreate(Bundle bundle)
    {
        // ...
    
        AddPersonButton.SetCommand("Click", Vm.AddPersonCommand);
        RemovePersonButton.SetCommand("Click", Vm.RemovePersonCommand);
    }


In the View Model we simply have two Relay Commands which are initialized in the Constructor:


    public MainViewModel()
    {
        AddPersonCommand = new RelayCommand(AddPerson);
        RemovePersonCommand = new RelayCommand(RemovePerson);
    }
    
    public RelayCommand AddPersonCommand { get; set; }
    public RelayCommand RemovePersonCommand { get; set; }


When invoking the commands a person is added or removed from the observable collection which will automatically will be propagated to the presentation in the <font face="Consolas">ListView</font>.

# 

# Be aware of the Activity Lifecycle

Note there is a possibility of a memory leak when it comes to the adapter provided by MVVM Light, this has to do with an internal setup of how the Adapter is created. When the Activity is destroyed and recreated e.g. when you turn the device and change the orientation. Internally the activity will be hooked to an event of the observable collection in the View Model. This connection will last even after a new activity is created. Therefore the old activity still hangs around as it is always hooked to the Observable collection in the View Model. This is why in the View Model on every <font face="Consolas">InitAsync</font> the Observable Collection will be replaced with a new instance. This will clean up the trail and allow the no longer used Activity to be garbage collected.


    public async Task InitAsync()
    {
        if (People != null)
        {
            // Prevent memory leak in Android
            var peopleCopy = People.ToList();
            People = new ObservableCollection<Person>(peopleCopy);
            return;
        }
    
        People = new ObservableCollection<Person>();
    
        var people = await InitPeopleList();
        People.Clear();
        foreach (var person in people)
        {
            People.Add(person);
        }
    }


# Conclusion

In this post we saw how we can bind a collection to a ListView and how updates to the collection are automatically propagated to the view. Further we saw how to handle the lifecycle of an Activity properly without generating a memory leak. Thanks to MVVM Light the complexity displaying collections in an Android app are greatly reduced.



The entire sample can be found under [GitHub](https://github.com/mallibone/MvvmLightSamples/tree/master/Android/MvvmLightListBindings.Droid).

This post was previously posted on the [Noser Engineering Blog](http://blog.noser.com/xamarin-android-mvvm-light-listview-bindings/).
