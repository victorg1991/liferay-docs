# Creating iOS Screenlets [](id=creating-ios-screenlets)

The built-in
[Screenlets](/develop/reference/-/knowledge_base/7-0/screenlets-in-liferay-screens-for-ios)
cover common use cases for mobile apps that use Liferay. They
authenticate users, interact with Dynamic Data Lists, view assets, and more.
What if, however, there's no Screenlet for *your* use case? No problem! You can
create your own. Extensibility is a key strength of Liferay Screens. 

This tutorial explains how to create your own Screenlets. As an example, it
references code from the sample
[Add Bookmark Screenlet](https://github.com/liferay/liferay-screens/tree/master/ios/Samples/Bookmark), 
that saves bookmarks to Liferay's Bookmarks portlet.

To understand the components that comprise a Screenlet, you might want to first
analyze the
[architecture](/develop/tutorials/-/knowledge_base/7-0/architecture-of-liferay-screens-for-ios)
of Liferay Screens for iOS. Without further ado, let the Screenlet creation
begin! 

## Determining Your Screenlet's Location [](id=determining-your-screenlets-location)

Where you should create your Screenlet depends on how you plan to use it. If you
want to reuse or redistribute it, you should create it in an empty Cocoa Touch
Framework project in Xcode. You can then use CocoaPods to publish it. The
section
[Publish Your Themes Using CocoaPods](/develop/tutorials/-/knowledge_base/7-0/creating-ios-themes#publish-your-themes-using-cocoapods)
in the tutorial
[Creating iOS Themes](/develop/tutorials/-/knowledge_base/7-0/creating-ios-themes)
explains how to publish an iOS Screenlet. Even though that section refers to
Themes, the steps for preparing Screenlets for publishing are the same. If you
don't plan to reuse or redistribute your Screenlet, create it in your app's
Xcode project.

## Analyzing Your Screenlet's use cases [](id=analyzing-your-screenlets-use-cases)

The first step when creating a Screenlet is asking ourselves: What should my Screenlet do?

In this case, the answer is simple, our screenlet should add bookmark entries to a folder in our Liferay Portal. With that in mind, we can draw the following conclusions:

-  Our Screenlet will have to respond only to one **action**: add a bookmark.
-  If you take a look at the service used to add a bookmark, you'll see that the Screenlet needs, at least, the following **parameters**: on one hand, the id of the bookmark folder and the group id, which the user won't know. On the other hand, the name and url of the bookmark, which will have to be provided by the user.

## Creating the Screenlet [](id=creating-the-screenlet)

The creation of a Screenlet is an easy process and can be done in the following steps.

### Creating the Screenlet class
In Xcode, create a Screenlet class that extends
    [`BaseScreenlet`](https://github.com/liferay/liferay-screens/blob/master/ios/Framework/Core/Base/BaseScreenlet.swift) and include, optionally, any `@IBInspectable` properties you need. The name of this class should follow the [Naming Convention](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices#ios-naming-convention) specified in the [Best Practices](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices).

For example, let's create the class `AddBookmarkScreenlet`. In the previous [chapter](#analyzing-your-screenlets-use-cases) we conclude that the Screenlet will need the groupId and the bookmark folderId. The first one can be obtained from the file created in the section [Configuring Communication with Liferay](/develop/tutorials/-/knowledge_base/7-0/preparing-ios-projects-for-liferay-screens#configuring-communication-with-liferay) of the tutorial [Preparing iOS Projects for Liferay Screens](/develop/tutorials/-/knowledge_base/7-0/preparing-ios-projects-for-liferay-screens). For the second one we will create a `folderId` property in the Screenlet which can be declared as `@IBInspectable` to facilitate the testing of the Screenlet.
    
    import UIKit
    import LiferayScreens
    
    
    //Screenlet used for adding bookmarks to a certain folder
    public class AddBookmarkScreenlet: BaseScreenlet {
        
    	@IBInspectable var folderId: Int64 = 0
    
    }

### Creating the Screenlet View

Create a new XIB file and construct your Screenlet's UI with Interface Builder. The name of this class should follow the [Naming Convention](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices#ios-naming-convention) specified in the [Best Practices](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices).

Almost each *action* received by the Screenlet has to be triggered from the view. In order to achieve that, make sure to use a `restorationIdentifier` property to assign an unique ID to each UI component that triggers an action. That ID will be the action name that we recover in the Screenlet class. That action will be triggered "automagically" once an user interacts with the component. If the action only needs to change the UI's state (that is, change the component's properties), then you can associate that component's event to an `IBAction` method as usual. Actions using `restorationIdentifier` are intended for use by actions that need an Interactor (such as actions that make server requests or retrieve data from a database). 

For example, the Add Bookmark Screenlet's XIB file [`AddBookmarkView_default.xib`](https://github.com/liferay/liferay-screens/tree/master/ios/Samples/Bookmark/AddBookmarkScreenlet/Basic/Themes/AddBookmarkView_default.xib) specifies text box fields for a bookmark's URL and title and a button (with `restorationIdentifier="add-bookmark"`) to save the bookmark. 

![Figure 1: Here's the sample Add Bookmark Screenlet's XIB file rendered in Interface Builder.](../../../images/screens-ios-xcode-add-bookmark.png)

Create a new View class that extends from [`BaseScreenletView`](https://github.com/liferay/liferay-screens/blob/master/ios/Framework/Core/Base/BaseScreenletView.swift). The name of this class should follow the [Naming Convention](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices#ios-naming-convention) specified in the [Best Practices](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices). Use standard `@IBOutlet`s and `@IBAction`s to wire in all the UI components and events from the XIB.

Implement getters and setters to get and set values from the UI components. Also make sure to implement any required animations or front-end logic. 

For example, the sample Screenlet's [`AddBookmarkView_default`](https://github.com/liferay/liferay-screens/tree/master/ios/Samples/Bookmark/AddBookmarkScreenlet/Basic/Themes/AddBookmarkView_default.swift) keeps references to the UI components and implements getters for them, in order to be able to recover the user data from the Screenlet.

    import UIKit
    import LiferayScreens

    class AddBookmarkView_default: BaseScreenletView {

	   @IBOutlet weak var URLTextField: UITextField?
	   @IBOutlet weak var titleTextField: UITextField?

	   var URL: String? {
		  return URLTextField?.text
	   }

	   var title: String? {
		  return titleTextField?.text
	   }
    }

Specify the View class as your XIB file's custom class. In the Add Bookmark Screenlet, for example, `AddBookmarkView_default` is set as the `AddBookmarkView_default.xib` file's custom class.

If you're using CocoaPods, make sure to explicitly specify a valid module for the custom class--the grayed-out *Current* default value only suggests a module. 

![Figure 2: This XIB file's custom class's module is NOT specified.](../../../images/screens-ios-theme-custom-module-wrong.png)

![Figure 3: The XIB file is bound to the custom class name, with the specified module.](../../../images/screens-ios-theme-custom-module-right.png)

### Creating the Interactor

Create an Interactor class for each use case of your Screenlet. Each Interactor must extend the Liferay Screens [`Interactor`](https://github.com/liferay/liferay-screens/blob/master/ios/Framework/Core/Base/Interactor.swift) class. The name of this class should follow the [Naming Convention](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices#ios-naming-convention) specified in the [Best Practices](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices).

Interactors work synchronously, but, you can use Callbacks (delegates) or Connectors to run their operations in the background. Here's what you need to know to implement an Interactor: 

- You must receive all needed properties, along with the Screenlet in the Interactor initializer.
- You must override `Interactor`'s `start` method to perform operations (e.g., invoke a Liferay portal operation via a Liferay Mobile SDK service). 
- You can save a response to a property and make its value accessible. 
- You must invoke methods `callOnSuccess` or `callOnFailure` to execute closures `onSuccess` or `onFailure`, respectively. 

The sample Add Bookmark Screenlet's Interactor class [`AddBookmarkInteractor`](https://github.com/liferay/liferay-screens/tree/master/ios/Samples/Bookmark/AddBookmarkScreenlet/Basic/Interactor/AddBookmarkInteractor.swift) adds a bookmark to the Liferay instance:

    import UIKit
    import LiferayScreens
    
    
    public class AddBookmarkInteractor: Interactor, LRCallback {
    	
    	public var resultBookmarkInfo: [String:AnyObject]?
    
    	var folderId: Int64
    	var title: String
    	var url: String
    
    
    	//MARK: Initializer
    
    	public init(screenlet: BaseScreenlet, folderId: Int64, title: String, url: String) {
    		self.folderId = folderId
    		self.title = title
    		self.url = url
    		super.init(screenlet: screenlet)
    	}
    
    
    	//MARK: Interactor
    
    	override public func start() -> Bool {
    		let session = SessionContext.createSessionFromCurrentSession()
    		session?.callback = self
    
    		let service = LRBookmarksEntryService_v7(session: session)
    
    		do {
    			try service.addEntryWithGroupId(LiferayServerContext.groupId,
    			                                folderId: folderId,
    			                                name: title,
    			                                url: url,
    			                                description: "Added from Liferay Screens",
    			                                serviceContext: nil)
    			
    			return true
    		}
    		catch {
    			return false
    		}
    	}
    
    
    	//MARK: LRCallback
    
    	public func onFailure(error: NSError!) {
    		self.callOnFailure(error)
    	}
    
    	public func onSuccess(result: AnyObject!) {
    		//Save result bookmark info
    		resultBookmarkInfo = (result as! [String:AnyObject])
    
    		self.callOnSuccess()
    	}
    
    }

In this example, we extend the Interactor from `LRCallback` class in order to make the interaction run asynchronously. This is the method provided by the Mobile SDK to run asyncronous services. Please refer to [this](/develop/tutorials/-/knowledge_base/7-0/creating-ios-screenlets-advanced#using-connector-instead-callback) chapter of the [advanced tutorial](/develop/tutorials/-/knowledge_base/7-0/creating-ios-screenlets-advanced) if you want to use Connectors instead of Callbacks.

### Getting user data and running interaction

In your Screenlet class override the `createInteractor` method to return an instance of the requested Interactor. If you have multiple Interactors, you can write additional private methods to retrieve and return the applicable type of Interactor instance requested. You must handle each Interactor's `onSuccess` and `onFailure` closures. 

As an example, the `createInteractor` method of the Add Bookmark Screenlet class is shown below. It returns an Interactor instance of the unique Screenlet use case. 

    override public func createInteractor(name name: String?, sender: AnyObject?) -> Interactor? {

		let view = self.screenletView as! AddBookmarkView_default

		let interactor = AddBookmarkInteractor(screenlet: self,
		                                       folderId: folderId,
		                                       title: view.title!,
		                                       url: view.URL!)

		//Called when interactor finish succesfully
		interactor.onSuccess = {
			let bookmarkName = interactor.resultBookmarkInfo!["name"] as! String
			print("Bookmark \"\(bookmarkName)\" saved!")
		}

		//Called when interactor finish with error
		interactor.onFailure = { _ in
			print("An error occurred saving the bookmark")
		}

		return interactor
	}


For reference, the sample Add Bookmark Screenlet's final code is on [GitHub](https://github.com/liferay/liferay-screens/tree/master/ios/Samples/Bookmark/AddBookmarkScreenlet/Basic/AddBookmarkScreenlet.swift). 

You're done! Your Screenlet is a ready-to-use component that you can add to your storyboard. You can even [package](/develop/tutorials/-/knowledge_base/7-0/creating-ios-themes#publish-your-themes-using-cocoapods) it to contribute to the Screens project or distribute with CocoaPods. Now you know how to create Screenlets for iOS.

If you want to go deeper into the development of a Screenlet, please refer to our [advanced tutorial](/develop/tutorials/-/knowledge_base/7-0/creating-ios-screenlets-advanced), where we teach cool things like: multiview support, event notification from your Screenlet, and much more!

## Related Topics [](id=related-topics)

[Creating iOS Screenlets (Advanced)](/develop/tutorials/-/knowledge_base/7-0/creating-ios-screenlets-advanced)

[Using Screenlets in iOS Apps](/develop/tutorials/-/knowledge_base/7-0/using-screenlets-in-ios-apps)

[Architecture of Liferay Screens for iOS](/develop/tutorials/-/knowledge_base/7-0/architecture-of-liferay-screens-for-ios)

[Creating iOS Themes](/develop/tutorials/-/knowledge_base/7-0/creating-ios-themes)

[Creating Android Screenlets](/develop/tutorials/-/knowledge_base/7-0/creating-android-screenlets)
