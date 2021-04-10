---
layout: single
title: "Setting up an Idenity Server for your Xamarin app"
date: 2021-04-01 15:03
tags: ["Xamarin", "Xamarin.Forms", "OIDC", "Authentication"]
slug: "gnabber"
---



Ever wondered how hard it would be to setup a minimal viable authenticaiton server which uses industry standards and can be integrated into your mobile Xamarin application? Well I have had that question and I think I found a solution that can be a great starting point and will allow you to expand the solution should you ever have the need to do so.

One common industry standard that is used by many companies is [OpenID](https://openid.net/developers/specs/) / [OAuth2](https://tools.ietf.org/html/rfc6749) which provides a standardized authentication mechanism that further allows to identity a user in a secure and realiable fashion. You can think of the identity service as a web server which has the job of identifying a user and providing the client (website/mobile app etc.) with a way to authenticate itself with another application server that is used by said client.

IIIMAGE

While the OAuth standard is open to anyone with a computer and an internet connection, I generally do not recommend writing your own implementation. My goto solution for setting up my own identity provider is the [Identity Server](https://identityserver4.readthedocs.io/en/latest/) by [Dominick Bayer](https://twitter.com/leastprivilege) and [Brock Allen](https://twitter.com/BrockLAllen). Identity Server is built based on the OAuth spec and is built on .Net Core - the official version of Identity Server requires quite some knowhow to get the configurations and other settings ready for use though. Luckely my good friend [Damien Bowden](https://twitter.com/damien_bod) has created a quick start solution which you can install via the dotnet command line and then create your own server. You can find the repository [here](https://github.com/damienbod/IdentityServer4AspNetCoreIdentityTemplate) on GitHub. So after following the [install instructions](https://github.com/damienbod/IdentityServer4AspNetCoreIdentityTemplate#using-the-template) we can create a server with the following command:

```bash
dotnet new sts -n Xamarin.OIDC
```

The solution is pretty much ready to go but let's have a look at the configuration of the identity server in `Config.cs` and make some adjustments in the `GetClients`  method. Based on the commented code we will want to set the following parameters:

* asdf

Which leaves us with the following final configuration:

CCCODE

The users are usually stored in a database which is a good thing because having an in memory storage usually causes quite some frustration when restarting a service. The quickstart solution we have configured comes with a SQLite LLLINK database per default. While you might outgrow the SQLite database at some point, if you only have a couple of users using it, it actually works very good. To use SQLite you will have to set the XXX to YYY of the ZZZZ or it will not be deployed during a build.

IIIMAGE

Next we will want to deploy our solution so it can be called by a client. Since the identity service is basically a ASP.NET Core web app we can deploy it to any webserver. I will be using an Azure Web Service for this. Once the server is deployed you can visit it, register new users and authenticate your clients.

And that's it - with these few changes you have a fully running Identity Server running in the cloud and you can now start integrating it into your apps, knowing that you are using industry standards so authenticate and identify your users.

HTH

