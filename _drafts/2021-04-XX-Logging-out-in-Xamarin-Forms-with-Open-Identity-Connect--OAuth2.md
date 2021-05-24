---
layout: single
title: "Logging out in Xamarin Forms with Open Identity Connect / OAuth2"
date: 2021-05-01 15:03:00
tags: ["Xamarin", "Xamarin Forms", "OIDC", "OAuth2"]
slug: "xamarin-oidc"
---

Who would have thought that logging out in your Xamarin.Forms app when using Open Identity Connect (OIDC) and OAuth would deserve it's own blogpost. But it just so happens that it took me quite a bit longer than I initially thought it would take. If you are interested in OAuth and Xamarin you might also be interested in my blog posts on [getting started](https://mallibone.com/post/xamarin-oidc), [setting up an IdentityServer](https://mallibone.com/post/xamarin-identity-server) for your mobile app and [refreshing your tokens](https://mallibone.com/post/xamarin-oidc-refresh ). 

But back to logging out - first of all let's have a look at what makes a logged in user differ from one that is not. It is actually not that much. Basically it comes down to having a access token or not. So in theory if we simply remove the access token from the app and from the HttpClient header we can no longer make any requests to an API without receiving a [401](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes#4xx_client_errors). We could implement the logout action as follows:

```c#
private async void Logout()
{
    _credentials = null;
    // ...
}
```

With that we can no longer have a valid token to call any API. The user is forced to log in again. But when open the browser for logging in somethign strange happens:

GGGIFFF

What just happened?! We would have expected a login dialog to appear but it seems the user is just magically logged back in again. While a cool party trick if you have a shared device and users should log in and out with different accounts, this is deffinitley not what we had in mind. So how did we end up here?

It is recommended that mobile apps use the OAuth2 code flow. The code flow will open a browser where the users enters his credentials and the server will then call back to the client via a defined callback URL. It is said browser that is responsible for the effects we just have witnessed. Within the browser there is a session cookie which "remembers" the user and will not prompt for credentials as long as the session cookie is still valid. Cool, but what does this imply for us?

In short additionally to deleting the token and cleaning up the request headers, we will also have to logout via the browser. We can do this in a similar fashion as we did the login using the [OIDC client](https://www.nuget.org/packages/IdentityModel.OidcClient/) library.

```c#
public async Task<LogoutResult> Logout(string? identityToken)
{
        OidcClient oidcClient = CreateOidcClient();
        LogoutResult logoutResult = await oidcClient.LogoutAsync();
        return logoutResult;
}
```

When we invoke this code on logout the user will then be requested to confirm the logout in the browser and he also has to manually close the browser window.

GGGIFFF

It works(tm), but it feels a bit yucky. For one the user already confirmed the logout in the app. And the browser window does not close automatically. Perhaps some users will not even notice how to close the browser window at first sight. So let's improve that flow.

## Configuring IdentityServer

To enable this the Identity Server also has to be configured accordingly so let's start there. First we will want to allow automatic redirects which is configured in the `AccountOptions` class:

```c#
public class AccountOptions
{
    // More account options
    public static bool AutomaticRedirectAfterSignOut = true;
}
```

Then in the configuration (`Config` class) we need to define a postlogout URL:

```c#
// mobile client
new Client
{
    // Other configuration itmes
    PostLogoutRedirectUris = new List<string>
    {
        "oidcxamarin101:/signout-callback-oidc",
    },
    // More configuration items
}
```

Note that you can define it as you like. Important is that you make sure that this deep link will work on your mobile platform. If possible I would recommend using the same prefix as the with the login redirect. And on iOS you will want to update the  `info.plist` file. But before we go to the mobile client there is still one adjustment in the `Logout` method in the `AccountController`:

```c#
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Logout(LogoutViewModel model)
{
    // ...

    return vm.AutomaticRedirectAfterSignOut &&
           !string.IsNullOrWhiteSpace(vm.PostLogoutRedirectUri)
        ? (IActionResult) Redirect(vm.PostLogoutRedirectUri)
        : View("LoggedOut", vm);
}

```

With these adoptions to the IdentityServer we can now focus on the client.

## Configuring the client

The OIDC Client provides a method XXX which takes all steps necessary. Since we are using the code flow the logout will happen in the browser of the mobile client. We can reuse the browser implementation used for the login LLLINK. The one thing we want to add to our configuraiton is the `PostLogoutRedirectUri` in the client configuration:

```c#
var options = new OidcClientOptions
{
  // More options ...
  PostLogoutRedirectUri = "oidcxamarin101:/signout-callback-oidc",
  Browser = new WebAuthenticatorBrowser()
};
```

Note this is the same URL we already specified on the server. Also note that this will be a deep link into your app. Since the base of the URL stays the same we will only have to register the callback on iOS in the info.plist file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<plist version="1.0">
  <dict>
  <!-- Other Plist entries -->
    <key>CFBundleURLTypes</key>
    <array>
      <dict>
        <key>CFBundleURLName</key>
        <string>authentication</string>
        <key>CFBundleURLSchemes</key>
        <array>
          <!-- login redirect -->
          <string>oidcxamarin101:/authenticated</string>
          <!-- logout redirect -->
          <string>oidcxamarin101:/signout-callback-oidc</string>
        </array>
        <key>CFBundleTypeRole</key>
        <string>Editor</string>
      </dict>
    </array>
  </dict>
</plist>
```

We could leave it at this. But the user will always be prompted in the webview if the logout was intended. This may be exactly what you are looking for. However, should you be looking for a way to logout when tapping logout, we still have to forward the ID Token with the logout request:

```c#
public async Task<LogoutResult> Logout(string? identityToken)
{
        OidcClient oidcClient = CreateOidcClient();
        LogoutResult logoutResult = await oidcClient.LogoutAsync(new LogoutRequest{IdTokenHint = identityToken});
        return logoutResult;
}
```

When the user now logs out and then tries to login again. Everything works as you might have expected.

GGGGIFFFF



## Aftermath

While the code provides additional security and decreases the number of attack vectors, it also provides a few interesting considrations. For one not only your app will remember the authentication but also the browser used on the system the authentication happened. This is also why we will want to open a browser when logging out, because it is the system browser which also has to delete the session or we will see that nice effect from the beginning of the blog post.

Remember to still delete the tokens you stored locally since they might still be valid.

You can find a sample application with all the code on GitHub.

HTH