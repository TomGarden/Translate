## 自定义 ProgressBar

```xml
    <ProgressBar
        style="@android:style/Widget.ProgressBar.Horizontal"
        android:layout_width="match_parent"
        android:layout_height="5dp"
        android:progressDrawable="@drawable/layer_list_custom_seekbar_2"
        android:max="10000"
        android:progress="120" />
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<scale xmlns:android="http://schemas.android.com/apk/res/android"
    android:scaleWidth="100%"
    android:scaleGravity="left">
    <shape>
        <size android:height="5dp" />
        <corners android:radius="3dp" />
        <solid android:color="@color/color4789F1" />
    </shape>
</scale>
```

`android:scaleWidth="100%"` 表示将这个 scaleDrawable 缩放到最小 。

`android:scaleWidth="0%"` 表示这个 scaleDrawable 完全不进行缩放 。

-   这个最小如何界定呢？
    在这个例子中的最小值由  `android:max="10000"` `android:progress="120"` 共同决定
    - 为什么我的例子中恰好是 10000 和 120 两个值？ 
        因为这个 sacle 有一个圆角，在这两个值的限制下得到的恰好是有一个小圆点。
    - 在 10000 和 120 两个值给定的情况下
        - `android:scaleWidth="100%"` 会显示小圆点
        - `android:scaleWidth="0%"` 会显示铺满整个 ProgressBar 的 scaleDrawable


