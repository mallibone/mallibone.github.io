---
layout: single
title: "Working offline with Xamarin"
title: Working offline with Xamarin
date: 2014-12-13
tags: ["Xamarin"]
slug: "working-offline-with-xamarin"
---

I’ll admit it I like working on the go. Whenever I’m not responsible in the transportation of myself e.g. sitting in a train I love packing out my Surface Pro 3 and do something productive on it. And one thing is continuing working on my mobile apps written with Xamarin. But that can be a short and frustrating experiencing when Xamarin requires you to connect to the internet to validate your certificate. But luckily there is a solution to that.

## Get your certificates

To work offline you will have to download your licenses from your Xamarin account. So head to the Xamarin website, login and navigate to computers.

[![theRightMenuOptionForCerts]({{ site.url }}{{ site.baseurl }}/images/4da34cbe-ffad-4f73-a601-e132b6616a46.png "theRightMenuOptionForCerts")]({{ site.url }}{{ site.baseurl }}/images/20c64478-2e16-4fa0-b0cc-22f5d3a336e2.png)

[![saveCert]({{ site.url }}{{ site.baseurl }}/images/0b6e9f85-d3a9-483f-9ccf-9c97eadd4f0d.png "saveCert")]({{ site.url }}{{ site.baseurl }}/images/04623410-e266-4c13-8138-fdbabec1941b.png)



Select the licenses you need to download, for me that was iOS and Android.

## Move them to the correct location

Now that we have the licenses we still need to move them into the correct location. Under Windows you will have to move your Android license to:




    C:\ProgramData\Mono for Android\License


And your iOS license to:




    C:\ProgramData\MonoTouch\License


Now the only thing hindering you from getting work done on the move while your offline is that you are cut of the any online reference.

HTH
