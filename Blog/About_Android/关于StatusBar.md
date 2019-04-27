## 0x00、 关于 Status bar 

我们的需求是什么？
1. 实时修改 StatusBar 颜色
2. 实时修改 StatusBar 位置文字颜色
3. 不显示 StatusBar 

## 0x01、 设置 Status bar 颜色

### 1.1、 在 API ≥ 21 的设备上
```xml
    <style name="AppTheme" >
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item> <!-- 修改此值即可 -->
        <item name="colorAccent">@color/colorAccent</item>
    </style>
```

### 1.2、 在 API < 21 的设备上


## 0x02、 设置 Status bar 字体颜色
在完成 `0x01` 之后我们的问题就变成了，当前 status bar 的颜色和其中文字的颜色相近，此时如何修改 status bar 中文字/图表颜色？

```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
    getWindow().getDecorView().setSystemUiVisibility(View.SYSTEM_UI_FLAG_LIGHT_STATUS_BAR);
}
```

上述代码设置 status bar 上部文字图表颜色

## 0x03、 进一步的问题
我们的主题色是白色也就是说 colorPrimaryDark 是白色，这时候我们的问题就是 背景和文字都是白色。导致混淆。

在 API ≥ 23 的时候我们可以使用 `0x02` 的方式修改 status bar 文字颜色。但是在这之前的版本我们怎么做？
1. 可以：我们使用半透明 status bar
2. 也可以替换


########################################################################################################

## 0x01、 设置 Status bar 颜色

### 1.1、 API ≥ 23

1. 设置 status bar 颜色
```xml
    <style name="AppTheme" >
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item> <!-- 修改此值即可 -->
        <item name="colorAccent">@color/colorAccent</item>
    </style>
```

2. 设置 status bar 中文字颜色
```java
//白底黑字
getWindow().getDecorView().setSystemUiVisibility(View.SYSTEM_UI_FLAG_LIGHT_STATUS_BAR);
```

### 1.2、 23 > API ≥ 21

1. 只设置 status bar 颜色，只是其不同于 API ≥ 23 时候的颜色。



## 参考
1. https://juejin.im/post/5a52023b6fb9a01c9c1ed937
2. http://www.10tiao.com/html/169/201803/2650825153/1.html
