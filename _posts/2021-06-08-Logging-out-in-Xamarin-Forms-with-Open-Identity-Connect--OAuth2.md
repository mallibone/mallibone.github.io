---
layout: single
title: "Logging out in Xamarin Forms with Open Identity Connect / OAuth2"
date: 2021-06-08 11:03:00
tags: ["Xamarin", "Xamarin Forms", "OIDC", "OIDC Client", "OAuth", "OAuth2"]
slug: "xamarin-oidc"
---

Who would have thought that logging out of your Xamarin Forms app using Open Identity Connect (OIDC) and OAuth would deserve its own blog post? But it just so happens that it took me quite a bit longer than I initially thought it would take. So if you are interested in OAuth and Xamarin, you might also be interested in my blog posts on [getting started](https://mallibone.com/post/xamarin-oidc), [setting up an IdentityServer](https://mallibone.com/post/xamarin-identity-server) for your mobile app and [refreshing your tokens](https://mallibone.com/post/xamarin-oidc-refresh).

But back to logging out - first of all, let's look at what makes a logged-in user differ from one that is not. It is not that much. It comes down to having an access token or not. So, in theory, if we remove the access token from the app and the HttpClient header, we can no longer make any requests to an API without receiving a [401](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes#4xx_client_errors). Hence, we could implement the logout action as follows:

<!--more-->

```c#
private async void Logout()
{
    _credentials = null;
    // ...
}
```

With that, we can no longer have a valid token to call any API. Consequently, the user is forced to log in again. But when open the browser for logging in, something strange happens:

![Logout fails to logout]({{ site.url }}{{ site.baseurl }}/assets/images/2021-06-Logout-Fail.gif "Logout fails to logout")

What just happened?! We would have expected a login dialogue to appear, but it seems the user magically logs back in again. While a cool party trick, if you have a shared device and users should log in and out with different accounts, this is not what we had in mind. So how did we end up here?

Mobile apps are encouraged to use the OAuth2 code flow. The code flow will open a browser where the users enter their credentials, and the server will then call back to the client via a defined callback URL. It is said browser that is responsible for the effects we just have witnessed. There is a session cookie within the browser that "remembers" the user and will not prompt for credentials as long as the session cookie is still valid. Cool, but what does this imply for us?

In short, in addition to deleting the token and cleaning up the request headers, we will also have to log out via the browser. We can do this as we did the login using the [OIDC client](https://www.nuget.org/packages/IdentityModel.OidcClient/) library.

```c#
public async Task<LogoutResult> Logout(string? identityToken)
{
        OidcClient oidcClient = CreateOidcClient();
        LogoutResult logoutResult = await oidcClient.LogoutAsync();
        return logoutResult;
}
```

When we invoke this code on logout, the user will then be requested to confirm the logout in the browser, and he also has to close the browser window manually.

![Logout works in theory but not a great UX experience]({{ site.url }}{{ site.baseurl }}/assets/images/2021-06-Logout-Workingish.gif "Logout works in theory but not a great UX experience.")

It works(TM), but it feels a bit yucky. For one, the user already confirmed the logout in the app. And the browser window does not go away automatically. Perhaps some users will not even notice how to close the browser window at first sight. So let's improve that flow.

## Configuring IdentityServer

The Identity Server also has to be configured accordingly to enable this, so let's start there. But, first, we will want to allow automatic redirects by allowing them in the `AccountOptions` class:

```c#
public class AccountOptions
{
    // More account options
    public static bool AutomaticRedirectAfterSignOut = true;
}
```

Then in the configuration (`Config` class), we need to define a post logout URL:

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

Note that you can define it as you like. You must make sure that this deep link will work on your mobile platform. If possible, I would recommend using the same prefix as the with the login redirect. And on iOS, you will want to update the `info.plist` file. But before we go to the mobile client, there is still one adjustment in the Logout method in the `AccountController`:

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

With these adoptions to the IdentityServer, we can now focus on the client.

## Configuring the client

The OIDC Client provides the `LogoutAsync`, which will open the browser to log out. To improve the user experience, we will want to define the `PostLogoutRedirectUri` in the client configuration:

```c#
var options = new OidcClientOptions
{
  // More options ...
  PostLogoutRedirectUri = "oidcxamarin101:/signout-callback-oidc",
  Browser = new WebAuthenticatorBrowser()
};
```

Note this is the same URL we already specified on the server. Furthermore, note that this will be a deep link into your app. Since the URL base stays the same, we will only have to register the callback on iOS in the info.plist file:

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

The last thing left to do is to pass along the ID Token when we invoke the logout request:

```c#
public async Task<LogoutResult> Logout(string? identityToken)
{
        OidcClient oidcClient = CreateOidcClient();
        LogoutResult logoutResult = await oidcClient.LogoutAsync(new LogoutRequest{IdTokenHint = identityToken});
        return logoutResult;
}
```

When the user now logs out and then tries to log in again. Everything works as you might have expected.

![Working logout]({{ site.url }}{{ site.baseurl }}/assets/images/2021-06-Logout-Final-Solution.gif "Working Logout")



## Aftermath

While the code provides additional security and decreases the number of attack vectors, it also provides a few exciting considerations. For one, your app will remember the authentication and the browser used on the system the authentication happened. This is also why we will want to open a browser when logging out because it is the system browser that also has to delete the session or see that nice effect from the beginning of the blog post.

Don't forget to delete the tokens you stored since they might still be valid and lead to unintended side effects.

You can find a sample application with all the code on [GitHub](https://github.com/mallibone/XamarinIdentity101).

HTH