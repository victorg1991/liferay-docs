# iOS Best Practices [](id=ios-best-practices)

This document includes best practices that every iOS developer using Liferay
Screens should follow. This are good practices in order to achieve readability,
and get cleaner and less buggy code. For code style conventions (on Swift),
please refer to the [Swift Style Guide](https://github.com/liferay/liferay-screens/blob/master/ios/swift-style-guide.md).

## Naming Convention [](id=ios-naming-convention)

All your components should follow the same naming convention. Not only in order
to packaging or distributing them, but also for a better understanding of the
Screens library. If you get used to this nomenclature, it will be much easier
to locate and understand files in Screens.

### Screenlet folder

First of all, all Screenlet components
have to be placed inside the Screenlet folder, which should be named in
accordance to its content (there are a few exceptions that we'll review later).

For example the comment add Screenlet is placed inside a folder named *"Add"*,
which is in another folder named *"Comment"*, so when you read it, you can
understand the contents of that folder.

    object/action/action...

### Screenlets

Screenlets are the main component of the Screens library, and for that, naming
them is a matter of great importance. Screenlets should (class and file) should
be named referencing its (principal) action as prefix and always ending it with
the word *"Screenlet"*. So, for example, a Screenlet uses to login users is a
`LoginScreenlet` or a Screenlet used to display an image gallery is a
`ImageGalleryScreenlet`.

    ActionScreenlet

### View Models

ViewModels should be placed along with the Screenlet (in the Screenlet root
folder) and with the same name as the Screenlet but changing the word
`Screenlet` for `ViewModel`. For example, the `ForgotPasswordScreenlet` has a
`ForgotPasswordViewModel`.

    ActionViewModel

### Interactors

Interactors should be placed in a folder name `Interactors` inside the Screenlet
root folder. Its name should indicate the use case they handle, and should have
in it the name of the object of the interaction (which usually matches the
Screenlet class prefix), and end it with the word *"Interactor"*. For example,
`RatingScreenlet` has three interactors inside the `Interactors` folder:
`UpdateRatingInteractor`, `DeleteRatingInteractor` and `LoadRatingInteractor`.

    Interactors/ActionObjectInteractor
    Interactors/ObjectActionInteractor

### Connectors

Connectors should be named exactly with the same rules as the Interactors, but,
changing the word *"Interactor"* for *"Connector"*. In the case that the
connector perform a call to a Liferay Service, the *"Connector"* suffix should
be preceded by the word *"Liferay"*. So, for example, the connector for adding
comments to a Liferay instance is called `CommentAddLiferayConnector`, but, a
connector used to get the title from a webpage should be called:
`GetWebsiteTitleConnector`.

    Connectors/ActionObject(Liferay)Connector
    Connectors/ObjectAction(Liferay)Connector

### Views/Themes

Views should be put in a *"Views"* or *"Themes"* folder inside the Screenlet
root folder. Except when you are creating several views of the same Theme, for
different Screenlets. In that case you should place all your views inside your
Theme's folder. Views (both XIBs and classes) should be called like the
Screenlet, but changing the word *"Screenlet"* for *"View"* and adding the
following suffix: *"_themeName"*, where `themeName` is the name of the theme of
this view, the default suffix should be *"_default"*.

    Views/ActionView_themeName
    Default/ActionView_default
    Blue/ActionView_blue

## Avoid Hard Coded Elements [](id=avoid-hard-coded-elements)

Bugs in hard coded elements are really difficult to track down so you should avoid
using them. And how to do that? By using constants. Whenever you need to use
a hard coded string, int, float... you should put a constant instead,
as simply as that.

A good example for this, is the name of the actions, which (if you follow the
[Creating iOS Screenlets (Advanced) tutorial](/develop/tutorials/-/knowledge_base/7-0/creating-ios-screenlets-advanced#ios-adding-another-use-case))
you should know they are established by the `restorationIdentifier` of a UI
component. So, for this case it's better to create a constant inside
Screeenlet's class referencing the action, and use that constant inside the
`didSet` of the outlet of your Screenlet inside the view than hard coding it
inside the Interface Builder. For example:

Instead of doing this:

![Figure 1: Don't hard code restoration identifiers.](../../../images/screens-ios-harcoded-strings.png)

Do this:

    public class AddBookmarkScreenlet: BaseScreenlet {
        
        static let AddBookmarkAction = "add-bookmark"
        ...
    }

    class AddBookmarkView_default: BaseScreenletView, AddBookmarkViewModel {

        @IBOutlet weak var addBookmarkButton: UIButton? {
            didSet {
                addBookmarkButton?.restorationIdentifier = AddBookmarkScreenlet.AddBookmarkAction
            }
        }

        ...
    }

Another good example would be the Screenlets `@IBInspectables` properties.
Setting this properties directly in a Storyboard or a XIB file is great when you
are testing, but, they are not so great when you are debugging why your
Screenlet is using a wrong id. In order to avoid this kind of bugs, you should
add all your ids, classnames... to the .plist file that sets the server context.
You should be familiar with this file if you followed the
[Configuring Communication with Liferay chapter](/develop/tutorials/-/knowledge_base/7-0/preparing-ios-projects-for-liferay-screens#configuring-communication-with-liferay)
of the [Preparing iOS Projects for Liferay Screens](/develop/tutorials/-/knowledge_base/7-0/preparing-ios-projects-for-liferay-screens).

So, for example, your .plist file could look like this:

![Figure 2: Don't hard code ids inside IB.](../../../images/screens-ios-harcoded-ids.png)

And how can you retrieve those ids? You can retrieve them by using the method
`propertyForKey` of the class `LiferayServerContext`. For example, doing
something like this:

    @IBOutlet weak var imageGalleryScreenlet: ImageGalleryScreenlet? {
        didSet {
            imageGalleryScreenlet?.delegate = self
            imageGalleryScreenlet?.presentingViewController = self

            imageGalleryScreenlet?.repositoryId = LiferayServerContext.groupId
            if let folderId = LiferayServerContext.propertyForKey("galleryFolderId") as? NSNumber {
                imageGalleryScreenlet?.folderId = folderId.longLongValue
            }
        }
    } 