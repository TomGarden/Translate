# Autosizing TextViews

在 Android 8.0(API level 26) 和更高版本，你能指示一个 TextView 让它自动扩大或者收缩从而使用 TextView 的 characteristics 和 boundaries 填充它的布局。
这种设置使得在不同屏幕上操作动态内容更加方便。

Support Library 26.0 提供了全面对 自动 TextView 的全面支持。
这个库从 Android 4.0(API level 14) 开始到之后的版本都会提供支持。
`android.support.v4.widget` 包中的 `TextViewCompat` 提供向后的兼容。
对应于 AndroidX ：`androidx.core.widget.TextViewCompat`

## 0x01、 设置自动调整尺寸的 TextView

你可以使用 framework 或者 支持库，通过手动编程或者 XML 配置文件使用 autosize TextView 。
要设置 XML 属性，你可以使用 AndroidStudio 的 Properties 窗口 。

这里有三种设置 autosizing TextView 的方式 。

> ☆ 注意 : 如果你在 XML 文件中设置自动尺寸，推荐使用 “wrap_content” 设置 `TextView` 的 `layout_width` 或者 `layout_height` 。

## 0x02、 默认的

默认设置的情况下，TextView 的缩放在 水平和竖直方向的缩放是均匀分布的。

-   要通过编程的方式调整默认设置，调用 `setAutoSizeTextTypeWithDefaults(int autoSizeTextType) ` 函数。
    可以提供的参数 `AUTO_SIZE_TEXT_TYPE_NONE` 关闭自动调整功能，也可以使用 `AUTO_SIZE_TEXT_TYPE_UNIFORM` 设置水平和竖直方向均匀缩放。

    >☆ NOTE : 在均匀缩放情况下的默认尺寸是 `minTextSize = 12sp`, `maxTextSize = 112sp`, `granularity = 1px` 。

-   要使用 XML 设置，使用 Android 命名空间设置 autoSizeTextTye 属性值为 *note* 或者 *uniform*
    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <TextView
    android:layout_width="match_parent"
    android:layout_height="200dp"
    android:autoSizeTextType="uniform" />
    ```

## 0x03、 使用支持库

-   要通过编程的方式调整默认设置，调用 `TextViewCompat.setAutoSizeTextTypeWithDefaults(TextView textview, int autoSizeTextType)` 方法。
    提供一个 TextView 实例，和一个文字类型 ： `TextViewCompat.AUTO_SIZE_TEXT_TYPE_NONE ` 或 ` TextViewCompat.AUTO_SIZE_TEXT_TYPE_UNIFORM`

-   要使用支持库并且在 XML 文件中定义默认设置，使用 `app` 命名空间设置 `autoSizeTextType` 的值为 *note* 或 *uniform* 。
    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="200dp"
        app:autoSizeTextType="uniform" />

    </LinearLayout>
    ```

## 0x04、 粒度

你能定义 textSize 的最小或最大值，以及指定每个步骤大小的范围 。
TextView 在 最小值和最大值 之间均匀缩放 。
每一步的增量根据粒度中设置的步长进行 。

-   要通过编程的方式定义文字大小的范围，调用 `setAutoSizeTextTypeUniformWithConfiguration(int autoSizeMinTextSize, int autoSizeMaxTextSize, int autoSizeStepGranularity, int unit)` 方法。
    提供任意 TypedValue 单位指定的 最大值，最小值，粒度值 。

-   要在 XML 文件中定义文字尺寸的大小范围，使用 android 命名空间并且设置下面的属性
    1.  使用 autoSizeText 属性设 `none` 或者 `uniform` 。
        - `none` 是默认值， `uniform` 使得 TextView 的内容在横向或者纵向均匀缩放。
    2.  设置 `autoSizeMinTextSize` , `autoSizeMaxTextSize` 和 `autoSizeStepGranulatity` 属性定义 TextView 在自动调整文字大小时候的调整步长 。
        ```xml
        <?xml version="1.0" encoding="utf-8"?>
        <TextView
            android:layout_width="match_parent"
            android:layout_height="200dp"
            android:autoSizeTextType="uniform"
            android:autoSizeMinTextSize="12sp"
            android:autoSizeMaxTextSize="100sp"
            android:autoSizeStepGranularity="2sp" />
        ```

### 4.1、 使用支持库 

后文 暂略 ......