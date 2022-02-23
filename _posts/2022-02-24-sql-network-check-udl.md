---

layout: single
title: "Using UDL files to check your SQL Server network connection"
date: 2022-02-24 14:03:00
tags: ["Windows Server", "SQL Server", "DevOps"]
slug: "udl-sqlserver-check"
---

Did you ever have that moment when you are just about to deploy your new (web) app for the first time on a Windows Server and have the information of connecting to your SQL server at hand? You deploy your app, but nothing works due to some incorrect configuration. Does this sound familiar? While there are many steps involved in getting everything set up and configured correctly, one of the essential parts is usually connecting your app to its database. And if that does not work, your app usually does not work either. And this is where [Universal Data Link](https://docs.microsoft.com/en-us/sql/connect/oledb/help-topics/data-link-pages?view=sql-server-ver15) (UDL) files come in handy.

<!-- more -->

UDL files allow you to verify the connection string from a Windows machine to a Microsoft Microsoft OLE DB Driver compatible data source, e.g. Microsoft SQL Server. Plus, they come out of the box, so no additional setup is required. 

> You will need the Microsoft OLE DB Driver installed on the Windows machine. However, this is usually installed by default.

The steps required are to go to any folder you want (I usually do this on the desktop). Create a new file and rename it to `whatever-you-want.udl`. Notice that the critical piece is the `.udl` file ending. Windows will prompt if we are sure - and of course, we are. 😉 If we now double click on the file, we are greeted with the following dialogue:

![UDL dialog with file named gnabber.udl next to it.]({{ site.url }}{{ site.baseurl }}/assets/images/2022-02-udl.png "UDL dialog with file named gnabber.udl next to it.")

We can now start filling in the information: 

- SQL Server instance name. There is a good chance your IT provided this.
- Username and Password: If you are not using Windows Auth, which is often the case when developing a web app.

At this point, you should already be able to select a database on the SQL Server, which is kind of a spoiler since we didn't even have a chance to press the check connection button. 🙃 Nevertheless, select the database you intend to use and press the check connection button. If successful, you will now know that the connection string you are using in your app is working correctly, and any error must be at a different point.

> Note that if you are using Entity Framework Core, you let Code First Migrations create your database and that it will not be listed. Though there is an argument here of how much power the database user of your app requires. But that argument is for another blog post topic.

## Conclusion

UDL files are a great helper to have up your sleeve if you are deploying an app the first time on a new Windows Server machine and want to quickly figure out if everything has been set up correctly before waltzing through error log files of your web application.

You can find more detailed information on configuring the UDL in the official [Microsoft docs.](https://docs.microsoft.com/en-us/sql/connect/oledb/help-topics/data-link-pages?view=sql-server-ver15)
