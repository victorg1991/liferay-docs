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

Note that we use a constant `let BookmarkCellId = "bookmarkCell"` instead of a
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

## Use Collection View instead of Table View [](id=use-collection-view-instead-table-view)

Liferay Screens lets you use [the iOS `UICollectionView` class](https://developer.apple.com/reference/uikit/uicollectionview)
to create additional Views for your list Screenlets. Using `UICollectionView` in
your View lets your list Screenlet present in a much more flexible list-based UI.

This tutorial shows you how to create such a View using the sample Bookmark List
Screenlet as an example.

### Creating the Cell

First, we'll need to create the cell for each element of our Collection View. In
consecuence, your cell's class must extend `UICollectionViewCell`. When you
create your cell's class in Xcode, select the option that creates a XIB file
too, and call it `BookmarkCell_default-collection.xib`.

![Figure 1: The Collection View cell XIB.](../../../images/screens-ios-collectionview-cell.png)

Now, add all the view logic to your cell's view class:

    import UIKit
    import LiferayScreens

    public class BookmarkCell_default_collection: UICollectionViewCell {


        //MARK: Outlets

        @IBOutlet weak var centerLabel: UILabel?

        @IBOutlet weak var urlLabel: UILabel?


        //MARK: Public properties

        public var bookmark: Bookmark? {
            didSet {
                if let bookmark = bookmark, url = NSURL(string: bookmark.url),
                        firstLetter = url.host?.remove("www.").characters.first {
                    self.centerLabel?.text = String(firstLetter).uppercaseString

                    self.urlLabel?.text = bookmark.url.remove("http://").remove("https://").remove("www.")
                }
            }
        }


        //MARK: UICollectionViewCell

        override public func prepareForReuse() {
            super.prepareForReuse()

            centerLabel?.text = "..."
            urlLabel?.text = "..."
        }
    }

### Creating the view

First create a new XIB file called `BookmarkListView_default-collection.xib`.
Use Interface Builder to construct your Screenlet's UI in this file. Since the
Screenlet must show a list of items, you should add a `UICollectionView` to this
XIB.

Now create a new View class with a name that matches that of the XIB file's
prefix. The name of both the XIB and the Swift files should follow
the [Naming Convention](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices#ios-naming-convention)
specified in the [Best Practices](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices).

Since the XIB uses UICollectionView, your View class must extend
`BaseListCollectionView. For example, this is Bookmark Collection List
Screenlet's View class declaration:

    public class BookmarkListView_default_collection : BaseListCollectionView { ...

In Interface Builder, set this new class as your XIB's Custom Class, and assign
the `collectionView` Outlet to your UICollectionView component.

After doing this, you can start adding code to the view class, the first thing
to do is to register the cell you created in the previous section.

To register the cell you have to override the `doRegisterCellNibs` method like
this:

    public override func doRegisterCellNibs() {
        let cellNib = UINib(nibName: "BookmarkCell_default-collection", bundle: nil)
        collectionView?.registerNib(cellNib, forCellWithReuseIdentifier: BookmarkCellId)
    }

Note that we use a constant `let BookmarkCellId = "bookmarkCell"` instead of a
hardcoded string, as suggested in the [Avoid Hardcoded Strings](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices#avoid-hardcoded-strings)
chapter of the [Best Practices](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices).

You also have to specify the identifier that you use to register the cell in the
method `doGetCellId`:

    public override func doGetCellId(indexPath indexPath: NSIndexPath, object: AnyObject?) -> String {
        return BookmarkCellId
    }

Next, you must override the methods in the View class that fill the table cells'
contents

First let's override `doFillLoadedCell` you have to set the Bookmark received
in the object argument into the cell:

    public override func doFillLoadedCell(
            indexPath indexPath: NSIndexPath,
            cell: UICollectionViewCell,
            object: AnyObject) {

        if let cell = cell as? BookmarkCell_default_collection, bookmark = object as? Bookmark {
            cell.bookmark = bookmark
        }
    }

## Creating the layout [](id=creating-layout)

The layout object is a fundamental part in the UICollectionView, it controls the
position of the elements, size, etc.

You can customize it for your screenlet view in the method `doCreateLayout`:

    public override func doCreateLayout() -> UICollectionViewLayout {
        let layout = UICollectionViewFlowLayout()
        layout.itemSize = CGSize(width: 110, height: 110)
        layout.minimumLineSpacing = 10
        layout.minimumInteritemSpacing = 10

        return layout
    }

In this example, you use the UICollectionViewFlowLayout which is the basic
layout that allows you to customize in an easy way things like size of the
items, spacing between items, scroll direction, etc.

Cool! You're done! If you want to use this new view you will have to go to the
storyboard where you put your Screenlet and change its theme name variable to
`default-collection`.

And this is it! Your screenlet is now more prepared than ever to be used
(and reused) in a real environment. Now more than ever you should
[package](/develop/tutorials/-/knowledge_base/7-0/creating-ios-themes#publish-your-themes-using-cocoapods)
it to contribute to the Screens project or distribute with CocoaPods.

## Related Topics [](id=related-topics)

[Creating iOS List Screenlets](/develop/tutorials/-/knowledge_base/7-0/creating-ios-list-screenlets)

[Packaging iOS Themes](/develop/tutorials/-/knowledge_base/7-0/packaging-ios-themes)

[Using Themes in iOS Screenlets](/develop/tutorials/-/knowledge_base/7-0/using-themes-in-ios-screenlets)
