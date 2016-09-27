# Gallery Screenlet for iOS [](id=gallery-screenlet-for-ios)

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

Gallery Screenlet can show lists of images from a `folderId` from a Liferay instance. Also, you can upload and delete photos. The Screenlet also implements [fluent pagination](http://www.iosnomad.com/blog/2014/4/21/fluent-pagination) with configurable page size, and supports i18n in asset values. 

## Module [](id=module)

- None

## Themes [](id=themes)

The Default Theme uses a standard `UICollectionView` to show the scrollable list as grid. 
Other Themes may use a different component, such as `UITableView` or 
others, to show the contents.

![Figure 1: Gallery Screenlet using the Default (`default`) Theme.](../../images/screens-ios-gallery.png)

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
| `repositoryId` | `number` | The ID of the site (group) where the image gallery exists. |
| `folderId` | `number` | The ID of the image gallery folder to be displayed. |
| `mimeTypes` | `string` | Comma separated mimeTypes that Gallery Screenlet supports. |
| `filePrefix` | `string` | Prefix for the image title when we are in uploading process. |
| `offlinePolicy` | `string` | Offline mode type. See [Offline](#offline) section. First, the screenlet takes *cache-first* but then, the default value is *remote-first*. |
| `autoLoad` | `boolean` | Whether the list should automatically load when the Screenlet appears in the app's UI. The default value is `true`. |
| `refreshControl` | `boolean` | Whether a standard [`UIRefreshControl`](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIRefreshControl_class/) is shown when the user does the pull to refresh gesture. The default value is `true`. |
| `firstPageSize` | `number` | The number of items to display on the first page. The default value is `50`. |
| `pageSize` | `number` | The number of items to display on second and subsequent pages. The default value is `25`. |
| `obcClassName` | `string` | The `OrderByComparator` class name to sort the results. If you don't want to sort the results, you can omit this property. See [ImageGallery comparators](https://github.com/liferay/liferay-portal/tree/master/portal-impl/src/com/liferay/portlet/documentlibrary/util/comparator). You can only use classes that extend `OrderByComparator<DLFileEntry>`. |

## Methods [](id=methods)

| Method | Return | Explanation |
|-----------|-----------|-------------| 
| `loadList()` | `boolean` | Starts the request to load the web content list. This list is shown when the response is received. Returns `true` if the request is sent successfully. |

## Delegate [](id=delegate)

Web Content List Screenlet delegates some events to an object that conforms to 
the `WebContentListScreenletDelegate` protocol. This protocol lets you implement 
the following methods: 

- `- screenlet:onImageEntriesResponse:`: Called when a page of contents is 
  received. Note that this method may be called more than once: one call for 
  each page received.

- `- screenlet:onImageEntriesError:`: Called when an error occurs in the 
  process. The `NSError` object describes the error. 

- `- screenlet:onImageEntrySelected:`: Called when an item in the list is 
  selected.
  
- `- screenlet:onImageEntryDeleted:`: Called when an item in the list is 
  deleted.

- `- screenlet:onImageEntryDeleteError:`: Called when an error occurs in the deleted process. The `NSError` object describes the error.
  
- `- screenlet:onImageUploadStart:`: Called when an item is prepared to be upload.

- `- screenlet:onImageUploadProgress:`: Called when an item is uploading.
  
- `- screenlet:onImageUploadError:`: Called when an error occurs in the uploaded process. The `NSError` object describes the error.

- `- screenlet:onImageUploaded:`: Called when an item has finished uploading.
  
- `- screenlet:onImageUploadDetailViewCreated:`: Called when the detail upload view has to be created.