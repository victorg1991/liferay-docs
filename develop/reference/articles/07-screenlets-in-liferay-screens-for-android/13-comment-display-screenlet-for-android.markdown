# Comment Display Screenlet for Android [](id=comment-display-screenlet-for-android)

## Requirements [](id=requirements)

- Android SDK 4.0 (API Level 15) or above
- Liferay 7.0 CE, Liferay DXP
- Liferay Screens Compatibility Plugin
  ([CE](http://www.liferay.com/marketplace/-/mp/application/54365664) or 
  [EE](http://www.liferay.com/marketplace/-/mp/application/54369726), 
  depending on your portal edition). This app is preinstalled in Liferay 7.0 CE 
  and Liferay DXP instances. 

## Compatibility [](id=compatibility)

- Android SDK 4.0 (API Level 15) and above

## Features [](id=features)

Comment Display Screenlet can show one comment of an `Asset`. Also allows to update or delete the comment by the user.

## Module [](id=module)

- None

## Views [](id=views)

The Default View uses `UserPortraitScreenlet`, `TextView` and `ImageButton` elements to show a comment of an `Asset`. 
Other Views may use a different component to show the comment.

![Figure 1: Comment Display Screenlet using the Default (`default`) Theme.](../../images/screens-android-commentdisplay.png)

## Offline [](id=offline)

This Screenlet supports offline mode so it can function without a network 
connection. 

| Policy | What happens | When to use |
|--------|--------------|-------------|
| `REMOTE_ONLY` | The Screenlet loads the list from the portal. If a connection issue occurs, the Screenlet uses the listener to notify the developer about the error. If the Screenlet successfully loads the list, it stores the data in the local cache for later use. | Use this policy when you always need to show updated data, and show nothing when there's no connection. |
| `CACHE_ONLY` | The Screenlet loads the list from the local cache. If the list isn't there, the Screenlet uses the listener to notify the developer about the error. | Use this policy when you always need to show local data, without retrieving remote information under any circumstance. |
| `REMOTE_FIRST` | The Screenlet loads the list from the portal. If this succeeds, the Screenlet shows the list to the user and stores it in the local cache for later use. If a connection issue occurs, the Screenlet retrieves the list from the local cache. If the list doesn't exist there, the Screenlet uses the listener to notify the developer about the error. | Use this policy to show the most recent version of the data when connected, but show an outdated version when there's no connection. |
| `CACHE_FIRST` | The Screenlet loads the list from the local cache. If the list isn't there, the Screenlet requests it from the portal and notifies the developer about any errors that occur (including connectivity errors). | Use this policy to save bandwidth and loading time in case you have local (but probably outdated) data. |

## Required Attributes [](id=required-attributes)

- `commentId`

## Attributes [](id=attributes)

| Attribute | Data type | Explanation |
|-----------|-----------|-------------|
| `layoutId` | `@layout` | The layout to use to show the View.|
| `autoLoad` | `boolean` | Whether the list should automatically load when the Screenlet appears in the app's UI. The default value is `true`. |
| `cachePolicy` | `string` | Offline mode type. See [Offline](#offline) section. |
| `commentId` | `number` | The primary key parameter for displaying the comment. |
| `editable` | `boolean` | Lets the user edit the comment or not. |

## Methods [](id=methods)

| Method | Return | Explanation |
|-----------|-----------|-------------| 
| `load()` | `void` | Starts the request to load the specified action. |

## Listener [](id=listener)

The `CommentDisplayScreenlet` delegates some events to a class that implements `BaseCacheListener` named `CommentDisplayInteractorListener` and lets you implement the following methods:

- `onLoadCommentSuccess(CommentEntry commentEntry)`: Called when the operation finished successfully and the comment has been loaded.

- `onDeleteCommentSuccess()`: Called when the operation finished successfully and the comment has been deleted.

- `onUpdateCommentSuccess(CommentEntry commentEntry)`: Called when the operation finished successfully and the comment has been updated.
