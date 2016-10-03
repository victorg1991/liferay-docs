# Gallery Screenlet for iOS [](id=gallery-screenlet-for-ios)

## Requirements [](id=requirements)

- Xcode 7.2
- iOS 9 SDK
- Liferay 7.0 CE, Liferay DXP 
- Liferay Screens Compatibility Plugin 
  ([CE](http://www.liferay.com/marketplace/-/mp/application/54365664) or 
  [DE](http://www.liferay.com/marketplace/-/mp/application/54369726), 
  depending on your portal edition). This app is preinstalled in Liferay 7.0 CE 
  and Liferay DXP instances. 

## Compatibility [](id=compatibility)

- iOS 8 and above

## Features [](id=features)

Gallery Screenlet shows a list of images from a Documents and Media folder in a 
Liferay instance. You can also use Gallery Screenlet to upload images to and 
delete images from the same folder. The Screenlet implements 
[fluent pagination](http://www.iosnomad.com/blog/2014/4/21/fluent-pagination) 
with configurable page size, and supports i18n in asset values. 

## Module [](id=module)

- None

## Themes [](id=themes)

The Default Theme uses a standard iOS `UICollectionView` to show the scrollable 
list as a grid. Other Themes may use a different component, such as 
`UITableView` or others, to show the contents. 

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
| `mimeTypes` | `string` | The comma-separated list of MIME types for the Screenlet to support. |
| `filePrefix` | `string` | The prefix to use on image titles when uploading images. |
| `offlinePolicy` | `string` | The offline mode setting. First, the Screenlet takes `cache-first` but then, the default value is `remote-first`. See the [Offline section](/develop/reference/-/knowledge_base/7-0/gallery-screenlet-for-ios#offline) for details. |
| `autoLoad` | `boolean` | Whether the list loads automatically when the Screenlet appears in the app's UI. The default value is `true`. |
| `refreshControl` | `boolean` | Whether a standard [iOS `UIRefreshControl`](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIRefreshControl_class/) appears when the user does the pull to refresh gesture. The default value is `true`. |
| `firstPageSize` | `number` | The number of items to display on the first page. The default value is `50`. |
| `pageSize` | `number` | The number of items to display on the second and subsequent pages. The default value is `25`. |
| `obcClassName` | `string` | The name of the `OrderByComparator` class to use to sort the results. Omit this property if you don't want to sort the results. Note that you can only use comparator classes that extend `OrderByComparator<DLFileEntry>`. Liferay contains no such comparator classes. You must therefore create your own by extending `OrderByComparator<DLFileEntry>`. To see examples of some comparator classes that extend other Document Library classes, [click here](https://github.com/liferay/liferay-portal/tree/master/portal-impl/src/com/liferay/portlet/documentlibrary/util/comparator). Note, however, that these classes can't be used with `obcClassName` because they don't extend `OrderByComparator<DLFileEntry>`. |

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