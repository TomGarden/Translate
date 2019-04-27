

## 0x00、 获取联系人列表

### 写在开始
在阅读本文前请知晓： 本文的技术已经不建议被使用了，它的替代技术为 [ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel#loaders)

Reference：https://stackoverflow.com/questions/51408098/what-is-the-appropriate-replacer-of-deprecated-getsupportloadermanager

----
正文开始


本文展示了获取匹配搜索内容的联系人列表。
使用如下技术点：

1. 匹配联系人姓名
    -   通过匹配(联系人姓名)搜索字段获取一个联系人列表。
        Content provider 允许相同名字的多个实例，所有此技术能返回一个匹配列表。
2.  匹配指定类型的数据，例如电话号码
    -   通过匹配特定类型(例如邮件地址)数据获得一个联系人列表。
        例如，此技术允许你列出所有与搜索字段相匹配的邮件所绑定的联系人的信息。
3.  匹配任何类型的数据
    -   通过搜索字段匹配任何类型(姓名、电话、地址、邮箱、等等)的数据从而获得一个联系人列表。
        此技术允许你与所有类型的数据进行匹配并从而获得一个联系人列表。

> **☆ 注意：** 本节例子中使用一个 `CursorLoader` 从 `ContentProvider` 获取数据。
    一个 `CursorLoader` 在子线程中进行检索动作。
    这确保了检索动作不会拖慢 UI 线程的节奏从而降低用户体验。
    有关于此的更多信息请查看 `Loading Data in the Background` 。

## 0x01、 请求读取 provider 的权限

要从 `ContentProvider` 进行任何类型的数据检索你的 app 都必须体检具有 `READ_CONTACTS` 权限。
要请求这个权限需要在 manifest 文件中 `<manifest>` 节点下增加 `<uses-permission>` 元素：

```xml
<uses-permission android:name="android.permission.READ_CONTACTS" />
```

## 0x02、 通过姓名进行匹配并列出结果列表

此技术尝试从 ContentProvider 的 `ContactsContract.Contacts` 表中通过 string 去 和姓名进行匹配。
你通常希望将匹配结果显示在一个 ListView 中，从而允许用户从结果中进行选择。

### 2.1、 定义 ListView 和 item 布局

要在 ListView 中显示搜索结果，你需要一个主布局文件定义一个包含 ListView 的 UI ，和一个 ListView 要显示的 item 的 UI 。
例如你可以创建通过如下代码创建 主布局文件：

```xml
<?xml version="1.0" encoding="utf-8"?>
<ListView xmlns:android="http://schemas.android.com/apk/res/android"
          android:id="@android:id/list"
          android:layout_width="match_parent"
          android:layout_height="match_parent"/>
```

XML 使用了 Android 内建的 ListView 控件 `android:id/list` 。

使用下面的代码定义 item 布局文件 contacts_list_item.xml ：

```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
          android:id="@android:id/text1"
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:clickable="true"/>
```

XML 使用了 Android 内建的 TextView 控件 `android:text1`

> **☆ 注意：** 本节未描述获取用户搜索内容的 UI ，因为你可能并非直接获取检索字段。
    例如，你可以给用户一个选项即从已经输入的文本信息匹配一个联系人。
    
这两个布局文件定义了展示 `ListView` 的 UI 。
下面的代码将用这个 UI 展示搜索内容。

### 2.2、 定义一个 Fragment 展示 list contacts。

要展示联系人列表，从定义一个展示在 Activity 中的 Fragment 开始。
使用 Fragment 是一个非常灵活的技术，因为你能使用一个 Fragment 展示 list，当用户选中一个 contact 后使用另一个 Fragment 展示联系人详情。
通过这个方法，你能将本文中的技术结合起来用于获取联系人详情。

要学习如何在 Activity 使用一个或多个 Fragment 对象，可以读 [Build a dynamic UI with Fragments](https://developer.android.com/training/basics/fragments/index.html)

为了帮助开发者针对 Contacts Provider 进行查询，Android framewor 提供了 `ContactsContaract` ,它定义了有助于访问 provider 的内容和方法。
当你使用这个类的时候，你不必定义自己的常量(content URIs, table names, or columns.) 。

此类导入：

```java
import android.provider.ContactsContract;
```

因为这个类使用 CursorLoader 从 provider 获取数据，所以你必须实现接口 `LoaderManager.LoaderCallbacks` 。
另外，当用户从结果列表中选定了联系人 `AdapterView.OnItemClickListener` 此类帮我们截获此事件：

```java
...
import android.support.v4.app.Fragment;
import android.support.v4.app.LoaderManager.LoaderCallbacks;
import android.widget.AdapterView;
...
public class ContactsFragment extends Fragment implements
        LoaderManager.LoaderCallbacks<Cursor>,
        AdapterView.OnItemClickListener {
```

### 2.3、 定义全局变量

```java
    ...
    /*
     * Defines an array that contains column names to move from
     * the Cursor to the ListView.
     */
    @SuppressLint("InlinedApi")
    private final static String[] FROM_COLUMNS = {
            Build.VERSION.SDK_INT
                    >= Build.VERSION_CODES.HONEYCOMB ?
                    ContactsContract.Contacts.DISPLAY_NAME_PRIMARY :
                    ContactsContract.Contacts.DISPLAY_NAME
    };
    /*
     * Defines an array that contains resource ids for the layout views
     * that get the Cursor column contents. The id is pre-defined in
     * the Android framework, so it is prefaced with "android.R.id"
     */
    private final static int[] TO_IDS = {
           android.R.id.text1
    };
    // Define global mutable variables
    // Define a ListView object
    ListView contactsList;
    // Define variables for the contact the user selects
    // The contact's _ID value
    long contactId;
    // The contact's LOOKUP_KEY
    String contactKey;
    // A content URI for the selected contact
    Uri contactUri;
    // An adapter that binds the result Cursor to the ListView
    private SimpleCursorAdapter cursorAdapter;
    ...
```

> **☆注意：** 因为 `Contacts.DISPLAY_NAME_PRIMARY` 需要 Android3.0(API 11) 或之后的版本，如果你的 minSdkVersion `<=` 10 AndroidStudio 会发出警告，可以通过 `@SuppressLint("InlinedApi")` 关闭此警告。

### 2.4、 初始化 Fragment

初始化 Fragment 。
Android 系统要求我们增加空的 public 的构造方法，并且在回调方法 `onCreateView()` 中 inflate 布局。

```java
    // Empty public constructor, required by the system
    public ContactsFragment() {}

    // A UI Fragment must inflate its View
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
            Bundle savedInstanceState) {
        // Inflate the fragment layout
        return inflater.inflate(R.layout.contact_list_fragment,
            container, false);
    }
```

### 2.5、 为 ListView 设置 CursorAdapter

将绑定了搜索结果的 SimpleCursorAdapter 设置到 ListView 中。
获取用于展示内容的 ListView 对象，你需要使用 Fragment 的父 activity 调用 Activity.findViewById() 。
当使用 setAdapter() 的时候你需要使用父 Activity 的 Context 。

```java
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        ...
        // Gets the ListView from the View list of the parent activity
        contactsList =
            (ListView) getActivity().findViewById(R.layout.contact_list_view);
        // Gets a CursorAdapter
        cursorAdapter = new SimpleCursorAdapter(
                getActivity(),
                R.layout.contact_list_item,
                null,
                FROM_COLUMNS, TO_IDS,
                0);
        // Sets the adapter for the ListView
        contactsList.setAdapter(cursorAdapter);
    }
```

### 2.6、 设置 ListView 的 item 的选择监听

当你展示了搜索结果，你通常允许用户进一步选择一个已经展示的条目。
例如，当用户点击了一个 联系人 你能在一个 map 中展示用户地址。
要提供这样的功能，你首先需要通过实现 `AdapterView.OnItemClickListener` 定义当前 Fragment 作为 点击事件的监听者，这些内容在 [Define a Fragment that displays the list of contacts](https://developer.android.com/training/contacts-provider/retrieve-names.html#Fragment) 。

在 `onActivityCreated()` 中通过调用 `setOnItemClickListener()` 将 listener 与 ListView 绑定起来。

```java
    public void onActivityCreated(Bundle savedInstanceState) {
        ...
        // Set the item click listener to be the current fragment.
        contactsList.setOnItemClickListener(this);
        ...
    }
```

因为你指定了当前 Fragment 是 ListView 的 OnItemClickListener ，所以你需要在 Fragment 中实现 `onItemClick()` 方法，此方法用户捕获点击事件。

### 2.7、 定义一个 映射

定义一个常量，其中包含你希望通过查询返回的 列。
ListView 中的每一个 item 展示 联系人的名字(这是联系人对象的一个主要的参数)。
姓名的列名为
-   Android3.0(API 11)和之后的版本 ：`Contacts.DISPLAY_NAME_PRIMARY`
-   之前的版本 ： `Contacts.DISPLAY_NAME`

列 `Contacts._ID` 被 `SimpleCursorAdapter` 进行绑定。
`Contacts._ID` 和 `LOOKUP_KEY` 一起用于为用户选择的联系人构造 URI 内容。

```java
...
@SuppressLint("InlinedApi")
private static final String[] PROJECTION =
        {
            Contacts._ID,
            Contacts.LOOKUP_KEY,
            Build.VERSION.SDK_INT
                    >= Build.VERSION_CODES.HONEYCOMB ?
                    ContactsContract.Contacts.DISPLAY_NAME_PRIMARY :
                    ContactsContract.Contacts.DISPLAY_NAME

        };
```

### 2.8、 为 Cursor 列索引定义常量

要从 Cursor 中指定的列获取数据，你需要知道他们在 Cursor 中的索引。
你可以为 Cursor 的列索引定义常量，因为这列索引的顺序和你映射的顺序是相同的：

```java
// The column index for the _ID column
private static final int CONTACT_ID_INDEX = 0;
// The column index for the CONTACT_KEY column
private static final int CONTACT_KEY_INDEX = 1;
```

### 2.9、 指定选择标准

你想要特定的数据，创建一个文本变量表达式的组合，它将告诉 provider 要检索的列和要发现的值。
对于文本表达式，定义一个常量，它包含了要检索的列。
虽然这个表达式也能包含值，最佳实践是用 `？` 作为占位符替换值。
在检索 期间，占位符会被数组中的值替换。
使用 `？` 作为占位符可以确保规范是通过绑定生成，而非通过编译生成。
这个实践消除了恶意 SQL 注入的极大可能。 
```java
    // Defines the text expression
    @SuppressLint("InlinedApi")
    private static final String SELECTION =
            Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB ?
            Contacts.DISPLAY_NAME_PRIMARY + " LIKE ?" :
            Contacts.DISPLAY_NAME + " LIKE ?";
    // Defines a variable for the search string
    private String searchString;
    // Defines the array to hold values that replace the ?
    private String[] selectionArgs = { searchString };
```

### 2.10、 定义 onItemClick() 函数

在前面的小节中，你为 ListView 设置了 item 点击事件。还没有实现点击回调方法：

```java
    @Override
    public void onItemClick(
        AdapterView<?> parent, View item, int position, long rowID) {
        // Get the Cursor
        Cursor cursor = parent.getAdapter().getCursor();
        // Move to the selected contact
        cursor.moveToPosition(position);
        // Get the _ID value
        contactId = cursor.getLong(CONTACT_ID_INDEX);
        // Get the selected LOOKUP KEY
        contactKey = cursor.getString(CONTACT_KEY_INDEX);
        // Create the contact's content Uri
        contactUri = Contacts.getLookupUri(contactId, mContactKey);
        /*
         * You can use contactUri as the content URI for retrieving
         * the details for a contact.
         */
    }
```
### 2.11、 初始化 loader

因为你使用了 CursorLoader 去获取数据，你必须初始化后台线程，和其他异步检索的变量。
在 `onActivityCreated()` 中进行初始化操作，该方法在 Fragment UI 出现之前调用。

```java
public class ContactsFragment extends Fragment implements
        LoaderManager.LoaderCallbacks<Cursor> {
    ...
    // Called just before the Fragment displays its UI
    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        // Always call the super method first
        super.onActivityCreated(savedInstanceState);
        ...
        // Initializes the loader
        getLoaderManager().initLoader(0, null, this);
```

### 2.12、 实现 onCreateLoader()

当你调用 `initLoader()` 方法后 `onCreateLoader()` 会被立即调用，我们要实现这个方法。

在 `onCreateLoader()` 方法中设置模糊搜索。
要将字符串设置为模糊搜索，可以将  `%` 作为 >= 0 个字符， `_` 作为一个字符。
例如， "%Jefferson%"  能匹配到类似 "Thomas Jefferson" 和 "Jefferson Davis" 这样的结果。

方法 return 一个新的 CursorLoader。
content URI 使用 Contacts.CONTENT_URI 。
这个 URI 代表了一个实体表。

```java
    ...
    @Override
    public Loader<Cursor> onCreateLoader(int loaderId, Bundle args) {
        /*
         * Makes search string into pattern and
         * stores it in the selection array
         */
        selectionArgs[0] = "%" + searchString + "%";
        // Starts the query
        return new CursorLoader(
                getActivity(),
                ContactsContract.Contacts.CONTENT_URI,
                PROJECTION,
                SELECTION,
                selectionArgs,
                null
        );
    }
```

### 2.13、 实现 onLoadFinished() 和 onLoaderReset()

实现 `onLoadFinished()` 函数。
当 Contacts Provider 返回了结果实体后，会调用 `onLoadFinished()` .
在这个方法中将 结果 `Cursor` 绑定到 `SimpleCursorAdapter` .
浙江自动更新 ListView 为新的检索结果。

```java
    @Override
    public void onLoadFinished(Loader<Cursor> loader, Cursor cursor) {
        // Put the result Cursor in the adapter for the ListView
        cursorAdapter.swapCursor(cursor);
    }
```

当 loader framework 发现结果 Cursor 中包含过期(不新鲜了)的值的时候调用。
删除 `SimpleCursorAdapter` 对现有 Cursor 的引用。
如果你不这么做， loader framework 将不会回收 Cursro，这将造成内存泄漏：

```java
    @Override
    public void onLoaderReset(Loader<Cursor> loader) {
        // Delete the reference to the existing Cursor
        cursorAdapter.swapCursor(null);

    }
```

## 0x03、 通过特定类型数据匹配联系人

这个技术允许你指定你想要匹配的数据类型。
获取姓名是按类型搜索的一个特殊的例子，你能对联系人相关的任何类型的数据进行指定类别的搜索。
例如，你能获取有指定邮政编码的联系人；
在这种情况下，搜索字符串必须匹配邮政编码。

要实现按类别搜索，首先应该实现下面的代码：

上面章节提到的：
- Request Permission to Read the Provider.
- Define ListView and item layouts.
- Define a Fragment that displays the list of contacts.
- Define global variables.
- Initialize the Fragment.
- Set up the CursorAdapter for the ListView.
- Set the selected contact listener.
- Define constants for the Cursor column indexes.
  Although you're retrieving data from a different table, the - order of the columns in the projection is the same, so you can - use the same indexes for the Cursor.
- Define the onItemClick() method.
- Initialize the loader.
- Implement onLoadFinished() and onLoaderReset().

下面的步骤展示了你要完成针对特定数据类型进行匹配与检索并展示搜索结果所需要做的额外的事情：

### 3.1、 选择数据类型和数据表

要针对数据的特定类型进行搜索，你必须知道数据的 MIME 类型值。
每一种数据类型都有独一无二的 MIME 类型值，它通过子类 `ContactsContract.CommonDataKinds` 中的 `CONTENT_ITEM_TYPE` 定义相关的数据类型。
子类的名字代表了数据类型；例如 email 数据对应的子类为 `ContactsContract.CommonDataKinds.Email` 邮件的自定义 MIME 类型被常量 `Email.CONTENT_ITEM_TYPE` 表示。

使用 `ContactsContract.Data` 表完成你的搜索。 
你的 projection，selection 和 sort order 都在此表中定义或继承。

### 3.2、 定义 projection

要定义一个 projection ，选择一个或多个列定义在 ContactsContract.Data 或者它的内部类中。
Contacts Provider 在 `ContactsContract.Data` 和 其他表之间做了隐式链接。
```java
    @SuppressLint("InlinedApi")
    private static final String[] PROJECTION =
        {
            /*
             * The detail data row ID. To make a ListView work,
             * this column is required.
             */
            ContactsContract.Data._ID,
            // The primary display name
            Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB ?
                    ContactsContract.Data.DISPLAY_NAME_PRIMARY :
                    ContactsContract.Data.DISPLAY_NAME,
            // The contact's _ID, to construct a content URI
            ContactsContract.Data.CONTACT_ID,
            // The contact's LOOKUP_KEY, to construct a content URI
            ContactsContract.Data.LOOKUP_KEY // A permanent link to the contact
        };
```

### 3.3、 定义搜索标准

要针对特定数据类型使用 string 进行搜索，遵从下面条目中构造 select 子句：
-   列名包含你的检索字段。
    变量名体现数据类型，所以你查找 `ContactsContract.CommonDataKinds` 的子类，它对应数据类型，然后从它的子类中选择列名。
    例如要搜索 email 地址，使用 Email.ADDRESS 列。
-   在 selection 语句中搜索字符串本身被表示为 `？` 。
-   column 名字包含自定义 MIME 数据类型值。
    这个名字总是 `Data.MIMETYPE` 。
-   为数据类型之定义 MIME 类型。
    如前所述，内容 `CONTENT_ITEM_TYPE` 在 `ContactsContract.CommonDataKinds` 子类中。
    例如，email 数据的 MIME 类型值是 `Email.CONTENT_ITEM_TYPE` 。
    搜索语句需要被 `'`(单引号) 包含；另外， provider 将值解释为变量类型而不是字符串。
    您不需要为此值使用占位符，因为您使用的是常量而不是用户提供的值。

```java
    /*
     * Constructs search criteria from the search string
     * and email MIME type
     */
    private static final String SELECTION =
            /*
             * Searches for an email address
             * that matches the search string
             */
            Email.ADDRESS + " LIKE ? " + "AND " +
            /*
             * Searches for a MIME type that matches
             * the value of the constant
             * Email.CONTENT_ITEM_TYPE. Note the
             * single quotes surrounding Email.CONTENT_ITEM_TYPE.
             */
            ContactsContract.Data.MIMETYPE + " = '" + Email.CONTENT_ITEM_TYPE + "'";
```

接下来定义包含selection 参数的变量
```java
    String searchString;
    String[] selectionArgs = { "" };
```

### 3.4、 实现 onCreateLoader()

现在你已经指定了你希望查找的数据，在 `onCreateLoader()` 中定义一个查询吧。
本方法使用你的 projection\selection 返回一个 `CursorLoader` 对象，并且 selection 数组提供查询参数细节。
 content RUI 使用  `Data.CONTENT_RUI` :
```java
@Override
    public Loader<Cursor> onCreateLoader(int loaderId, Bundle args) {
        // OPTIONAL: Makes search string into pattern
        searchString = "%" + searchString + "%";
        // Puts the search string into the selection criteria
        selectionArgs[0] = searchString;
        // Starts the query
        return new CursorLoader(
                getActivity(),
                Data.CONTENT_URI,
                PROJECTION,
                SELECTION,
                selectionArgs,
                null
        );
    }
```

这些代码段是基于特定类型的详细数据进行简单反向查找的基础。如果您的应用程序关注特定类型的数据（如电子邮件），并且您希望允许用户获取与某个数据相关联的名称，那么这是最好的方法。

## 0x04、 匹配任意任意类型的 contact

不限制数据类型进行数据检索。
这将产生大量的搜索结果。
例如，如果你的检索字符串是 `Doe` ，会返回名为 `John Doe` 或者街道为 `Doe Street` 之类的数据。

要实现这种操作，首先需要按上文所述实现代码：

- Request Permission to Read the Provider.
- Define ListView and item layouts.
- Define a Fragment that displays the list of contacts.
- Define global variables.
- Initialize the Fragment.
- Set up the CursorAdapter for the ListView.
- Set the selected contact listener.
- Define a projection.
- Define constants for the Cursor column indexes.
  For this type of retrieval, you're using the same table you used in the section Match a contact by name and list the results. Use the same column indexes as well.
- Define the onItemClick() method.
- Initialize the loader.
- Implement onLoadFinished() and onLoaderReset().

下面的步骤展示了要做全类型数据查询你需要做的额外的事情。

### 4.1、 移除 selection 标准

不要定义 SELECTION 内容，不要定义 mSelectionArgs 变量。
他们不用于此种类型的搜索。

### 4.2、实现 onCreateLoader()

实现 `onCreateLoader()` 方法，返回一个新的 `CursorLoader` 。
你不需要转换搜索字符串为模糊搜索，因为 Contacts Provider 会自动完成它。
使用 `Contacts.CONTENT_FILTER_URI` 作为基础 URI ，通过调用 `Uri.withAppendedPath()` 将你的搜索字符串追加上去。
使用这个 URI 自动触发搜索任意类型的数据，如下所示：

```java
    @Override
    public Loader<Cursor> onCreateLoader(int loaderId, Bundle args) {
        /*
         * Appends the search string to the base URI. Always
         * encode search strings to ensure they're in proper
         * format.
         */
        Uri contentUri = Uri.withAppendedPath(
                Contacts.CONTENT_FILTER_URI,
                Uri.encode(searchString));
        // Starts the query
        return new CursorLoader(
                getActivity(),
                contentUri,
                PROJECTION,
                null,
                null,
                null
        );
    }
```

这些代码片段是对联系人提供商进行广泛搜索的应用程序的基础。这项技术对于想要实现类似于People应用程序联系人列表屏幕的功能的应用程序很有用。