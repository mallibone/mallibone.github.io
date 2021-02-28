---
layout: single
title: "MVVM Light bindings under Xamarin.iOS"
title: MVVM Light bindings under Xamarin.iOS
date: 2015-07-01
tags: ["Xamarin", "iOS", "MVVM Light"]
slug: "mvvm-light-bindings-under-xamarin.ios"
---

In this weeks blog post lets look at how we can enable bindings with [MVVM Light](http://www.mvvmlight.net/) under iOS using a Storyboard with Xamarin. So lets create a simple application that has a label, button and text entry field.

[![MvvmLightiOSBinding](http://www.mallibone.com/posts/files/1d1469c9-88c1-40fb-89b4-03bcadbe54c9.png "MvvmLightiOSBinding")](http://www.mallibone.com/posts/files/dced9eb7-53fb-4f33-a647-69d3578a4c0f.png)

The label will display the previous message which was entered in the text entry field and submitted with the button. The entire UI is designed in the Storyboard designer and then hooked up to the ViewModel in the code behind.

# Wiring up the View to the ViewModel

Under iOS the bindings have to be defined within the code, so for the label that gets updated from the ViewModel the binding looks as follows:


    _textLabelBinding = this.SetBinding (    () => Vm.TheResponse,    () => TextLabel.Text);


The button is wired via a command binding:


    SubmitTextButton.SetCommand("TouchUpInside", Vm.SubmitTextCommand);


When it comes to the text field we have to register ourselfs to an event of the UI control on which we are invoked. Then we pass the value of the text field to our **MainViewModel** i.e. the property.


    _textFieldBinding = this.SetBinding (() => EntryTextField.Text).ObserveSourceEvent ("EditingDidEnd").WhenSourceChanges (() => Vm.TheInput = EntryTextField.Text);




So as you can see there is a bit more work to do for using ViewModels under iOS than under Xamarin.Forms, Android or Windows – but it can be done ![Smile](http://www.mallibone.com/posts/files/219dbe76-6622-4063-a6ad-3be710c22551.png)

# Taking a peek at the ViewModel

The **MainViewModel** is pretty straight forward and does not differ from ViewModels you may use for Windows applications e.g. Universal Windows Apps.


    public class MainViewModel : ViewModelBase{    private string _theInput;    private string _theResponse;    private RelayCommand _submitTextCommand;    public MainViewModel ()    {        Names = new ObservableCollection<string>();        SubmitTextCommand = new RelayCommand (HandleSubmitTextCommand, () => CanExecuteSubmitText);        CanExecuteSubmitText = true;        TheResponse = "Awaiting your input";    }    public string TheInput {        get {            return _theInput;        }        set {            _theInput = value;            RaisePropertyChanged (() => TheInput);        }    }    public string TheResponse {        get {            return _theResponse;        }        set {            if (value == _theResponse) return;            _theResponse = value;            RaisePropertyChanged (() => TheResponse);        }    }    public RelayCommand SubmitTextCommand {        get {            return _submitTextCommand;        }        private set {            _submitTextCommand = value;        }    }    public bool CanExecuteSubmitText {        get;        set;    }    public ObservableCollection<string> Names { get; set; }    private void HandleSubmitTextCommand ()    {        CanExecuteSubmitText = false;        TheResponse = "You just entered: " + TheInput;        CanExecuteSubmitText = true;    }}


When the command is invoked we simply register a handler for which in the case of the sample code is the **HandleSubmitTextCommand** method. You can also use set the **CanExecute** property of an **[ICommand](https://msdn.microsoft.com/en-us/library/system.windows.input.icommand%28v=vs.110%29.aspx)** and it will prevent the Command from re-executing while the handler is still working on a call from the user.

# Conclusion

So as you could see the MVVM Light framework from [Laurent Bugnion](http://www.galasoft.ch/) can be used to implement a MVVM client architecture under iOS using Storybaords. The biggest difference from the standard approach is that the developer has to handle wiring up the bindings and ensure that the bindings are attached to corresponding events in case of reacting to new values from the UI. Other then that the code is the same as you would expect it to be from Windows implementations.

You can find the sample code on [GitHub](https://github.com/mallibone/XamarinIosMvvmLightBindings).

## References

Laurent Bugnions Xamarin [Evolve talk](http://blog.galasoft.ch/posts/2014/10/announcing-mvvm-light-v5-for-windows-and-xamarin/).

Getting started with MVVM Light and Xamarin.Forms [Post](http://www.mallibone.com/category/mvvm+light).
