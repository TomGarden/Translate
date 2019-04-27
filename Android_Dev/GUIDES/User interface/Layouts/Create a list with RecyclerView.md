## 0x00、 [用 RecyclerView 创建一个列表](https://developer.android.com/guide/topics/ui/layout/recyclerview#java)

## 0x01、 RecyclerView 概览

RecyclerView 比 ListView 更优秀更灵活。

在 RecyclerView 中，几种不同的组件一起工作展示你的数据。
你的用户界面整体上是一个 RecyclerView 容器对象。
RecyclerView 通过你提供的 *LayoutManager* 中的内容填充它自身。
你能使用一个标准的 *LayoutManager* (例如 LinearlayoutManager 或者 GridLayoutManager )，也可以自己实现一个。

列表中的 view 被 `view holder` 对象代表和持有。
这些对象是你继承自 `RecyclerView.ViewHolder` 类的一个实例。
每一个 viewHoler 负责展示一个 item 。
例如，如果你的列表展示音乐集合，每一个 view holder 或许会代表一个专辑。
RecyclerView 仅创建当前屏幕需要展示的动态内容，和一点额外的东西。
当用户滚动显示列表的时候，RecyclerView 将从屏幕内消失的 item view 重新绑定上即将出现的 item 的数据并加以显示。

View holder 对象被 adapter 管理， adapter 是你创建的一个继承自 RecyclerView.Adapter 的子类对象。
adapter 会在需要的时候创建 view holder 。
adapter 也负责将 view holder 与其对应的数据相绑定。
它通过将 ViewHolder 分配到指定位置并且调用 `onBindViewHolder()` 来实现。
这个方法使用 view holder 的位置去决定应该展示什么内容。

RecyclerView 也做了大量优化工作使你不必：
-   当列表首次被填充的时候，它在列表的边界处创建并绑定了一些 view holder。
    例如，如果 view 当前展示的 list 是从 0 到 9 ， RecyclerView 将会创建并绑定这些 viewHolder ，可能也会创建第 10 个 viewHolder。
    这样的话，如果用户触发滚动事件，下一个 item 是处于已经准备好的状态的。
-   当用户 scroll 列表的时候，如果有必要 RecyclerView 会创建一个新的 viewHolder 。
    它也保存者已经滚处屏幕局的 viewHolder ， 从而可重用它们。
    如果用户改变滚动方向，这些已经滚出屏幕的 viewHolder 就能被重新使用了。
    另一方面，如果用户一直在一个方向滚动屏幕，已经滚出屏幕局的 viewHolder 也可以重新绑定数据。
    ViewHolder 不需要被创建或者有它自己的 view 加载器，而是仅需要更新匹配新 item 的绑定数据。
-   当展示的 item 发生改变的时候，你需要在合适的时候调用 `RecyclerVie.Adapter.notify…()` 通知 adapter 。
    adapter 的会重新构建和绑定新的数据。

## 0x02、 增加 support library

要使用 RecyclerView 控件，你需要添加 v7 Support Libraries 到你的项目中：
1. 打开你 app moudle 中的 build.gradle
2. 在 `dependencies` 节增加条目
    ```jva
    dependencies {
        implementation 'com.android.support:recyclerview-v7:28.0.0'
    }
    ```

## 0x03、 在布局文件中添加 RecycleView

现在你需要将 RecyclerView 添加到你的布局文件中。
例如，下面的代码将 RecyclerView 作为一个布局中唯一的 view 。
```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- A RecyclerView with some commonly used attributes -->
<!-- 一个 RecyclerView 使用了一些常用属性 -->
<android.support.v7.widget.RecyclerView
    android:id="@+id/my_recycler_view"
    android:scrollbars="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>
```

一旦你在布局中添加了 RecyclerView 控件，你需要，获取一个持有 RecyclerView 的对象，将其与 LayoutManager 链接起来，为它附加一个数据集 adapter :
```java
public class MyActivity extends Activity {
    private RecyclerView recyclerView;
    private RecyclerView.Adapter mAdapter;
    private RecyclerView.LayoutManager layoutManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.my_activity);
        recyclerView = (RecyclerView) findViewById(R.id.my_recycler_view);

        // use this setting to improve performance if you know that changes
        // in content do not change the layout size of the RecyclerView
        //如果你知道内容更改不会修改布局 item 大小，可以这样设置提高性能
        recyclerView.setHasFixedSize(true);

        // use a linear layout manager
        layoutManager = new LinearLayoutManager(this);
        recyclerView.setLayoutManager(layoutManager);

        // specify an adapter (see also next example)
        mAdapter = new MyAdapter(myDataset);
        recyclerView.setAdapter(mAdapter);
    }
    // ...
}
```

## 0x04、 添加一个 adapter 

要把你所有的数据输入到列表中，你必须继承 `RecyclerView.Adapter` 类。这个对象为 item 创建 view ，并且当最初的 item 不再显示的时候，使用新数据替换其中的内容。

下面的代码示例展示了一个数据集的简单实现，它含有与一个 string 数组，通过 TextView 控件进行展示。

