---
layout: single
title: "Using addresses, maps and geocoordinates in your Xamarin Forms apps"
title: Using addresses, maps and geocoordinates in your Xamarin Forms apps
date: 2020-02-12
tags: ["Xamarin.Forms"]
slug: "xamarin-maps-addresses-geocoords"
---

[![00_TitleImage]({{ site.url }}{{ site.baseurl }}/images/fa8b38b3-5a2a-4d8b-8b01-8c1d3a455058.jpg "00_TitleImage")]({{ site.url }}{{ site.baseurl }}/images/bad03f25-d450-4278-ac55-69ca9780a8c5.jpg)

Ever had to develop an app where the position of something was of interest. An address of a user, store, restaurant or some other point of interest? The straight forward version might be to provide the user with a simple form.

[![Address entry form]({{ site.url }}{{ site.baseurl }}/images/12a03657-6545-411a-9d6a-2a211da91186.png "Address entry form")]({{ site.url }}{{ site.baseurl }}/images/89cdb3df-7181-406a-a3e4-c86aa3cfb554.png)

While the form is easy to implement and reliable, it can be cumbersome. Plus what if you want to navigate to the location? Or why not use the position of the device? Tap on a map to get the address? Well, all of these things can be achieved quite easily with a few helpful libraries provided by the Xamarin team.

### Geocoding like a pro

Whenever you need to convert between coordinates and addresses, [Geocoding](https://docs.microsoft.com/en-us/xamarin/essentials/geocoding?tabs=android) is your best friend. You can show up with either an address or position, and you will get back the opposite:


    var location = (await Geocoding.GetLocationsAsync($"{street}, {city}, {country}")).FirstOrDefault();
    
    if (location == null) return;
    
    Position = new Position(location.Latitude, location.Longitude);


So if we had a form where the user can enter the address and we wanted to navigate to that position, we could convert the address to geo coordinates and then, using [Xamarin Essentials](https://docs.microsoft.com/en-us/xamarin/essentials/maps?context=xamarin%2Fandroid&amp;tabs=android), open the maps app on the platform and navigate to that position.


    var location = new Location(Position.Latitude, Position.Longitude);
    var options = new MapLaunchOptions { NavigationMode = NavigationMode.Driving };
    await Xamarin.Essentials.Map.OpenAsync(location, options);


Quite impressive for a bunch of entries and a few lines of code right?

### Getting an address with one tap

But typing in addresses can be cumbersome. Why not just select the position on the map and get the address from that position? Xamarin Forms has supported the maps control for quite some time. Once you have added the [NuGet package](https://www.nuget.org/packages/Xamarin.Forms.Maps), it will allow you to present the native map platform, i.e. Google Maps on Android.


    <?xml version="1.0" encoding="utf-8" ?>
                 ...
                 xmlns:maps="clr-namespace:Xamarin.Forms.Maps;assembly=Xamarin.Forms.Maps"
                 ... >
    
        <Grid>
            <Grid.RowDefinitions>
                <RowDefinition Height="2*" />
                <RowDefinition Height="*" />
            </Grid.RowDefinitions>
            <maps:Map x:Name="MapControl" />
            ...
        </Grid>
    
    </ContentPage>





> Note there are some setup steps required depending on the platform you intend to target. So be sure to check out the official docs from Microsoft on those details. Here we will continue to focus on the fun stuff


The map has an event `MapClicked` which fires when the user taps on the map with the geo-position.


    Observable.FromEventPattern<MapClickedEventArgs>(
        mc => MapControl.MapClicked += mc, 
        mc => MapControl.MapClicked -= mc)
        .Subscribe(ev => ViewModel.ExecuteSetAddress.Execute(ev.EventArgs.Position));


If you are not familiar with Rx.Net, here is the equivalent code with an event handler:


    protected override void OnAppearing()
    {
       // ...
    
        MapControl.MapClicked += HandleMapClicked;
    }
    
    private void HandleMapClicked(object sender, MapClickedEventArgs e)
    {
        var postion = e.Position;
        // ...
    }


To give the user some feedback, we could even present the taped position with a pin:


    private Unit SetPin(Position position)
    {
        Pin pin = new Pin
        {
            Label = "The Place",
            Address = $"{ViewModel.Street}, {ViewModel.City}, {ViewModel.Country}",
            Type = PinType.Place,
            Position = position
        };
    
        var latDegrees = MapControl.VisibleRegion?.LatitudeDegrees ?? 0.01;
        var longDegrees = MapControl.VisibleRegion?.LongitudeDegrees ?? 0.01;
        MapControl.MoveToRegion(new MapSpan(position, latDegrees, longDegrees));
        MapControl?.Pins?.Clear();
        MapControl?.Pins?.Add(pin);
        return Unit.Default;
    }


This code will always place the pin at the new location. If the user taps the pin, it will display the information provided to the pin.


> Side note: Managing to present that info is quite a challenge given this code, 9 out of 10 times I only managed to place the pin at a new position. But perhaps that's just me


Using the geocoder, we can now get the address for those coordinates:


    private async Task SetAddress(Position p)
    {
        var addrs = (await Geocoding.GetPlacemarksAsync(new Location(p.Latitude, p.Longitude))).FirstOrDefault();
        Street = $"{addrs.Thoroughfare} {addrs.SubThoroughfare}";
        City = $"{addrs.PostalCode} {addrs.Locality}";
        Country = addrs.CountryName;
        // ...
    }


But now you have it, going from Coordinates to an address and back the other way. Easy right? Well, there is still one more thing; the default coordinates of the map point to Rome. And I get it. Rome has a lot to offer, but if your user is not currently in Rome, it might be quite a bit of pinching and swiping until the map will be useful. So long story short, why not use the user's position?

### Marco? Polo!

Getting the users position is a piece of cake again, thanks to [Xamarin Essentials](https://docs.microsoft.com/en-us/xamarin/essentials/geolocation?context=xamarin%2Fandroid&amp;tabs=android). That is once you have required all the permissions as described in the [link](https://docs.microsoft.com/en-us/xamarin/essentials/geolocation?context=xamarin%2Fandroid&amp;tabs=android#get-started). With just a few lines we can get the address and set the position:


    try
    {
        var location = await Geolocation.GetLastKnownLocationAsync()var position = new Position(location.Latitude, location.Longitude);Position = position;// ... set address and stuff
    }
    // many exception handlers according to docs here





> The user will get prompted the first time this code runs if they want to share their position. Just as a side note if you do not want to smother the user right of the batt with permission dialogues, you might want to put some thought into when this code should first be run.


So when the view appears, we can try to get the users position and tell the map control what part of the world to display:


    Geolocation.GetLastKnownLocationAsync()
                .ToObservable()
                .Catch(Observable.Return(new Location()))
                .SubscribeOn(RxApp.MainThreadScheduler)
                .Subscribe(async location => { 
                    var position = new Position(location.Latitude, location.Longitude);
                    Position = position;
                    await SetAddress(position);
                });


I hope you could see that it does not take much to go from tiresome form to a "fun" maps control.

[![GifRecordingSmall]({{ site.url }}{{ site.baseurl }}/images/22099ce0-7096-40b0-bbe3-0372018dcf9c.gif "GifRecordingSmall")]({{ site.url }}{{ site.baseurl }}/images/eab4d0b1-27ff-4149-a2e7-834f0ee0ef3e.gif)

And all by using standard libraries offered to you by Microsoft. You can find the complete sample on [GitHub](https://github.com/mallibone/XamarinAddressPosition).

HTH
