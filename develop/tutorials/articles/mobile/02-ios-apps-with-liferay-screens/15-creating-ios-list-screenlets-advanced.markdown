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

## Related Topics [](id=related-topics)

[Creating iOS List Screenlets](/develop/tutorials/-/knowledge_base/7-0/creating-ios-list-screenlets)

[Packaging iOS Themes](/develop/tutorials/-/knowledge_base/7-0/packaging-ios-themes)

[Using Themes in iOS Screenlets](/develop/tutorials/-/knowledge_base/7-0/using-themes-in-ios-screenlets)
