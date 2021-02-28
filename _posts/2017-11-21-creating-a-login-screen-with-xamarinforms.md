---
layout: single
title: "Creating a login screen with Xamarin.Forms"
title: Creating a login screen with Xamarin.Forms
date: 2017-11-21
tags: ["Xamarin.Forms", "Xamarin"]
slug: "creating-a-login-screen-with-xamarinforms"
---

[![pexels-photo-277803](https://mallibone.com/posts/files/1b20fcbe-941f-4fef-beea-afcdf4bc9e7c.jpg "pexels-photo-277803")](https://mallibone.com/posts/files/c85b8f48-62da-4286-a0e2-fb749466beb6.jpg)

When writing an app that only allows access to certain or all parts of the app when a user is logged in requires a login screen which can be presented to the user at every screen in the app I.e. as soon as he is required to login or re-login.

In this post you will see how to create a view that can be used to enter the username and password. Further we will look at how we can use this screen regardless of which screen is currently displayed to the user.

# Writing a login view

Let’s consider a view as follows.

[![LoginScreen](https://mallibone.com/posts/files/78932983-cb59-4a1c-b4d4-4610b521e5f4.gif "LoginScreen")](https://mallibone.com/posts/files/763c9659-1cfd-427f-93f4-42055ed22fec.gif)

It requires a username and password, the user can confirm his entry by hitting a button which will validate his entry.

<script src="https://gist.github.com/mallibone/4d003cf3192c6702ddec263c0f0d053d.js"></script>

After entering a correct login the user will be presented with the apps content. In our case a simple screen containing a logout button.

<script src="https://gist.github.com/mallibone/cabfd841a03e6fcb83fc8823607cccd5.js"></script>

So far so good, but how will we ensure that the user only sees the content after she has logged in? How do we prevent the user form simply dismissing the page? Well let’s dive into this topic as next.

# Login sites and Navigation

So implementing the view and even the business logic of a login site are quite straight forward but how do can we pop up the login view whenever the user is required to authenticate himself I.e. has to re-authenticate? And how do we prevent him from leaving the screen. Luckily all this can be solved by using the modal navigation backed in to [Xamarin Forms](https://developer.xamarin.com/guides/xamarin-forms/application-fundamentals/navigation/modal/ "Xamarin Forms Modal Navigation Documentation"). Utilizing a simple navigation service from a [previous post](https://mallibone.com/post/a-simple-navigation-service-for-xamarinforms "Simple Xamarin.Forms Navigation Service"), we can ensure invoke the navigation to the login page from any page or even when resuming the app or on start up:  
<script src="https://gist.github.com/mallibone/02e2db97dec491e8c63f5a8951950179.js"></script>

Using modal navigation with a simple  navigation service allows us to implement a login dialog that can be pop over any view currently displayed and return the user to the sensitive content once he is properly authenticated.

### 

### No way back

Though modal pages do not provide the user with a software button in the navigation bar to return to the previous page. The dedicated OS back button on Android and Windows 10 can still be used by the user. To ensure the user can not leave the page via the OS button the <font face="Consolas">OnBackButtonPressed</font> method has to be override as follows:

<script src="https://gist.github.com/mallibone/82b74da9c16c436bc7851d60d1a554ea.js"></script>

### Instant Login View navigation

When requiring the user to log in on the initial start up of the app or resume. Often it is desired to simply overlay the login view over the page that should be displayed when the user is successfully authenticated. Want we do not want is the user to ever see the landing page before being logged in. To achieve this effect we can want to insert our check before the page is displayed to the user and then navigate without animation to the login page:

<script src="https://gist.github.com/mallibone/1f857fd1787f02539f7888f37a69879c.js"></script>

# Improve user input experience

When creating the user login what would be nice is if the user could simply navigate from the username to the password field via the enter button on the keyboard. After completing the password it could directly verify the username and password when pressing enter. This can be done in the code behind of the login view as follows:

<script src="https://gist.github.com/mallibone/1b938d008acd18d3e7f70534618bc720.js"></script>

# Conclusion

In this blog post we saw how we can create a view for a login page and also implement the logic behind it. Through modal navigation we saw how we can “capture” a user on a view and prevent him from leaving the view before he has entered some valid credentials.

We then improved the UX by showing the login page instantly when resuming or starting the app. Plus improving the entry of username and password.

You can find a sample of the login view on [GitHub](https://github.com/mallibone/LoginViewSample "GitHub link to sample repo").
