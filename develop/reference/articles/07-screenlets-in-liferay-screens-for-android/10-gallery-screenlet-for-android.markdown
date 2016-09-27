# Gallery Screenlet for Android [](id=gallery-screenlet-for-ios)

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

Gallery Screenlet can show lists of images from a `folderId` from a Liferay instance. Also, you can upload and delete photos. The Screenlet also implements [fluent pagination](http://www.iosnomad.com/blog/2014/4/21/fluent-pagination) with configurable page size, and supports i18n in asset values. 

## Module [](id=module)

- None

## Views [](id=views)

The Default Views use a standard `RecyclerView` to show the scrollable list. 
Other Views may use a different component, such as `ViewPager` or others, to 
show the items.

![Figure 1: Gallery Screenlet using the Default (`default`) Theme.](../../images/screens-android-gallery.png)

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

- `folderId`
- `repositoryId`

## Attributes [](id=attributes)

| Attribute | Data type | Explanation |
|-----------|-----------|-------------|
| `repositoryId` | `number` | The ID of the site (group) where the image gallery exists. |
| `folderId` | `number` | The ID of the image gallery folder to be displayed. |
| `cachePolicy` | `string` | Offline mode type. See [Offline](#offline) section. |
| `firstPageSize` | `number` | The number of items to display on the first page. The default value is `50`. |
| `pageSize` | `number` | The number of items to display on second and subsequent pages. The default value is `25`. |
| `mimeTypes` | `string` | Comma separated mimeTypes that Gallery Screenlet supports. |
| `autoLoad` | `boolean` | Whether the list should automatically load when the Screenlet appears in the app's UI. The default value is `true`. |
| `layoutId` | `@layout` | The layout to use to show the View.|
| `obcClassName` | `string` | The `OrderByComparator` class name to sort the results. If you don't want to sort the results, you can omit this property. See [ImageGallery comparators](https://github.com/liferay/liferay-portal/tree/master/portal-impl/src/com/liferay/portlet/documentlibrary/util/comparator). You can only use classes that extend `OrderByComparator<DLFileEntry>`. |

## Methods [](id=methods)

| Method | Return | Explanation |
|-----------|-----------|-------------| 
| `loadPage(pageNumber)` | `void` | Starts the request to load the specified page of assets. The page is shown when the response is received. |

## Listener [](id=listener)

The `GalleryScreenlet` delegates some events to a class that implements `BaseListListener` named `GalleryListener` and lets you implement the following methods:

- `onListPageFailed(int startRow, Exception e)`: Called 
  when an error occurs in the process.
  
- `onListPageReceived(int startRow, int endRow, 
  List<Record> records, int rowCount)`: Called when a page of records is 
  received. Note that this method may be called more than once; once for each 
  page received.

- `onListItemSelected(Record records, View view)`: Called when an 
  item in the list is selected.
  
- `onImageEntryDeleted(long imageEntryId)`: Called when an item in the list is deleted.

- `onImageUploadStarted(String picturePath, String title, String description, String changelog)`: Called when an item is prepared to be upload.

- `onImageUploadProgress(int totalBytes, int totalBytesSent)`: Called when an item is uploading.

- `onImageUploadEnd(ImageEntry entry)`: Called when an item has finished uploading.

- `showUploadImageView(String actionName, String picturePath, int screenletId)`: Called when the detail upload view has to be shown.

- `provideImageUploadDetailView()`: Called when the detail upload view has to be created.