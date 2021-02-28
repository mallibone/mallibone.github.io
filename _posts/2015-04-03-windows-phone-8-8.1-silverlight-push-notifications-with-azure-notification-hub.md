---
layout: single
title: "Windows Phone 8/8.1 Silverlight push notifications with Azure Notification Hub"
title: Windows Phone 8/8.1 Silverlight push notifications with Azure Notification Hub
date: 2015-04-03
tags: ["Azure", "Windows Phone", "Xamarin", "Xamarin.Forms", "Mobile"]
slug: "windows-phone-8/8.1-silverlight-push-notifications-with-azure-notification-hub"
---

Technorati Tags: [windows phone](http://technorati.com/tags/windows+phone),[windows](http://technorati.com/tags/windows),[push](http://technorati.com/tags/push),[azure](http://technorati.com/tags/azure),[azure notification hub](http://technorati.com/tags/azure+notification+hub),[push messages](http://technorati.com/tags/push+messages),[push message](http://technorati.com/tags/push+message),[wp8](http://technorati.com/tags/wp8),[wp8 silverlight](http://technorati.com/tags/wp8+silverlight)
[![PushNotification](http://mallibone.com/posts/files/6b8db687-70f2-4366-acff-a604a9f45148.png "PushNotification")](http://mallibone.com/posts/files/1308b685-543c-4ad1-a9ca-4dc7c27dd631.png)
Push notifications are a great way to inform users about an update. This might be a chat message, breaking news or simply put alert your users that something of interest has occurred and is probably worth her while to check it out. Now when we look at push notifications there is obviously the receiver which resembles the Phone. On the sending part you will find some backend service e.g. a web server that initiates the push notification. I'll be using a Xamarin.Forms application for this demo, but all these steps apply without any difference for a standard Windows Phone Silverlight application.
[![NotificationHubBigPicture](http://mallibone.com/posts/files/82f55a9c-8b60-4dac-a0dd-dd515c10c6ba.png "NotificationHubBigPicture")](http://mallibone.com/posts/files/845dfcc8-40e9-4f2e-ad9b-53ce4911b445.png)
# Setting up the backend
 
Now lets first setup the backend so we can send the push notification messages, which will require a valid [Azure Account](http://azure.microsoft.com/en-us/pricing/free-trial/). Log on to [Azure](http://azure.microsoft.com/en-us/) and perform the following steps:
 
1. Click **+NEW**
2. Click **APP SERVICES**, then **SERVICE BUS** and then select **NOTIFICATION HUB**
3. Enter name, select the region and if you haven't done so already enter your namespace (this option does not appear if you already have created a namespace).

[![CreateNotificationHub](http://mallibone.com/posts/files/148701b9-d90d-447e-a4cb-a21c8b5e9455.png "CreateNotificationHub")](http://mallibone.com/posts/files/72e1204b-fac5-4a0e-9349-0dfd58e04d9d.png)
After the notification hub is created we will have to configure it for use with Windows Phone Silverlight. Select your notification hub (namespace), which you will find under the **SERVICE BUS** tab and then select the notification hub you just created.
 

> Note you may have multiple notification hubs under the same namespace, this can be really handy once your app is in the field and you are developing on the next version you can separate the Dev and Production notification hubs very easy this way.

 
![NHMPNSSetup](http://mallibone.com/posts/files/8858eb4c-a1d3-40b6-b790-c02398823074.png "NHMPNSSetup")Now for development the easiest way is to simply **Enable unauthenticated push notifications.** and save the changes. which will not require registering the app at this point. If you already have done so you can also upload the certificate you can download from the [Windows Developer Portal](http://developer.windows.com/en-us).
 
# Enabling your App to receive push notifications
 
Creating a Xamarin.Forms project is [done really easily](http://mallibone.com/post/xamarin.forms-xaml-hello-world). We will now perform the following steps:

1. Permissions for Remote Notifications
2. Install the Azure Messaging NuGet Package
3. Register your app on startup
4. Sending the first message

 
## Permissions for Remote Notifications
 
Open the **WMAppManifest.xml** file which is located under the **Properties** of the Windows Phone Project. Under Capabilities ensure that **ID\_CAP\_PUSH\_NOTIFICATION** is enabled.
[![mobile-app-enable-push-wp8](http://mallibone.com/posts/files/796b5ca8-960f-49ee-845c-df4e6c031ee9.png "mobile-app-enable-push-wp8")](http://mallibone.com/posts/files/d755795d-c0fd-4d9d-a4e5-f39794c2d50d.png)
## Install the Azure Messaging NuGet Package
 
Open the NuGet Package Manager by right clicking on the Windows Phone project and select **Manage NuGet Packages...**. In the **Online** tab search for the package *WindowsAzure.Messaging.Managed* and install it in your Windows Phone project.
 
## Register your app on startup
 
Now lets wire the backend service together with our mobile app. This is done in the **App.xaml.cs** file which is located in the root of your Windows Phone Project. Within the **Application\_Launching** method add the following code:


    var channel = HttpNotificationChannel.Find("ChannelName");if (channel == null){    channel = new HttpNotificationChannel("ChannelName");    channel.Open();    channel.BindToShellToast();}channel.ChannelUriUpdated += new EventHandler<NotificationChannelUriEventArgs>(async (o, args) =>{    var hub = new NotificationHub("<hub name>", "<connection string>");    await hub.RegisterNativeAsync(args.ChannelUri.ToString());});


Which will require the following usings:


    using Microsoft.Phone.Notification;using Microsoft.WindowsAzure.Messaging;


Note that the Channel name is individual to your application and is used to check if your app has already registered for a push notification. We are currently only binding to the ShellToast notification type. Now lets enter the *hub name* and *connection string* for your messaging hub. Due to the superb integration of azure within Visual Studio open the Server Explorer (if not already present on the left side you can find it under the **VIEW** menu), open the **Azure** dropdown, then the **Notification Hubs** and select the notification hub you created before in my example it's named *mallibone* which resembles the hub name. In your properties sub-view you will see the connection strings. You will have to open the view by clicking on the three points to get the full connection string, for the client we will only need the listening connection string as we do not intend to write push notifications from the mobile client in this app.

Copy the connection string and insert it in the notification hub creation within the **Application\_Launching** method.


    private void Application_Launching(object sender, LaunchingEventArgs e){    var channel = HttpNotificationChannel.Find("MalliboneHelloWorldChannel");    if (channel == null)    {        channel = new HttpNotificationChannel("MalliboneHelloWorldChannel");        channel.Open();        channel.BindToShellToast();    }    channel.ChannelUriUpdated += new EventHandler<NotificationChannelUriEventArgs>(async (o, args) =>    {        var hub = new NotificationHub("mallibone", "Endpoint=sb://mallibone.servicebus.windows.net/;SharedAccessKeyName=DefaultListenSharedAccessSignature;SharedAccessKey=SOME_MUMBOJUMBO_AKA_THE_KEY        await hub.RegisterNativeAsync(args.ChannelUri.ToString());    });}


## Sending the first message

You could now send a notification and would receive toast notification if your app is not open. And by tapping on the toast notification your app would open. But for our first message lets have the app running and have the app vibrate when a message comes in.

### Handling push notifications when the app is running

Within **App.xaml.cs** add the following line to the **Application\_Launching** method:


    channel.ShellToastNotificationReceived += (o, args) => VibrationDevice.GetDefault().Vibrate(TimeSpan.FromMilliseconds(300));


Which will require the *using*:


    using Windows.Phone.Devices.Notification;


Ensure you can launch your application before proceeding.

### Sending the message

In the server explorer double click on your notification hub which will open a window that allows you to send push notifications and check on your device registrations (all within VS :)). Now under type select **Windows Phone MPNS** and then **Toast**. For now we do not have to alter the message so if your app successfully started and is humming beside you just click on **Send**.

# Adding parameters to the notification

Sending notifications is a great way to inform your users but often you do not only want a fancy Toast tile and then just start your app to the root page but rather give the user the content he just selected by tapping on the toast notification. So lets extend the message by adding a parameter to the toast message:


    <?xml version="1.0" encoding="utf-8"?><wp:Notification xmlns:wp="WPNotification">    <wp:Toast>        <wp:Text1>NotificationHub</wp:Text1>        <wp:Text2>Test message</wp:Text2>        <wp:Param>/MainPage.xaml?helloParameter=1234&amp;secondParameter=SomeKey</wp:Param>    </wp:Toast></wp:Notification>


## Parsing parameters on the client

Now when we start the app from the background i.e. by clicking on the toast notification, we will be starting the app with and can parse for the parameters in the **MainPage.xaml.cs** in the **OnNavigetTo** method:


    protected override void OnNavigatedTo(NavigationEventArgs e){    base.OnNavigatedTo(e);    const string helloParameter = "helloParemeter";    const string secondParameter = "secondParameter";    if (NavigationContext.QueryString.ContainsKey(helloParameter) && NavigationContext.QueryString.ContainsKey(secondParameter))    {        var message = string.Format("Received toast with HelloParameter {0} and SecondParemter {1}",            NavigationContext.QueryString[helloParameter], NavigationContext.QueryString[secondParameter]);        MessageBox.Show(message);    }}


Parsing the parameters when the app is running is done, as you might have already guessed in the **App.xaml.cs** class by extending the event handler.

# Conclusion

Adding push notifications can be a rich addition to your application as it allows you to interact with your user and give him news within seconds after something has happened. By using Azure you can really streamline the integration of push services across multiple platforms as I will show you in future posts.

### References

[http://azure.microsoft.com/en-us/documentation/articles/notification-hubs-windows-phone-get-started/](http://azure.microsoft.com/en-us/documentation/articles/notification-hubs-windows-phone-get-started/ "http://azure.microsoft.com/en-us/documentation/articles/notification-hubs-windows-phone-get-started/")
