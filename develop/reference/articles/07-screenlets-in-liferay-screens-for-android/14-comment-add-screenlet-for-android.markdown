# Comment Add Screenlet for Android [](id=comment-add-screenlet-for-android)

## Requirements [](id=requirements)

- Android SDK 4.0 (API Level 15) or above
- Liferay 7.0 CE, Liferay DXP
- Liferay Screens Compatibility Plugin
  ([CE](http://www.liferay.com/marketplace/-/mp/application/54365664) or 
  [DE](http://www.liferay.com/marketplace/-/mp/application/54369726), 
  depending on your Liferay edition). This app is preinstalled in Liferay 7.0 CE 
  and Liferay DXP instances. 

## Compatibility [](id=compatibility)

- Android SDK 4.0 (API Level 15) and above

## Features [](id=features)

Comment Add Screenlet can add a comment to an asset in a Liferay instance. 

## Module [](id=module)

- None

## Views [](id=views)

The Default View uses Android's `EditText` and `Button` elements to show an add 
comment dialog. Other Views may use different components to show this dialog. 

![Figure 1: Comment Add Screenlet using the Default (left) and Material (right) Views.](../../images/screens-android-commentadd.png)

## Offline [](id=offline)

This Screenlet supports offline mode so it can function without a network 
connection. 

| Policy | What happens | When to use |
|--------|--------------|-------------|
| `REMOTE_ONLY` | The Screenlet loads the data from the Liferay instance. If a connection issue occurs, the Screenlet uses the listener to notify the developer about the error. If the Screenlet successfully loads the data, it stores it in the local cache for later use. | Use this policy when you always need to show updated data, and show nothing when there's no connection. |
| `CACHE_ONLY` | The Screenlet loads the data from the local cache. If the data isn't there, the Screenlet uses the listener to notify the developer about the error. | Use this policy when you always need to show local data, without retrieving remote information under any circumstance. |
| `REMOTE_FIRST` | The Screenlet loads the data from the Liferay instance. If this succeeds, the Screenlet shows the data to the user and stores it in the local cache for later use. If a connection issue occurs, the Screenlet retrieves the data from the local cache. If the data doesn't exist there, the Screenlet uses the listener to notify the developer about the error. | Use this policy to show the most recent version of the data when connected, but show an outdated version when there's no connection. |
| `CACHE_FIRST` | The Screenlet loads the data from the local cache. If the data isn't there, the Screenlet requests it from the Liferay instance and notifies the developer about any errors that occur (including connectivity errors). | Use this policy to save bandwidth and loading time in case you have local (but probably outdated) data. |

## Required Attributes [](id=required-attributes)

- `className`
- `classPK`

## Attributes [](id=attributes)

| Attribute | Data type | Explanation |
|-----------|-----------|-------------|
| `layoutId` | `@layout` | The layout to use to show the View.|
| `className` | `string` | The asset's fully qualified class name. For example, a blog entry's `className` is [`com.liferay.blogs.kernel.model.BlogsEntry`](https://docs.liferay.com/portal/7.0/javadocs/portal-kernel/com/liferay/blogs/kernel/model/BlogsEntry.html). The `className` and `classPK` attributes are required to instantiate the Screenlet. | 
| `classPK` | `number` | The asset’s unique identifier. The `className` and `classPK` attributes are required to instantiate the Screenlet. |
| `cachePolicy` | `string` | The offline mode setting. See [the Offline section](/develop/reference/-/knowledge_base/7-0/comment-add-screenlet-for-android#offline) for details. |

## Listener [](id=listener)

Comment Add Screenlet delegates some events to a class that implements 
`CommentAddListener`. This interface lets you implement the following methods: 

- `onAddCommentSuccess(CommentEntry commentEntry)`: Called when the Screenlet 
  successfully adds a comment to the asset. 
