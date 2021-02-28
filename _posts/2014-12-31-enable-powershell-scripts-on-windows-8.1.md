---
layout: single
title: "Enable PowerShell Scripts on Windows 8.1"
title: Enable PowerShell Scripts on Windows 8.1
date: 2014-12-31
tags: ["General", " PowerShell"]
slug: "enable-powershell-scripts-on-windows-8.1"
---

When ever you want to automate some mundane work or setup steps on your Windows computer the word PowerShell seems to be starting to pop up. But before the fun even can begin one will hit a wall pretty quickly.

[![FailureRunningScript]({{ site.url }}{{ site.baseurl }}/images/a47559ed-fe11-4afa-9597-a0a8d1178de1.png "FailureRunningScript")]({{ site.url }}{{ site.baseurl }}/images/d097830b-db83-4015-b5d8-1458d3a60b84.png)

# Get scripting enabled

So how do we get scripts enabled? On the [Technet site](http://technet.microsoft.com/library/hh847748.aspx) you will find a long list of possible options on what levels can be set to:

- **Restricted:** Default execution policy, does not run scripts, interactive commands only. *This is what the default setup is on Windows 8.1, Windows 8 and even good old Windows 7.*
- **All Signed**: Runs scripts; all scripts and configuration files must be signed by a publisher that you trust; opens you to the risk of running signed (but malicious) scripts, after confirming that you trust the publisher.
- **Remote Signed**: Local scripts run without signature. Any downloaded scripts need a digital signature, even an UNC path.
- **Unrestricted**: Runs scripts; all scripts and configuration files downloaded from communication applications such as Microsoft Outlook, Internet Explorer, Outlook Express and Windows Messenger run after confirming that you understand the file originated from the Internet; no digital signature is required; opens you to the risk of running unsigned, malicious scripts downloaded from these applications



> This list was originally taken from [HowTo-Geek](http://www.howtogeek.com/106273/how-to-allow-the-execution-of-powershell-scripts-on-windows-7/) on 31. December 2014.


For now I'll go with the Remote Signed level. As I'm not planning on downloading any scripts from the web or using scripts from a network share. If you do intend to use any scripts from the web or any network share that are not signed you might want to use the Unrestricted level. So enough talk lets change the level, open a PowerShell with administrative rights and run the command:


    Set-ExecutionPolicy RemoteSigned


[![SetupRemoteSigned]({{ site.url }}{{ site.baseurl }}/images/c83367fe-a3e1-4809-b4c0-849e13379061.png "SetupRemoteSigned")]({{ site.url }}{{ site.baseurl }}/images/9b0c9fd9-5993-49f1-a519-b8d26b67d4c1.png)

Now you should be able to start writing the little helpers for your system with the help of PowerShell.

HTH
