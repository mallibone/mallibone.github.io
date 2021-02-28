---
layout: single
title: "Using custom fonts in Xamarin.iOS"
title: Using custom fonts in Xamarin.iOS
date: 2018-01-19
tags: ["Xamarin.iOS", "Xamarin"]
slug: "using-custom-fonts-with-xamarin-ios"
---

[![pexels-photo-316465]({{ site.url }}{{ site.baseurl }}/assets/images/059a46f3-4dc4-459a-99b2-80940039e7a9.jpg "pexels-photo-316465")]({{ site.url }}{{ site.baseurl }}/assets/images/aff3840c-e17b-46fc-8aff-e140e6156cdb.jpg)



When developing an app your design might require to use a font that is not available from iOS out of the box. So let’s see how a custom font can be added to ones app and how to use it in a Storyboard or Code.

## Adding the font

Assuming you already have the font, note that iOS supports fonts that are stored in TTF or OTF formats. Custom fonts are copied into the Resource folder of your iOS project.

[![image of ios project, showing the custom font file in a subfolder named fonts in the resoucres folder]({{ site.url }}{{ site.baseurl }}/assets/images/db01b14b-0e86-4165-b13f-46ebf788fa8c.png "image of ios project, showing the custom font file in a subfolder named fonts in the resoucres folder")]({{ site.url }}{{ site.baseurl }}/assets/images/c8f01385-4c27-49e4-98d2-97c35375a369.png)


> Ensure that the properties of the font file are set to *BundleResource.*


Creating the Fonts subfolder is optional. But since the Resources projects tends to collect a couple of items in larger projects. Let’s tidy things up from the start ![Smile]({{ site.url }}{{ site.baseurl }}/assets/images/dc3de230-94db-4cfe-96c7-0b06e0819e9a.png)

To use the font we will have to update the <font face="Consolas">Info.plist</font> file with the following lines:

[![image]({{ site.url }}{{ site.baseurl }}/assets/images/7e608fe7-adf4-4cc4-a763-826e8af7db43.png "image")]({{ site.url }}{{ site.baseurl }}/assets/images/471a87a1-b7e4-4e98-8f20-44fbd12b05d4.png)


> Note: At the time of writing in Visual Studio 15.5.4 you will have to open the <font face="Consolas">Info.plist</font> file in an XML Editor. For this right click the file and select “*Open with…*” then choose the *XML (Text) Editor*. In Visual Studio for Mac the GUI Info.plist editor supports editing the source directly.


## Using custom fonts in Storyboards

After adding for example a <font face="Consolas">UILabel</font> to the storyboard, select it. In the options click on the font and choose the custom font.

[![Storyboard]({{ site.url }}{{ site.baseurl }}/assets/images/e8ed476c-8894-4371-9343-5682c4c0950d.png "Storyboard")]({{ site.url }}{{ site.baseurl }}/assets/images/c4eb5723-7cb1-45d5-8aa3-5d48676b6b19.png)

## Using custom fonts in Code

In code behind using a custom font is pretty straight forward. For example in a label we can set the Font attribute as follows:

[![image]({{ site.url }}{{ site.baseurl }}/assets/images/554c2cd5-4da7-469d-86bf-29b0ebd5cca2.png "image")]({{ site.url }}{{ site.baseurl }}/assets/images/155a6e61-59bd-4e43-b623-999a8ebccb03.png)

The result when running app with the combined storyboard and new label is:

[![Custom Font Screenshot]({{ site.url }}{{ site.baseurl }}/assets/images/91449354-238c-48da-a8c2-067bc367151c.jpg "Custom Font Screenshot")]({{ site.url }}{{ site.baseurl }}/assets/images/b08b6923-d09f-4572-9287-654f17f640e9.jpg)

## Conclusion

In this blogpost we went through the steps that have to be taken to add a custom font. Note that fonts are loaded during the start up of the app and if you go bonkers with them you might notice some performance impact.

You can find a small sample on [GitHub](https://github.com/mallibone/XamarinCustomFonts.git "Sample GitHub Repo"). For setting the constraints in the code defined UI [PureLayout.Net](https://mallibone.com/post/purelayout-for-xamarin-with-purelayoutnet) was used.
