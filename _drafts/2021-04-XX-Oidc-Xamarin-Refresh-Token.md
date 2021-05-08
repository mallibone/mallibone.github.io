---
layout: single
title: "Using OIDC client in Xamarin Forms to refresh your access token"
date: 2021-05-01 13:10:00
tags: ["Xamarin", "Xamarin Forms", "Authentication", "OIDC", "OAuth"]
slug: "xamarin-oidc-refresh"
---

IMAGE

So you decided to take use OpenIdentity Connect / OAuth for authenticating your users in your Xamarin app. The great thing is that your client application will never see or process the username and password, which reduces the possible attack vector. Instead, all your app ever receives is an access token that grants to access APIs that require authenticated users. But there is a minor caveat. The access token often is only valid for a short time. With IdentityServer, the default is 60 minutes. Which would, in theory, mean that your user has to authenticate - read enter username and password - every hour. That is seriously uncool, and there surely must be a better way. There is. The app can renew the access token with a refresh token on the user's behalf without any interaction required.

## The backstory

I covered how to integrate OIDC into a Xamarin Forms app in a former [post](https://mallibone/post/xamarin-oidc). As described, when the user authenticates him- or herself, we pass along the scopes we require. If we add the `offline_access` scope to the request and the server allows the scope request, we will receive a refresh token. You can configure on the IdentityServer how long a refresh token is valid and if it is enabled.

```c#
AllowOfflineAccess = true, // allow refresh tokens
AbsoluteRefreshTokenLifetime = 2592000, // in seconds ~> 30 days
```

XXXWith the above configuration the `LoginResult` will contain a valid refresh token. Which is cool, but what now? When and how do we use this token? Let's first look at when we want to us the refresh token. The client (in our case the Xamarin Forms app) knows exacthly when the access token is about to expire with the `AccessTokenExpiration` property on the `LoginResult`. So we could check before every request if the token is still valid. Perhaps subtract a few seconds to be on the safe side.

```c#
if(_loginResult.AccessTokenExpiration.ToUniversalTime().AddSeconds(-30) > DateTime.UtcNow)
{
    await _oidcIdentityService.Refresh(_loginResult.RefreshToken)
}
```

Another option would be to let the backend tell us that we no longer have a valid access token. We can do this by calling the API and if the call returns a [401 response](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes#4xx_client_errors) - we know that the access token is expired and we need will probably want to refresh the token.

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

> There is another 400 code that is often associated with authentication and that is the [403](https://en.wikipedia.org/wiki/HTTP_403). Other than the 401, the 403 indicates that while the access token is valid the user does not have the right permission to access the resource. So if a user wanted to access some admin function, a 403 response would be expected.

Both methods work and it is up to you which one you prefer more. When the time has come to refresh the tokens we can request a new set of tokens.

## Asking for a refresh

By default we can only use the refresh token one time to request a new access token. Fortunately the result of the refresh contains not only a new access token but also a new refresh token. So we can request once more a new set of tokens should the time come. The OIDC client provides a method for requesting a new sets of tokens for a given refresh token:

```c#
OidcClient oidcClient = CreateOidcClient();
RefreshTokenResult refreshTokenResult = await oidcClient.RefreshTokenAsync(refreshToken);
```

The result of this request is a `RefreshTokenResult` which contains sort of the same information as the `LoginResult` that is returned after authenticating a user. Unfortunately the do not share their same properties in a base class so I would recommend to you create your own `Credentials` class to store the information of the tokens:

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

You can then simply convert the result into your own credentials model, which will make it easier to handle this information later on in the app.

```c#
RefreshTokenResult refreshTokenResult = await oidcClient.RefreshTokenAsync(refreshToken);
return refreshTokenResult.ToCredentials();
```

Be aware that the refresh might also fail. The main reason usually being that the refresh token has already expired. In this case there is no way around it and you will have to present the user with the beautiful login dialog of yours. 😉

Other than the access token the server stores the refresh token per client. This also allows to revoke a refresh token on the server. Since every client has a refresh token some websites will show which clients/apps have access to your account. By revoking the access you then remove the refresh token, forcing the client to reauthenticate (via the login form).

## Storing tokens

With refresh tokens you can create apps that only require users to authenticate once (or at least fewer times) because the access token can be renewed automatically in the background while the user is using the app. The refresh token does not contain any information and should not be parsed for information. Neverless should a valid request token fall into the wrong hands a potential attacker could impersonate a user.

So don't just store the tokens and the refresh token somewhere in the local storage, but use the [Xamarin Essentials secure storage](https://docs.microsoft.com/en-us/xamarin/essentials/secure-storage?tabs=ios). This will ensure that the tokens are stored on safest place available on the device. And once you need them you can read them again, verify if it's time to renew the tokens and if so. Send of a HTTP request for a new batch of tokens.



## Aftermath

And that is how you can use OAuth refresh tokens to request new tokens without having to prompt your user over and over with an authentication form. Note that the feature has to be enabled on the server and it is also said server that will be able to determine how long a refresh token is valid.

When writing a mobile app using refresh tokens is highly recommended since it allows using a strong and secure authentication mechanism while providing the user with a nice experience.

You can find code sample [here](https://github.com/mallibone/XamarinIdentity101) on GitHub.

HTH