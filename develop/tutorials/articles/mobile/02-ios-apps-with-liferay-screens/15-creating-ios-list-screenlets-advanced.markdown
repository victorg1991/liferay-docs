# Creating iOS List Screenlets (Advanced) [](id=creating-ios-list-screenlets-advanced)

If you followed our previous [creating ios list screenlets tutorial](/develop/reference/-/knowledge_base/7-0/creating-ios-list-screenlets)
you'll have a nice Screenlet to list bookmarks from a Liferay's Bookmarks Folder
right in your app. But, what if you want to customize each cell, or want to use
`UICollectionView` instead of `UITableView`?

This tutorial explains how to improve your Screenlets with more advanced code. As
an example, it references code from the sample [List Bookmark Screenlet](https://github.com/liferay/liferay-screens/tree/master/ios/Samples/Bookmark/BookmarkListScreenlet).

If you didn't follow that tutorial or don't have the code right now, you can
clone the code from the sample [List Bookmark Screenlet](https://github.com/liferay/liferay-screens/tree/develop/ios/Samples/Bookmark/ListBookmarkScreenlet).

So... are you prepared to fully master your list Screenlet?

## Using Custom Cells with List Screenlets [](id=using-custom-cells-with-list-screenlets)

Liferay Screens's Default theme uses the [`UITableView` class](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UITableView_Class/)
as a UI component for list Screenlets. Although this works fine for showing
simple lists of items, you may want to use custom cells to spruce things up a
bit or show more complex content in the list.

To do this, you must [create a custom theme](/develop/tutorials/-/knowledge_base/7-0/creating-ios-themes)
that extends the Default theme of the list Screenlet you want to use.

For example, for our `ListBookmarkScreenlet`, create a new theme in your
Screenlet's folder following the steps described in the
[Creating iOS Themes tutorial](/develop/tutorials/-/knowledge_base/7-0/creating-ios-themes).

Then create the XIB file for your custom cell. As usual, create this file and
its companion class, and create as many outlets and actions as you need. If you
want to use different layouts for different rows, create several XIB files and
companion classes. The name of both the XIB and the Swift files should follow
the [Naming Convention](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices#ios-naming-convention)
specified in the [Best Practices](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices).

For example, our `BookmarkCell_default-custom.xib` could look like this:

![Figure 1: Custom cell XIB view.](../../../images/screens-ios-xcode-custom-cell.png)

And, on the other hand, the associated class:

    import UIKit

    class BookmarkCell_default_custom: UITableViewCell {

        @IBOutlet weak var nameLabel: UILabel?
        @IBOutlet weak var urlLabel: UILabel?

        var bookmark: Bookmark? {
            didSet {
                nameLabel?.text = bookmark?.name
                urlLabel?.text = bookmark?.url
            }
        }

    }

Now you're ready to implement your theme's functionality. In your
`*ListView_mytheme` class, override the `doRegisterCellNibs` method to register
your custom cell:

In the `BookmarkListView_default-custom`:

    public override func doRegisterCellNibs() {
        let nib = UINib(nibName: "BookmarkCell_default-custom", bundle: NSBundle.mainBundle())

        tableView?.registerNib(nib, forCellReuseIdentifier: BookmarkCellId)
    }

Note that we use a constant `let BookmarkCellId = "bookmarkCell"` instead of an
hardcoded string, as suggested in the [Avoid Hardcoded Strings](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices#avoid-hardcoded-strings)
chapter of the [Best Practices](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices).

Next, get the cell ID for each row, by override the `doGetCellId` method.

In this example we just return the previous String constant:

    override public func doGetCellId(row row: Int, object: AnyObject?) -> String {
        return BookmarkCellId
    }

To fill the cell with the row's data, override the `doFillLoadedCell` method.
Note that this method isn't called for in-progress cells; it's only called for
cells with data. Also note that the source data is stored in the method's
`object` argument. This is a generic object that you must cast to the specific
element type.

In the case of this example:

    override public func doFillLoadedCell(row row: Int, cell: UITableViewCell, object:AnyObject) {
        if let bookmarkCell = cell as? BookmarkCell_default_custom, bookmark = object as? Bookmark {
            bookmarkCell.bookmark = bookmark
        }
    }

Remember that you also have the typical [`UITableViewDelegate` protocol](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UITableViewDelegate_Protocol/)
and [`UITableViewDataSource` protocol](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UITableViewDataSource_Protocol/)
methods available to you, so you can override any of them if you need to (check
first to make sure they're not already overridden). For example, implement the
following method to use a different cell height for one row:

    public func tableView(tableView: UITableView, heightForRowAtIndexPath indexPath: NSIndexPath) -> CGFloat {
        return 80
    }

Great! Now you know how to implement your own custom cells for use in list
Screenlets.

## Sorted List by Comparator [](id=list-sorted-comparator)

As you can see in your list Screenlet properties, you can add an `obcClassName`.
With this parameter you can set an `OrderByComparator`. This class allows to
sort the results. If you want to set this comparator, you must add the full
className in your `@IBInspectable` property named `obcClassName`.

For example, in our `BookmarkListScreenlet` if you want to sort the results by
URL, you must set `obcClassName` to
`"com.liferay.bookmarks.util.comparator.EntryURLComparator"`.

But you can sort it by name, date, etc., with the proper comparator. This is an
optional property, so you can omit it. Be careful because `obcClassName` is
different in 6.2 and 7.0 version. Also, if there isn't the comparator you want,
you can create it yourself.

For creating a new comparator, you must create a class in your Liferay Portal
that extends `OrderByComparator<E>`. This `E` has to be the object model that
your list manages. After that, you have to override the methods you want for
your sorted list. For example, `BookmarkListScreenlet` can use
`EntryURLComparator`. This comparator class sort the results by URL:

    public class EntryURLComparator extends OrderByComparator<BookmarksEntry> {

        public static final String ORDER_BY_ASC = "BookmarksEntry.url ASC";

        public static final String ORDER_BY_DESC = "BookmarksEntry.url DESC";

        public static final String[] ORDER_BY_FIELDS = {"url"};

        public EntryURLComparator() {
            this(false);
        }

        public EntryURLComparator(boolean ascending) {
            _ascending = ascending;
        }

        @Override
        public int compare(BookmarksEntry entry1, BookmarksEntry entry2) {
            String url1 = StringUtil.toLowerCase(entry1.getUrl());
            String url2 = StringUtil.toLowerCase(entry2.getUrl());

            int value = url1.compareTo(url2);

            if (_ascending) {
                return value;
            }
            else {
                return -value;
            }
        }

        @Override
        public String getOrderBy() {
            if (_ascending) {
                return ORDER_BY_ASC;
            }
            else {
                return ORDER_BY_DESC;
            }
        }

        @Override
        public String[] getOrderByFields() {
            return ORDER_BY_FIELDS;
        }

        @Override
        public boolean isAscending() {
            return _ascending;
        }

        private final boolean _ascending;

    }

## List with Sections [](id=list-with-section)

One common pattern in iOS list is split its elements between sections. You can
achieve it in a very simple way. Linking with the previous example, imagine you
want to group our bookmarks by host. An **important** thing to notice is that
you have to order the content according to the sections you want to make, how we
just explained in the previous part of the tutorial
[previous part of the tutorial](#list-sorted-comparator)

In order tu support sections you need to revisit our brand new
`BookmarkListPageLoadInteractor` and add an extra method. This method is
`func sectionForRowObject(object: AnyObject) -> String?`
in this method, we need to return the section for the object argument, in our
case we will return the host for the current bookmark url.

The complete method will be like this:

    public override func sectionForRowObject(object: AnyObject) -> String? {
        guard let bookmark = object as? Bookmark else {
            return nil
        }

        let host = NSURL(string: bookmark.url)?.host?.lowercaseString

        return host?.stringByReplacingOccurrencesOfString("www.", withString: "")
    }

And that's all, from now you will see your list grouped by bookmark hosts.

## Related Topics [](id=related-topics)

[Creating iOS List Screenlets](/develop/tutorials/-/knowledge_base/7-0/creating-ios-list-screenlets)

[Packaging iOS Themes](/develop/tutorials/-/knowledge_base/7-0/packaging-ios-themes)

[Using Themes in iOS Screenlets](/develop/tutorials/-/knowledge_base/7-0/using-themes-in-ios-screenlets)
