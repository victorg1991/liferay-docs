# Creating iOS List Screenlets (Advanced) [](id=creating-ios-list-screenlets-advanced)

The 
[basic iOS list Screenlet creation tutorial](/develop/tutorials/-/knowledge_base/7-0/creating-ios-list-screenlets), 
shows you how to create a simple list Screenlet that displays a list of entities 
from a Liferay instance. The example Screenlet in that tutorial, Bookmark List 
Screenlet, displays a list of bookmarks from the Bookmarks portlet in a Liferay 
instance. But what if you want your list Screenlet to do more? What if you want 
to customize the cells in the list? What if you want to create a more complex 
list with 
[iOS's `UICollectionView`](https://developer.apple.com/reference/uikit/uicollectionview)? 
Then you're in luck! Screens can handle such functionality without breaking a 
sweat. This tutorial explains how to do these things, and more. As an example, 
it references code from the example 
[Bookmark List Screenlet](https://github.com/liferay/liferay-screens/tree/master/ios/Samples/Bookmark/BookmarkListScreenlet). 

First, you'll learn how to create custom cells with your list Screenlets. 

## Using Custom Cells with List Screenlets [](id=using-custom-cells-with-list-screenlets)

Liferay Screens's Default Theme uses the 
[iOS's `UITableView` class](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UITableView_Class/) 
as a UI component for list Screenlets. Although this works fine for showing
simple lists of items, you may want to use custom cells to spruce things up a
bit or show more complex content in the list. To do this, you must 
[create a custom theme](/develop/tutorials/-/knowledge_base/7-0/creating-ios-themes)
that extends the Default theme of the list Screenlet you want to use. 

First, create your new theme in your Screenlet's project following the steps 
described in the 
[Creating iOS Themes tutorial](/develop/tutorials/-/knowledge_base/7-0/creating-ios-themes). 
Then create your custom cell's XIB file. As usual, create your XIB's companion 
class and create as many outlets and actions as you need. Note that if you want 
to use different layouts for different rows, you must create several XIB files 
and accompanying companion classes. The name of both the XIB and Swift files 
should follow 
[the naming conventions](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices#ios-naming-convention)
specified in the 
[best practices tutorial](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices). 

The following screenshot shows Bookmark List Screenlet's XIB file 
`BookmarkCell_default-custom.xib`. This Screenlet needs to show a bookmark's 
name and URL, so it contains two labels. 

![Figure 1: The custom cell XIB for Bookmark List Screenlet.](../../../images/screens-ios-xcode-custom-cell.png)

The XIB's class, `BookmarkCell_default_custom`, contains an `@IBOutlet` for each 
label. Its `bookmark` variable also contains a `didSet` observer that sets the 
bookmark's name and URL to the appropriate label: 

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

Now you're ready to implement your Theme's functionality. In your 
`*ListView_mytheme` class, override the `doRegisterCellNibs` method to register
your custom cell. For example, Bookmark List Screenlet does this in its 
`BookmarkListView_default-custom` class: 

    public class BookmarkListView_default_custom: BookmarkListView_default {

      let BookmarkCellId = "bookmarkCell"


      //MARK: BaseListTableView

      public override func doRegisterCellNibs() {
          let nib = UINib(nibName: "BookmarkCell_default-custom", bundle: NSBundle.mainBundle())

          tableView?.registerNib(nib, forCellReuseIdentifier: BookmarkCellId)
    }

Note that this uses the constant `let BookmarkCellId = "bookmarkCell"` instead 
of a hardcoded string, as suggested in the 
[Avoid Hardcoded Strings](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices#avoid-hardcoded-strings)
section of 
[the best practices tutorial](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices).

Next, override the `doGetCellId` method to get the cell ID for each row. For 
example, this method in Bookmark List Screenlet returns the previous string 
constant:
<!-- In which class should this be done? -->

    override public func doGetCellId(row row: Int, object: AnyObject?) -> String {
        return BookmarkCellId
    }

To fill the cell with the row's data, override the `doFillLoadedCell` method.
Note that this method isn't called for in-progress cells; it's only called for
cells with data. Also note that the source data is stored in the method's
`object` argument. This is a generic object that you must cast to the specific
element type. For example, Bookmark List Screenlet's `doFillLoadedCell` method 
casts the `object` argument to a `Bookmark`. It then sets the `Bookmark` as the 
cell's bookmark: 
<!-- In which class should this be done? -->

    override public func doFillLoadedCell(row row: Int, cell: UITableViewCell, object:AnyObject) {
        if let bookmarkCell = cell as? BookmarkCell_default_custom, bookmark = object as? Bookmark {
            bookmarkCell.bookmark = bookmark
        }
    }

Remember that you also have the typical 
[`UITableViewDelegate` protocol](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UITableViewDelegate_Protocol/) 
and 
[`UITableViewDataSource` protocol](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UITableViewDataSource_Protocol/) 
methods available to you, so you can override any of them if you need to (check
first to make sure they're not already overridden). For example, Bookmark List 
Screenlet implement's the following method to use a different cell height for 
one row:
<!-- In which class should this be done? -->

    public func tableView(tableView: UITableView, heightForRowAtIndexPath indexPath: NSIndexPath) -> CGFloat {
        return 80
    }

Great! Now you know how to implement your own custom cells for use in list
Screenlets. 

Next, you'll learn how to sort your list's contents. 

## Sorting Your List

If you want to sort your list Screenlet's list, you must use a *comparator 
class* in your Liferay instance. A comparator class implements the logic 
required to sort your entities. You can create your own comparator 
class, or use ones that already exist in Liferay. 

To create a new comparator, you must create a class that extends Liferay's 
[`OrderByComparator` class](https://docs.liferay.com/portal/7.0/javadocs/portal-kernel/com/liferay/portal/kernel/util/OrderByComparator.html) 
with your entity as a type argument. Then you must override the methods required 
to implement the sort. For example, Liferay's 
[`EntryURLComparator` class](https://github.com/liferay/liferay-portal/blob/7.0.x/modules/apps/collaboration/bookmarks/bookmarks-api/src/main/java/com/liferay/bookmarks/util/comparator/EntryURLComparator.java) 
sorts bookmarks in the Bookmarks app by URL. 

<!-- 
Do Screenlet developers have to add support for the `obcClassName` 
property? If so, how? 
-->
To use the comparator, you must set the list Screenlet's `obcClassName` property 
to the comparator's fully qualified class name. You do this in Interface Builder 
when inserting the Screenlet in an app, just as you would set any other 
Screenlet property. 
<!--
"If you want to set this comparator, you must add the full className in your 
`@IBInspectable` property named `obcClassName`."

Does this have to be done in one of the Screenlet's classes? If so, we should 
include example code.
-->

For example, to set Bookmark List Screenlet to sort its results by URL, you must 
set `obcClassName` to 
`"com.liferay.bookmarks.util.comparator.EntryURLComparator"`. 

Be careful because `obcClassName` is different in 6.2 and 7.0 version. 
<!-- How is this property different between 6.2 and 7.0? -->

## Create Sections for Your List

Splitting lists into sections that contain like elements is common in iOS apps. 
To do this in list Screenlets, you must first use a comparator as described in 
the preceding section to sort the list by the criteria you'll use to create the 
sections. You must then override the `sectionForRowObject` method in your 
Interactor. This method is called for each item in the list, and should return 
the information necessary to place the item in a section. 

For example, you can create sections for grouping Bookmark List Screenlet's 
results by hostname. To do so, first sort the results with `EntryURLComparator` 
as detailed in the preceding section. In the `BookmarkListPageLoadInteractor` 
class, then override the `sectionForRowObject` method to return the URL's 
hostname: 

    public override func sectionForRowObject(object: AnyObject) -> String? {
        guard let bookmark = object as? Bookmark else {
            return nil
        }

        let host = NSURL(string: bookmark.url)?.host?.lowercaseString

        return host?.stringByReplacingOccurrencesOfString("www.", withString: "")
    }

Next, you'll learn how to use the `UICollectionView` class to create more 
complex lists in your list Screenlets. 

## Creating Complex Lists

Up to this point, your list Screenlets' Views have used 
[iOS's `UITableView` class](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UITableView_Class/) 
to display simple lists. As an experienced iOS developer, you probably know that 
although `UITableView` is great at creating simple lists, it's not so great at 
creating more complex lists like grids or stacks. For special list types like 
this, or your own custom lists, you should use 
[iOS's `UICollectionView` class](https://developer.apple.com/reference/uikit/uicollectionview) 
in your list Screenlet's View. 

Using the sample Bookmark List Screenlet as an example, this section shows you 
how to create such a View. First, you'll create the list's cell.

### Creating the Cell

You'll create your View's cell with the same sequence of steps that you use to 
create any cell for use in a list Screenlet. Note, however, that the steps' 
content is a bit different than usual: 

1. Create your cell's XIB file. Define your UI in this file. Because this cell 
   is part of a View that uses `UICollectionView`, you can shape it however you 
   want. For example, here's the `BookmarkCell_default-collection.xib` file for 
   the cell in Bookmark List Screenlet's custom View. It's a simple square that 
   displays the bookmark's URL and the URL's first letter. 

    ![Figure 1: The XIB file for the cell in Bookmark List Screenlet's custom View.](../../../images/screens-ios-collectionview-cell.png)

2. Create your XIB file's class by extending `UICollectionViewCell`. In this 
   class, create as many outlets and actions for your UI components as you need. 
   This class should also contain the logic required for your cell's UI to 
   function. For example, the XIB file's class in Bookmark List Screenlet is 
   `BookmarkCell_default_collection`. This class extends `UICollectionViewCell` 
   and contains outlets for the URL (`urlLabel`) and the URL's first letter 
   (`centerLabel`). Its `bookmark` variable also contains a `didSet` observer 
   that sets the bookmark's name and URL to the appropriate label. Note that the 
   `prepareForReuse` method is overridden to reset the labels when they need to 
   be reused: 

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

Now that your cell exists, you can create the View. 

### Creating the View

You'll create your View with the same sequence of steps you use to create a View 
in any list Screenlet. Because your View uses `UICollectionView` instead of 
`UITableView`, however, how you execute these steps is a bit different: 

1. Create your View's XIB file and define your UI in this file. Add a 
   `UICollectionView` instead of a `UITableView` to this file. 

2. Create the View class. Instead of extending `BaseListTableView`, this class 
   must extend Screens's 
   [`BaseListCollectionView` class](https://github.com/liferay/liferay-screens/blob/master/ios/Framework/Core/Base/BaseListScreenlet/CollectionView/BaseListCollectionView.swift). 
   This class implements most of the code necessary to use `UICollectionView` in 
   your Screenlet. By extending it, you can focus on the code unique to your 
   Screenlet. 

The name of both the XIB and the Swift files should follow 
[the naming convention](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices#ios-naming-convention) 
specified in 
[the best practices tutorial](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices). 
For example, Bookmark List Screenlet's XIB file and View class are 
`BookmarkListView_default-collection.xib` and 
`BookmarkListView_default_collection`, respectively. 

After creating your XIB, create your View class as indicated in step two above. 
For example, here's Bookmark List Screenlet's View class declaration: 
<!-- 
Provide the complete code for this View class
-->

    public class BookmarkListView_default_collection : BaseListCollectionView { ...

In Interface Builder, set this new class as your XIB's Custom Class, and assign 
an outlet to your `UICollectionView` component. Then register the cell you 
created in the previous section. To do this, you must override the 
`doRegisterCellNibs` method to create an 
[iOS `UINib`](https://developer.apple.com/reference/uikit/uinib) 
instance, and then register it to your collection view outlet. For example, the 
`doRegisterCellNibs` method in `BookmarkListView_default_collection` creates a 
`UINib` instance and registers it to its `UICollectionView` outlet 
(`collectionView`). Also note that this method uses the constant 
`let BookmarkCellId = "bookmarkCell"` for its cell reuse ID, instead of a 
hardcoded string. This is suggested in the 
[Avoid Hardcoded Strings](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices#avoid-hardcoded-strings)
section of 
[the best practices tutorial](/develop/tutorials/-/knowledge_base/7-0/ios-best-practices): 
<!-- 
This constant isn't created anywhere in the above code. If you show a constant 
or variable being used, then you have to show it being created... otherwise, 
it's confusing. The same goes for collectionView... show this outlet being 
created in the code.
-->

    public override func doRegisterCellNibs() {
        let cellNib = UINib(nibName: "BookmarkCell_default-collection", bundle: nil)
        collectionView?.registerNib(cellNib, forCellWithReuseIdentifier: BookmarkCellId)
    }

You must also override the `doGetCellId` method to specify the ID that you use 
to register the cell. For example, the `doGetCellId` method in 
`BookmarkListView_default_collection` returns `BookmarkCellId`: 

    public override func doGetCellId(indexPath indexPath: NSIndexPath, object: AnyObject?) -> String {
        return BookmarkCellId
    }

Next, you must override the methods in the View class that fill the table cells' 
contents. First, override the `doFillLoadedCell` method to set the object to use 
as the cell's contents. For example, the `doFillLoadedCell` method in 
`BookmarkListView_default_collection` set a `Bookmark` object as the cell's 
content. This method is executed once for each object received from the server: 

    public override func doFillLoadedCell(
            indexPath indexPath: NSIndexPath,
            cell: UICollectionViewCell,
            object: AnyObject) {

        if let cell = cell as? BookmarkCell_default_collection, bookmark = object as? Bookmark {
            cell.bookmark = bookmark
        }
    }

<!-- 
What are the other methods? You said that there are methods (plural) that fill 
the table cells' contents.
-->

Next, you'll create the layout. 

### Creating the layout

<!-- 
Where is this done? Inside the same View class?
-->
The layout object is a fundamental part of `UICollectionView`. This object 
controls the position of the elements, their size, and more. You can customize 
this object in your View by overriding the `doCreateLayout` method. For example, 
the `doCreateLayout` method in Bookmark List Screenlet's View uses 
`UICollectionViewFlowLayout` as its layout object. This is a basic layout that 
gives you a simple way to customize things like item size, spacing between 
items, scroll direction, and more. This layout is then used to set the item's 
size and spacing: 

    public override func doCreateLayout() -> UICollectionViewLayout {
        let layout = UICollectionViewFlowLayout()
        layout.itemSize = CGSize(width: 110, height: 110)
        layout.minimumLineSpacing = 10
        layout.minimumInteritemSpacing = 10

        return layout
    }

Cool! You're done! To use your new View, set your Screenlet's theme name 
variable in Interface Builder to the last part of your XIB's file name. For 
example, `BookmarkListView_default-collection.xib` is Bookmark List Screenlet's 
XIB for this example View. To use this View, set the Screenlet's theme name 
variable in Interface Builder to `default-collection`. 

If you want to package your View to contribute it to the Liferay Screens project 
or distribute it with CocoaPods, see 
[this section](/develop/tutorials/-/knowledge_base/7-0/creating-ios-themes#publish-your-themes-using-cocoapods)
in the tutorial on creating a View. 

## Related Topics [](id=related-topics)

[Creating iOS List Screenlets](/develop/tutorials/-/knowledge_base/7-0/creating-ios-list-screenlets)

[Packaging iOS Themes](/develop/tutorials/-/knowledge_base/7-0/packaging-ios-themes)

[Using Themes in iOS Screenlets](/develop/tutorials/-/knowledge_base/7-0/using-themes-in-ios-screenlets)
