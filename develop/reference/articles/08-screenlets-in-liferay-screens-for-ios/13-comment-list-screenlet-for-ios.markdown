# Comment List Screenlet for iOS [](id=comment-list-screenlet-for-ios)

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

Comment List Screenlet can list all the comments of an `Asset`. Also allows to update or delete the comment by the user.

## Module [](id=module)

- None

## Themes [](id=themes)

The Default Theme use `UITableView` to show all the comments.
Other Views may use a different component, such as `UICollectionView` or others, to show the items.

![Figure 1: Comment List Screenlet using the Default (`default`) Theme.](../../images/screens-ios-commentlist.png)

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
| `className` | `string` | The class name of the `Asset` that we want to list its comments. It's required when we are instantiating the screenlet with `className` and `classPK`. For example, for `BlogsEntry` object in Liferay 7.0 version, its `className` is [`com.liferay.blogs.kernel.model.BlogsEntry`](https://github.com/liferay/liferay-portal/blob/master/portal-kernel/src/com/liferay/blogs/kernel/model/BlogsEntry.java). |
| `classPK` | `number` | This is the asset’s identifier and it's unique. This attribute is used only with `className`. |
| `offlinePolicy` | `string` | Offline mode type. See [Offline](#offline) section. The default value is *remote-first*. |
| `editable` | `boolean` | Lets the user edit the comment or not. |
| `autoLoad` | `boolean` | Whether the list should automatically load when the Screenlet appears in the app's UI. The default value is `true`. |
| `refreshControl` | `boolean` | Defines whether a standard [UIRefreshControl](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIRefreshControl_class/) is shown when the user does the pull to refresh gesture. The default value is `true`. |
| `firstPageSize` | `number` | The number of items retrieved from the server for display on the first page. The default value is `50`. |
| `pageSize` | `number` | The number of items retrieved from the server for display on the second and subsequent pages. The default value is `25`. |
| `obcClassName` | `string` | The `OrderByComparator` class name to sort the results. If you don't want to sort the results, you can omit this property. You can only use classes that extend `OrderByComparator<MBMessage>`. |

## Methods [](id=methods)

| Method | Return | Explanation |
|-----------|-----------|-------------| 
| `loadList()` | `boolean` | Starts the request to load the comments list. This list is shown when the response is received. Returns true if the request is sent. |

## Delegate [](id=delegate)

`CommentListScreenlet` delegates some events to an object that conforms to 
the `ComentListScreenletDelegate` protocol. This protocol lets you implement 
the following methods: 

- `- screenlet:onListResponseComments:`: Called when the comments are 
  received.
    
- `- screenlet:onCommentListError:`: Called when an error occurs in the 
  process. The `NSError` object describes the error. 

- `- screenlet:onSelectedComment:`: Called when a comment is 
  selected.
    
- `- screenlet:onDeletedComment:`: Called when a comment is 
  deleted.
  
- `- screenlet:onCommentDelete:`: Called when a comment is prepared for 
  deleting.
  
- `- screenlet:onUpdatedComment:`: Called when a comment is 
  updated.
  
- `- screenlet:onCommentUpdate:`: Called when a comment is prepared for updating.
