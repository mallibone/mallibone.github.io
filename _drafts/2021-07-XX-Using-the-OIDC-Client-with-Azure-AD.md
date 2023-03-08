---

layout: single
title: "Using the .NET OIDC Client in a .NET MAUI app with Azure AD"
date: 2023-01-02 00:03:00
tags: ["OIDC", "OAuth", "Azure AD", ".NET MAUI"]
slug: "oidc-client-aad"
---

The .NET OIDC Client is a great helper for solving the task of implementing an OAuth2/OIDC client LLLINK for a .NET app. Since it is based on standards one can use it for various Identity Services that are based on said standard. So how about Azure Active Directory aka AAD?

When working with Azure AD the usual choice is the Microsoft Authentication Library (MSAL) LLLINK, since it adds some functionality that is specific to Azure AD. But since Azure AD also supports OAuth2 or at least mostly, we can use the .NET OIDC Client.

## Setting up the mobile client

The basic Azure AD configuration for an application is independent of the library we choose. If we have some Ressource we can configure it for the OAuth Code-Flow LLLINK. With Azure AD configured we can configure the .NET MAUI app. First we will want to add the .NET OIDC client to the project. LLLINK

CCCCODE

Azure AD does mostly follow the OAuth2 standard. It does however use different URLs for the XXX and XXX, which is not according to the OAuth2 Standard. But we can take this under consideration when configuring the OIDC client in our client as follows:

CCCODE

With the configuration in place we still have to add a browser client to be used by the OIDC library. For this we add an additional class which implements the `IBrowser` interface:

CCCCODE

Now we can authenticate against Azure AD by instantiating a new authentication client using the above configuration and browser implementation.

CCCODE

With the basics met, lets look at some other topics that are just as relevant when implementing a mobile client.

## Refreshing Tokens

The number of mobile apps that require the user to authenticate are quite limited. Mobile banking comes to mind, but usually the user does not want to be bothered with having to authenticate on every start of the app. After the initial authentication we can request a Refresh Token from the identity service. With said token we can request a new set of tokens without the user having to authenticate.

The refresh token can be requested by adding the following claim to the configuration:

CCCCODE

With this change the authentication response now will include a Refresh Token. Which can be used to get a fresh set of tokens:

CCCODE

It usually is a good time to refresh the tokens once the access token expires. Or shortly before.

Logging out

The last part of this blog will focus on implementing a logout. Or sort of - Azure AD is. a bit special when it comes to logging out as in it does not work a straight forward way to implement for .NET MAUI / native apps. That being said should you have the requirement of a manual logout. The logout can be implemented as follows:

CCCCODE

After deleting the local tokens, the code opens up the system browser and allows the user to logout in the browser choosing the profile that should be logged out. As of writing there is unfortunately no easy way to simply logout the user directly from a call within the app.

## Conclusion

The .NET OIDC Client LLLLINK is a great helper library when it comes to implementing authentication via a OAuth/OIDC identity service into a .NET MAUI application. With a few modifications it can be adapted to be used with Azure AD. Note there are some differences that have to taken care of and some behaviour differs from other identity services that follow the OAuth 2 specs more diligently.

If you do not have any requirements that explicitly ask for the .NET OIDC Client be sure to check out the Microsoft Authentication Library (MSAL) since it is created to work with Azure AD. LLLINK

