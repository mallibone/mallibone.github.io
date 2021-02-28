---
layout: single
title: "Xamarin.Android remove previous Activity from navigation stack"
title: Xamarin.Android remove previous Activity from navigation stack
date: 2015-08-23
tags: ["Android", "Xamarin"]
slug: "xamarinandroid-remove-previous-activity-from-navigation-stack"
---

[![Android logo portraied in stack of squares with rounded edges.](http://mallibone.com/posts/files/8eef1a9f-001d-4107-a409-33d0ce30bfed.png "Android logo portraied in a navigation stack.")](http://mallibone.com/posts/files/f10c920f-8f3e-4671-a6ba-19ce7e47c355.png)
 
In this weeks blog post we will look at how we can remove an Activity from the Navigation Stack. This can be very useful if your are implementing an extended splash screen and don’t want to navigate back to it when the user selects the back button.
 
For this we will first create a welcome screen which will navigate to the main activity, in the process it will remove the welcome screen i.e. the activity behind from the navigation stack allowing the user to quit the app from the main screen.
 
# Creating a welcome screen
 
For the welcome screen we will create a normal linear layout page, so nothing special here. For navigating to the main activity we create an intent and configure it to create the target activity in a new task:


    var intent = new Intent(this, typeof(MainActivity));
    intent.SetFlags(ActivityFlags.NewTask);
    StartActivity(intent);


After navigating to the main activity we call the **Finish** method which will terminate the **WelcomeActivity** and remove it from the navigation stack.


    Finish();


We can place the code in the on created method as such:


    [Activity(Label = "RemoveActivityFromNavigationstack", MainLauncher = true, Icon = "@drawable/icon")]
    public class WelcomeActivity : Activity
    {
        protected override async void OnCreate(Bundle bundle)
        {
            base.OnCreate(bundle);
    
            SetContentView(Resource.Layout.Welcome);
    
            // Do some meaninthing stuff here, or just wait for 3 seconds...
            await Task.Delay(3000);
    
            var intent = new Intent(this, typeof(MainActivity));
            intent.SetFlags(ActivityFlags.NewTask);
            StartActivity(intent);
            Finish();
        }
    }



> Note that if the ActivityFlags.NewTask is not set, the targeted activity will also stop running and the user will only see a blank screen of the app.


# Conclusion

We saw how to setup navigation to remove an activity from the navigation stack. This method is ideally used when removing an extended splash screen or the like from the navigation stack allowing the user to close the app from the root screen.

You can find the entire sample on [GitHub](https://github.com/mallibone/XamarinAndroidSamples/tree/master/RemoveActivityFromNavigationstack).
