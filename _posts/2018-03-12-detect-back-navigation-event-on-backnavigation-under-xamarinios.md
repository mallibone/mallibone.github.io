---
layout: single
title: "Detect back navigation in your Xamarin.iOS apps"
title: Detect back navigation in your Xamarin.iOS apps
date: 2018-03-12
tags: ["Xamarin.iOS"]
slug: "detect-back-navigation-event-on-backnavigation-under-xamarinios"
---

[![Traffic lights at the crossroads](https://mallibone.com/posts/files/942a2190-a407-4e7a-b911-5fef2a98897a.jpg "Traffic lights at the crossroads")](https://mallibone.com/posts/files/8b53bc17-00f8-49dd-9a1c-0f3d73276899.jpg)

In this post we look at getting notified when the user chooses to navigate back. If you have to be in control of your navigation under iOS I strongly recommend you check out using a modal navigation. But what if you simply want to track when the navigation hits the back button. Or uses the swipe gesture to go back to a previous page.

While there is no direct event, there a few ways to achieve this.

# Custom NavigationController

One way is to subclass the `UINavigationController`:


    public class AwareNavigationController : UINavigationController
    {
        public event EventHandler PoppedViewController;
    
        public AwareNavigationController() : base() { }
        public AwareNavigationController(UIViewController rootViewController) : base(rootViewController) { }
        public AwareNavigationController(IntPtr intPtr) : base(intPtr) { }
        public AwareNavigationController(NSCoder coder) : base(coder) { }
        public AwareNavigationController(NSObjectFlag t) : base(t) { }
        public AwareNavigationController(string nibName, NSBundle bundle) : base(nibName, bundle) { }
        public AwareNavigationController(Type navigationBarType, Type toolbarType) : base(navigationBarType, toolbarType) { }
    
        public override UIViewController PopViewController(bool animated)
        {
            PoppedViewController?.Invoke(this, null);
            return base.PopViewController(animated);
        }
    }


You can then start using the new navigation controller:


    new AwareNavigationController(new ContainterViewController());


And hook up to it’s event:


    public override void ViewWillAppear(bool animated)
    {
        base.ViewWillAppear(animated);
        ((AwareNavigationController)NavigationController).PoppedViewController += ViewControllerPopped;
    }
    
    
    private void ViewControllerPopped(object sender, EventArgs e)
    {
        Console.WriteLine("Going back Shell");
    }


Use this approach if you want to register the navigation at a single point and provide this information as an event or even message. If you are writing a navigation service this approach will most probably be your best choice.


> A couple of side notes when using this approach. Make sure to deregister the event handler. You will want to deregister the event handler in the ViewWillDisappear method. If you try to do this at a later stage I.e. ViewDidDisappear the navigation controller reference will already be null.


# WillMoveToParentViewController

Alternatively to creating a custom navigation view controller you can override the following method in your `ViewController`:


    public override void WillMoveToParentViewController(UIViewController parent)
    {
        base.WillMoveToParentViewController(parent);
        if(parent == null) Console.WriteLine("Going back Shell");
    }


If the passed in parent view controller is null, the view is being removed from the view stack.  Note that if you have child view controllers you will have to extend the method above or the children will not be notified:


    public override void WillMoveToParentViewController(UIViewController parent)
    {
        base.WillMoveToParentViewController(parent);
    
        if (parent == null)
        {
            var childVCs = ChildViewControllers;
            foreach(var childVC in childVCs)
            {
                childVC.RemoveFromParentViewController();
            }
    
            Console.WriteLine("Method override: Going back Shell");
        }
    }


I would recommend this approach when you are only need to detect the navigation in one or two `ViewController`'s I.e. but do not require this event globally in the app.

# Not recommended approach

If you have stumbled over the suggestion to simply check the attribute `IsMovingFromParentViewController` in the `ViewWillDisappear` / `ViewDidDisappear`, be aware that this is not recommended. While it might work at first, as soon as you need to detect the back navigation in a child view controller this attribute will always be set to true. Even if navigating to another view controller and not backwards.


    public override void ViewDidDisappear(bool animated)
    {
        base.ViewWillDisappear(animated);
        // not recommended!
        if(IsMovingFromParentViewController) Console.WriteLine("Going back Shell");
    }


Since one can’t forbid a `ViewController` to be used as child view controller, I would strongly recommend to stay away from this approach.

# Conclusion

In this post the different ways of getting notified of a back navigation in progress have been shown. If you have to block the user from navigating back I strongly recommend you use a modal navigation in the first place.

Find a small sample app on [GitHub](https://github.com/mallibone/iOSNavigation).

Know another option not mentioned here? Or have any thoughts on the approaches. Let me know by posting a comment.
