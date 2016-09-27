# Asset List Screenlet for Android [](id=assetlistscreenlet-for-android)

## Requirements [](id=requirements)

- Android SDK 4.0 (API Level 15) or above
- Liferay Portal 6.2 (CE or EE), Liferay 7.0 CE, Liferay DXP
- Liferay Screens Compatibility Plugin
  ([CE](http://www.liferay.com/marketplace/-/mp/application/54365664) or 
  [EE](http://www.liferay.com/marketplace/-/mp/application/54369726), 
  depending on your portal edition). This app is preinstalled in Liferay 7.0 CE 
  and Liferay DXP instances. 

## Compatibility [](id=compatibility)

- Android SDK 4.0 (API Level 15) and above

## Features [](id=features)

The `AssetListScreenlet` can be used to show [asset](/tutorials/-/knowledge_base/7-0/asset-framework) 
lists from a Liferay instance. For example, you can use the Screenlet to show a 
scrollable list of assets. It also implements [fluent pagination](http://www.iosnomad.com/blog/2014/4/21/fluent-pagination) 
with configurable page size. The `AssetListScreenlet` can show assets belonging 
to the following classes:

- `Group`
- `Layout`
- `Organization`
- `User`
- `UserGroup`
- `BlogsEntry`
- `BookmarksEntry`
- `BookmarksFolder`
- `CalendarEvent`
- `DLFileEntry`
- `DLFileEntryMetadata`
- `DLFileEntryType`
- `DLFileRank`
- `DLFileShortcut`
- `DLFileVersion`
- `DDLRecord`
- `DDLRecordSet`
- `JournalArticle` (Web Content)
- `JournalFolder`
- `MBMessage`
- `MBThread`
- `MBCategory`
- `MBDiscussion`
- `MBMailingList`
- `WikiPage`
- `WikiPageResource`
- `WikiNode`

The `AssetListScreenlet` also supports i18n in asset values.

## Module [](id=module)

- None

## Views [](id=views)

The Default Views use a standard `RecyclerView` to show the scrollable list. 
Other Views may use a different component, such as `ViewPager` or others, to 
show the items.

![`AssetListScreenlet` using the Default and Material Viewsets.](../../images/screens-android-assetlist.png)

## Portal Configuration [](id=portal-configuration)

Dynamic Data Lists (DDL) and Data Types should be configured properly in the 
portal. Refer to the 
[Creating Data Definitions](/discover/portal/-/knowledge_base/7-0/creating-data-definitions)  
and 
[Creating Data Lists](/discover/portal/-/knowledge_base/7-0/creating-data-lists) 
sections of the User Guide for more details.

Also, to allow remote calls without the `userId`, the Liferay Screens 
Compatibility app must be installed in your Liferay instance. You can find this 
app on 
[Liferay Marketplace](https://web.liferay.com/marketplace). 

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

- `classNameId`

## Attributes [](id=attributes)

| Attribute | Data type | Explanation |
|-----------|-----------|-------------| 
| `layoutId` | `@layout` | The layout to use to show the View.|
| `autoLoad` | `boolean` | Whether the list should be loaded when it's presented on the screen. The default value is `true`. |
| `groupId` | `number` | The asset's group (site) ID. If this value is `0`, the `groupId` specified in `LiferayServerContext` is used. The default value is `0 `. |
| `cachePolicy` | `string` | Offline mode type. See [Offline](#offline) section. |
| `portletItemName` | `string` | The archive name you used in the Asset Publisher. To use this feature, add an Asset Publisher to one of your site's pages (it may be a hidden page), configure the Asset Publisher filter (Asset Selection on configuration page), and then use the *Archive Setup* option to save this configuration with a name. Use this name in this attribute. |
| `classNameId` | `number` | The asset class name's ID. Use values from the portal's `classname_` database table. |
| `firstPageSize` | `number` | The number of items to retrieve from the server for display on the list's first page. The default value is `50`. |
| `pageSize` | `number` | The number of items to retrieve from the server for display on the second and subsequent pages. The default value is `25`. |
| `labelFields` | `string` | The comma-separated names of the DDL fields to show. Refer to the list's data definition to find the field names. For more information on this, see [Creating Data Definitions](/discover/portal/-/knowledge_base/7-0/creating-data-definitions). Note that the appearance of these values in your app depends on the `layoutId` set. |
| `customEntryQuery` | `HashMap` | The set of keys (string) and values (string or number) to be used in the [AssetEntryQuery object](https://docs.liferay.com/portal/6.2/javadocs/com/liferay/portlet/asset/service/persistence/AssetEntryQuery.html). These values filter the assets returned by the portal. | 
| `obcClassName` | `string` | The `OrderByComparator` class name to sort the results. If you don't want to sort the results, you can omit this property. See [AssetList comparators](https://github.com/liferay/liferay-portal/tree/master/portal-impl/src/com/liferay/portlet/asset/util/comparator). You can only use classes that extend `OrderByComparator<AssetEntry>`. |

## Methods [](id=methods)

| Method | Return | Explanation |
|-----------|-----------|-------------| 
| `loadPage(pageNumber)` | `void` | Starts the request to load the specified page of assets. The page is shown when the response is received. |

## Listener [](id=listener)

The `AssetListScreenlet` delegates some events to an object or a class that implements `BaseListListener` and lets you implement the following methods: 

- `onListPageFailed(int startRow, Exception e)`: Called 
  when an error occurs in the process.
  
- `onListPageReceived(int startRow, int endRow, 
  List<Record> records, int rowCount)`: Called when a page of records is 
  received. Note that this method may be called more than once; once for each 
  page received.

- `onListItemSelected(Record records, View view)`: Called when an 
  item in the list is selected.