```java
public class MyAdapter extends RecyclerView.Adapter<MyAdapter.MyViewHolder> {
    private String[] mDataset;

    // Provide a reference to the views for each data item
    // Complex data items may need more than one view per item, and
    // you provide access to all the views for a data item in a view holder
    //为每一个 item 提供一个 view 引用，复杂的 item 或许需要多个 view ，你可以访问 view holder 中的所有 view 。
    public static class MyViewHolder extends RecyclerView.ViewHolder {
        // each data item is just a string in this case
        // 本例中，每一个 item 的数据就是一个 string
        public TextView textView;
        public MyViewHolder(TextView v) {
            super(v);
            textView = v;
        }
    }

    // Provide a suitable constructor (depends on the kind of dataset)
    // 提供合适的构造函数(取决于数据集的类型)
    public MyAdapter(String[] myDataset) {
        mDataset = myDataset;
    }

    // Create new views (invoked by the layout manager)
    //创建 view(被 LayoutManager 调用)
    @Override
    public MyAdapter.MyViewHolder onCreateViewHolder(ViewGroup parent,
                                                   int viewType) {
        // create a new view
        //创建有一个新 view 
        TextView v = (TextView) LayoutInflater.from(parent.getContext())
                .inflate(R.layout.my_text_view, parent, false);
        ...
        MyViewHolder vh = new MyViewHolder(v);
        return vh;
    }

    // Replace the contents of a view (invoked by the layout manager)
    //替换 view 中的内容(被 LayoutManager 调用)
    @Override
    public void onBindViewHolder(MyViewHolder holder, int position) {
        // - get element from your dataset at this position
        // - replace the contents of the view with that element
        holder.textView.setText(mDataset[position]);

    }

    // Return the size of your dataset (invoked by the layout manager)
    @Override
    public int getItemCount() {
        return mDataset.length;
    }
}
```

LayoutManager 调用 adapter 的 `onCreateViewHolder()` 函数。
这个函数需要构造一个 `RecyclerView.ViewHolder` 并且设置 view 用于展示它的内容。
ViewHolder 的类型必须必须匹配 Adapter 中声明的签名。
通常它将通过加载 XML 布局文件进行设置。
因为 view holder 尚未被分配任何特定的数据，这个方法没有实质设置 view 的内容。

接下来 LayoutManager 将 view 对应的数据与 view 想绑定。
它通过调用 `onBindViewHolder()` 完成这件事(参数中包括 item 在 RecyclerView 中的索引)。
`onBindViewHolder()` 函数需要取来合适的数据并填充 view holder 的布局。
例如，如果 RecyclerView 正在展示一个名字列表，这个方法可能从 list 中获取合适的名字然后将他填充到 view holder 中的 TextView 控件。

如果 list 需要更新，要调用 adapter 的通知函数，例如 `notifyItemChanged()` 。
LayoutManager 然后重新绑定受影响的 view holder, 从而更新响应的数据。

> **Tip:**  你应该了解到，当你的 list 更新的时候 ListAdapter 类对于知晓你的 list 中哪一个 item 需要更新是有用的。

## 0x05、 自定义你的 RecyclerView 

你能自定义 RecyclerView 来满足你特定的需求。
标准类提供了开发者将会用到的所有方法；在多数情况下，你唯一需要自定义的是设计每一个 view holder 并且写代码用于使用合适的数据更新这些 view 。
然而，如果你的 app 已经指定了需求，你能修改通过击中方式修改标准行为。
下面介绍了其他几种常见的自定义的情况。

### 5.1、 修改布局

RecyclerView 使用一个 LayoutManager 去指定屏幕上的个别 item 并且决定什么时候重用长时间不用的 item 对应的 view 。
要重用(或回收)一个 view ， LayoutManager 或许会要求 adapter 从数据集中拿出不同的元素替换 view 中的内容。
用这种方式回收 view 通过避免创建不必要的 view 或者执行昂贵的 `findViewById()` 循环来提高效率。
Android Support Library 包含三个标准 LayoutManager，他们每一个都提供了多个自定义选项。
-   LinearLayoutManager 将 item 整理为一维列表。
    使用 RecyclerView 配合 LinearlayoutManager 提供的功能类似于 ListView 布局。
-   GridLayoutManager 将 item 整理为二维列表，就像棋盘上的方块。。
    使用 RecyclerView 配合 LinearlayoutManager 提供的功能类似于 GridView 布局。
-   StaggeredGridLayoutManager 将 item 整理为一个 二维列表，每一列都有轻微的偏移，像美国旗上的星星。

如有没有一个 LayoutManager 满足你的需求，你可以通过继承 RecyclerView.LayoutManager 创建你自己的类。

### 5.2、 添加 item 动画

当 item 改变的时候， RecyclerView 使用一个 动画去改变外观。
这个动画是一个继承了 RecyclerView.ItemAnimator 类的对象。
默认的， RecyclerView 使用 DefaultItemAnimator 提供动画。
如果你想提供自定义动画，你可以继承 `RecyclerView.ItemAnimator` 实现自己的动画类。

### 5.3、 使 list-item 可选

[recyclerview-selection](https://developer.android.com/reference/androidx/recyclerview/selection/package-summary.html) library 使用户能通过触摸或者鼠标输入选择 RecyclerView 中的 item 。
你保留对所选项目的视觉显示的控制。
也能保持对选中项目的政策控制，例如什么样的 item 能被选择，以及能选择多少个 item 。

要增加选择支持，需要遵循下面的步骤：
1.  Determine which selection key type to use, then build a `ItemKeyProvider`.
    