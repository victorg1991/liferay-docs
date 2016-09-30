# Rating Screenlet for iOS [](id=rating-screenlet-for-ios)

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

Rating Screenlet can show the rating for an `Asset`. Also allows to update or delete the rating by the user. The screenlet comes with different views: thumbs, stars, like and emojis.

## Module [](id=module)

- None

## Themes [](id=themes)

The Default Theme use `CosmosView` to show user rating and average rating of an `Asset`.
Other Views may use a different component, such as `UIButton` or others, to show the items.

![Figure 1: Rating Screenlet using the Default (`default`) Theme.](../../images/screens-ios-ratings.png)

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
| `layoutId` | `@layout` | The layout to use to show the View.|
| `autoLoad` | `boolean` | Whether the list should automatically load when the Screenlet appears in the app's UI. The default value is `true`. |
| `editable` | `boolean` | Lets the user change the rating or not. |
| `entryId` | `number` | The primary key parameter for displaying the `Asset` rating. |
| `className` | `string` | The class name of the `Asset`. It's required when we are instantiating the screenlet with `className` and `classPK`. For example, for `BlogsEntry` object in Liferay 7.0 version, its `className` is [`com.liferay.blogs.kernel.model.BlogsEntry`](https://github.com/liferay/liferay-portal/blob/master/portal-kernel/src/com/liferay/blogs/kernel/model/BlogsEntry.java). |
| `classPK` | `number` | This is the asset’s identifier and it's unique. This attribute is used only with `className`. |
| `groupId` | `number` | The ID of the group where the `Asset` exists. |
| `ratingsGroupCount` | `number` | Number of ratings that the user can select. |
| `offlinePolicy` | `string` | Offline mode type. See [Offline](#offline) section. |

## Methods [](id=methods)

| Method | Return | Explanation |
|-----------|-----------|-------------| 
| `loadRatings()` | `boolean` | Starts the request to load the `Asset` ratings. |

## Delegate [](id=delegate)

`RatingScreenlet` delegates some events to an object that conforms to 
the `RatingScreenletDelegate` protocol. This protocol lets you implement 
the following methods: 

- `- screenlet:onRatingRetrieve:`: Called when the ratings are 
  received.
  
- `- screenlet:onRatingDeleted:`: Called when a rating is 
  deleted.
  
- `- screenlet:onRatingUpdated:`: Called when a rating is 
  updated.
  
- `- screenlet:onRatingError:`: Called when an error occurs in the 
  process. The `NSError` object describes the error. 