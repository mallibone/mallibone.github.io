---
layout: single
title: "Universal Windows Platform (UWP) apps adapting to multiple devices"
title: Universal Windows Platform (UWP) apps adapting to multiple devices
date: 2016-01-31
tags: ["Windows", "UWP", "Mobile", "Windows 10"]
slug: "unviersal-windows-platform-uwp-apps-adapting-to-multiple-devices"
---

[![Overview of UWP Apps Platforms]({{ site.url }}{{ site.baseurl }}/assets/images/21b42d15-6444-44a7-9c0b-0d71b0dbd43d.png "Overview of UWP Apps Platforms")]({{ site.url }}{{ site.baseurl }}/assets/images/3d3c67f4-2dee-461d-ac8b-c7ae9adc1569.png)
 
With UWP apps it has become easier then ever to write a single app that will run natively on all Windows 10 devices be it a phone, tablet, PC, Xbox et. al. Now we don’t just want our apps to be able to run on a device we want them to shine while they are running. In this blog post we will look at the basics on how to adopt to different screen sizes within your UWP app.
 

 
# The devices and screen sizes
 
UWP runs on many different platforms some might not even come with a screen by default e.g. Internet of Things (IoT) devices or others that require a unique UI such as [HoloLens](https://www.microsoft.com/microsoft-hololens/en-us) which should be available for a few lucky developers later this year. Now while those platforms are very interesting in certain scenarios a wide array of apps will most probably be developed for mobile devices such as phones, tablets and phablets aka BAPHs (Big a\*\* Phones). The unique thing about the UWPs in comparison to Android and iOS apps is that you apps will not only run on mobile devices but on desktop machines, laptops and even larger screens such as the new [Surface Hub](https://www.microsoft.com/microsoft-surface-hub/en-us).
 
[![Windows 10 Devices]({{ site.url }}{{ site.baseurl }}/assets/images/6631697a-1134-4bf8-9cb5-fc77dc54effa.png "Windows 10 Devices such as Phone, Tablet, Laptop and Surface Hub")]({{ site.url }}{{ site.baseurl }}/assets/images/c23aef10-6de3-458c-b1db-881fd1913e20.png)  
Now write once run everywhere and look like crap is easy to do. But the great benefit of creating a UWP app is that you can adopt to the different screen sizes and even adopt to different user input such as touch vs mouse and keyboard. But before we dive into the details of how we can write responsive UIs with XAML lets have a look at some basic layout rules.
 
## Devices and screen sizes
 
A general split up of the devices can be made as follows.
 

  |  **Category** |  **Screen size** |  **Width in Effective Pixels** |  **Inputs** |
| --- | --- | --- | --- |
 |  Phone |  4” to 6” |  340+ |  Touch & Voice |
 |  Phablet |  6+” to 7” |  720+ |  Touch & Voice |
 |  Tablet |  7” to 13.3” |  1024+ |  Toch, Stylus, External keyboard\*, Mouse\*, Voice\* |
 |  PCs and Laptops |  13” and greater |  1024+ |  Mouse, Keyboard, Touch\*, Gamepads |
 |  Surface Hub Devices |  55” and 84” |  1024+ |  Touch, Pen, Voice, Keyboard, remote Touchpad |

 

> \*: occasionally

 
# Basic layout guides
 
Since UWP apps are built from the ground up to run on all kinds of different devices i.e. form factors the platform comes with many helpers that make life easier to design for multiple platforms. One basic concept are effective pixels. Effective pixels differ from actual physical pixels so 24 effective pixels on a phone will scale up to a larger physical appearance on a larger screen. The scaling of effective to physical pixels is all done by the framework and does not require any special attendance by the developer i.e. designer.
 
[![Image showing the diferent scale factors accroding to the device.]({{ site.url }}{{ site.baseurl }}/assets/images/5f0ebba0-6b8c-482f-bb8d-0af2291338fb.png "Image showing the diferent scale factors accroding to the device.")]({{ site.url }}{{ site.baseurl }}/assets/images/43a725a6-9bfe-44e3-9b80-5cfee8dc144f.png)
 

> Note: Often small screens do have very large Dots Per Inch ([DPI](https://en.wikipedia.org/wiki/Dots_per_inch "DPI explanation on Wikipedia")), this is also handled by the platform so even if you are running on a small device with a cheap display i.e. low DPI the text will still be readable as clearly as on a higher end display with more pixels i.e. a higher DPI.

 
The scaling is based on multiple of fours (4), so when defining layouts make sure they are a multiple of four e.g. 16. This will ensure that the scaling will not lead to strange effects. When using icons, rather than using images try to use a font (for example the new *Segoe MDL2 Assets*) that provides the icon you are looking for (or close enough). Though Windows 10 does provide a mechanism that chooses between different resolutions of an image according to the screen size, a font will always scale without any further effort.
 
# Implementing responsive layouts that adapt to the available screen space
 
When working with different resolutions there are multiple layout adjustments that can be performed to improve general User Experience (UX) and – Interaction. UWP offer a variety of ways to adjust the layout of an app according to the available screen space.
 
1. Changing layout of elements on a page according to form factor and available screen space
2. Reflow e.g. adding text columns when the screen size increases
3. Reveal information/functionality depending on the device and it’s resolution
4. Replace elements such as navigation bars with fly out menus to accommodate smaller screen estate
5. Change screen flow e.g. of master detail sites

 
So lets dive into how we can change the layout of UI elements according to the screen space.
 
## Responsive Layout
 
[![ResponsiveDesign]({{ site.url }}{{ site.baseurl }}/assets/images/fe384d1d-1ab4-4b6a-b806-652d5c57b0f4.png "ResponsiveDesign")]({{ site.url }}{{ site.baseurl }}/assets/images/bcf7f3de-5fae-4f90-94c3-4d6bc3ea4da5.png)
 
One new feature of UWP apps is the possibility to adapt the layout of the controls according the available screen size i.e. the effective pixels width of the app windows. This lets the UI adapt to different screen sizes e.g. phones and tablets but also lets the UI adapt to the size a Window has on a desktop.
 
Lets assume you have the following screen flow and layout in your app:
 
[![Shows app running with fewer than 720 effective pixels.]({{ site.url }}{{ site.baseurl }}/assets/images/d44c4d9b-7ec5-472d-838c-d97a946a2215.png "Shows app running with fewer than 720 effective pixels.")]({{ site.url }}{{ site.baseurl }}/assets/images/03704f3a-7289-4144-9368-a3850cb9f111.png)
 
The basic Layout of the app looks as follows:


    <RelativePanel>
        <Rectangle x:Name="Image" Width="80" Height="80" Fill="Gray" Margin="8"></Rectangle>
        <Rectangle x:Name="TextLine1" Height="16" Width="200" Fill="Gray" RelativePanel.RightOf="Image" RelativePanel.Below="" Margin="8,8,8,4"/>
        <Rectangle x:Name="TextLine2" Height="16" Width="200" Fill="Gray" RelativePanel.RightOf="Image" RelativePanel.Below="TextLine1" RelativePanel.AlignLeftWith="TextLine1" Margin="8,8,8,4"/>
        <Rectangle x:Name="TextLine3" Height="16" Width="168" Fill="Gray" RelativePanel.RightOf="Image" RelativePanel.Below="TextLine2" RelativePanel.AlignLeftWith="TextLine1" Margin="8,8,32,4"></Rectangle>
    </RelativePanel>


**Note:** that rendering the items in a <font face="Consolas">RelativePanel</font> allows the elements within i.e. the Rectangle to align themselves relative to one another. This is a new layout feature available in Windows 10 UWP apps which previously had to be done with a <font face="Consolas">Grid</font> or <font face="Consolas">StackPanel</font>. The benefit of using relative layouts is when we focus on how we might want to realign the content for a different screen(size).

For instance when displayed on a tablet you might not just want to show a small image next to the text but go with a banner. Using a <font face="Consolas">VisualStateManager </font>one can easily define trigger points for the minimal window Width and  if the trigger fires adopt the layout of the predefined layout:


    <VisualStateManager.VisualStateGroups>
        <VisualStateGroup>
            <VisualState x:Name="wideView">
                <VisualState.StateTriggers>
                    <AdaptiveTrigger MinWindowWidth="720" />
                </VisualState.StateTriggers>
                <VisualState.Setters>
                    <Setter Target="Image.(Width)" Value="720"/>
                    <Setter Target="TextLine1.(Width)" Value="720"/>
                    <Setter Target="TextLine2.(Width)" Value="720"/>
                    <Setter Target="TextLine3.(Width)" Value="600"/>
                    <Setter Target="TextLine1.(RelativePanel.Below)" Value="Image"/>
                    <Setter Target="TextLine1.(RelativePanel.AlignLeftWith)" Value="Image"/>
                </VisualState.Setters>
            </VisualState>
            <VisualState x:Name="narrowView">
                <VisualState.Setters>
                    <!-- Adjust view for narrow view -->
                </VisualState.Setters>
                <VisualState.StateTriggers>
                    <AdaptiveTrigger MinWindowWidth="0" />
                </VisualState.StateTriggers>
            </VisualState>
        </VisualStateGroup>
    </VisualStateManager.VisualStateGroups>


The name of the Target matches the <font face="Consolas">x:Name</font> given to the controls. The page will now re-render if the app has more then 720 effective pixels:

[![Shows app running with more than 720 effective pixels.]({{ site.url }}{{ site.baseurl }}/assets/images/182ec6bd-9224-41a4-b3f5-6a4f23b60616.png "Shows app running with more than 720 effective pixels.")]({{ site.url }}{{ site.baseurl }}/assets/images/623b6c2b-c013-4395-a644-e8ab3371ac7f.png)

The page will also adjust when resizing the window on a Windows 10 desktop machine which allows the user to have multiple applications open at once with them seamlessly adapting to new windows sizes.

## Conclusion

In this post we saw that Universal Windows Platform Apps not only run on multiple devices but that the platform provides the developers and designers with the necessary tooling of creating a great User Experience no matter the screen size and resolution without having to create multiple page layouts from the ground up and having to somehow figure out the size of the screen itself.

You can find the entire project on [GitHub](https://github.com/mallibone/UWP.Layouting.git "GitHub Sample Link uri").
