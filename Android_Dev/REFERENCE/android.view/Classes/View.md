## 0x00、 android.view.View
原文：https://developer.android.com/reference/android/view/View

这个类是 UI 组件的基本构建模块。
一个 View 占据屏幕上的一个规则的区域，该区域可以展示绘画和响应各种事件。
View 是各种控件(widgets)的基类，各种控件用于创建和用户交互的 UI 组件(例如 Button、text、等)。
View 的子类 ViewGroup 是布局(layout)的基类，ViewGroup 的子类中可以包含其他 View(或者 ViewGroup) 并且布局属性。

>☆ Devekoper Guides <br/>
>关于使用这个类开发自己应用程序的 UI ，可以阅读 [User Interface](https://developer.android.com/guide/topics/ui/index.html) 开发指南。

## 0x01、 使用 View
所有的 view 都被组织到一个 tree 结构上。
你可以通过代码或者指定 XML 布局文件中的 view 向 tree 结构上添加 view 。
View 有许多独特的子类作为控件，展示文字、图片或者其他内容。

一旦你使用 View 创建了一个 tree 结构，这里有一些你能用得到的常见操作：
- **Set properties:(设置属性)** 
    例如设置 TextView 。
    对于 View 不同的子类，设置属性所使用的方法可能有很大差别。
    注意：可以在 XML 布局文件中设置那些在构建的时候已知的属性。
- **Set focus:(设置焦点)**
    framework 会随着用户的操作切换焦点所在的 view 。
    要为特定的 view 手动设置焦点可以使用 `requestFocus()`
- **Set up listeners:(设置监听)**
    View 允许设置监听以便于特定交互事件发生的时候可以及时进行通知。
    例如，所有的控件都可以为焦点得/失设置监听事件。
    你可以使用类似方法注册监听事件：`setOnFocusChangeListener(android.view.View.OnFocusChangeListener)`。
    许多的 View 子类中都有特定的监听事件。
    例如 Button 允许设置监听事件响应自身的点击事件。
- **Set visibility:(设置可见性)**
    你可以使用 `setVisibility(int)` 方法隐藏/展示一个view。

*注意：
Android framework 负责管理 view 的 measrue/layout/draw 。
除非你实现了一个 ViewGroup，否则你不应该在你的 view 中调用和执行上述方法。*

## 0x02、 实现一个自定义视图(Custom View)
要实现一个自定义 View ，你通常需要着手覆写一些 framework 会调用的标准方法。
你不需要覆写所有的标准方法。
事实上，你能从覆写 `onDraw(android.graphics.Canvas)` 开始。

|Category|Methods|Description|
|--------|-------|-----------|
|Creation|Constructors|当 View 对象被实例化的时候调用。当从 XML 布局文件中加载一个 View 的时候，需要解析并使用文件中声明的属性|
||onFinishInflate()|XML 布局文件中一个 view 和它所有的子 view 被加载完成的时候调用|
|Layout|onMeasure(int,int)|调用它确定本视图和本视图所有孩子视图对于尺寸的要求|
||onLayout(boolean,int,int,int,int)|当本 view 需要为其所有的 子view 指明大小和位置的时候调用|
||onSizeChanged(int,int,int,int)|当前 view 的大小发生改变的时候调用|
|Drawing|onDraw(android.graphics.Canvas|当 view 需要绘制其需要显示的内容的时候调用|
|Event processing|onKeyDown(int,KeyEvent)|硬件按钮按下事件出现的时候调用|
||onKeyUp(int,KeyEvent)|硬件按钮松开的时候调用|
||onTrackballEvent(MotionEvent)|当触摸球运动的时候调用|
||onTouchEvent(MotionEvent)|当屏幕没触摸的时候调用|
|Focus|onFocusChange(boolean,int,android.graphics.Rect)|当 view 得/失焦点的时候调用|
||onWindowFocusChanged(boolean)|当包含 view 的 window 得/失焦点的时候调用|
|Attaching|onAttachedToWindow()|当 view 被附加到 window 的时候调用|
||onDetachedFromWindow()|当 view 与其所在的 window 分离的时候调用|
||onWindowVisibilityChanged(int)|当包含 view 的 window 的能见度发生改变的时候调用|

## 0x03、 IDs
View 或许有一个和它相关的整数类型的 ID。
这个 ID 通常在 XML 文件中进行设置，并且应用于在 view tree 中寻找特定的 view 。
一个常见的形式是这样的：
- 在 XML 布局文件中定义一个 Button 并且指定一个独一无二的 ID
    ```xml
    <Button
        android:id="@+id/my_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/my_button_text"/>
    ```

- 在一个 Activity 中的 onCreate 方法中，找到该 Button :
    ```java
    Button myButton = findViewById(R.id.my_button);
    ```

View 的 ID 不必在所有的 view tree 中都是独一无二的，但是一个好的实践至少要保证该 ID 在其所在的 view tree 中独一无二。

## 0x04、 Position(位置)
View 的几何类型是矩形。
View 的位置，用以左上角为原点的坐标系表示;View 的尺寸，用 width 和 height 表示。
位置和尺寸的单位是 pixel(像素)。

要得知 view 的位置可以通过调用 `getLeft()` 和 `getTop()` 获得。
前者返回 left，或者称之为矩形的 X 轴坐标。
后者返回 top， 或者称之为矩形的 Y 轴坐标。
这两个方法所返回的数据都是相对于父 view 的。
例如，getLeft() 得到的数据为 20 ，这意味着，view 位于距离它直接所在的父 view 左边界 20 pixles 的位置。

还有，一些方便的方法处于避免不必要的计算的目的被提供：`getRight()` 和 `getButthm()`。
这两个方法返回矩形所代表的 view 的右侧和底部边界坐标值。
例如，调用 `getRight()` 的结果和 `getLeft() + getWidth()` 的结果相同(关于 width 查看 Size 可以了解更多)。

## 0x05、 Size,padding and margins(尺寸，填充 和 边距)
View 的大小可以用一个 width 和一个 height 表示。
实际上一个 View 具有两对 widht 和 height 的值。

第一对是 measured width(测量宽度) 和 measured height(测量高度)。
这对数据决定了一个 view 要在其父 view 中占据多大的位置([Layout](https://developer.android.com/reference/android/view/View#Layout)有更多细节)，测量尺寸可以通过 `getMeasuredWidth() ` 和 ` getMeasuredHeight()` 获得。

第二对是刚讨论过的 width 和 height，有时也称之为 drawing width(绘制宽度) 和 drawing height(绘制高度) 。
这些尺寸定义了一个一个 view 在屏幕展示的、在绘制时使用的、在布局时放置的实际的尺寸。
这些值可以但是不必须在 measured(测量) width 和 height 的时候设置。
width 和 height 可以通过调用 `getWidht()` 和 `getHeight()` 获得。

要测量 view 的尺寸，需要将 View 的 padding 计算在内。
padding 和 left,top,right,bottom 一样被表示为像素。
pdding 可用于使 view 中的内容偏移指定的像素。
例如，left padding 为 2 将会使 view 的内容从左到右推进 2 像素。
padding 可以通过 `setPadding(int, int, int, int) ` 或 ` setPaddingRelative(int, int, int, int)` 进行设置；通过调用 `getPaddingLeft(), getPaddingTop(), getPaddingRight(), getPaddingBottom(), getPaddingStart(), getPaddingEnd()` 进行查询。
即使 view 可以定义 padding ，但是 padding 并未为 margins(边距) 提供任何支持。
然而，视图组提供了这样的支持。
从 [ViewGroup](https://developer.android.com/reference/android/view/ViewGroup.html) 和 [ViewGroup.MarginLayoutParams](https://developer.android.com/reference/android/view/ViewGroup.MarginLayoutParams.html) 可以获得更多信息。

## 0x06、 Layout (布局)
Layout 是两个过程：一个 measure(测量) 过程，一个 layout(布局) 过程。
measuring(测量) 过程的实现在 `measure(int,int)` ，它会自上而下的遍历 view tree。
Each view pushes dimension specifications down the tree during the recursion.(递归期间每一个 view 的尺寸都被存储了起来。)
在 measure(测量) 过程的结束阶段，每一个 view 的数据都被保存在 measurements 中。
第二个过程从 view tree 自上而下的发生在 `layout(int,int,int,int)` 。
在这个过程中，每一个父 view 主管负责使用在 measure 过程中测量的尺寸完成其所有子 view 的定位。

当一个 view 的 measure() return，它的 `getMeasuredWidth()` 和 `getMeasuredHeight()` 的值都必须被设置，以及该 View 所有后代的值。
一个 View 的 mreasured width 和 measured height 必须接受来自父 view 的强制性约束。
这保证了在 measure 过程完成后，所有的父 view 都完全接收到了其子 view 的 所有measurements(测量)。
一个父 view 可以在其子 view 上多次调用 measure()。
例如，父 view 可能在不确定子 view 尺寸的时候测量一遍所有子 view 的尺寸确定他们的大小；然后如果所有子 view 为受限制的尺寸之和过大或者过小父 view 会带着实际的数值再次调用子 view 中的 measure() 。

measure(测量) 过程使用两个类计算尺寸。
`View.MeasureSpec` 类被子 view 用于通知其父 view 它希望如何测量和定位。
`LayoutParams` 类仅仅描述 view 的 width 和 height 的大小。
对于每一个尺寸，都可以是如下选项中的一个：
- 一个精确的数字
- MATCH_PARENT,这意味着，子 view 希望和它的父 view 有一样的尺寸(减去 padding)
- WRAP_CONTENT,这意味着 ，子 view 希望自己的大小恰好足够包含自身的内容即可(加上 padding)

对不同的 ViewGroup 来讲有不同的 LayoutParams 子类。
例如，AbsoluteLayout 有其自己的 LayoutParams 子类用于增加 X 和 Y 值。

MeasureSpecs 用于将父 view 的请求传达给子 view 。
一个 MeasureSpec 能够使一下三种模式中的一种：
- UNSPECIFIED(未指定):
    被父 view 用于决定子 view 所需要的尺寸。
    例如，一个 LinearLayout 可能在调用 height 为 UNSPECIFIED 和 widht 为 EXACTLY 240 的子 view 中的 measure(),去确定希望自身 width 为 240 pixels 的子 view 的 height 应该是多少。
- EXACTLY(明确的):
    这种模式被用于父 view 强制子 view 具有一个明确的尺寸。
    子 view 必须使用这个尺寸，并且保证其所有的后代根据这个尺寸来修正自身。
- AT_MOST(最多):
    被用于父 view 强制设置子 view 的最大尺寸。
    子 view 必须使用这个尺寸，并且保证其所有的后代根据这个尺寸来修正自身。

要初始化一个布局，调用 requestLayout() 。
这个方法在 view 确定其范围不在修改后被 view 自身调用。

## 0x07、 Drawing (绘制)
Drawing 动作被正在运行的 view tree 处理，并且会记录任何需要更新的 view 的绘制命令。
此后，绘制 tree 的命令被发送到屏幕上，剪切到新的损坏的地方。

tree view 大部分是按顺序绘制和记录的，在父 view 按照之前的记录绘制他们的子 view ，子 view 的绘制顺序是按照他们出现在 tree 中的顺序进行排列的。
如果你为一个 view 设置了一个背景图片，这个 view 建辉在调用 onDraw() 方法之前绘制这个背景。
ViewGroup 中的子 view 的绘制顺序能被改变： [自定义绘制顺序](https://developer.android.com/reference/android/view/ViewGroup.html#setChildrenDrawingOrderEnabled(boolean))，使用 `setZ(float)` 自定义 View 的 Z 值。

要强制绘制调用 `invalidate()` 。

## 0x08、 Event Handling and Threading (事件处理和线程)
一个 view 的基本周期如下：
1.  一个事件到来并被分配到适当的 view 。
    这个 view 处理这个事件并且通知响应的 listeners 。
2.  如果要调整第一步中的处理过程，view 的边界或许需要调整，调用 requestLayout()。
3.  类似的，如果要调整第一步中的处理过程，view 的外观或许需要调整，调用 invalidate()。
4.  如果 `requestLayout()` 或 `invalidat()` 北调用，framework 将会根据 的 measuring ，laying ，重新 drawing view tree 。

*注意：
view tree 实体是在单独的线程中运行的。
你必须总是在 UI 线程中调用 view 的方法。
如果你在其他线程中计算数据，并且希望用这个数据更新 view 你需要使用 Handler 完成线程键通信。*

## 0x09、 Foucs Handling(焦点处理)
framework 将会处理在用户交互过程中处理常规的焦点事件。
这些常规事件包括：当 view 被移除或隐藏的时候的焦点转移和插入新 view 时候的焦点转移。
可以通过 isFocusable() 方法来查看一个 view 当前是否持有焦点。
可以通过 setFocusable(boolean) 方法来为一个 view 设置焦点。
在触摸模式下 view 可以通过 isFocusableInTouchMode() 表名是否持续持有拣点，也可以通过 setFocusableInTouchMode(boolean) 方法改变 view 的焦点。

焦点移动所依据的算法是最近的邻居 view 所指出的防线。
在极少数情况下默认的算法和开发者的预期不相符。
在这种情况下，你可以在 XML 文件中使用属性明确的提出自己的意图。
```
 nextFocusDown
 nextFocusLeft
 nextFocusRight
 nextFocusUp
```

要指定的 view 获取焦点需要调用 `requestFocus()` 。

## 0x10、 Touch Mode(触摸模式)
当用户使用类似 D-pad 的方向键改变操作 UI 的时候，将焦点移交到正在操作的项目是有必要的，例如移交到一个 Button 或者 EditText ，从而用户可以看到自己操作所带来的变化。
如果设备具有触控功能，当用户正在通过触摸和一个控件交互，这时候保持这个控件高亮或者保持它的焦点是没有必要的。
这种激励模式称之为 ‘touch mode’。

对于支持触摸的设备，一旦用户触摸屏幕，设备就会确定为 touch mode 。
有一点，只有 view 的 siFocusableInTouchMode() 的值为 true ，这个 view 才能获取焦点，例如 EditText 控件。
其他的控件例如 Button 也能触摸但是无法获取焦点，只能触发 onClick 事件。

任何时候，用户开始使用方向键(例如 D-pad)的时候，设备会退出 touch mode ，并且定位到当前正在操作的 view 为其设置焦点，从而用户可以继续在不触摸屏幕的情况下和控件交互。

当 Activit 调用 isInTouchMode() 可以查看当前设备是否处于 touch mode 。

## 0x11、 Scrolling(滚动)
framework 对于 view 内容的滚动提供了基本的支持。
包括跟踪 X 和 Y 滚动便宜从而绘制滚动条的机制。
查看 `scrollBy(int,int) ,scrollTo(int,int) ,awakenScrollBars()` 了解更多信息。

## 0x12、 Tags(标签)
不同于 ID tag 不用于标识 vies 。
tag 实际上是和 view 相关的额外信息。
他们经常被用于存储和 view 相关的信息，使用它存储信息比将 view 放到别的结构体重更加方便。

tab 可以在 XML 布局文件中通过 android:tag 属性被指定一个值也可以通过 `<tag>` 子标签制定多个 tag ：
```xml
     <View ...
           android:tag="@string/mytag_value" />
     <View ...>
         <tag android:id="@+id/mytag"
              android:value="@string/mytag_value" />
     </View>
```

tag 也可以通过代码指定：
- setTag(Object)
- setTag(int,Object)

## 0x13、 Themes(主题)
默认的 View 在创建的时候使用 Context 对象提供给其构造方法的主题；
然而可以通过在 XML 布局文件中使用 android:theme 或者通过 ContextThemeWrapper 将自定义代码传递到 view 。

当在 XML 文件中使用 android:theme 属性的时候，指定的 theme 就被应用到顶级加载 context ，这时候被设置的 view 和其子 view 都将应用这个主题。

下面的例子，所有的 view 都将通过 Material dark 主题被创建；
然而，因为一个子集中仅设置了 android:colorAccent 的主题覆盖了默认主题，这个属性将被存储到加载上下文的主题中。
```xml
     <LinearLayout
             ...
             android:theme="@android:theme/ThemeOverlay.Material.Dark">
         <View ...>
     </LinearLayout>
```

## 0x14、 Properties(属性)
View 类公开 [ALPHA](https://developer.android.com/reference/android/view/View.html#ALPHA) 属性,和一些 transform-related(转换相关)的属性，例如 TRANSLATION_X 和 TRANSLATION_Y 。
这些属性在 Property 表单和相关的 setter/getter 方法中都可用(例如 setAlpha(float) 和 ALPHA 相关)。
这些属性能被用于设置 View 的和渲染相关的持久的状态。
这些属性和方法能和 Anmator base animations(基础动画)关联，更多的细节描述查看下一节。

## 0x15、 Animation(动画)
从 Android 3.0 开始动画 view 的应用方式使用 android.animation 包中的 API。
这些基于动画的类更改了 view 对象的实际属性，例如 alpha 和 translationX。
这些行为和 3.0 之前作对比，动画基类替代了在展示阶段的绘制。
特别是 ViewPropertyAnimater 类使 view 动画变的简单高效。

特别的 ，在 pre-3.0 你可以使用动画呈现一个 View。
你可以使用 setAnimation(Animation) 或者 startAnimation(Animation) 为 view 附加一个 Animation。
动画可以是 scale(缩放)、 rotation(旋转)、 translation 和 alpha(渐变)。
如果一个动画被附加到一个有子 view 的 view 上，其子 view 也会收到影响。
当一个动画开始的时候，framework 将会适当地重绘 view 知道动画结束。

## 0x16、 Security(安全)
有时候应用程序校验某些行为是否被用户全面了解并同意执行是必要的，例如请求一个权限去购买付费或者点击一个广告。
不幸的是，一些恶意应用总是尝试使用 view 欺骗用户执行某些用户未知的操作。
作为补救措施，framework 提供的触摸过滤机制能够被 view 用于安全的访问方法。

要打开触摸过滤可以调用 setFilterTouchesWenObscured(boolean) 或者在 xml 文件中设置 android:filterTouchesWhenObscured 属性为 true 。
当打开此机制，framework 将会丢弃一个控件被另一个控件所覆盖时候的触摸事件。
此时，只要 view 上出现了 toast 胡总恶化其他提示框和窗口 view 将不会接收到触摸事件。

要更好的控制安全风险，可以覆写 onFilterTouchEventForSecurity(MotionEvent) 方法实现自己的安全策略。
更多信息查看：[MotionEvent.FLAG_WINDOW_IS_OBSCRUED](https://developer.android.com/reference/android/view/MotionEvent.html#FLAG_WINDOW_IS_OBSCURED).

这个方法必须在 UI 元素的创建线程内调用。
这通常是你 APP 的主线程。

## 0x17、 我们关心的几个方法
### 17.1、 构造方法
> public View (Context context, 
                AttributeSet attrs, 
                int defStyleAttr, 
                int defStyleRes)

执行此方法从 XML 文件中加载布局，可以指定 view 的风格通过 theme 属性或者 style 资源。
这个构造方法允许子类使用从而在加载 view 的时候使用它们自己的 style 。

当决定最终的属性值的时候，这里有 4 个输入可以发回作用：
1.  任何通过 AttributeSet 给定的属性值。
2.  通过 style 资源指定的 AttributeSet。
3.  默认 style 通过 defStyleAttr 指定。
3.  默认 style 通过 defStyleRes 指定。
5.  在当前 theme 中的基础值。

这里的每一个输入都会按顺序被考虑，首先洗出的输入内容优先于下述内容。
换句话说，如果 AttributeSet 你已经设置了 `<Button  ***  textColor="#ff000000"` ，那么接下来 Button 中的文字将会是黑色，而不是后续主题或者风格中所指定的颜色。

## 0x18 、 选择性方法译注

>`public void setDrawingCacheEnabled (boolean enabled)`

