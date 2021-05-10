## 0x00、 android.view.Window 

顶级窗口外观和行为策略的抽象基类。
这个类被用于添加到 window manager 的顶级视图。
它提供标准 UI 策略，例如 背景 ，标题区域，默认秘钥等。

这个类唯一已存的实现类是 ` android.view.PhoneWindow` 在你需要 Window 的时候进行实例化。

### 0.1、 属性

`public static final int FEATURE_ACTION_BAR_OVERLAY`
-   flag 用于请求 Action bar 覆盖窗口内容。
    通常一个 Action Bar 将位于 window 中的内容上方，但是如果被请求的功能仅仅是 `FEATURE_ACTION_BAR` 它将分层覆盖窗口内容本身。


`public void addFlags (int flags)`
- 根据setFlags（int，int）设置flags中指定的标志位的便捷功能。

`public void setFlags (int flags, int mask)`
-   由 `WindowManager.LayoutParams` 中的 Flag 设置 window 的 flag。

    注意，一些 flag 必须在 window 装饰者被创建之前进行设置（也就是说在调用`setContentView(android.view.View, android.view.ViewGroup.LayoutParams)` 或者 `getDecorView()` 之前被设置）：`WindowManager.LayoutParams#FLAG_LAYOUT_IN_SCREEN` 和 `WindowManager.LayoutParams#FLAG_LAYOUT_INSET_DECOR`
    这些将根据 `R.attr.windowIsFloating` 属性为您设置。

-   **参数**
    - **flags** 新的 window flag 请查看 `WindowManager.LayoutParams`
    - **mask** 哪一个 window flag 将会被修改



## 0x01、 android.view.WindowManager.LayoutParams

public static final int **FLAG_FULLSCREEN**
-   *Window flag :* 当 window 显示的时候隐藏左右屏幕装饰(例如 status bar)。
    这允许即将显示的 window 单独使用整个展示空间 —— 当一个位于顶层的时候 app window 使用了这个 flag 的时候 status bar 将会被隐藏。
    一个全屏 window 将会忽略 `SOFT_INPUT_ADJUST_RESIZE` (window 的 softInputMode 属性)；这个窗口将保持全屏不重新设置窗口大小。

    这个 flag 能在你的 theme 中通过 `R.attr.windowFullscreen` 属性进行控制；这个属性在下面的标准全屏 theme 中被默认设置了：
    -   `R.style.Theme_NoTitleBar_Fullscreen`
    -   `R.style.Theme_Black_NoTitleBar_Fullscreen`
    -   `R.style.Theme_Light_NoTitleBar_Fullscreen`
    -   `R.style.Theme_Holo_NoActionBar_Fullscreen`
    -   `R.style.Theme_Holo_Light_NoActionBar_Fullscreen`
    -   `R.style.Theme_DeviceDefault_NoActionBar_Fullscreen`
    -   `R.style.Theme_DeviceDefault_Light_NoActionBar_Fullscreen`

public static final int **FLAG_TRANSLUCENT_STATUS**
-   *Window flag：* 请求一个半透明的 status bar ，系统提供的背景保护最小。

    这个 flag 能在 theme 中通过属性 `R.attr.windowTranslucentStatus` 进行控制。
    在下面的标准主题中都默认都默认设置了这个属性：
    -   `R.style.Theme_Holo_NoActionBar_TranslucentDecor`
    -   `R.style.Theme_Holo_Light_NoActionBar_TranslucentDecor`
    -   `R.style.Theme_DeviceDefault_NoActionBar_TranslucentDecor`
    -   `R.style.Theme_DeviceDefault_Light_NoActionBar_TranslucentDecor`

    当一个 window 将这个 flag 设置为可用的时候，将会自动设置 system UI 可见 flag ： 
    -   `View#SYSTEM_UI_FLAG_LAYOUT_STABLE` 
    -   `View#SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN`

public static final int **FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS**
- 

## 0x02、 android.view.View

`public void setFitsSystemWindows (boolean fitSystemWindows)`
-   设置这个 view 是否应该作为屏幕装饰者被系统计算在内(例如 status bar 和其中的内容)；也就是说，控制 `fitSystemWindows(android.graphics.Rect)` 方法的默认实现是否会被执行(可以到这个方法 `fitSystemWindows(..)` 中查看更多细节)。

    注意，如果你自己实现了 `fitSystemWindows(android.graphics.Rect)` ，就无需设置这个 flag 为 true 了，你的实现将会覆盖默认的 falg 校验。

    相关 XML 属性 ： `android:fitsSystemWindows`
    
`public void setSystemUiVisibility (int visibility)`
-   请求可见的 status bar 或者其他 screen/window 装饰发生改变。

    这个方法用于将整个设备的 UI 设置为临时模式，在这种情况下用户能更专注于应用内容，通过降低亮度或隐藏内容周围的系统 UI 。
    这通常结合 `Window#FEATURE_ACTION_BAR_OVERLAY` 使用，允许将应用程序内容放在操作栏后面（并使用这些标记其他系统可用性），以便可以在隐藏和显示它们之间进行平滑过渡。

    使用系统 UI 可见性实现内容浏览器和视频播放器的相关例子。

    略

-   **参数**
    -   *visibility：*
        -  `SYSTEM_UI_FLAG_LOW_PROFILE`           (添加于 API 14)   
            View 请求 system UI 进入不显眼的低调模式
        -  `SYSTEM_UI_FLAG_HIDE_NAVIGATION`       (添加于 API 14)
            View 请求 navigation 暂时隐藏。
        -  `SYSTEM_UI_FLAG_FULLSCREEN`            ()     
        -  `SYSTEM_UI_FLAG_LAYOUT_STABLE`         ()        
        -  `SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION`()               
        -  `SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN`     ()            
        -  `SYSTEM_UI_FLAG_IMMERSIVE`             ()    
        -  `SYSTEM_UI_FLAG_IMMERSIVE_STICKY`      ()
        -  `SYSTEM_UI_FLAG_LIGHT_STATUS_BAR`      (添加与 API 23)
            请求状态栏中内容的绘制与 light status bar 背景相兼容
