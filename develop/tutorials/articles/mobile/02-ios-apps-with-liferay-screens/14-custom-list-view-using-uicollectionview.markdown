# Creating Views for list Screenlets

Liferay Screens lets you use 
[the iOS `UICollectionView` class](https://developer.apple.com/reference/uikit/uicollectionview) 
to create additional Views for your list Screenlets. Using `UICollectionView` in 
your View lets your list Screenlet present a much more flexible list-based UI. 

This tutorial shows you how to create Such a View using the sample Bookmark List 
Screenlet as an example. You can find the finished code here in 
[here in GitHub](https://github.com/liferay/liferay-screens/tree/master/ios/Samples/Bookmark/BookmarkListScreenlet).  
Note that because this tutorial only explains creating the View, you should read 
[the list Screenlet creation tutorial](/develop/tutorials/-/knowledge_base/7-0/creating-ios-list-screenlets) 
before beginning here. The list Screenlet creation tutorial walks you through 
the creation of the sample Bookmark List Screenlet. 
<!-- Replace the BookmarkListScreenlet GitHub link with a link to the View -->

## Creating the Cell

First, you must create the cell you'll use to show each item in your list. 
Because you want your View to use `UICollectionView`, your cell's class must 
extend `UICollectionViewCell`. When you create your cell's class in Xcode, 
select the option that creates an XIB file too, and call it 
`BookmarkCell_default_collection.xib`. 

![Figure 1: Here's the collection view cell finished.](../../../images/screens-ios-collectionview-cell.png)

Now, you must create two outlets, one for the big letter in the middle of the 
cell and one for the url below. 

    public class BookmarkCell_default_collection: UICollectionViewCell {

        @IBOutlet weak var centerLabel: UILabel!

        @IBOutlet weak var urlLabel: UILabel!
    }

## Creating the view [](id=creating-the-view)

First create a new XIB file called `BookmarkListView_default-collection.xib`. 
Use Interface Builder to construct your Screenlet's UI in this file. Since the 
Screenlet must show a list of items, you should add a `UICollectionView` to this 
XIB.

Now create a new View class with a name that matches that of the XIB file's 
prefix. Since the XIB uses UICollectionView, your View class must extend 
BaseListCollectionView. For example, this is Bookmark Collection List 
Screenlet's View class declaration:

    public class BookmarkListView_default_collection : BaseListCollectionView { ...
	
In Interface Builder, set this new class as your XIB's Custom Class, and assign 
the collectionView outlet to your UICollectionView component.

After doing this, you can start adding code to the view class, the first thing 
to do is to register the cell you created in the previous section. In this 
example there is only one cell type but if you want more than one you should 
register them at this point.

To register the cell you have to override the `doRegisterCellNibs` method like 
this:

    public override func doRegisterCellNibs() {
        let cellNib = UINib(nibName: "BookmarkCell_default-collection", bundle: nil)
        collectionView?.registerNib(cellNib, forCellWithReuseIdentifier: "bookmarkCell")
    }

You also have to specify the identifier that you use to register the cell in the 
method `doGetCellId`:

    public override func doGetCellId(indexPath indexPath: NSIndexPath,
        object: AnyObject?) -> String {

        return "bookmarkCell"
    }

Next, you must override the methods in the View class that fill the table cells' 
contents. There are two methods for this, depending on the type of cell being 
filled: 

- **Normal cells:** the cells that show the entities. These cells typically use 
  `UILabel`, `UIImage`, or any other UI component to show the entity's 
  attributes. Override the `doFillLoadedCell` method to fill this type of cell. 

- **Progress cell:** the cell that show the entities that are not already 
  loaded. Override the `doFillInProgressCell` method to fill this type of cell. 
  
First let's override `doFillLoadedCell` you have to set the values of the 
Bookmark received in the object argument into the cell:
  
    public override func doFillLoadedCell(indexPath indexPath: NSIndexPath,
        cell: UICollectionViewCell, object: AnyObject) {

        let bookmarkCell = cell as! BookmarkCell_default_collection
        let bookmar = object as! Bookmark

        bookmarkCell.centerLabel.text = bookmark.name.characters.first!
        bookmarkCell.urlLabel.text = bookmark.url

    }

And then override the method `doFillInProgressCell` where you have to fill the 
cell with default values: 
			
    public override func doFillLoadedCell(indexPath indexPath: NSIndexPath,
        cell: UICollectionViewCell) {

        let bookmarkCell = cell as! BookmarkCell_default_collection

        bookmarkCell.centerLabel.text = "..."
        bookmarkCell.urlLabel.text = "Loading..."

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

## Related Topics [](id=related-topics)

[Creating iOS List Screenlets](/develop/tutorials/-/knowledge_base/7-0/creating-ios-list-screenlets)

[Packaging iOS Themes](/develop/tutorials/-/knowledge_base/7-0/packaging-ios-themes)

[Using Themes in iOS Screenlets](/develop/tutorials/-/knowledge_base/7-0/using-themes-in-ios-screenlets)
