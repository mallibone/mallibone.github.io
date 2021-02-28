---
layout: single
title: "Beta-Testing on iOS by simply sharing a link"
title: Beta-Testing on iOS by simply sharing a link
date: 2016-04-03
tags: ["Xamarin.Forms", "Xamarin", "iOS", "testing"]
slug: "simply-share-a-link-which-allows-user-to-download-your-app-on-their-ios-device"
---

[![Blog Title Image showing ios logo and Beta Testing with a ruber stamp effect.](https://mallibone.com/posts/files/c5b6b331-1d73-4bc4-aa40-29862f1c6554.png "Blog Title Image showing ios logo and Beta Testing with a ruber stamp effect.")](https://mallibone.com/posts/files/d87e3699-f65c-49da-ada2-9958080b838e.png)
 
When developing your latest and greatest (Xamarin) iOS app there will come a time when you are ready to ship the app to users for some beta testing. This allows you to get some feedback before shipping it to the official store. First thing needed is the application package also known by it’s file ending the *IPA*. If you are using Xamarin.iOS you can find a more details on building the IPA and setting up the certificates [here](https://developer.xamarin.com/guides/ios/deployment%2c_testing%2c_and_metrics/app_distribution/ipa_support/ "Instructions on how to distribute an IPA with Xamarin."). Assuming that you have an IPA ready for distribution the most common instruction found on the internet is to use iTunes to side load the app. Now this has a bit of an indirection to it, since the user must be sitting in front of a Laptop which allows him to install the app. In case of a company Laptop with IT policies this may end up being rather complicated. But there is an easier way, you can send a link which the user can select on an iOS Device which will install the app directly onto the device.
 

> For the user to be able to use the app the certificates have to be correctly configured. If not, the user will be able to install the app but unable to launch it, which shows itself with by either closing right after the splash screen or a greyed out app icon.

 
# 
 
# Sharing your app via link
 
Now since it would be way to easy to just simply send a link to a file share which allows you to install the app, we jump through an additional hoop to comply with Apples “process”. We need to send the user(s) a link to a plist file that includes a link to the actual app i.e. IPA file:
<script src="https://gist.github.com/mallibone/a56cc79479d0944af22c470477d85272.js"></script> 
Now lets go through this step by step:
 
1. Upload IPA to the file share of your choosing and get a public URL for the IPA
2. Edit the plist template above to match the requirements of your app (note the comments)
3. Upload the plist file to Fileshare and get a public URL to the plist-File
4. Send link to plist-File to whomever you would like to have the app installed

 
Since I was met with a confused look the first time I explained this to a co-worker see the graphic bellow and just think of the link to the plist file as a pointer of a pointer ![Winking smile](https://mallibone.com/posts/files/fc4d5d0b-d8de-4c92-a7e1-702160dd40fc.png):
 
[![Shows image where beta users taps on a link that is pointed towards the plist file (lying on a file share). The plist itself contains a link to the ipa file which will get installed on the users device.](https://mallibone.com/posts/files/543ba9b7-4e4c-4e4b-a88f-cd8b7d87db1a.png "Shows image where beta users taps on a link that is pointed towards the plist file (lying on a file share). The plist itself contains a link to the ipa file which will get installed on the users device.")](https://mallibone.com/posts/files/3ea6c8ea-83b0-4291-97c0-8072c5ba8f07.png)
 
This is a super easy way to share your app with testers. Then again these steps have to be performed for each update of the software and there is no user monitoring included with this approach. So you can not see if the app crashed nor can you know if a user already installed the latest version of the app. Or simply inform the user while he is using the app that there is an update. Now all this can be included in an app but it requires quite some effort to do so and then again we are talking about code that is only used during beta testing. So if you are developing a larger app with a longer lifetime or multiple beta releases you might want to consider a third party service e.g. [Hockey App](http://hockeyapp.net/features/ "link to hockey app website").
 
# Conclusion
 
Setting yourself or your company up for running beta tests for iOS is as easy as sharing a link to a file (with some overhead for the publisher). No additional accounts needed or third party services. You could even share this thing from a NAS. That being said I would recommend you check out Hockey App as they do provide a free tier for up to [two apps for free](http://hockeyapp.net/pricing/#business "Link to pricing page of hockey app") (at time of writing) and the process of adding it to your app is a couple of lines of code and a NuGet package.
 
HTH
