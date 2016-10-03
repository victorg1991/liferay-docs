# Asset Display Screenlet for Android [](id=asset-display-screenlet-for-android)

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

Asset Display Screenlet can render an `Asset` without knowing what kind of `Asset` is. Nowadays, the screenlet displays `DLFileEntry` (image, video, audio and PDF), `BlogsEntry` and `WebContent`. You can display your custom `Asset` with `AssetDisplayScreenlet` because exists a listener method for it (see [Listener](#listener) section).

## Module [](id=module)

- None

## Views [](id=views)

The Default View uses different element to show an `Asset` depending on its type. For example, for images, the screenlet uses `ImageView`, for blogs entry uses `TextView`, `UserPortraitScreenlet` and others.

This screenlet renders another screenlet:

- Images: `ImageDisplayScreenlet`.
- Videos: `VideoDisplayScreenlet`.
- Audios: `AudioDisplayScreenlet`.
- PDFs: `PdfDisplayScreenlet`.
- `BlogsEntry`: `BlogsEntryDisplayScreenlet`.
- `WebContent`: `WebContentDisplayScreenlet`.

For images, videos, audios and PDFs see [BaseFileDisplayScreenlet](../base-file-display-screenlet-for-android).

These screenlets can be used alone without `AssetDisplayScreenlet`.

![Figure 1: Asset Display Screenlet using the Default (`default`) Theme.](../../images/screens-android-assetdisplay.png)

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
- `entryId`

Or...

- `className`
- `classPK`

## Attributes [](id=attributes)

| Attribute | Data type | Explanation |
|-----------|-----------|-------------|
| `layoutId` | `@layout` | The layout to use to show the View.|
| `autoLoad` | `boolean` | Whether the list should automatically load when the Screenlet appears in the app's UI. The default value is `true`. |
| `entryId` | `number` | The primary key parameter for displaying the `Asset`. | 
| `className` | `string` | The class name of the `Asset` that we want to render. It's required when we are instantiating the screenlet with `className` and `classPK`. For example, if we want show a blog entry, for `BlogsEntry` object in Liferay 7.0 version, its `className` is [`com.liferay.blogs.kernel.model.BlogsEntry`](https://github.com/liferay/liferay-portal/blob/master/portal-kernel/src/com/liferay/blogs/kernel/model/BlogsEntry.java). |
| `classPK` | `number` | This is the asset identifier and it's unique. This attribute is used only with `className`. |
| `cachePolicy` | `string` | Offline mode type. See [Offline](#offline) section. |
| `imageLayoutId` | `@layout` | The layout to use to show an image. |
| `videoLayoutId` | `@layout` | The layout to use to show a video. |
| `audioLayoutId` | `@layout` | The layout to use to show an audio. |
| `pdfLayoutId` | `@layout` | The layout to use to show a PDF. |
| `blogsLayoutId` | `@layout` | The layout to use to show a `BlogsEntry`. |
| `webDisplayLayoutId` | `@layout` | The layout to use to show a `WebContent`. |

## Listener [](id=listener)

The `AssetDisplayScreenlet` delegates some events to a class that implements `BaseCacheListener` named `AssetDisplayListener` and lets you implement the following methods:

- `onRetrieveAssetSuccess(AssetEntry assetEntry)`: Called when the operation finished successfully and the asset has been loaded.

Also, exists another listener called `AssetDisplayInnerScreenletListener` for configuring child screenlet (the screenlet which will be rendered inside `AssetDisplayScreenlet`) or render a custom `Asset`:

- `onConfigureChildScreenlet(AssetDisplayScreenlet screenlet, BaseScreenlet innerScreenlet, AssetEntry assetEntry)`: Called when the child screenlet has been created successfully and we want to configure or customize it. For example:

	```
	@Override
	public void onConfigureChildScreenlet(AssetDisplayScreenlet screenlet,
		BaseScreenlet innerScreenlet, AssetEntry assetEntry) {
		if ("blogsEntry".equals(assetEntry.getObjectType())) {
			innerScreenlet.setBackgroundColor(ContextCompat.getColor(this,
				R.color.light_gray));
		}
	}
	
- `onRenderCustomAsset(AssetEntry assetEntry)`: Called when we want to render a custom `Asset`. For example:

	```
	@Override
	public View onRenderCustomAsset(AssetEntry assetEntry) {
		if (assetEntry instanceof User) {
			View view = getLayoutInflater().inflate(R.layout.user_display, null);
			User user = (User) assetEntry;

			TextView username = (TextView)
				view.findViewById(R.id.liferay_username);

			username(user.getUsername());
			
			return view;
		}
		return null;
	}