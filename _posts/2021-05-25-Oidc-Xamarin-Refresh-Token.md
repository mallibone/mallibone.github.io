---
layout: single
title: "Using OIDC client in Xamarin Forms to refresh your access token"
date: 2021-05-25 08:10:00
tags: ["Xamarin", "Xamarin Forms", "Authentication", "OIDC", "OAuth"]
slug: "xamarin-oidc-refresh"
---

![Image showing a smartphone with a recycle symbol on it]({{ site.url }}{{ site.baseurl }}/assets/images/2021-05-RefreshToken-Title.jpeg "Image showing a smartphone with a recycle symbol on it")

So you decided to take use OpenIdentity Connect / OAuth for authenticating your users in your Xamarin app. The great thing is that your client application will never see or process the username and password, which reduces the possible attack vector. Instead, all your app ever receives is an access token that grants to access APIs that require authenticated users. But there is a minor caveat. The access token often is only valid for a short time. With IdentityServer, the default is 60 minutes. Which would, in theory, mean that your user has to authenticate - read enter username and password - every hour. That is seriously uncool, and there surely must be a better way. There is. The app can renew the access token with a refresh token on the user's behalf without any interaction required.
<!--more-->



## The backstory

I covered how to integrate the [OIDC client](https://github.com/IdentityModel/IdentityModel.OidcClient) into a Xamarin Forms app in a former [post](https://mallibone/post/xamarin-oidc). As described, when the user authenticates him- or herself, we pass along the scopes we require. If we add the `offline_access` scope to the request and the server allows the scope request, we will receive a refresh token. You can configure on the IdentityServer how long a refresh token is valid and if it is enabled.

```c#
AllowOfflineAccess = true, // allow refresh tokens
AbsoluteRefreshTokenLifetime = 2592000, // in seconds ~> 30 days
```

With the above configuration, the `LoginResult` will contain a valid refresh token. Which is cool, but what now? When and how do we use this token? Let's first look at when we want to use the refresh token. The client (in our case, the Xamarin Forms app) knows precisely when the access token is about to expire with the `AccessTokenExpiration` property on the `LoginResult`. So we could check before every request if the token is still valid. Perhaps subtract a few seconds to be on the safe side.

```c#
if(_loginResult.AccessTokenExpiration.ToUniversalTime().AddSeconds(-30) > DateTime.UtcNow)
{
    await _oidcIdentityService.Refresh(_loginResult.RefreshToken)
}
```

Another option would be to let the backend tell us that we no longer have a valid access token. We can do this by calling the API, and if the call returns a [401 response](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes#4xx_client_errors) - we know that the access token is expired, and we need will probably want to refresh the token.

```c#
var response = await _httpClient.GetAsync(url);
if (response.StatusCode == HttpStatusCode.Unauthorized)
{
    if (string.IsNullOrEmpty(_loginResult?.RefreshToken))
    {
        // no valid refresh token exists => authenticate
        // ...
    }
    else
    {
        // we have a valid refresh token => refresh tokens
        // ...
    }
}
```

> There is another 400 code that is often associated with authentication - the [403](https://en.wikipedia.org/wiki/HTTP_403). While the 401 indicates that the access token is invalid. The 403 means that the user does not have the proper permission to access the resource. So if a non-admin user wanted to access some admin function, a 403 response is expected.

Both methods work, and it is up to you which one you prefer more. When the time has come to refresh the tokens, we can request a new set of tokens.

## Asking for a refresh

By default, we can only use the refresh token one time to request a new access token. Fortunately, the result of the refresh contains not only a new access token but also a new refresh token. So we can request once more a new set of tokens should the time come. The OIDC client provides a method for asking a new set of tokens for a given refresh token:

```c#
OidcClient oidcClient = CreateOidcClient();
RefreshTokenResult refreshTokenResult = await oidcClient.RefreshTokenAsync(refreshToken);
```

It returns a `RefreshTokenResult`, which contains kind of the same information as the `LoginResult`. But, unfortunately, they do not share their same properties in a base class, so that I would recommend to you create your `Credentials` class to store the information of the tokens:

```c#
public class Credentials
{
    public string AccessToken { get; set; } = "";
    public string IdentityToken { get; set; } = "";
    public string RefreshToken { get; set; } = "";
    public DateTime AccessTokenExpiration { get; set; }
    public string Error { get; set; } = "";
    public bool IsError => !string.IsNullOrEmpty(Error);
}
```

By writing the following extension methods:

```c#
public static Credentials ToCredentials(this LoginResult loginResult)
    => new Credentials
    {
        AccessToken = loginResult.AccessToken,
        IdentityToken = loginResult.IdentityToken,
        RefreshToken = loginResult.RefreshToken,
        AccessTokenExpiration = loginResult.AccessTokenExpiration
    };

public static Credentials ToCredentials(this RefreshTokenResult refreshTokenResult)
    => new Credentials
    {
        AccessToken = refreshTokenResult.AccessToken,
        IdentityToken = refreshTokenResult.IdentityToken,
        RefreshToken = refreshTokenResult.RefreshToken,
        AccessTokenExpiration = refreshTokenResult.AccessTokenExpiration
    };
```

You can then convert the result into your own credentials model, which will make it easier to handle this information later on in the app.

```c#
RefreshTokenResult refreshTokenResult = await oidcClient.RefreshTokenAsync(refreshToken);
return refreshTokenResult.ToCredentials();
```

Be aware that the refresh might also fail. The main reason usually being that the refresh token has already expired. Unfortunately, there is no way around it in this case, and you will have to present the user with your beautiful login dialogue. 😉

Other than the access token, the server stores the refresh token per client. This also allows revoking a refresh token on the server. Since every client has a refresh token, some websites will show which clients/apps have access to your account. By revoking the permit, you remove the refresh token, forcing the client to reauthenticate (via the login form).



## Storing tokens

With refresh tokens, you can create apps that only require users to authenticate once (or at least fewer times) because the access token can be renewed automatically in the background while the user is using the app. Furthermore, the refresh token does not contain any information. Nevertheless, should a valid request token fall into the wrong hands, a potential attacker could impersonate a user.

So don't just store the tokens and the refresh token somewhere in the local storage, but use the [Xamarin Essentials secure storage](https://docs.microsoft.com/en-us/xamarin/essentials/secure-storage?tabs=ios). This will ensure that you keep the tokens in the safest place available on the device. And once you need them, you can read them, verify if it's time to renew the tokens and if so. Then, send an HTTP request for a new batch of tokens.



## Aftermath

And that is how you can use OAuth refresh tokens to request new tokens without having to prompt your user over and over with an authentication form. Note that the feature has to be enabled on the server, and it is also the said server that will be able to determine how long a refresh token is valid.

Writing a mobile app using refresh tokens is highly recommended since it allows using a solid and secure authentication mechanism while providing the user with a pleasant experience.

You can find the code sample [here](https://github.com/mallibone/XamarinIdentity101) on GitHub.

HTH

Titlephoto by **[ready made](https://www.pexels.com/@readymade?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)** from **[Pexels](https://www.pexels.com/photo/mobile-phone-with-green-recycling-sign-and-mesh-bag-3850512/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)**
