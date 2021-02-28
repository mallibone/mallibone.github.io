---
layout: single
title: "Creating a Fabulous Xamarin app on a budget using F# Data"
title: Creating a Fabulous Xamarin app on a budget using F# Data
date: 2019-12-18
tags: ["F#", "Fabulous", "Xamarin.Forms"]
slug: "fabulous-budget"
---

[![TitleImage](https://mallibone.com/posts/files/7805b5d6-ed39-430c-a6d0-0974a6040f34.png "TitleImage")](https://mallibone.com/posts/files/60324200-638a-4cf1-a420-95ee974ccc62.png)

A while back, I used the F# type providers to create a conversion table. That post gave me the idea if it were possible to write an app that gets its data from a website. Perhaps you have also received the request for an app. Nothing expensive and actually all the data that should be displayed is already present on the web, i.e. this website right over here. So the question is: Could such an app be created without having to write a single line of backend code?

In this blog post, I will try to create an app for one of my favourite Xamarin conferences - the [Xamarin Expert Day](https://expertday.forxamarin.com/). So let's see if we can create our Fabulous Xamarin Expert Day App.

## Getting the data

Type providers are a delightful feature of F#. During compilation type providers generate the data models represented in a data source. I have written a [blogpost](https://mallibone.com/post/ral-colour-table-with-fsharp) before on the topic of parsing HTML. This time we will use another toolset that comes with the F# Data [NuGet](https://www.nuget.org/packages/FSharp.Data) package.

Since we want to get the information about the Xamarin Experts Day conference, we can try parsing the website directly. So we could use the following line to do this:


    HtmlDocument.Load "https://expertday.forxamarin.com/"


Unfortunately, we live in the modern ages of JavaScript. I don't want to go on a tangent here, but just state the fact that the Xamarin Experts Day website seems to be loading the information about the talks and the tracks after the initial HTML has been loaded. Luckily when loading the page in a browser, we get an HTML version which contains all of the information we are looking for. So instead of loading the data directly from the website, we can load the data from a file. Budget projects have their limitations... 🙃


    HtmlDocument.Parse("ExpertXamarin.html")


When we look at the HTML of the website (in the browser) we can see that the speakers are listed under the following HTML structure:


    <li data-speakerid="df2bc5ca-5a6b-48a9-87ac-71c817d7b240" class="sz-speaker sz-speaker--compact ">
    <div class="sz-speaker__photo">
      <a href="#" onclick="return ...');">
        <img src="...894db1.jpg">
      </a>
    </div>
    <h3 class="sz-speaker__name">
      <a href="#" onclick="return ...');">Mark Allibone</a>
    </h3>
    <h4 class="sz-speaker__tagline">Lead Mobile Developer Rey Automation, Microsoft MVP</h4>
    </li>


So we know that we can get the image, name, tagline and id of every speaker. So let's create a record to store that information:


    type Speaker = {Id:string; Name:string; Photo:string; Tagline:string}





> Creating the record is not strictly necessary. But it does make working with the data a bit easier later on. Another plus is that we could capsule the type provider code in a .Net Standard library and then share it with non F# .Net code. No, you can't access type provider data types directly from C# and while some features in C# get inspired by F#. From what I have heard, I would not hold my breath, hoping to see type providers in C# anytime soon...


With the record in place, all that is left to be done is extracting the data from the HTML. Good thing that F# Data comes along with the HTML CSS selector. The HTML CSS selector allows filtering after ids, classes and tag types. So if we wanted to get the speakers name, we can filter the component and then extract the value as follows:


    // .. other parsing methods
    let private getName (htmlNode:HtmlNode) =
        htmlNode.CssSelect("h3.sz-speaker__name > a") |> Seq.map (fun h -> h.DirectInnerText()) |> Seq.head
    
    let getSpeakers (html:string) =
        HtmlDocument.Parse(html)
            .CssSelect("li.sz-speaker")
            |> Seq.map (fun s -> {Id = (getId s); Name = (getName s); Photo = (getPhoto s); Tagline = (getTagline s)})


Similarly, the rest of the data can be accessed to fill the other fields in our record. Same goes for the tracks, again we would first create a record where we store the name, time, room and speaker id of every track. We will be able to link a track to a speaker with the id:


    type Track = {Room:string; Time:string; Title:string; SpeakerId:string option}
    
    let getTracks (html:string) =
        HtmlDocument.Parse(html)
            .CssSelect("div.sz-session__card")
            |> Seq.map(fun s -> {Room = (getRoom s); Time = (getTime s); Title = (getTitle s); SpeakerId = (getSpeakerId s) })


Now that we have all the data in place, it is time to get cracking on the app.

## Fabulous Xamarin Experts App

Before we start writing our UI code, there is still that shortcut we took above with loading the information out of a file. While this works great when using a script in the mobile world, this means we have to pack that HTML doc into the app. There are two approaches: either put it into the Assets folder on Android and in the Resources folder (you can also use XCAssets...) on iOS or make an Embedded Resource in the .Net Standard library. While the first option would be what Apple and Google intended you to use when adding docs, you want to ship with your app. You will have to jump through some hoops to access the document. So let's again save some time and just pack the file as an `Embedded Resource` in our .Net Standard project. Embedded Resources are packed into your apps binary. This results in an awkward fashion of accessing the data. While described in the official docs [here](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/data-cloud/data/files?tabs=windows&amp;WT.mc_id=DT-MVP-5002881#loading-files-embedded-as-resources), this is how it is implemented in the Xamarin Experts Day Conference App (we need a shorter name...):


    let loadFile filename =
        let assembly = IntrospectionExtensions.GetTypeInfo(typedefof<Model>).Assembly;
        let stream = assembly.GetManifestResourceStream(filename);
        use streamReader = new StreamReader(stream)
        streamReader.ReadToEnd()


With that out of the way. Let's create a list of all the talks with the title, time and room. When selecting a track, we will display the Information of the presentation along with the speaker info.

So the list we can put together like this:


    let showTrackCell track =
        View.ViewCell( view =
            View.StackLayout(children = [
                View.Label (text = track.Title, 
                            fontSize = FontSize 22.)
                View.Label (text = track.Time + " in " + track.Room, 
                            fontSize = FontSize 14.,
                            fontAttributes = FontAttributes.Italic)
                ]))
    
    let view (model: Model) dispatch =
    
        View.ContentPage(
            content = match model.SelectedTrack with 
                        | Some track -> showTrackInfo track model dispatch
                        | None -> View.ListView(
                                        rowHeight = 80,
                                        hasUnevenRows = true,
                                        margin = Thickness(8.,0.,0.,0.),
                                        items = (model.Tracks |> List.map showTrackCell),
                                        selectionMode = ListViewSelectionMode.Single,
                                        itemSelected = (fun args -> dispatch (TrackSelected args))
                                        )
            )


And the "detail view" would be done like this:


    let showTrackInfo track (model:Model) dispatch =
        let speaker = match track.SpeakerId with
                      | Some speakerId -> model.Speakers |> Seq.tryPick(fun s -> if s.Id = speakerId then Some s else None)
                      | None -> None
    
        let addSpeakerInfo (speaker:Speaker) =
            View.StackLayout(margin = Thickness(0.,32.,0.,0.), children = [
                    View.Label (text = "Speaker", fontSize = FontSize 22. )
                    View.Image (source = (Image.Path speaker.Photo))
                    View.Label (text = "Presenter: " + speaker.Name)
                    View.Label (text = "Tagline: " + speaker.Tagline)
                ])
            
        let speakerViewElements = match (speaker |> Option.map addSpeakerInfo) with
                                  | Some speakerInfo -> speakerInfo
                                  | None -> View.Label(text = "Brought to you by the Organizers");
    
        View.Grid (margin = Thickness(8.,8.,8.,16.),
                    rowdefs = [Star; Auto],
                    children = [
                        View.StackLayout(children = [
                            View.Label (text = track.Title, fontSize = FontSize 22.)
                            View.Label (text = "In: " + track.Room, fontSize = FontSize 14.)
                            View.Label (text = "At: " + track.Time, fontSize = FontSize 14., margin = Thickness(0.,-4.,0.,0.))
                            speakerViewElements
                            ])
                        (View.Button (text = "Back", command = (fun () -> dispatch (TrackSelected None)))).Row(1)
                    ])


You might have noticed that the talk description is missing. The website has a JavaScript function retrieve that additional information. I think it would be possible to replicate the JavaScript call to the backend and then parse through the answer

JSON/HTML answer. But INSERT-LAME-STATEMENT-WHY-I-AM-NOT-LAZY-HERE.

The app still feels a bit ruff I think I might have to follow up in another blog post and make it [pretty 😎](https://mallibone.com/post/creating-beautiful-xamarin-forms-apps-using-atomic-design-f-and-fabulous)

## Conclusion

In this little experiment, we set out to see if it would be possible to write a mobile app that displays the same information as already present on a website. And while there were some bumps in the road - I am looking at you JavaScript. It was indeed possible to write an app for the Xamarin Experts Day that runs on Android and iOS.

Though I really should get started on my next post and make the app pretty 😇

You can check out the entire app on [GitHub](https://github.com/mallibone/FabulousBudget).

This post is part of theF# advent calendar. Be sure to check out the other [posts](https://sergeytihon.com/2019/11/05/f-advent-calendar-in-english-2019/).

HTH
