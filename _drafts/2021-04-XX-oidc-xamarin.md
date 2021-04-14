---
layout: single
title: "Authentication in Xamarin Forms with Open Identity Connect / OAuth2"
date: 2021-04-01 15:03
tags: ["Xamarin", "Xamarin Forms", "OIDC", "OAuth2"]
slug: "xamarin-oidc"
---
Authenticating users in a Xamarin Forms app using the OpenID/OAuth standard... When overhearing people talking about authentication it almost feels like there is something mistacal to it. A secret club with handshakes unkown to the outstanders. At least that was how I rememberd it when I started looking into adding an Authenticaiton layer to an app I was working on. But as with many things, the more time you invest the more mysteries suddenly seem to make sense and before you know it, you are cruising through the authentication related tasks. Now authentication is not a new concept but it is one that you might have heard is best to follow a standard. Use a proven library and not implement your own. And we will follow those words by using OpenID/OAuth2 for authentication in a mobile Xamarin Forms application.

OAuth2 is a standard LLLINK that provides different flows how the user can authenticate himself. The recommended flow when using a mobile app is the code flow. Which looks like this if we plot it on a diagram:

IIIMAGE

You can see that the authentication is ruffly split up into four steps.

1. Authenticate
2. Callback into the app with a code
3. Requesting an access token (and identity token) with the code from the callback
4. Refreshing the token

And in the app the first three steps look something like this.

GIFFFFF

You might have noticed that the user actually authenticates using a browser window. This is by intent. The code flow does not allow the user to login using a native view in the app. The reason being that this flow ensures that the username and password are never seen by the client (except the browser which is part of the OS system - aka we trust it). You could enable using a native login view with the XXX flow. But this is also an attack vector. If someone makes a fraud duplicate of your application and tricking users to enter their credentials. The fraudulent app could store those credentials inbetween. You just got to enjoy those tin-foil-hat moments when doing security right? 🙃 In other words using the code flow does not give an attacker that opportunity and therefore is the recommended option.

With that brief introduction to the authentication flow and why the code flow is recommended let's start assembling the pieces of code which will bring the chart to life.

## Authenticating the user

The goal of this step is to get a code which will allow us to request the identity server for a token. Understanding the OAuth2 specs to build such a client from scratch is no small feat. Luckily Dominick Bayer LLINK and Brock Allen LLLINK have done a lot of heavy lifting for us on the client side by providing us with the IdentityModel.OidcClient library. The library provides the `OidcClient` class which will greatly simplify the calls to authenticate our user. We will require a few parameters to make the request:

| Parameter    | Description                                                  |
| ------------ | ------------------------------------------------------------ |
| Authority    | The URL of the Identity Server.                              |
| ClientId     | The id of the client which is defined on the server.         |
| Scope        | The scopes you want the tokens to have access to. The scopes are defined on the server. |
| RedirectUrl  | The login form will forward to this URL once the login has been successfull. We will have a closer look at this parameter in a second. |
| ClientSecret | A secret only known by the client and the server. I am listing this to tell you that you can add this but hard coding this value into your app will not make a real secret. Decompile an app and you will find it. If the server has set a secret (usually a string) then you can set it. But generally view this parameter as optional. The undefined value is `null` which is also the default value later on. |

The parameters described above usually differ from app to app. They probably will even differ from environment to environment i.e. staging vs production. Generally the parameters will be defined on the Identity Server e.g. the backend.

> Authority: This URL points the root URL of the identity server. The OidcClient library knows by convention which endpoints it has to call for the authentication and requesting the tokens later on by calling the https://path-to-the-identity-server.ch/.well-known/openid-configuration.

With the parameters at hand we can create a new `OidcClient` and pass in the parameters via a `OidcClientOptions` object as follows:

CCCODE

There is one additional parameter that was not mentioned before. The `Browser` parameter. Since the code flow will open a browser window for the user to log in, we can provide here the browser it will use. Luckely we can use the XXX from the Xamarin.Essentials LLLINK package which will provide exactly this feature. All that we have to do is embed it in an `IBrowser` interface provided by the OidcClient library:

CCCODE

After all this configuration we can start the authentication process by invoking the following line of code:

CCCODE

XXXX



Intro

big picture (what happens during OIDC)

Different auth flows -> why code

Explaining the auth flow

Getting the code (Xamarin Essentials)

Getting the token

Refreshing a token

Storing tokens in your app (Xamarin.Essentials)

Conclusion