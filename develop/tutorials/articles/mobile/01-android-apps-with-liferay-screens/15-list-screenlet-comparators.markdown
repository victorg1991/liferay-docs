# List Screenlet Comparators

List Screenlets contain a `getPageRowsRequest` method that has a parameter named 
query. With this parameter, you can set an `OrderByComparator` class. This class 
allows to sort the results. If you want to set this comparator, you must add the 
`obcClassName` property in your screenlet:

	<com.liferay.mobile.screens.listbookmark.BookmarkListScreenlet
		android:id="@+id/bookmarklist_screenlet"
		android:layout_width="match_parent"
		android:layout_height="match_parent"
		app:cachePolicy="REMOTE_FIRST"
		app:folderId="@string/liferay_bookmark_folder"
		app:layoutId="@layout/list_bookmarks"
		app:obcClassName="com.liferay.bookmarks.util.comparator.EntryURLComparator"
		/>
		
In the example, the results are sorted by URL. But you can sort it by name, 
date, etc., with the proper comparator. This is an optional property, so you can 
omit it. Be careful because `obcClassName` is different in 6.2 and 7.0 version. 
Also, if there isn't the comparator you want, you can create it yourself.

For creating a new comparator, you must create a class that extends 
`OrderByComparator<E>`. This `E` has to be the object model that your list 
manages. After that, you have to override the methods you want for your sorted 
list. For example, `BookmarkListScreenlet` uses `EntryURLComparator`. This 
comparator class sort the results by URL:

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