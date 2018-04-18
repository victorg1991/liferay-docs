# Using OAuth 2 in Liferay Screens for iOS [](id=using-oauth-2-in-liferay-screens-for-ios)

Liferay Screens lets you use 
[OAuth 2](https://oauth.net/2/) 
for authentication with 
[Login Screenlet](/develop/reference/-/knowledge_base/7-0/loginscreenlet-for-ios). 
You can use the following 
[OAuth 2 grant types](https://oauth.net/2/grant-types/): 

-   [**Authorization Code (PKCE for native apps):**](https://oauth.net/2/grant-types/authorization-code/) 
    Redirects users to a page in their mobile browser where they enter their 
    credentials. Following login, the browser redirects users back to the mobile 
    app. User credentials can't be compromised via the app because it never 
    accesses them--it uses a token that can be easily revoked. This is also 
    useful if users don't want to enter their credentials in the app. For 
    example, users may not want to enter their Twitter credentials directly in a 
    3rd-party Twitter app, preferring instead to authenticate via Twitter's 
    official site. Note that the site you redirect to for authentication must 
    have OAuth 2 implemented. 

-   [**Resource Owner Password:**](https://oauth.net/2/grant-types/password/) 
    Users authenticate by entering their credentials directly in the app. 

-   [**Client Credentials:**](https://oauth.net/2/grant-types/client-credentials/)
    Authenticates without requiring user interaction. This is useful when the 
    app needs to access its own resources, not those of a specific user. 

This tutorial shows you how to use all 3 of these OAuth 2 grant types. 

## Authorization Code (PKCE) [](id=authorization-code-pkce)

Follow these steps to use the Authorization Code grant type with Login 
Screenlet: 

1.  Set Login Screenlet's `loginMode` attribute to `oauth2Redirect`. There are 2 
    ways to do this: 

    -   In code, as the Login Screenlet instance's `authType` or `loginMode` 
        property:

            loginScreenlet.authType = .oauth2Redirect
            // or
            loginScreenlet.loginMode = "oauth2redirect"

        Note that `oauth2redirect` must be a string when set to `loginMode`. 

    -   In Interface Builder, as the value of the *Login Mode* attribute. You do 
        this the same way you set other Screenlet attributes (via the Attributes 
        inspector, with the Screenlet selected in the storyboard). Be sure to 
        enter `oauth2redirect` with no period preceding it. 

2.  Set Login Screenlet's `oauth2clientId` attribute to the ID of the portal's 
    OAuth 2 application. You can find this value in the portal's OAuth 2 Admin 
    portlet. 

3.  Set Login Screenlet's `oauth2redirectUrl` attribute to the URL that the 
    mobile browser will redirect the user to after successful login. You must 
    configure this in the portal's OAuth 2 Admin portlet, and associate the URL 
    with the application. 

4.  Notify Liferay Screens when the redirect has been performed. Do this in your 
    `AppDelegate` as follows: 

        func application(_ app: UIApplication, open url: URL, 
            options: [UIApplicationOpenURLOptionsKey : Any] = [:]) -> Bool {
                return SessionContext.oauth2ResumeAuthorization(url: url)
        }

Note that you can cancel the authorization at any time by calling 
`SessionContext.oauth2Cancel()`. 

## Resource Owner Password [](id=resource-owner-password)

Follow these steps to use the Resource Owner Password grant type with Login 
Screenlet: 

1.  First, you must set Login Screenlet's `loginMode` attribute to 
    `oauth2UsernameAndPassword`. There are 2 ways to do this: 

    -   In code, as the Login Screenlet instance's `authType` or `loginMode` 
        property:

            loginScreenlet.authType = .oauth2UsernameAndPassword
            // or
            loginScreenlet.loginMode = "oauth2UsernameAndPassword"

        Note that `oauth2UsernameAndPassword` must be a string when setting 
        `loginMode`. 

    -   In Interface Builder, as the value of the *Login Mode* attribute. You do 
        this the same way you set other Screenlet attributes (via the Attributes 
        inspector, with the Screenlet selected in the storyboard). Be sure to 
        enter `oauth2UsernameAndPassword` with no period preceding it. 

2.  Set Login Screenlet's `oauth2clientId` attribute to the ID of the OAuth 2 
    application in the portal. You can find this value in the portal's OAuth 2 
    Admin portlet. 

3.  Set Login Screenlet's `oauth2clientSecret` attribute to the client secret of 
    the portal's OAuth 2 application. You can find this value in the portal's 
    OAuth 2 Admin portlet. 

## Client Credentials [](id=client-credentials)

The Client Credentials grant type in OAuth 2 authenticates without requiring 
user interaction. This is useful when the app needs to access its own resources, 
not those of a specific user. 

+$$$

**Warning:** The Client Credentials grant type poses a security risk to the 
portal. To use this grant type, the mobile app must contain the OAuth 2 
application's client ID and client secret, which are used to authenticate 
without user credentials. Anyone who can access those values via the mobile app 
can also authenticate without user credentials. 

$$$

Follow these steps to use the Client Credentials grant type in your Screens app: 

1.  Follow the 
    [iOS Mobile SDK instructions](/develop/tutorials/-/knowledge_base/7-0/using-oauth-2-in-the-ios-mobile-sdk#client-credentials) 
    for using the Client Credentials grant type. 

2.  The session object's `authentication` property now contains a valid 
    authentication object. Cast it to `LROAuth2Authentication` and pass the 
    result to the `authentication` argument of the `SessionContext` method 
    `loginWithOAuth2`: 

        let auth = session.authentication as! LROAuth2Authentication

        SessionContext.loginWithOAuth2(authentication: auth, userAttributes: [:])

    This initializes the Screens `SessionContext` object, authenticating any 
    Screenlets that you use in the iOS app. 

## Related Topics [](id=related-topics)

[Using OAuth 2 in the iOS Mobile SDK](/develop/tutorials/-/knowledge_base/7-0/using-oauth-2-in-the-ios-mobile-sdk)

[Using Screenlets in iOS Apps](/develop/tutorials/-/knowledge_base/7-0/using-screenlets-in-ios-apps)
