---

layout: single
title: "Using UDL files to check your SQL Server network connection"
date: 2022-02-23 10:03:00
tags: ["Windows Server", "SQL Server", "DevOps"]
slug: "udl-sqlserver-check"
---

Did you ever have that moment when you are just about to deploy your new (web) app the first time on a Windows Server and have the information of connecting to your SQL server at hand. You deploy your app but nothing happens because something was not configured correctly. While there are many steps involved in getting this step right one of the essential parts is usually connecting your app to it's database. And if that does not work, well your app usually does not work either. And this is where UDL files come in handy.

UDL files allow you to check if the SQL Server is setup, the user is working and if the firewall/network is properly configured without having to install a single tool. Steps required are go to any folder you want (I usually just do this on the desktop). Create a new file and rename it to `whatever-you-want.udl`. Notice that the important piece is the file ending, Windows will prompt if we are sure - and of course we are. 😉 If we now double click on the file we are greeted with the following dialog:

IIIIMAGE

We can now start filling in the information: 

* SQL Server instance name (good chance this was provided by your IT)
* Username and Password (if you are not using Windows Auth which is often the case when developing a web app)

At this point you should already be able to select a database on the SQL Server, which is kind of a spoiler since the check connection button has not yet been pressed. 🙃 Nevertheless select the database you intend to use and then select check connection. If successful you will now know that the connection string you are using in your app is working correctly and any error must be at a different point.

> Note that if you are using Entity Framework Core it may be that you let Code First Migrations create your database and that it will not be listed. That being said I would not recommend this approach since it will give your app the permission to go around dropping databases. If you use the `DBOwner` you have to create the DB by hand but the tables and all can still be created by Entity Framework Core.

## Conclusion

UDL files are a great helper to have up your sleve if you are deploying an app the first time on a new Windows Server machine and want to quickly figure out if everything has been setup correctly before walzing through error log files of your web application.
