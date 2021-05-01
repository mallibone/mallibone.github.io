---
layout: single
title: "Setting up an Idenity Server for your Xamarin app"
date: 2021-05-11 12:03
tags: ["Xamarin", "Xamarin.Forms", "IdentityServer", "OIDC", "Authentication"]
slug: "xamarin-identity-server"
---

![Image showing a server rack]({{ site.url }}{{ site.baseurl }}/assets/images/2021-05-server.jpeg "Image showing a server rack")

Have you ever wondered how hard it would be to set up a minimal viable authentication server that uses industry standards and usable from your mobile Xamarin application? Well, I have, and I believe in having found a solution that can be a great starting point and will allow you to expand the answer should you ever need to do so.

<!--more-->

One common industry standard is [OpenID](https://openid.net/developers/specs/) / [OAuth2](https://tools.ietf.org/html/rfc6749), which provides a standardized authentication mechanism that allows user identification securely and reliably. You can think of the identity service as a web server that identifies a user and provides the client (website/mobile app, etc.) to authenticate itself with another application server that said client uses. 

![OAuth Code Flow Diagram]({{ site.url }}{{ site.baseurl }}/assets/images/202104_OIDC_Flows.png "OIDC Flow")

While the OAuth standard is open to anyone with a computer and an internet connection, I generally do not recommend writing your own implementation. My go-to solution for setting up an identity provider is the [IdentityServer](https://identityserver4.readthedocs.io/en/latest/) by [Dominick Bayer](https://twitter.com/leastprivilege) and [Brock Allen](https://twitter.com/BrockLAllen).

> At the time of writing this post the IdentityServer5 has been released by [Duende Software](https://duendesoftware.com/). A company founded by Dominick Bayer and Brock Allen. With that come some changes to licenses for companies. You can still use Duende IdentityServer for free for certain scenarios which you can find here. For this post however we will be sticking with IdentityServer 4.

[IdentityServer4](https://github.com/IdentityServer/IdentityServer4) is built based on the OAuth spec. It is built on the trusted ASP.NET Core but requires quite some know-how to get the configurations and other settings ready for use. Luckily my good friend [Damien Bowden](https://twitter.com/damien_bod) has created a quickstart template that you can install via the dotnet command line and then make your server. You can find the repository [here](https://github.com/damienbod/IdentityServer4AspNetCoreIdentityTemplate) on GitHub. After following the [install instructions](https://github.com/damienbod/IdentityServer4AspNetCoreIdentityTemplate#using-the-template), we can create a server with the following command:

    dotnet new sts -n XamarinIdentity.Auth

The solution is pretty much ready to go but let\'s look at the configuration of the IdentityServer in `Config.cs` and make some adjustments in the `GetClients` method. Based on the template, let\'s make some changes that leave us with the following final configuration:

```c#
public static IEnumerable<Client> GetClients(IConfigurationSection stsConfig)
{
    return new List<Client>
    {
        // mobile client
        new Client
        {
            ClientName = "mobileclient-name-shown-in-logs",
            ClientId = "the-mobileclient-id-of-your-choice",
            AllowedGrantTypes = GrantTypes.Code,
            AllowOfflineAccess = true, // allow refresh tokens
            RequireClientSecret = false,
            RedirectUris = new List<string>
            {
                "com.mallibone.oidcsample/authorized"

            },
            PostLogoutRedirectUris = new List<string>
            {
                "com.mallibone.oidcsample/unauthorized",
            },
            AllowedScopes = new List<string>
            {
                "openid",
                "role",
                "profile",
                "email"
            }
        }
    };
}
```

Generally, you can set the `ClientName`, `ClientId`, `RedirectUris` and `PostLogoutRedirectUris` to values of your choosing. The scopes represent the defaults. Further note that by setting `AllowOfflineAccess` to true, the user can request [refresh tokens](https://identityserver4.readthedocs.io/en/latest/topics/refresh_tokens.html) which means that as long as the refresh token is valid, the user will not have to log in but can use said refresh token to request a new access token. In mobile apps, this is generally the prefered behaviour since users usually have their personal device and therefore expect the app to \"store\" their login.

The only thing left before we can deploy this puppy is making sure we store our users.

Storing the login credentials in a database is good because having in-memory storage usually causes quite some frustration when restarting a service. The quickstart solution we have configured comes with a [SQLite](https://sqlite.org/index.html) database per default. While you might outgrow the SQLite database at some point, if you only have a couple of users using it, it works very well. To use SQLite, you will have to set the *Copy to output directory* to *Copy if newer* of the `userdatabase.sqlite` or it will not deploy during a build. And please note that the SQLite database will be created anew on every deploy - so this is purely for development purposes. 😉

> Please note that using the SQLite version is a quick and easy way to get your IdentityServer up and running locally and in the cloud. However, I am not saying you should not use MS SQL, Postgresql, MySQL or any other database that you might have already running in production. If you plan to use a different database anyway, feel free to configure the required changes in the ASP.NET Core startup.

Next, we will want to deploy our solution so a client can call it. Since the identity service is an ASP.NET Core web app, we can deploy it to any web server. For example, you could use Azure Web Service for this. Once the server is deployed, you can visit it, register new users and authenticate your clients. Of course, you can further edit the server to your needs and wishes, in which case the [official documentation](https://identityserver4.readthedocs.io/en/latest/) will be of great value.

And that\'s it - with these few changes, you have a fully running IdentityServer running in the cloud, and you can now start integrating it into your apps, knowing that you are using industry standards to authenticate and identify your users. You can find a sample on [GitHub](https://github.com/mallibone/XamarinIdentity101/tree/main/XamarinIdentity.Auth).

HTH

Title photo by **[panumas nikhomkhai](https://www.pexels.com/@cookiecutter?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)** from **[Pexels](https://www.pexels.com/photo/black-and-gray-mining-rig-1148820/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)**