## include 
略

## Use `<merge>` tag

当你的布局中包含了从其他地方 include 来的视图的情况下， `<merge/>` 可以帮助消除冗余的视图组。
例如，如果你的 main 布局是 vertical 的 LinearLayout ，根布局中中包含了多个可重用的 view ，可重用的布局，我们可以就这样把他们放在 LinearLayout 中然后重用的是时候 LinearLayout 也被重用了。
这样这个额外的 LinearLayout 也被重用了(但它是不必要的)。

要避免使用 `include` 重用 view 组，可以使用 `merge` 元素。

```xml
<merge xmlns:android="http://schemas.android.com/apk/res/android">

    <Button
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:text="@string/add"/>

    <Button
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:text="@string/delete"/>

</merge>
```

现在当你使用 inclue 重用这个布局文件的时候，系统会忽略 `merge` 直接将两个 view 元素应用到你要重用布局的地方。