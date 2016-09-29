# Comment Display Screenlet for iOS [](id=comment-display-screenlet-for-ios)

## Requirements [](id=requirements)

- Xcode 7.2
- iOS 9 SDK
- Liferay 7.0 CE, Liferay DXP 
- Liferay Screens Compatibility Plugin 
  ([CE](http://www.liferay.com/marketplace/-/mp/application/54365664) or 
  [EE](http://www.liferay.com/marketplace/-/mp/application/54369726), 
  depending on your portal edition). This app is preinstalled in Liferay 7.0 CE 
  and Liferay DXP instances. 

## Compatibility [](id=compatibility)

- iOS 8 and above

## Features [](id=features)

Comment Display Screenlet can show one comment of an `Asset`. Also allows to update or delete the comment by the user.

## Module [](id=module)

- None

## Themes [](id=themes)

The Default Theme uses `UserPortraitScreenlet` and `UILabel` elements to show a comment of an `Asset`. 
Other Views may use a different component to show the comment.

![Figure 1: Comment Display Screenlet using the Default (`default`) Theme.](../../images/screens-ios-commentdisplay.png)

## Offline [](id=offline)

This Screenlet supports offline mode so it can function without a network 
connection. 

| Policy | What happens | When to use |
|--------|--------------|-------------|
| `remote-only` | The Screenlet loads the list from the Liferay instance. If a connection issue occurs, the Screenlet uses the delegate to notify the developer about the error. If the Screenlet successfully loads the list, it stores the data in the local cache for later use. | Use this policy when you always need to show updated data, and show nothing when there's no connection. |
| `cache-only` | The Screenlet loads the list from the local cache. If the list isn't there, the Screenlet uses the delegate to notify the developer about the error. | Use this policy when you always need to show local data, without retrieving remote information under any circumstance. |
| `remote-first` | The Screenlet loads the list from the Liferay instance. If this succeeds, the Screenlet shows the list to the user and stores it in the local cache for later use. If a connection issue occurs, the Screenlet retrieves the list from the local cache. If the list doesn't exist there, the Screenlet uses the delegate to notify the developer about the error. | Use this policy to show the most recent version of the data when connected, but show a possibly outdated version when there's no connection. |
| `cache-first` | The Screenlet loads the list from the local cache. If the list isn't there, the Screenlet requests it from the Liferay instance and notifies the developer about any errors that occur (including connectivity errors). | Use this policy to save bandwidth and loading time in case you have local (but possibly outdated) data. |

## Attributes [](id=attributes)

| Attribute | Data type | Explanation |
|-----------|-----------|-------------|
| `commentId` | `number` | The primary key parameter for displaying the comment. |
| `autoLoad` | `boolean` | Whether the list should automatically load when the Screenlet appears in the app's UI. The default value is `true`. |
| `editable` | `boolean` | Lets the user edit the comment or not. |
| `offlinePolicy` | `string` | Offline mode type. See [Offline](#offline) section. The default value is *remote-first*. |

## Methods [](id=methods)

| Method | Return | Explanation |
|-----------|-----------|-------------| 
| `load()` | none | Starts the request to load the requested comment. |

## Delegate [](id=delegate)

`CommentDisplayScreenlet` delegates some events to an object that conforms to 
the `CommentDisplayScreenletDelegate` protocol. This protocol lets you implement 
the following methods:

- `- screenlet:onCommentLoaded:`: Called when the comment are loaded.

- `- screenlet:onLoadCommentError:`: Called when an error occurs in the process. The `NSError` object describes the error.
  
- `- screenlet:onSelectedComment:`: Called when a comment is selected. 
     
- `- screenlet:onDeletedComment:`: Called when a comment is deleted. 
   
- `- screenlet:onCommentDelete:`: Called when a comment is prepared for deleting. 
   
- `- screenlet:onUpdatedComment:`: Called when a comment is updated. 
   
- `- screenlet:onCommentUpdate:`: Called when a comment is prepared for updating. 
