---
layout: single
title: "OpenIdentity Connect and refreshing your access token in Xamarin Forms"
date: 2021-05-01 13:10:00
tags: ["Xamarin", "Xamarin Forms", "Authentication", "OIDC", "OAuth2"]
slug: "xamarin-oidc"
---

IMAGE

So you decided to take use OpenIdentity Connect / OAuth for authenticating your users in your Xamarin app. Great thing is that your client no longer requires to ever see the username and password, which reduces the possible attack vector. Instead all your app ever receives is an access token which allows you to access APIs that only provide access to authenticated users. But there is a minor caviat. The access token often is only valid for a short time. With IdentityServer the default is 60 minutes. Which would in theory mean that your user has to authenticate - read enter username and password - every hour. That is seriously uncool and there surely must be a better way. There is in fact. By using refresh tokens the app can renew the access token on the users behalf, without the user having to tap a thing.

## Refreshing

I covered how to integrate OIDC into a Xamarin Forms app in a former [post](https://mallibone/post/xamarin-oidc). In that post we see that, when we make the authentication request. The request where the user has to enter his username and password. We pass along the scopes we request. If we add the `offline_access` scope to the request and the server allows the scope request we will receive a refresh token. You can configure on the IdentityServer how long a refresh token is valid and if it is enabled.

CCCCODE

With the above configuration the `LoginResult` will contain a valid refresh token. Which is cool, but what now? When and how do we use this token? Let's first look at when we want to us the refresh token. The client (in our case the Xamarin Forms app) knows exacthly when the access token is about to expire with the `AccessTokenExpiration` property on the `LoginResult`. So we could check before every request if the token is still valid. Perhaps subtract a few seconds to be on the safe side.

CCCCODE

Another option would be to let the backend tell us that we no longer have a valid access token. We can do this by calling the API and if the call returns a [401 response](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes#4xx_client_errors) - we know that the access token is expired and we need will probably want to refresh the token.

CCCCODE

> There is another 400 code that is often associated with authentication and that is the [403](https://en.wikipedia.org/wiki/HTTP_403). Other than the 401, the 403 indicates that while the access token is valid the user does not have the right permission to access the resource. So if a user wanted to access some admin function, a 403 response would be expected.

Both methods work and it is up to you which one you prefer more. When the time has come to refresh the tokens we can request a new set of tokens as follows:

CCCCODE

By default we can only use the refresh token one time to request a new access token. Fortunately the result of the refresh contains not only a new access token but also a new refresh token. So we can request once more a new set of tokens should the time come.

Be aware that the refresh might also fail. The main reason usually being that the refresh token has already expired. In this case there is no way around it and you will have to present the user with the beautiful login dialog of yours. 😉

## Taking a closer look

XXX

Intro

When to refresh a token (expiration date/401 response)

Refresh

Taking a look at the refresh token, how to store it and know that you can revoke it

Conclusion