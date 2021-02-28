---
layout: single
title: "A simple Navigation Service for Xamarin.Forms"
title: A simple Navigation Service for Xamarin.Forms
date: 2017-09-25
tags: ["Xamarin", "Xamarin.Forms"]
slug: "a-simple-navigation-service-for-xamarinforms"
---

[![pexels-photo-297755](https://mallibone.com/posts/files/7688f273-4350-4f14-b896-a4340628f48d.jpg "pexels-photo-297755")](https://mallibone.com/posts/files/d088738a-d705-4b86-bb1a-571a2bc4cbff.jpg)

If you have ever written a Xamarin Forms app and wanted to navigate from within a View Model to another page. Then you have come over the issue that the navigation logic usually resides in the view. Since every best practice blog post, course and video tells us that the view and business/control logic should be separate. We are often left with the question how to integrate a View Model based navigation.

## What we want to do

The goal of this post is to show you how we can initiate a navigation call from the View Model. Without adding dependencies to your view or writing call back methods.

## Navigation 101

In Xamarin Forms there are two fundamental ways of navigating between pages:

- Push
- Modal


**Push** navigation is the most straight forward way for navigating in an app. The main page in the <font face="Consolas">App.xaml.cs</font> is a <font face="Consolas">NavigationPage</font> which contains the main page displayed to the user:

<script src="https://gist.github.com/mallibone/3e2df0a87c785ad7e8af9505f6796f6a.js"></script>

When navigating to the next page, the page gets pushed onto the navigation stack. The user can then navigate back via Screen (iOS & Windows) or Hardware button (Android & Windows). Navigating back will dispose the current page, pop a page from the navigation stack and display it to the user.

When navigating to a page using **Modal**. The target page is displayed in a separate navigation context. This results in the behaviour that the user is no longer displayed with a software back button from the OS. The back navigation call differs from Push. Thus it requires invoking a different method of the Navigation Page.


> Modal navigation is a great option to capture the user on a view. This is not generally what we intend to do with a user. But if the user has to login or the users consent is required (are you really sure...) filling out a form. Then modal navigation allows keeping the user on a page and handling the navigation via custom defined logic.


**Note:** Modal navigation is an iOS concept in it’s origins. Under Android and Windows the user has system buttons to navigate the app. If the user should not be able to leave the page and return to the previous page I.e. user has to login first. The developer will have to override the <font face="Consolas">OnBackButtonPressed</font> with a return true in the code behind of the view.

### Navigation TL;DR;

We have two navigation methods in Xamarin.Forms. These are Push and Modal. Each has different ways of invoking the navigation. Which also applies to the back navigation.

## Using a navigation service

In general the navigation service will have to contain a Navigation Page to handle the navigation stack. Further it needs to offer a Method for navigating to a page via push or modal navigation and allow navigating back.

<script src="https://gist.github.com/mallibone/5de9e9d0f6ff76c40305a1a5cd8e963e.js"></script>

<script src="https://gist.github.com/mallibone/15b2279599e8b589713ebd226dbf5957.js"></script>

The **initialisation** of the navigation service can now be done in the <font face="Consolas">App.xaml.cs</font> by registering all the views with a corresponding key.

<script src="https://gist.github.com/mallibone/9135329987612d9904e25c143f67687a.js"></script>

In a view model we can now navigate to a page by providing the service with the key of the target page. If required we can also pass in a parameter. Since the parameter is of type object there are no limitations there.

<script src="https://gist.github.com/mallibone/7cc74a01ab76c709567ebb63108b3d09.js"></script>

As mentioned before **modal** navigation leaves the current navigation stack and displays the page on it’s own. If we want to navigate from a modal root page via push to a child page. The modal root page has to be in a Navigation Page itself. This is what the navigation service does automatically. It further removes the navigation header of the modal root page.

**Back** navigation checks what back navigation is intended for the current page. A root page in a navigation stack is either the root page of the app or a root page in a modal navigation stack. In either case it will choose the appropriate back navigation method.  
If the page is not the root page we simple navigate back to the previous page that was pushed onto the navigation stack.

## Conclusion

Using the navigation service described in this blog post. It is possible to navigate from the view model to other pages. Doing so we do not add dependencies to the view stack e.g. Xamarin Forms. Nor do we have to write cumbersome call back methods for each navigation action over and over again.

I would like to thank [Laurent Bugnion](http://www.galasoft.ch/ "Laurent's website") at this point who laid down the base of the described navigation service above in with his initial implementation of a Navigation Service for Xamarin Forms when using MVVM Light.

You can find a running sample on [GitHub](https://github.com/mallibone/XamarinFormsNavigationService "Basic Sample App").
