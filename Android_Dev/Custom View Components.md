## 0x00、 [Custom view components](https://developer.android.com/guide/topics/ui/custom-components)

Android 基于基本的布局类 `View` 和 `ViewGroup` 为 UI 构建提供了复杂高效的组件。
首先，平台包含了丰富的 View 和 ViewGroup 的子类 —— 称之为控件和布局，你能分别使用它们来构建自己的 UI 。

一部分可用的控件：`Button`, `TextView`, `EditText`, `ListView`, `CheckBox`, `RadioButton`, `Gallery`, `Spinner`,还有一些专用控件：`AutoCompleteTextView`, `ImageSwitcher`, and `TextSwitcher` 。

可用的布局包括： `LinearLayout`, `FrameLayout`, `RelativeLayout` 等。
要查看更多布局请参阅 [Common Layout Object](https://developer.android.com/guide/topics/ui/layout-objects.html)

如果没有预定义的控件和布局能满足你的需求，你可以创建你自己的 View 子类。
如果你只需要对已存在的 widget 或者 layout 做微调，你可以简单的继承当前已存的类，并且覆写相关的方法。

创建你的 View 子类，你就能更精确的控制屏幕元素的外观和动作。
关于对 view 的控制，这里有几个例子，或许能给你带来灵感：
1.  你能创建一个完全自定义的 View 类型，例如，模仿电子控制设备，通过 2D 绘图完成的音量控制旋钮。
2.  你能结合一系列 View 组件，完成一个组合控件，或许可以制作一个 ComboBox(下拉列表框)(文本输入控件与列表弹窗的组合)，一个二级选择控件(包含左侧和右侧两个面板，每一个面板都展示一个列表，只是一侧展示的是另一侧的子列表)，或者其他什么东西。
3.  你能够重写 EditText 在屏幕上的显示方式([Notepad Tutorial](https://android.googlesource.com/platform/development/+/master/samples/NotePad)使用这种方式创建了效果不错的笔记页)。
4.  你能捕获其他的事件(例如屏幕事件)并按照自己的意图处理这些事件。


略去一小部分

## 0x02、 Fully Custom Components
你可以使用完全自定义组件来创建你所希望的显示任意图像的组件。
或许一个图形声音控制器看起来想一个老式的模拟量表，或者长文本视图上有一个弹跳求在文字间移动，从而你可以配合一个卡拉OK机器唱歌。
两种方式，无论你如何组合他们，你都需要内置组件不具备的能力。

幸好，你可以方便的创建你喜欢的外观和行为的组件，存在的限制或许只有你的想象力，屏幕大小和可用的进程资源(记住，最终你的 app 或许不得不在比桌面工作站少得多的系统资源下运行)。

去创建一个完全自定义组件：
1.  你可以继承的最通用的 view 毋庸置疑是 View(class)，所以你通常从继承 View 开始创建你的超级组件。
2.  你可以提供一个构造方法用于接收来自 XML 的属性和参数，你也可以使用自己提供的属性和参数(例如，音量控制器的颜色和绘制区域，或者宽度和指针，等等)。
3.  你可能希望创建自己的事件监听者，特别是关于数据访问和修改的监听者，在你的自定义组件类中可能有更复杂的行为。
4.  如果你希望你的 View 展示一些内容，通常你会想要覆写 onMeasure() 和 OnDraw() 。当有默认的实现的时候上述两个方法可以不用覆写，默认的 onDraw() 什么都不做，oMeasure() 默认返回的尺寸为 100x100,这些默认动作可能不是你想要的。
5.  其他 `on...` 方法也可能被重写。

## 0x03、 扩展 onDraw() 和 onMeasure()
`onDraw()` 传递给你一个 Canvas 对象，你可以使用这个对象绘制任何 2D 图形，其他独立的自定义组件，不同风格的 text 或者任意你希望的内容。

>☆ **NOTE:** 这里不支持 3D 绘图，如果你想要 3D 绘图，你必须继承 SurfaceView，然后从独立的线程进行绘制。
    GLSurfaceViewActivity sample有更多关于此的细节。

**有关于其中的 onMeasure().** 你的 View 和其他控件组合的时候 onMeasure() 是一个关键的部分。
onMeasure() 应该被覆写用于准确高效的 return 其所处的 view 所包含内容的测量结果。
要根据父控件的限制(这一限制会被传入 onMeasure())以及使用计算出的 width height 调用 setMeasureDimension() 完成对测量结果的计数这件事略微有点复杂。
如果你在 onMeasure() 方法中调用这个方法失败，将会在测量的时候发生异常。

高水平的实现 onMeasure() 需要知道以下内容：
1.  覆写的 onMeasure() 携带参数为 widht 和 height 
    (widthMeasureSpec 和 heightMeasureSpec 参数都是 int 类型代表尺寸) 这两个数据应该被你测量的 width 和 height 进行修正。
    在View.onMeasure(int,int) 下的参考文档中可以找到对这些规范可能要求的限制类型的完整引用(这些引用文档对整个的测量操作有很好的解释)。
2.  你组件的 onMeasure() 方法应该计算测量 width 和 height，这
    在渲染组件的时候将会被需要。
    它应该尽量保持在传入的范围规范之内，虽然可以选择拓展他们(在这种情况下，通过携带不同的测量规范父控件的选择包括剪裁、滚动、抛出异常、或者再次调用 onMeasure() )
3.  一旦 widht 和 height 计算出来，
    setMeasuredDimension(int widht,int height) 方法就必须被调用去计算尺寸。
    上述两个值计算失败将会抛出异常。

下面是 framework 会在 view 中调用的其他标准方法的概览：

|类型|方法|描述|
|---|----|---|
|Creation|Constructors|当 view 使用 java 代码或者从 xml 文件中被加载的时候会调用构造方法。从 xml 文件加载 view 并调用构造方法应该解析 xml 文件中声明的属性。|
||onFinishInflate()|当一个 view 和它是所有子 view 被加载完成的时候将会被调用。|
|Layout|onMeasure(int,int)|为 view 本身和它是子 view 所调用的请求 view 尺寸的方法|
||onLayout(boolean,in,int,int,int)|当 view 应该为它是子 view 提供尺寸和位置的时候调用|
||onSizeChanged(int,int,int,int)|当当前view 的 尺寸发生改变的时候调用|
|Drawing|onDraw(Canvas)|当 view 需要渲染内容的时候调用|
|Event processing|onKeyDown(int,KeyEvent)|当按键按下的时候调用|
||onKeyUp(int,KeyEvent)|当按键开的时候付调用|
||onTrackballEvent(MotionEvent)|当触摸求移动的时候调用|
||onTouchEvent(MotionEvent)|当触摸屏幕的时候调用|
|Focus|onFocusChanged(boolean,int,Rect)|当 view 获得或者失去焦点的时候调用|
||onWindowFocusChange(boolean)|当 window 获得或时区焦点的时候调用|
|Attaching|onAttachedToWindow()|当 view 被附加到 window 的时候调用|
||onDetachedFromWindow()|当取消 view 对 window 附加的时候调用|
||onWindowVisibilityChanged(int)|当 window 的科级及监督发生改变的时候调用|


## 0x04、 复合控制
如果你不希望创建一个完全自定义的组件，而是正在寻找一个由一组现成的组件组成的可复用的组件，创建一个 Compound Control(符合控制)符合你的需求。
简言之，这是将多个原子控制器(或者说多个 view )，放到一个逻辑上的组合内，这个组合将被视为一个单独的事物。
例如，一个 Combo Box 是一个 Editext 和一个绑定了了 PopuList 的经过调整的 Button 的组合。
如果你按下这个 button 并且选择其列表中的某一个，该字段会填充到 EditText，但是用户也可以按他们希望的自行想 EditText 中填充内容。

在 Android 中，有两个 View 是这种实现机制：Spinner 和 AutoCompliteTextView，蚕食 Combo Box 是一个更容易理解的例子。

创建一个复合组件
1.  通常的起点是某种类型的布局，所以创建一个继承了 Layout(布局) 的类。
    或许 Combo Box 使用的就是一个 LinearLayout(水平布局的)。
    记住，布局可以被嵌套，所以组合控件能的复杂度是未知的。
    就像一个 Activity，你可以使用 XML 创建复合组件，也可以在程序中使用代码创建组合布局。
2.  有关在新类的构造方法，首先在超累中获取其构造方法的参数然后应用到构造方法上。
    然后你可以设置其他 view 和你的组件一起使用，在这里你可以创建 EditText 字段和 PopuList。
    注意，你应该制造你自己的用于 XML 文件的属性，从而可以使用它们。
3.  你可应该为组合中的控件所产生的事件创建监听者，例如,
    List Item 的点击事件用于当 Item 被选中的时候更新 Editext 中的内容。
4.  你也应该创建你自己的属性用于访问和修改，例如，
    在组合控件中允许 EditText 的值被初始化和被查询。
5.  在继承 Layout 的情况下，你无需覆写 onDraw 和 onMeasure 方法，
    因为 layout 默认的代码就工作的很好。
    然而如果需要你依然可以覆写他们。
6.  你或许要覆写其他 `on...` 方法，像 `onKeyDown()` ,可能用于确定默认值当 combo box 的列表被按下的时候。

小结，使用 Layout 作为自定义控制的基础有几个优点包括：
-   你可以通过在 XML 中声明指定布局，就像在 Activit 中使用布局一样，
    你可以以编程的方式创建视图，也可以在用 XML 的方式使用它。
-   onDraw() 和 onMeasure() 方法(和其他的 `on...` 方法)都有适当的实现，你不必覆写他们。
-   最后，你能非常快速的构造任意复杂复合 view 并且如果他们是被设计为一个组合，你可以复用它们。

## 0x05、 修改已经存在的 View 类型
在某些情况下，这是一种非常简单的创建自定义 view 的方法。
如果继承的是一个已经存在的控件，你能非常简单的将其修改为你希望的样子，仅需要覆写你需要改变的行为即可。
你能做一个完全自定义控件能做的所有的事，但是要从 view 层次结构中更专业的类开始，你也能随意或许所有可能产生的行为从而扩展他们。

作为一个例子 [NotePad application](https://android.googlesource.com/platform/development/+/master/samples/NotePad) 演示了 Android 平台的多个方面。
将 EditText View 改造成内置笔记本软件。
这不是一个完美的例子，其中的 API 可能也发生改变了，但是它展示了原理。

如果你还没有准备着手去做，导入 NotePad 到 AndroidStudio(或许看一下它的源码)。
特别注意，在 NoteEditor.java 文件定义的 LinedEditText 。

接下来介绍该笔记本项目。
略...