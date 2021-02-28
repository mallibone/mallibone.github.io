---
layout: single
title: "MVVMLight Messenger"
title: MVVMLight Messenger
date: 2015-06-24
tags: ["Mobile", "Windows Phone", "MVVM Light"]
slug: "mvvmlight-messenger"
---

[![message](http://www.mallibone.com/posts/files/89a89efc-6dd1-42a0-b92b-b2c8b29371b2.png "message")](http://www.mallibone.com/posts/files/2f645e44-ad0d-4b9c-b029-4c5257474e7a.png)
 
Messages are a common pattern used in the Model View ViewModel (MVVM) pattern approach. They allow the ViewModel and Model to communicate with the view without having to couple them to it. Messages should generally be used to communicate changes within the system. Similar to events they provide a looser coupling and the message provides all the functionality of a C# object if need be. In this post I’ll show you how a message can be created, sent and finally how to register a handler within an object for a defined message.
 
# Setting up the project
 
The demonstration project will be a Windows Phone 8.1 solution. You will have to install the [MVVM Light](http://www.mvvmlight.net/) NuGet package. We will have a simple view containing a label and a button:


    <Page    x:Class="Messenger.MainPage"    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"    xmlns:local="using:Messenger"    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"    mc:Ignorable="d"    DataContext="{Binding Source={StaticResource Locator}, Path=Main}"    Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">    <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center">        <TextBlock Text="{Binding InvokedMessage}" Style="{StaticResource MessageDialogContentStyle}"                   Margin="0,10" />        <Button Content="Invoke Service" HorizontalAlignment="Center" Command="{Binding InvokeServiceCommand}" Width="200" Margin="0,10" />    </StackPanel></Page>


The button triggers a command in the ViewModel, which invokes an update of the service:


    private void InvokeService(){    CanInvokeService = false;    _sampleService.NonBlockingAsyncMethod();}


The service is a async void method which means it can not be awaited. After the delay has passed a message shall be sent any receiver:


    public async void NonBlockingAsyncMethod(){    _invokedCount += 1;    await Task.Delay(3000);    GalaSoft.MvvmLight.Messaging.Messenger.Default.Send(new SampleMessage(_invokedCount));}


So lets see how the message is implemented.

## Creating a message

A message is a POCO (Plain Old CLR Object) that inherits from MessageBase. It is up to the developer how many properties or even functions shall be added to the message. In general a message should be treated as a Data Object.


    internal class SampleMessage:MessageBase{    public SampleMessage(int invokedCount):base()    {        InvokedCount = invokedCount;    }    public int InvokedCount { get; set; }}


For our sample we will simply add a count property that indicates the number of calls the service has been invoked.

# Sending a message

Sending a message is simply done by invoking the **Send** method on the default Messanger, as it is done in the **NonBlockingAsyncMethod** in the **SampleService**:


    GalaSoft.MvvmLight.Messaging.Messenger.Default.Send(new SampleMessage(_invokedCount));


So now all we have to do is register our receivers.

# Subscribing/Registering the message receiver

On the view model we can register a handler that receives the message and updates the label with an according call:


    GalaSoft.MvvmLight.Messaging.Messenger.Default.Register<SampleMessage>(this, ReceiveSampleMessage);




Which is done in the **MainViewModel**.

# Conclusion

Messaging allows to decouple objects that need to communicate with one another. In our sample the service communicates with the ViewModel but does not have any direct reference of the ViewModel and therefore is fully decoupled of it. Messaging can be a great help in writing maintainable code but be advised that though this method does allow to decouple objects it also adds complexity and therefore passing a method as call back function might be sometimes the more straight forward way of communicating to a developer.

You can find the entire code on [GitHub](https://github.com/mallibone/MvvmLightSamples.git).

## References

[Laurent Bugnion](http://www.galasoft.ch/) who is the creator and maintainer of the MVVM Light framework.
