---
layout: single
title: "Fixing the Visual Studio 2015 Android Emulator under Windows 10"
title: Fixing the Visual Studio 2015 Android Emulator under Windows 10
date: 2015-11-01
tags: ["Android", "General", "Mobile", "Surface", "Visual Studio"]
slug: "fixing-the-visual-studio-2015-android-emulator-under-windows-10"
---

This has been driving me bonkers the last couple of days so I just wanted to jot down the steps to get the Emulator up and running under Windows 10. In my case after upgrading from Win 8.1. So if you ever received one of the following errors:

- <font face="Consolas">[Critical] An internal virtual network switch is required for emulated devices to run.</font>
- <font face="Consolas">[Critical] XDE Exit Code: CouldntCreateInternalSwitch (16)</font>
- <font face="Consolas">[Critical] XDE Exit Code: CouldntStartVm (10)</font>
- <font face="Consolas">[Critical] XDE Exit Code: InvalidArguments (3)</font>
- <font face="Consolas">For UDP connection errors see update of July 2016 at the end.</font>


These steps should help you bring the Emulator back up and running:


> **Disclaimer:** Please note that you perform these steps at your own risk. Generally speaking all should run just fine but some steps will remove some Hyper-V settings and this might affect your network connectivity or virtual machine setup. So take these steps as worked on my machine advice.


1. Ensure no XDE.exe task is running (Task Manager)
2. Repair Android SDK – Open *Programs and Features* &gt; Microsoft Visual Studio Emulator for Android &gt; select *Change* in the menu bar and choose *Repair* in the Visual Studio dialog
3. Remove All Hyper-V virtual switches – open the  *Hyper-V Manager* &gt; select the Virtual Switch Manager… (located in the menu on the right) &gt; select each virtual switch and select remove &gt; click onto Apply
    1. If there is an error while removing a virtual switch, try restarting your computer
4. Run *XdeCleanup.exe* – Depends on your install but on the Surface Pro 3 you should find it under: "C:\Program Files (x86)\Microsoft XDE\10.0.10240.0"


Now try to get your Emulator up and running. If you still receive error messages try the following steps:

- (Surface Pro 3) Reinstall the network drivers &gt; Open the *Device Manager*&gt; under *View* select *Show hidden devices* &gt; Uninstall *Marvell AVASTAR Wireless-AC Network Controller* and *Surface Ethernet Adapter* &gt; Restart which will automatically reinstall the drivers from the recovery partition
- Clean up network bridges &gt; Open Network Connections &gt; Add Bridge (this option only showed up the first time) &gt; Remove the added bridge again


Now your Emulator should fire up.

# Remark

I had to repeat step 3 a couple of times – even after the emulator would start up the first time but maybe that is just a configuration issue on my machine…

Hope this helps and happy coding! ![Smile](https://mallibone.com/posts/files/5bbd1b8e-ff4b-4d0d-b029-93f37da303ef.png)

## Resources

[http://stackoverflow.com/questions/31613607/visual-studio-2015-emulator-for-android-not-working-xde-exe-exit-code-3](http://stackoverflow.com/questions/31613607/visual-studio-2015-emulator-for-android-not-working-xde-exe-exit-code-3 "http://stackoverflow.com/questions/31613607/visual-studio-2015-emulator-for-android-not-working-xde-exe-exit-code-3")



# Update - 20. July 2016

Managed to wreck my Android Emulator again yesterday. This time none of the above attempts fixed it for me. Since this is one of the more frequented blog posts I thought I would update to share another possible solution.

  


The Error message was that no UDP connection could be established with the Emulator. Following these steps solved the issue:

  



- Open Device Manager (On Win10 simply search for Device Manager in the Start Menu)
- Under *View*select *Show hidden devices*
- Delete all virtual network devices except your Network card (you do not have remove the Bluetooth devices)
- Open Hyper-V and remove all Hyper-V virtual network endpoints as described under Point 3
- Reboot


These steps solved the mentioned issue above and is based on the following Troubleshooting [article](https://msdn.microsoft.com/en-us/library/mt228282.aspx#XamarinPlayer)by Microsoft.

