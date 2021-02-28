---
layout: single
title: "Airport Extreme Printing via Windows 8.1 and Windows RT"
title: Airport Extreme Printing via Windows 8.1 and Windows RT
date: 2014-12-19
tags: ["General"]
slug: "airport-extreme-printing-via-windows-8-and-8.1"
---

[![windows_airportExtreme](http://mallibone-blog.azurewebsites.net/posts/files/8914e51e-deb8-4666-84b2-7e7c9092d2ab.png "windows_airportExtreme")](http://mallibone-blog.azurewebsites.net/posts/files/1394e2a7-873f-44ba-a1c7-66ef7da02e8f.png)

These 14 steps show in great detail how you can add a printer that is connected to your Airport Extreme Router and print via your Network or even wirelessly if you happen to use WiFi from you Windows Device.

1. Push the Windows-Key and W to enter **Devices and Printers** and push enter to open
2. Select **Add a Printer**
3. Don’t bother waiting until the scan has finished and just select next
4. Select the **Add a local printer** option – lets not get into a discussion about bad naming.. Let’s just get the printer setup and working
5. Click **Create a new port**, then select **Standard TCP/IP Port** from the drop-down list and finally click **Next**
6. Fill in the **Hostname or IP address**with the address of your Airport Extreme router.
    1. By default it should be 10.0.1.1
    2. If default fails push Windows-Key and S, enter cmd, as soon as the command line has opened enter ipconfig. The IP next to the Default Gateway will be your Routers IP.
7. Leave **Port name** at whatever it fills in, uncheck **Query the printer and automatically select the driver to use**, and click **Next**.
8. The wizard will say it’s **Detecting the TCP/IP port**. It should find the device. If not, you probably entered the wrong IP address. Check it and try again. If it still fails to detect it, don’t worry about it and continue anyway.
9. Select your printer manufacturer in the dropdown e.g. **Samsung Printer**. Click **Next**.
10. Select your printer’s **Manufacturer** from the list, then select your specific printer from the **Printers** list, then click **Next**. If your printer isn’t there, you’ll have to download a driver and use the **Have Disk…** option.
11. Leave the option **Use the driver that is currently installed (recommended)** and just click **Next**.
12. Fill in a **Printer name**, or leave the name it fills in for you alone. Click **Next**.
13. Decide whether to share the printer or not. Since other devices on your network can print directly to the Airport Extreme, why bother to share it? I selected **Do not share this printer** and clicked **Next**.
14. Decide whether to **Set as the default printer**, and try to **Print a test page**, then click **Finish**.




This worked on all my Windows 8.x machines even RT. This blog post leans very heavily on the [Blogpost](http://blog.engelke.com/2012/12/27/windows-printing-to-an-airport-extreme-connected-printer/) from [Charles Engelke](https://twitter.com/charlesengelke), whom originally wrote the insturctions for Windows 7.

HTH
