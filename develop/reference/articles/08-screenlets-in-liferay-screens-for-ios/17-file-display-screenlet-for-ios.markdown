# File Display Screenlet for iOS [](id=base-file-display-screenlet-for-ios)

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

File Display Screenlet displays a `DLFileEntry` object. This screenlet cannot be used alone, but rather, it's the superclass for: 

- `ImageDisplayScreenlet`: displays an image.

- `VideoDisplayScreenlet`: displays a video.

- `AudioDisplayScreenlet`: displays an audio.

- `PdfDisplayScreenlet`: displays a PDF file.

## Module [](id=module)

- None

## Themes [](id=themes)

The Default View uses different element to show an `Asset` depending on its type. For example, for images, the screenlet uses `UIImageView`, for PDF uses `UIWebView`, etc.

![Figure 1: Asset Display Screenlet using the Default (`default`) Theme.](../../images/screens-ios-assetdisplay.png)

## Attributes [](id=attributes)

This screenlet it's the superclass for other screenlets but all of them have the following required attributes:

- `entryId`

Or...

- `className`
- `classPK`

| Attribute | Data type | Explanation |
|-----------|-----------|-------------|
| `assetEntryId` | `number` | The primary key parameter for displaying the `Asset`. | 
| `className` | `string` | The class name of the `Asset` that we want to render. It's always the same for `DLFileEntry` object, in Liferay 7.0 version, it's [`com.liferay.document.library.kernel.model.DLFileEntry`](https://github.com/liferay/liferay-portal/blob/master/portal-kernel/src/com/liferay/document/library/kernel/model/DLFileEntry.java). |
| `classPK` | `number` | This is the asset identifier and it's unique. This attribute is used only with `className`. |
| `autoLoad` | `boolean` | Whether the list should automatically load when the Screenlet appears in the app's UI. The default value is `true`. |
| `offlinePolicy` | `string` | The offline mode setting. The default value is `remote-first`. See the [Offline](#offline) section for details. |


## Delegate [](id=delegate)

`FileDisplayScreenlet` delegates some events to an object that conforms to 
the `FileDisplayScreenletDelegate` protocol. This protocol lets you implement 
the following methods:

- `- screenlet:onFileAssetResponse:`: Called when the `DLFileEntry` object is received.

- `- screenlet:onFileAssetError:`: Called when an error occurs in the process. The `NSError` object describes the error.