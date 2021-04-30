---
layout: single
title: "Authentication in Xamarin Forms with Open Identity Connect and OAuth"
date: 2021-04-30 08:00:00
tags: ["Xamarin", "Xamarin Forms", "OIDC", "OAuth2"]
slug: "xamarin-oidc"
---

Authenticating users in a Xamarin Forms app using the OpenID/OAuth standard\... When overhearing people talking about authentication, it almost feels like there is something mystical to it, almost like a secret club with handshakes unknown to outsiders. At least, that was how I remember it when I started looking into adding an authentication layer to an app I was working on at the time. But as with many things, the more time you invest, the more mysteries suddenly seem to make sense, and before you know it, you are cruising through the authentication-related tasks. Now authentication is not a new concept, but it is one that you might have heard is best to follow a standard. Use a proven library and not implement your own. And we will follow those words using OpenID/OAuth2 for authentication in a mobile Xamarin Forms application.

OAuth2 is a [standard](https://oauth.net/2/) that provides different flows of how the user can authenticate himself. The recommended flow when using a mobile app is the code flow. Which looks like this if we plot it on a diagram:

![OAuth Code Flow Diagram]({{ site.url }}{{ site.baseurl }}/assets/images/202104_OIDC_Flows.png "OIDC Flow")

You can see that the authentication is ruffly split up into four steps.

1.  Authenticate

2.  Callback into the app with a code

3.  Requesting an access token (and identity token) with the code from the callback

4.  Refreshing the token

And in the app, the first three steps look something like this.

![Animated screen flow of the login process in the Xamarin Forms app.]({{ site.url }}{{ site.baseurl }}/assets/images/202104_XamarinOIDC_01_Screenflow.gif "OIDC Flow")

You might have noticed that the user authenticates using a browser window. This is by intent. The code flow does not allow the user to log in using a native view in the app. The reason being that this flow ensures that the username and password are never seen by the client (except the browser, which is part of the OS system - aka we trust it). You could enable using a native login view with the Resource Owner Password Credentials (ROPC) flow. But this is also an attack vector. Suppose someone makes a fraud duplicate of your application and tricking users into entering their credentials. The fraudulent app could store those credentials in-between. You just got to enjoy those tin-foil-hat moments when doing security. 🙃 In other words, using the code flow does not give an attacker that opportunity and therefore is the recommended option for mobile clients.

With that brief introduction to the authentication flow and why the code flow is recommended, let\'s start assembling the pieces of code, bringing the chart to life.

## Authenticating the user

This step aims to get a code that will allow us to request a token from the identity server. Understanding the OAuth2 specs to build such a client from scratch is no small feat. Luckily [Dominick Bayer](https://leastprivilege.com/) and [Brock Allen](https://brockallen.com/) have done a lot of heavy lifting for us on the client-side by providing us with the IdentityModel.OidcClient library. The library offers the `OidcClient` class, which will significantly simplify the calls to authenticate our user. We will require a few parameters to make the request:

| Parameter    | Description                                                  |
| ------------ | ------------------------------------------------------------ |
| Authority    | The URL of the Identity Server.                              |
| ClientId     | The id of the client.                                        |
| Scope        | he scopes you want to be included in the tokens. OpenId includes the scopes `openid` and `profile` by default. Another one is `offline`, which will request a refresh token from the backend---more on the refresh token in a bit. |
| RedirectUrl  | The login form will forward to this URL once the login has been successful. We will have a closer look at this parameter in a second. |
| ClientSecret | A secret that the client and server share. Since storing secrets in a mobile app are usually discovered relatively easy after unzipping and decompiling them. Regard this parameter as optional (no additional security benefits) for mobile apps. If, however, the server has defined a secret (usually a string), you have to set it on the client. The undefined value is `null`, which is also the default value and used later on. |

The parameters described above usually differ from app to app. They probably will even vary from environment to environment, i.e. staging vs production. The parameters get defined on the Identity Server, e.g. the backend and have to be inserted accordingly on the client.

> Authority: This URL points to the root URL of the identity server. The OidcClient library knows by a convention which endpoints it has to call for the authentication and requesting the tokens later on by calling the https://path-to-the-identity-server.ch/.well-known/openid-configuration.

With the parameters at hand, we can create a new `OidcClient` and pass in the parameters via an `OidcClientOptions` object as follows:

```c#
private OidcClient CreateOidcClient()
{
    var options = new OidcClientOptions
    {
        Authority = _authorityUrl,
        ClientId = _clientId,
        Scope = _scope,
        RedirectUri = _redirectUrl,
        ClientSecret = _clientSecret,
        Browser = new WebAuthenticatorBrowser()
    };

    var oidcClient = new OidcClient(options);
    return oidcClient;
}
```

There is one additional parameter that did not get mentioned before: The `Browser` parameter. Since the code flow will open a browser window for the user to log in, we need to configure the browse. Luckily we can use the Web Authenticator from the [Xamarin.Essentials](https://docs.microsoft.com/en-us/xamarin/essentials/) package which will provide exactly this feature. There are some [getting started steps](https://docs.microsoft.com/en-us/xamarin/essentials/web-authenticator?tabs=android) required, which you find in the official Microsoft docs. To invoke the All that we have to do is embed it in an `IBrowser` interface provided by the OidcClient library:

```c#
internal class WebAuthenticatorBrowser : IBrowser
{
    public async Task<BrowserResult> InvokeAsync(BrowserOptions options, CancellationToken cancellationToken = default)
    {
        try
        {
            WebAuthenticatorResult authResult =
                await WebAuthenticator.AuthenticateAsync(new Uri(options.StartUrl), new Uri(options.EndUrl));
            var authorizeResponse = ToRawIdentityUrl(options.EndUrl, authResult);

            return new BrowserResult
            {
                Response = authorizeResponse
            };
        }
        catch (Exception ex)
        {
            Debug.WriteLine(ex);
            return new BrowserResult()
            {
                ResultType = BrowserResultType.UnknownError,
                Error = ex.ToString()
            };
        }
    }

    public string ToRawIdentityUrl(string redirectUrl, WebAuthenticatorResult result)
    {
        IEnumerable<string> parameters = result.Properties.Select(pair => $"{pair.Key}={pair.Value}");
        var values = string.Join("&", parameters);

        return $"{redirectUrl}#{values}";
    }
}

```

The main interaction point is the `InvokeAsync` method. Here we use the Web Authenticator to open a browser. The end url passed to the server is the `RedirectUrl` we defined earlier in the options. This is a deep link back into your application, which is also part of the configuration of the Xamarin.Essentials Web Authenticator functionality. The response is a `WebAuthenticatorResult`. We are parsing the result into an HTTP request parameter string which we will use to request the tokens. We add the request string to the browser result. If anything goes wrong, we will return an error in the same object.

After all this configuration, we can start the authentication process by invoking the following line of code:

```c#
OidcClient oidcClient = CreateOidcClient();
LoginResult loginResult = await oidcClient.LoginAsync(new LoginRequest());
```

Starting the authentication will invoke our browser implementation `WebAuthenticatorBrowser`. Should the browser return a result, the OidcClient will automatically request the tokens from the server. If, however, there was an error at any stage, the result would provide us with the information. I.e. an exception occurred while executing the browser code.

The tokens returned from the server are the access token, an id token and if requested a refresh token. The id token is a JSON Web Token ([JWT](https://en.wikipedia.org/wiki/JSON_Web_Token)) and can be viewed as the response from the identity server. You can view the content of the token using a JWT parser like [this](https://jwt.ms/) one. It is important to note that while the access token is also a JWT token and can be parsed the OAuth spec does not define it as so. It is better to not depend on information in the access token as a client but to better read it from the id token, which you can and should view as the response of the identity server. For example the id token will contain the information when the access token expires.

## Making authenticated requests

Given a valid response from the server. Add the access token to a `HttpClient` header:

```c#
_httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("bearer", credentials.AccessToken);

```

Every call using this HttpClient will now provide the server with an access token. Should you want to remove the authentication header from your client, you can set it to `null`.

```c#
_httpClient.DefaultRequestHeaders.Authorization = credentials.IsError
    ? null
    : new AuthenticationHeaderValue("bearer", credentials.AccessToken);
```

Setting the value to `null` will also remove it from the request header of the  `HttpClient`.

## Refreshing the access token

An access token usually is only valid for a short time. How long depends on what on the server setting. The client receives the date and time when the access token expires. Either by parsing the ID Token (which is a [JWT Token](https://docs.microsoft.com/en-us/dotnet/api/System.IdentityModel.Tokens.Jwt.JwtSecurityToken?view=azure-dotnet&viewFallbackFrom=netstandard-2.0)), or we get a handy property `AccessTokenExpiration` on the response. The default with the identity server is 60 minutes. This would imply that the user will have to go through the authentication every hour. Do you remember a mobile app demanding this of you? Probably not. And if there were, you would probably be looking for an alternative. Luckily there is a way to prevent your user from having to provide the credentials over and over again. You can request a refresh token by adding the `offline_access` scope (this has to be allowed by the server). Then you can request a new access token from the server without asking the user for any information:

```c#
OidcClient oidcClient = CreateOidcClient();
RefreshTokenResult refreshTokenResult = await oidcClient.RefreshTokenAsync(refreshToken);
```

When requesting a new access token, you will also receive a new refresh token and identity token. The lifetime of a refresh token is defined (again) on the server. The default lifespan for the refresh token is 30 days with Identity Server. So if the user uses the app regularly, no login is required. However, should your app not receive the love it deserves, the user may be confronted every time with a login mask.

## Storing tokens

So far, we have received tokens after the user has logged in. Further, we looked at renewing toWe also saw how tokens could be kens without any interaction from the user\'s side. A good thing with this token-based approach is that your app will never be responsible for handling the user\'s credentials. Nevertheless, it can make sense to store the tokens, i.e., ensure the user does not have to log in after restarting the app. Since tokens impersonate a user, we should consider keeping them in a secure location. Again the Xamarin.Essentials library will be of great help, more precisely, the [Secure Storage](https://docs.microsoft.com/en-us/xamarin/essentials/secure-storage?tabs=android) feature of the library. It enables apps to store information encrypted or depending on phone and platform, even on a secure storage module.

## Recap

I hope that authentication has lost some of its mystery. The process may seem a bit complex at first, but it is there to ensure that our users\' credentials do not fall into the wrong hands. We also saw that with OidcClient and Xamarin.Essentials there are libraries ready to use and help to implement the Open Id / OAuth2 standard in your Xamarin app.

You can find a sample app implementing the described steps above on [GitHub](https://github.com/mallibone/XamarinIdentity101/tree/main/Mobile).

HTH
