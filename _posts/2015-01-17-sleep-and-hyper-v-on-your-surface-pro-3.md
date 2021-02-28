---
layout: single
title: "Sleep and Hyper-V on your Surface Pro 3"
title: Sleep and Hyper-V on your Surface Pro 3
date: 2015-01-17
tags: ["Surface", "General"]
slug: "sleep-and-hyper-v-on-your-surface-pro-3"
---

[![surfacepro3](http://mallibone-blog.azurewebsites.net/posts/files/bd4d8773-09af-464f-890a-ddc8ca36b330.jpg "surfacepro3")](http://mallibone-blog.azurewebsites.net/posts/files/c632f0c2-91c9-4b88-803f-e85a188a102d.jpg)
 
I got a Surface 3 Pro recently and I really love the fact that I can use it as a tablet and as a normal Laptop but am not bound to carrying two devices with me. One problem I received was after installing Visual Studio which installs and activates Hyper-V automatically. Hyper-V disables the connected Stand-By mode on your Surface. No problem there but that connected stand-by is also your sleep mode and living without a sleep mode on your tablet is a real pain point.
 
## The solution
 
Luckely we can easily change this behaviour by switching Hyper-V off when we do not need it with the following command:
 



    bcdedit /set hypervisorlaunchtype off




And if we do need Hyper-V for development we can enable it again with this line of code:




    bcdedit /set hypervisorlaunchtype auto




You will have to run each command in an elevated powershell as admin. Simply open Powershell rightclick on the Taskbar icon and select *Run as Administrator*.
  

HTH
