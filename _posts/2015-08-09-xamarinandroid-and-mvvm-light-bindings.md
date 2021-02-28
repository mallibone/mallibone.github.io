---
layout: single
title: "Xamarin.Android and MVVM Light bindings"
title: Xamarin.Android and MVVM Light bindings
date: 2015-08-09
tags: ["MVVM Light", "Xamarin", "Android"]
slug: "xamarinandroid-and-mvvm-light-bindings"
---

[![androidMvvmLight](http://mallibone.com/posts/files/e65c27c2-091e-4dd9-8a3b-118635050893.png "androidMvvmLight")](http://mallibone.com/posts/files/34a6e965-07f2-4694-9ca8-e295cac95191.png)
 
This week we’ll have a look at how we can use the MVVM pattern on Xamarin.Android using the MVVM Light framework from [Laurent Bugnion](http://www.galasoft.ch/).
 
# Creating the View
 
For this sample lets create a basic entry view with an **EditText**, **Button **and **TextView** to display the entry that was submitted via the button.


    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="vertical"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent">
        <EditText
            android:id="@+id/MessageText"
            android:hint="@string/MessageTextHint"
            android:layout_marginTop="10dp"
            android:layout_marginBottom="10dp"
            android:layout_width="fill_parent"
            android:layout_height="wrap_content" />
        <Button
            android:id="@+id/SubmitMessage"
            android:layout_marginBottom="10dp"
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:text="@string/SubmitMessage" />
        <TextView
            android:id="@+id/SubmittedMessage"
            android:layout_marginBottom="10dp"
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:text="@string/SubmittedMessageText" />
    </LinearLayout>


As you can observe this is a standard **LinearLayout** containing default Android controls. With MVVM Light we do not have the option as in MVVM Cross to set the binding code directly in the AXML but more how it’s done later. So lets setup the corresponding ViewModel.

# Creating the ViewModel

With in the ViewModel we need a property to store the entered message, another one to expose an ICommand which we can bind to the button and finally a third property where we can store the message that was entered at the time the button is being pushed:


    public class MainViewModel : ViewModelBase
    {
        private string _message;
        private string _previousMessage;
    
        public MainViewModel()
        {
            MessageCommand = new RelayCommand<string>(SubmitMessage);
        }
    
        public RelayCommand<string> MessageCommand { get; private set; }
    
        private void SubmitMessage(string message)
        {
            PreviousMessage = message;
        }
    
        public string PreviousMessage
        {
            get { return _previousMessage; }
            set
            {
                _previousMessage = value;
                RaisePropertyChanged(propertyName: nameof(PreviousMessage));
            }
        }
    
        public string Message
        {
            get { return _message; }
            set
            {
                _message = value;
                RaisePropertyChanged(propertyName: nameof(Message));
            }
        }
    }


If you have been using ViewModels before you will not notice anything out of the ordinary with this code which means you could reuse this ViewModel for a WPF, Windows 8, UWP apps or even Xamarin.iOS i.e. Xamarin.Forms app. (This is good ![Winking smile](http://mallibone.com/posts/files/ed272fe7-d8a4-4e5d-b5b4-c3dab96e2dcf.png))

## Setting up the Locator

Another step we want to make is to ensure that the ViewModel and any future services will be created and setup over the Inversion of Control container. MVVM Light provides us the SimpleIoC container which usually is setup in the ***ViewModelLocator.cs*** class. Currently we only use one view model so that is all we have to setup:


    public class ViewModelLocator
    {
        public ViewModelLocator()
        {
            ServiceLocator.SetLocatorProvider(() => SimpleIoc.Default);
    
            SimpleIoc.Default.Register<MainViewModel>();
        }
    
        public MainViewModel Main
        {
            get
            {
                return ServiceLocator.Current.GetInstance<MainViewModel>();
            }
        }
    
        public static void Cleanup()
        {
            // TODO Clear the ViewModels
        }
    }


As there is no fixed starting point in Android we have not **Main** method or such to configure our container on startup. I personally like the idea of a lazy container such in one that is only created when it is really used. So lets create an ***App.cs*** class in the root of our Android project. In this class we will instantiate the locator and provide access to this one instance:


    public static class App
    {
        private static ViewModelLocator _locator;
    
        public static ViewModelLocator Locator => _locator ?? (_locator = new ViewModelLocator());
    }


Now that we have all the basics covered lets move on to the juicy part of wiring up the bindings.

# Setting up the bindings

We have to manually set the bindings in the activity for the corresponding view. In this sample app we will do it in the ***MainActivity.cs***. First we wire up the **EditText:**


    _messageBinding = this.SetBinding(() => EditMessage.Text, BindingMode.TwoWay);


The command with the button:


    MessageButton.SetCommand("Click", Vm.MessageCommand, _messageBinding);


And the **TextView **to the ***PreviousMessage*** property:


    _textViewBinding = this.SetBinding(() => Vm.PreviousMessage, () => PreviousMessage.Text);


Now whenever we edit a text and select the button the text will appear on the **TextView** bellow.

# Conclusion

In this post we saw how we can create a simple form application and write & read from a ViewModel. We further saw how we can setup the dependency injection module for a Xamarin.Android application and ensure that even if the app should be started with a different activity the container is setup correctly.

You can find the entire little sample project up on [GitHub](https://github.com/mallibone/MvvmLightSamples/tree/master/Android/MvvmLightBindings.Droid).

## References

MVVMLight [flowers sample](http://mvvmlight.codeplex.com/SourceControl/latest#Samples/Flowers/) by Laurent Bugnion.
