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

Here are this method's parameters: 

-   `withRedirectURL`: the URL that the user is redirected to after successful 
    login in the browser. You must also configure this URL in the portal via the 
    OAuth 2 Admin portlet, and associate the URL with the application. 
    <!-- Which application? -->
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

1.  In your `AppDelegate`, create the `LROAuth2AuthorizationFlow` property. This 
    is the property that you'll set later when you call the 
    `LROAuth2SignIn.signIn` method:

        var authorizationFlow: LROAuth2AuthorizationFlow?

2.  In the view controller in which you'll call `LROAuth2SignIn.signIn`, define 
    the callback that runs with the authentication's result. In this example, if 
    the authentication succeeds, the callback prints a success message and calls 
    a sample method that tests the session's user credentials; otherwise it 
    prints an error message: 

        let oauth2Callback: (LRSession?, Error?) -> Void = { session, error in
            if let session = session {
                print("Login successful")
                testCredentials(session: session)
            }
            else {
                print(error!)
            }
        }

    Keep in mind that your callback can perform any action you need it to. 

3.  In the same view controller, call the `LROAuth2SignIn.signIn` method with 
    the above parameters. Then set the resulting `LROAuth2AuthorizationFlow` to 
    the `AppDelegate` property you created in step 1. This example does this in 
    a `loginWithRedirect()` method: 

    func loginWithRedirect() {
        let session = LRSession(server: LiferayServerContext.server)
        let redirectUrl = URL(string: "my-app://my-app")!
        let clientIdRedirect = "54321"

        let authorizationFlow = LROAuth2SignIn.signIn(withRedirectURL: redirectUrl,
            session: session, clientId: clientIdRedirect, scopes: [], callback: oauth2Callback)

        (UIApplication.shared.delegate as! AppDelegate).authorizationFlow = authorizationFlow
    }

4.  In your `AppDelegate`, call the `LROAuth2AuthorizationFlow` method 
    `resumeAuthorizationFlow` in the `application(_:open:options:)` method: 

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


