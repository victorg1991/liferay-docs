# Using OAuth 2 in the iOS Mobile SDK

The Liferay Mobile SDK for iOS lets you use OAuth 2 to authenticate to the 
portal. The following 3 OAuth 2 grant types are supported: 

-   [**Authorization Code (PKCE for native apps):**](https://oauth.net/2/grant-types/authorization-code/) 
    Redirects users to a page in their mobile browser where they must enter 
    their credentials. Following login, the browser redirects users back to the 
    mobile app. The app never accesses the credentials--it uses a token that can 
    be easily revoked. User credentials therefore can't be compromised via the 
    app. This type of authentication is also useful in cases where users may not 
    want to enter their credentials in the app. For example, users may not want 
    to enter their Twitter credentials directly in a 3rd-party Twitter app, 
    preferring instead to authenticate via Twitter's official site. Note that 
    the site you redirect to for authentication must have OAuth 2 implemented. 

-   [**Resource Owner Password:**](https://oauth.net/2/grant-types/password/) 
    Users authenticate by entering their credentials directly in the app. 

-   [**Client Credentials:**](https://oauth.net/2/grant-types/client-credentials/)
    Authenticates without requiring user interaction. This is useful when the 
    app needs to access its own resources, not those of a specific user. 

This tutorial shows you how to use these OAuth 2 grant types in your iOS app via 
the Mobile SDK. 

## Authorization Code (PKCE)

The Authorization Code grant type opens a mobile browser window containing the 
portal's login page, then redirects the user back to the app following login. 
You must define the redirect URL when implementing this in your app. To 
authenticate via the Authorization Code grant type, you must call the following 
`LROAuth2SignIn` method: 

    LROAuth2SignIn.signIn(withRedirectURL: URL, session: LRSession, clientId: String, 
        scopes: [String], callback: (LRSession?, Error?) -> Void) -> LROAuth2AuthorizationFlow

Here are descriptions of this method's parameters: 

-   `withRedirectURL`: the URL that the user is redirected to after successful 
    login in the browser. You must configure this URL in the portal via the 
    OAuth 2 Admin portlet, and 
    [associate the URL with the iOS app](https://developer.apple.com/documentation/uikit/core_app/communicating_with_other_apps_using_custom_urls). 
-   `session`: The session that you want to authenticate. It must have the 
    server property set. 
    <!-- What server property? -->
-   `clientId`: ID of the OAuth2 application. You can find this value in the 
    portal's OAuth 2 Admin portlet. 
-   `scopes`: The portal permissions to request. You can define a set of 
    permissions associated with an OAuth2 application in the portal's OAuth2 
    Admin portlet. Use this property to request a subset of those permissions. 
    <!-- Why doesn't the example app use this property? -->
-   `callback`: A function called with the result of the authentication. If 
    authentication succeeds, you receive a non-null session containing the 
    authentication; otherwise you receive an error. 

This `LROAuth2SignIn.signIn` method returns an `LROAuth2AuthorizationFlow` 
object, which represents an ongoing authentication request. You must save this 
as an `LROAuth2AuthorizationFlow` property in your `AppDelegate`. In your 
`AppDelegate`, you must then call the `LROAuth2AuthorizationFlow` method 
`resumeAuthorizationFlowWithURL` in the `application(_:open:options:)` method. 

Here's an example of this workflow: 

1.  Configure the redirect URL in the portal via the OAuth 2 Admin portlet. In 
    the portal, navigate to *Control Panel* &rarr; *OAuth 2 Admin* and select 
    the OAuth 2 application you want to use. Then enter the redirect URL in the 
    *Callback URIs* field. The redirect URL in this example is 
    `my-app://my-app`. 

    ![Figure 1: Enter the redirect URL in the OAuth 2 application in the portal.](../../../images/mobile-oauth2-redirect-url.png)

2.  In your iOS app, register your redirect URL via the *Info* tab in your 
    project's settings. For instructions on this, see the section *Register Your 
    URL Scheme* in 
    [Apple's documentation on using custom URLs](https://developer.apple.com/documentation/uikit/core_app/communicating_with_other_apps_using_custom_urls). 

    ![Figure 2: Register the redirect URL in your iOS app.](../../../images/ios-register-url.png)

3.  In your `AppDelegate`, create the `LROAuth2AuthorizationFlow` property. This 
    is the property that you'll set later when you call the 
    `LROAuth2SignIn.signIn` method:

        var authorizationFlow: LROAuth2AuthorizationFlow?

4.  In the view controller in which you'll call `LROAuth2SignIn.signIn`, define 
    the callback that runs with the authentication's result. In this example, if 
    the authentication succeeds, the callback prints a success message and calls 
    a sample method that tests the session's user credentials; otherwise it 
    prints an error message. Note that your callback can perform any action you 
    need it to: 

        let oauth2Callback: (LRSession?, Error?) -> Void = { session, error in
            if let session = session {
                print("Login successful")
                testCredentials(session: session)
            }
            else {
                print(error!)
            }
        }

5.  In the same view controller, call the `LROAuth2SignIn.signIn` method with 
    the above parameters. Then set the resulting `LROAuth2AuthorizationFlow` to 
    the `AppDelegate` property you created in step 3. This example does this in 
    a `loginWithRedirect()` method: 

        func loginWithRedirect() {
            let session = LRSession(server: LiferayServerContext.server)
            let redirectUrl = URL(string: "my-app://my-app")!
            let clientIdRedirect = "54321"

            let authorizationFlow = LROAuth2SignIn.signIn(withRedirectURL: redirectUrl,
                session: session, clientId: clientIdRedirect, scopes: [], callback: oauth2Callback)

            (UIApplication.shared.delegate as! AppDelegate).authorizationFlow = authorizationFlow
        }

6.  In your `AppDelegate`'s `application(_:open:options:)` method, call the 
    `LROAuth2AuthorizationFlow` method `resumeAuthorizationFlow` with the URL as 
    its argument. For more information on the `application(_:open:options:)` 
    method, see the section *Handle Incoming URLs* in 
    [Apple's documentation on using custom URLs](https://developer.apple.com/documentation/uikit/core_app/communicating_with_other_apps_using_custom_urls): 

        @UIApplicationMain
        class AppDelegate: UIResponder, UIApplicationDelegate {

            ...

            var authorizationFlow: LROAuth2AuthorizationFlow?

            func application(_ app: UIApplication, open url: URL,
                options: [UIApplicationOpenURLOptionsKey : Any] = [:]) -> Bool {

                if let authorizationFlow = authorizationFlow {
                    return authorizationFlow.resumeAuthorizationFlow(with: url)
                }
            }

            ...

        }

## Resource Owner Password

In the Resource Owner Password grant type, users authenticate by entering their 
credentials directly in the app. Implementing authentication for this grant type 
is similar to doing so for the PKCE grant type, except you don't need to 
configure a redirect URL. You instead handle the user's credentials directly in 
your iOS app via a slightly different `LROAuth2SignIn.signIn` method: 

    LROAuth2SignIn.signIn(withUsername: String, password: String, session: LRSession, clientId: String, 
        clientSecret: String, scopes: [String], callback: (LRSession?, Error?) -> Void) -> LRSession?

Compared to the `LROAuth2SignIn.signIn` method used for the PKCE grant type, 
this one requires the user's credentials instead of a redirect URL. It also 
requires the OAuth 2 application's client secret from the portal. 

Here are descriptions of this method's parameters: 

-   `withUsername`: the user's username. 
-   `password`: the user's password.
-   `session`: The session that you want to authenticate. It must have the 
    server property set. 
    <!-- What server property? -->
-   `clientId`: ID of the OAuth2 application. You can find this value in the 
    portal's OAuth 2 Admin portlet. 
-   `clientSecret`: The client secret of the OAuth 2 application. You can find 
    this value in the portal's OAuth 2 Admin portlet. 
-   `scopes`: The portal permissions to request. You can define a set of 
    permissions associated with an OAuth2 application in the portal's OAuth2 
    Admin portlet. Use this property to request a subset of those permissions. 
    <!-- Why doesn't the example app use this property? -->
-   `callback`: A function called with the result of the authentication. If 
    authentication succeeds, you receive a non-null session containing the 
    authentication; otherwise you receive an error. 

Note that you can call the `LROAuth2SignIn.signIn` method without a callback, 
instead passing `nil` as the `callback` argument. This causes the request to 
execute synchronously. If you provide a callback, the request is executed 
asynchronously in another thread and you receive the response in the callback. 

Follow these steps to call the `LROAuth2SignIn.signIn` method for the Resource 
Owner Password grant type: 

1.  If you want to provide a callback, define it now in the view controller in 
    which you'll call `LROAuth2SignIn.signIn`. In this example, if the 
    authentication succeeds, the callback prints a success message and calls a 
    sample method that tests the session's user credentials; otherwise it prints 
    an error message. Note that your callback can perform any action you need it 
    to: 

        let oauth2Callback: (LRSession?, Error?) -> Void = { session, error in
            if let session = session {
                print("Login successful")
                testCredentials(session: session)
            }
            else {
                print(error!)
            }
        }

2.  In the same view controller, call the `LROAuth2SignIn.signIn` method with 
    the above parameters. This example does this in a 
    `loginWithUsernameAndPassword()` method: 

        func loginWithUsernameAndPassword() {
            if password.isEmpty {
                fatalError("you have to enter the password")
            }

            let clientId = "12345"
            let clientSecret = "12345"

            _ = try? LROAuth2SignIn.signIn(withUsername: "test@liferay.com", password: password,
                        session: session, clientId: clientId, clientSecret: clientSecret, scopes: [], 
                        callback: oauth2Callback)
        }

## Client Credentials

The Client Credentials grant type in OAuth 2 authenticates without requiring 
user interaction. This is useful when the app needs to access its own resources, 
not those of a specific user. The iOS app authenticates via the OAuth 2 
application's client ID and client secret, which you can find in the OAuth 2 
application in the portal. 

+$$$

**Warning:** The Client Credentials grant type poses a security risk to the 
portal. To use this grant type, the mobile app must contain the OAuth 2 
application's client ID and client secret, which are used to authenticate 
without user credentials. Anyone who can access those values via the mobile app 
can also authenticate without user credentials. 

$$$

To authenticate with the Client Credentials grant type, you must call the 
`LROAuth2SignIn.signIn` method that lacks arguments for user credentials or 
redirect URLs: 

    LROAuth2SignIn.clientCredentialsSignIn(with: LRSession, clientId: String, 
        clientSecret: String, scopes: [String], callback: (LRSession?, Error?) -> Void)

Here are descriptions of this method's parameters: 

-   `with`: The session that you want to authenticate. It must have the 
    server property set. 
    <!-- What server property? -->
-   `clientId`: ID of the OAuth2 application. You can find this value in the 
    portal's OAuth 2 Admin portlet. 
-   `clientSecret`: The client secret of the OAuth 2 application. You can find 
    this value in the portal's OAuth 2 Admin portlet. 
-   `scopes`: The portal permissions to request. You can define a set of 
    permissions associated with an OAuth2 application in the portal's OAuth2 
    Admin portlet. Use this property to request a subset of those permissions. 
    <!-- Why doesn't the example app use this property? -->
-   `callback`: A function called with the result of the authentication. If 
    authentication succeeds, you receive a non-null session containing the 
    authentication; otherwise you receive an error. 

Note that you can call the `LROAuth2SignIn.signIn` method without a callback, 
instead passing `nil` as the `callback` argument. This causes the request to 
execute synchronously. If you provide a callback, the request is executed 
asynchronously in another thread and you receive the response in the callback. 

Follow these steps to call the `LROAuth2SignIn.signIn` method for the Client 
Credentials grant type: 

1.  If you want to provide a callback, define it now in the view controller in 
    which you'll call `LROAuth2SignIn.signIn`. In this example, if the 
    authentication succeeds, the callback prints a success message and calls a 
    sample method that tests the session's user credentials; otherwise it prints 
    an error message. Note that your callback can perform any action you need it 
    to: 

        let oauth2Callback: (LRSession?, Error?) -> Void = { session, error in
            if let session = session {
                print("Login successful")
                testCredentials(session: session)
            }
            else {
                print(error!)
            }
        }

2.  In the same view controller, call the `LROAuth2SignIn.signIn` method with 
    the above parameters. This example does this in a 
    `loginWithClientCredentials()` method: 

        func loginWithClientCredentials() {

            let clientId = "12345"
            let clientSecret = "12345"

            _ = try? LROAuth2SignIn.clientCredentialsSignIn(with: session, clientId: clientId, 
                        clientSecret: clientSecret, scopes: [], callback: oauth2Callback)
        }

## Related Topics

[Using OAuth 2 in Liferay Screens for iOS]()
