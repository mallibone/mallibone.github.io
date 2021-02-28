---
layout: single
title: "F# + Twilio = SMS Secret Santa"
title: F# + Twilio = SMS Secret Santa
date: 2020-12-29
tags: ["F#"]
slug: "fsharp-twilio-secret-santa"
---

[![Photograph showing christmas gifts](https://mallibone.com/posts/files/e9f5059a-74b4-4ee5-b6a5-e109919a90c9.jpg "00_TitlePhoto")](https://mallibone.com/posts/files/39281630-cc03-4a9a-900f-8772501bfb26.jpg)

So with a pandemic, Christmas was interesting, to say the least, this year. Following the COVID-19 guidelines, we were able to exchange our secret Santa gifts this year. There was only one problem. We usually have a little paper-based lottery for assigning the secret Santa. And we tend to do it on the same evening. But with COVID-19 doing its thing there was FaceTime and such involved this year. FaceTime was great but not so good when it comes to the lottery part, especially since no one should know who got who.

So what was there to be done for a successful Secret Santa 2021 without anyone knowing who picked whom? Why not do this on my own, break out [Visual Studio Code](https://code.visualstudio.com/) and write app a little app that will do this for us. So I broke the process down into the following steps:

- Read the input
- Assign randomly
- Send a text message(s)



> **Disclaimer:** I did not receive any compensation for using Twilio. I did this because it seemed like a fun project, and SMS just seemed to be the friendliest approach for everyone involved.


Now for reading the input, I went with a more or less standard CSV file format as follows:


    Person1;+15551234
    Person2;+15551235


I figured this would allow for some flexibility and allow me to share the code without sharing any private contact information. 😉

Reading the input can be done with a few helpers from the .Net Framework living in the `System` and `System.IO` namespace.


    type Person = { Name : string; MobileNumber : string }
    
    let parseToPerson (inputString:string) =
        let csvItems = inputString.Split(";")
        {Name = csvItems.[0]; MobileNumber = csvItems.[1]}


Now next to the algorithm which would assign a Secret Santa. In our family, the rules are quite simple. Everyone is the Secret Santa of someone else, and no one is his or her own Secret Santa. The random assignments can are achieved by ordering the list with a random number and then zipping it to the order of the CSV file:


    let random = Random()
    let rec assignSecretSantas (people : Person seq) =
        let secretSantasAssignments =
            people
            |> Seq.sortBy(fun x -> random.Next())
        
        let assignments = 
            secretSantasAssignments
            |> Seq.zip people
            |> Seq.toList
        
        if assignments |> Seq.exists (fun a -> fst a = snd a) then
            assignSecretSantas people
        else
            assignments


Lastly, the assignment is checked that no one is assigned to themselves. Should there be an illegal assignment, the method will execute recursively.

Now to the part which was new to me. Send a text message from code. I have heard of [Twilio](https://www.twilio.com/) before but never had the opportunity to play around with their service. Setting up an account is your run of the mill create an account scenario. They even provide you with a couple of bucks to try out their tutorials. So with my trial account and money at hand, I implemented the SMS part.

The first step is adding the [NuGet package](https://www.nuget.org/packages/Twilio). When developing this, I used an F# Script file. Why am I mentioning this, well because since [F# 5](https://docs.microsoft.com/en-us/dotnet/fsharp/whats-new/fsharp-50#package-references-in-f-scripts) adding/using a NuGet package in a script is as simple as adding the following lines to the top of your script file:


    #r "nuget: Twilio"
    
    open Twilio
    open Twilio.Rest.Api.V2010.Account


But in the final version, I opted for a console app, which uses the standard `dotnet add package Twilio` to add the package to your project.

Before being able to send messages, the client needs to be initialized.


    let initTwilio =
        let accountSid = configuration.GetSection("Twilio").["Sid"]
        let authToken = configuration.GetSection("Twilio").["AuthToken"]
        TwilioClient.Init(accountSid, authToken)


I must say the [docs](https://www.twilio.com/docs/sms) from Twilio were a great help here. Be sure to check them out if you are looking into more advanced scenarios.

With the client initialized, I wrote the code to send the message:


    let secretSantaPhoneNumber = "+15551236"
    let secretSantaMessage = "Testmessage"
    MessageResource.Create(
        Twilio.Types.PhoneNumber(secretSantaPhoneNumber), 
        body=secretSantaMessage, 
        from=Twilio.Types.PhoneNumber("Twilio-Sender-Phone-Number"))


Sending the message was a tad more complicated than initially expected. In comparison, everything worked with my phone number, which I had to provide when signing up. When trying to send a test message to someone else in the family, I got the error message to use a number that had previously been registered with Twilio's trial version. The fix was going for a paid account - but after that, I could implement the sending of Secret Santas:


    let rec informSecretSantas dryRun (santas: (Person * Person) list) =
        match santas with
        | [] -> ()
        | head::tail ->
            let santaName = (fst head).Name
            let giftReceiver = (snd head).Name
            let phoneNumber = (fst head).MobileNumber
            let body = $"The secret santa message"
            if dryRun then
                printfn "%s" body
                printfn "Sending to %s" phoneNumber
            else
                let msg = MessageResource.Create(Twilio.Types.PhoneNumber(phoneNumber), body=body, from=Twilio.Types.PhoneNumber("Twilio-Phone-Number"))
                printfn "%A" msg.Status
            informSecretSantas dryRun tail


And with that Christmas 2021 was saved. Now was this the best option? Well, there are quite a few services out there - some even support SMS and don't cost you a thing. But where would have the fun been in one of those? 🙃

For me, the experience of using F# Script files to carve out the different parts of the "app" was great and very intuitive. Running code in the F# [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) / [FSI](https://docs.microsoft.com/en-us/dotnet/fsharp/tools/fsharp-interactive/) allowed for quick iterations and trying out. An added benefit was using [Ionide](https://ionide.io/) - the plugin you want to install when developing F# with VS Code. And Ionide just got an update to support F# 5.0. 🥳

Oh, and another word of advice, if you keep the buttons pressed to execute code in the FSI - it will run multiple times. Ask how my family knows after receiving not one but four secret Santa text messages in the first "test" run. 🙈

You can find the entire code on [GitHub](https://github.com/mallibone/SecretSanta).



Title photo by **[Lucie Liz](https://www.pexels.com/@lulizler?utm_content=attributionCopyText&amp;utm_medium=referral&amp;utm_source=pexels)** from **[Pexels](https://www.pexels.com/photo/wrapped-presents-3298041/?utm_content=attributionCopyText&amp;utm_medium=referral&amp;utm_source=pexels)**
