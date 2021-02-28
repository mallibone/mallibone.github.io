---
layout: single
title: "Setting the back button in Xamarin.iOS"
title: Setting the back button in Xamarin.iOS
date: 2018-03-21
tags: ["Xamarin.iOS"]
slug: "setting-the-back-button-in-xamarinios"
---

Just stumbled over this one in a recent endeavour. The default behaviour of iOS is to use the title of the previous navigation controller as the text for the back button.

[![BackButtonNotSet](https://mallibone.com/posts/files/e0192a53-4c4e-4cb8-b13e-f9e15e1a18d0.gif "BackButtonNotSet")](https://mallibone.com/posts/files/692cb584-3429-4403-b514-7913eed36bea.gif)

If you want a different text such as back you might find this [recipe](https://developer.xamarin.com/recipes/ios/content_controls/navigation_controller/change_the_back_button/ "Change the Back Button") from Xamarin which works. But it does feel like a hack and it requires you to adopt this approach in every view.

Another approach is to set the `NavigationItem.BackBarButtonItem` attribute in the view controller:


    NavigationItem.BackBarButtonItem = new UIBarButtonItem {Title = "Back"};


The set text will be displayed as the back button of the ***next*** view controller. In other words previous view controller always sets the back button text of the following.

[![BackButtonSet](https://mallibone.com/posts/files/e7130565-946b-4bcd-99f8-d497a35d5b45.gif "BackButtonSet")](https://mallibone.com/posts/files/e99b209f-99a2-4abb-9021-53e3a01c9e3a.gif)

If you are using a central point to generate your views e.g. a navigation service or factory you can apply this to every view you create. Which makes this approach a lot less error prone.

If you are using storyboards you have a further option.  The title can be set directly in the designer under Navigation Item, Back Button.

[![Set back button title in Storyboard editor](https://mallibone.com/posts/files/26e44e5d-e2f8-463c-8f0f-58a3e10a4f96.png "Set back button title in Storyboard editor")](https://mallibone.com/posts/files/7c762c07-0ddc-4542-8ad6-48ebd0aec1da.png)


> Note this option is only available when you choose the (no longer recommended) push navigation for your segue.


## Conclusion

In this post we saw how we can set the title of the back button. The title of the back button is always set in the preceding view controller.

Using a storyboard the title can be set directly in the storyboard designer.

You can find an entire little app sample on [GitHub](https://github.com/mallibone/BackNavigationButton).
