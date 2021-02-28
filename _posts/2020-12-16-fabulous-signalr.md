---
layout: single
title: "Working with SignalR and Fabulous"
title: Working with SignalR and Fabulous
date: 2020-12-16
tags: ["F#", "Fabulous", "Xamarin"]
slug: "fabulous-signalr"
---

[![00_Title](https://mallibone.com/posts/files/cb40190b-4067-4a64-abdd-e9eed5239514.jpg "00_Title")](https://mallibone.com/posts/files/7cba682c-d343-40d8-857f-e4e6be2ac4e5.jpg)

This blog post is part of the [F# Advent Calendar 2020](https://sergeytihon.com/2020/10/22/f-advent-calendar-in-english-2020/). A big thank you to [Sergey](https://twitter.com/sergey_tihon) for organizing this year and be sure to check out the other blog posts - after reading this one .

Based on the Model View Update (MVU) pattern the Fabulous frameworks provides the means to write functional first mobile (and desktop), clients. If you are new to Fabulous, I would recommend you check out the [post](https://timothelariviere.com/2019/12/21/how-to-become-a-fabulous-developer/) by [Tim](https://twitter.com/Tim_Lariviere) (the maintainer of Fabulous) or the official [docs](https://github.com/fsprojects/Fabulous).

The backend is implemented as an ASP.NET Core web app which has SignalR enabled. You can see how to this [here](https://docs.microsoft.com/en-us/aspnet/core/signalr/introduction?view=aspnetcore-5.0).

To show how you can use SignalR with Fabulous, we will implement a chat application. And I felt like calling it "Fabulous Chat" - see how is easy it is to do fabulous things with this framework ? So let's ignore the question: "Does this world really need yet another chat app?!" in the back of our minds and let's have a look at what we will implement in this app to highlight how you could use SignalR within in a functional first mobile app.

Let's assume we have the following requirements:

- A user should identify himself by his name.
- The user should be able to send messages.
- A user should be able to read messages while the app is active.


In other words, two actions are entirely UI driven, the user enters his username and then enters the chat. The user types a message and then sends it by the push of a button. However, when it comes to being able to read messages, the event will be fired from the background. Further, the SignalR connection usually is established once and then used for the remainder of the session. So let's create a module in our app, which will contain all the operations regarding the SignalR interaction.


    module SignalR =
        let connectToServer =
            // connect to SignalR service
    
        let startListeningToChatMessages (connection: HubConnection) dispatch =
            // receive messages
    
        let sendMessage (connection: HubConnection) (message: ChatMessage) =
            // send message


We connect to the service after the user has provided his name. Not because it is required per se. But if at some later point we decide to add some proper authentication this will not change the flow of the app. Without further ado, let's implement the login view.

[![Image showing the login view](https://mallibone.com/posts/files/da072851-7565-45ef-90f6-582a06dba440.png "00_Login")](https://mallibone.com/posts/files/378e89d0-ef8e-41fc-9adc-5a748026b3fd.png)

The view we use for this part is as described below.


    let view model dispatch =
        View.ContentPage
            (title = "Login",
             content =
                 View.StackLayout
                     (verticalOptions = LayoutOptions.Center,
                      horizontalOptions = LayoutOptions.Center,
                      children =
                          [ View.Label(text = "Please enter your Username")
                            View.Entry
                                (text = model.Username,
                                 maxLength = 15,
                                 placeholder = "Please enter your username",
                                 completed = (fun _ -> (loginUser dispatch)),
                                 textChanged = fun e -> dispatch (UsernameChanged e.NewTextValue))
                            View.Button(text = "Login", command = fun _ -> (loginUser dispatch))
                            ]))


Once the user hits the *Login* button, we want to establish the SignalR connection.


    let connectToServer =
        let connection =
            HubConnectionBuilder()
                .WithUrl(Config.SignalRUrl)
                .WithAutomaticReconnect()
                .Build()
    
        async {
            do! connection.StartAsync() |> Async.AwaitTask
            return connection
        }


Since we will want to hold on to the connection, we will invoke a `Command` which will be evaluated in the Update method.


    let update msg (model: Model) =
        match msg with
        // ... other message handling
        | Connected connection ->
            { model with
                  SignalRConnection = Some connection
                  AppState = Ready },
            Cmd.none
        // ... more message handling


Now, after the user is connected to the chat, we will present the chat view. This view allows the user to type a message, send it and read the responses or questions by other users connected to the service.

[![Image showing the chat view in action](https://mallibone.com/posts/files/ede82d45-3b3d-462c-8b2b-39b7aec24fa1.png "01_Chat")](https://mallibone.com/posts/files/821ba9bd-000a-4626-ac22-e6b5a8bf7ded.png)

Let's start with writing messages. Similar as with the username we again have an `Entry` and a button for sending the messages. Once the user sends a message, we invoke `SendAsync` in our SignalR module:


    let sendMessage (connection: HubConnection) (message: ChatMessage) =
        async {
            let jsonMessage = JsonSerializer.Serialize(message)
    
            do! connection.SendAsync("SendMessage", jsonMessage)
                |> Async.AwaitTask
        }


So thus far, we have connected to the SignalR service, and we can send messages to the server. But we are still missing one essential part, and that is how we can receive messages from the backend service. What we need to do is register a listener to a specific SignalR-method (called `NewMessage`). We can implement our function as follows:


    let startListeningToChatMessages (connection: HubConnection) dispatch =
        let handleReceivedMessage (msg: string) =
            printfn "Received message: %s" msg
            dispatch (Msg.MessageReceived(JsonSerializer.Deserialize<ChatMessage>(msg)))
            ()
    
        connection.On<string>("NewMessage", handleReceivedMessage)


Now in the handler method, we will dispatch a command every time a new message is received from the SignalR service. So let's extend our login function, from the beginning, to not only create a connection but also register our receiver.


    let loginUser dispatch =
        async {
            let! connection = SignalR.connectToServer
            dispatch (Msg.Connected connection)
    
            SignalR.startListeningToChatMessages connection dispatch
            |> ignore
    
            dispatch (Msg.LoggedIn)
        }
        |> Async.StartImmediate


We pass in the dispatcher from the view method when registering. This allows us to dispatch a command, i.e. invoke the `update` method and add the new message to our list of chat messages:


    let update msg (model: Model) =
        match msg with
        // ... other message handling
        | MessageReceived chatMessage ->
            { model with
                  Messages = chatMessage :: model.Messages },
            Cmd.none
        // ... more message handling


And with that, the user will be able to send and receive chat messages with whoever is using the chat program at the current moment.

## Conclusion

And that is how we can use SignalR in a Fabulous app and create a Fabulous Chat app. Perhaps we still have to go over the design and security to really earn that name .

What you also saw is how you can work with events that do not originate from the user's input but happen in the background. This technique can be used whenever you are using some code that gets invoked in the background while your app is doing other fabulous stuff.

You can find the complete sample of the app on [GitHub](https://github.com/mallibone/FabulousChat/tree/v1.0).

Titlephoto by **[Tyler Lastovich](https://www.pexels.com/@lastly?utm_content=attributionCopyText&amp;utm_medium=referral&amp;utm_source=pexels)** from **[Pexels](https://www.pexels.com/photo/black-iphone-7-on-brown-table-699122/?utm_content=attributionCopyText&amp;utm_medium=referral&amp;utm_source=pexels)**
