# Using OAuth 2 in Liferay Screens for iOS

Liferay Screens lets you use 
[OAuth 2](https://oauth.net/2/) 
for authentication with 
[Login Screenlet](/develop/reference/-/knowledge_base/7-0/loginscreenlet-for-ios). 
You can do this via 
[Login Screenlet's attributes](/develop/reference/-/knowledge_base/7-0/loginscreenlet-for-ios#attributes). 
Specifically, you must set the `loginMode` attribute's value to one of these 
values, depending on the authentication type you want to use: 

-   `oauth2Redirect`: This specifies the 
    [OAuth 2 Authorization Code grant](https://oauth.net/2/grant-types/authorization-code/). 
    This redirects users to a page in their mobile browser where they must enter 
    their credentials. Following login, the browser redirects users back to the 
    mobile app. The app never accesses the credentials--it uses a token that can 
    be easily revoked. User credentials therefore can't be compromised via the 
    app. This type of authentication is also useful in cases where users may not 
    want to enter their credentials in the app. For example, users may not want 
    to enter their Twitter credentials directly in a 3rd-party Twitter app, 
    preferring instead to authenticate via Twitter's official site. Note that 
    the site you redirect to for authentication must have OAuth 2 implemented. 

-   `oauth2UsernameAndPassword`: This specifies the 
    [OAuth 2 Resource Owner Password grant](https://oauth.net/2/grant-types/password/). 
    Users authenticate by entering their credentials directly in the app. 

The following sections describe how to implement these authentication types in 
your app. 

## oauth2Redirect

Follow these steps to set and use `oauth2Redirect`: 

1.  First, you must set `oauth2Redirect`. There are 2 ways to do this: 

    -   In code, as the Login Screenlet instance's `authType` or `loginMode` 
        property:

            loginScreenlet.authType = .oauth2Redirect
            // or
            loginScreenlet.loginMode = "oauth2redirect"

        Note that `oauth2redirect` must be a string when setting the `loginMode` 
        property. 

    -   In Interface Builder, as the value of the *Login Mode* attribute. You 
        set this in Interface Builder the same way you set other Screenlet 
        attributes (via the Attributes inspector, with the Screenlet selected in
        the storyboard). Be sure to enter `oauth2redirect` with no period 
        preceding it. 

2.  Set Login Screenlet's `oauth2clientId` attribute to the ID of the OAuth 2 
    application. You can find this value in the portal's OAuth 2 Admin portlet. 

3.  Set Login Screenlet's `oauth2redirectUrl` attribute to the URL that the 
    mobile browser will redirect the user to after successful login. You must 
    configure this in the portal's OAuth 2 Admin portlet, and associate the URL 
    with the application. 

4.  You must also notify Liferay Screens when the redirect has been performed. 
    Do this in your `AppDelegate` as follows: 

        func application(_ app: UIApplication, open url: URL, 
            options: [UIApplicationOpenURLOptionsKey : Any] = [:]) -> Bool {
                return SessionContext.oauth2ResumeAuthorization(url: url)
        }

Note that you can cancel the authorization at any time by calling 
`SessionContext.oauth2Cancel()`. 

## oauth2UsernameAndPassword

Follow these steps to set and use `oauth2UsernameAndPassword`: 

1.  First, you must set `oauth2UsernameAndPassword`. There are 2 ways to do 
    this: 

    -   In code, as the Login Screenlet instance's `authType` or `loginMode` 
        property:

            loginScreenlet.authType = .oauth2UsernameAndPassword
            // or
            loginScreenlet.loginMode = "oauth2UsernameAndPassword"

        Note that `oauth2UsernameAndPassword` must be a string when setting the 
        `loginMode` property. 

    -   In Interface Builder, as the value of the `loginMode` attribute. You set 
        this in Interface Builder the same way you set other Screenlet 
        attributes. 

2.  Set Login Screenlet's `oauth2clientId` attribute to the ID of the OAuth 2 
    application. You can find this value in the portal's OAuth 2 Admin portlet. 

3.  Set Login Screenlet's `oauth2clientSecret` attribute to the client secret of 
    the OAuth 2 application. You can find this value in the portal's OAuth 2 
    Admin portlet. 

That's it! Now you know how to use OAuth 2 in Liferay Screens. 

## Related Topics

[Using OAuth 2 in the iOS Mobile SDK]()

[Using Screenlets in iOS Apps](/develop/tutorials/-/knowledge_base/7-0/using-screenlets-in-ios-apps)
