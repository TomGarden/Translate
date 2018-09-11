## 0x00、 创建一个 View 类
一个设计优秀的自定义 view 和其他的自定义类非常想像。
它包括一个特定的方法集以及一个简单的使用接口，它使用 CUP 和内存的效率高，并且速度快。
作为一个自定义 view 除了是一个设计优秀的类，还应该:
-   符合 Android 标准。
-   提供在 XML 布局文件运行良好的自定义风格属性。
-   发送可达的事件。
-   能兼容不同的 Android 版本。

Android 矿建提供了可以满足一切需求的一系列基础的类和 XML 标签帮助你创建一个 view。
这篇文字介绍如何使用 Android framework 创建 view 类的核心功能。

在开始学习本文之前你需要先了解一下：[自定义组件](https://developer.android.com/guide/topics/ui/custom-components.html)

## 0x01、 View 的子类
Android framework 中的所有所有的 view 类都是 View 的子类。
你的自定义 view 也能直接继承 View ，或者你为了节省时间也可以集成其他已经存在的 view 的子类，例如 Button 。

要允许 Android Strudio 和你的类进行交互，你至少需要提供一个用于接收 Context 和一个 AttributeSet 的构造方法。
这个构造方法允许 layout 编辑器创建和编辑你 view 的实例。

```java
class PieChart extends View {
    public PieChart(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
}
```

## 0x02、 定义自定义属性
要把 view 构建到你的 UI ，你需要在 layout 文件中通过一个 XML 元素和一些属性来控制 view 的外观和行为。
写的好的 view 也能通过 XML 定义风格。
要想你的 view 具备这些能力，你需要：
-   在资源文件众多  `<declare-styleable>` 节点下定义 view 的自定义属性。
-   在你的 XML layout 文件中为你定义的属性设置值。
-   在运行时检索属性和值。
-   将检索到的属性和值应用到你的 view 中。

要定义自定义属性，想你的项目中添加 `<declare-styleable>` 资源。
就要将该自定义资源内容添加到 `/res/values/attrs.xml` 文件。
这里有一个 `attrs.xml` 文件的例子：
```xml
<resources>
   <declare-styleable name="PieChart">
       <attr name="showText" format="boolean" />
       <attr name="labelPosition" format="enum">
           <enum name="left" value="0"/>
           <enum name="right" value="1"/>
       </attr>
   </declare-styleable>
</resources>
```

上述代码在一个名为 `PieChar` 的风格实体中声明了两个自定义属性，`showText` 和 `labelPosition`。
按照惯例，这个风格实体中的 name 字段和自定义 view 中的相同。
虽然没有强制要求遵循惯例，许多流行代码的编辑者尊徐这个惯例提供代码。

一旦你定义了自定义属性，你就可以在 layout 文件中的 XML 语法中像内建属性一样使用他们了。
唯一的不同是你自定义的属性术语不同的命名控件。
不是命名空间:`http://schemas.android.com/apk/res/android`;他们属于命名空间:`http://schemas.android.com/apk/res/[your package name]`。例如，使用定义域 `PieChar` 的属性：
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
   xmlns:custom="http://schemas.android.com/apk/res/com.example.customviews">
 <com.example.customviews.charting.PieChart
     custom:showText="true"
     custom:labelPosition="left" />
</LinearLayout>
```

为了避免不得不重复那么长的命名空间 URL ，这里使用了 `xmlns` 指令。
这个指令为命名空间为 `http://schemas.android.com/apk/res/com.example.customviews` 分配的是 `custom` 这个别名 。
你能选择任何你喜欢的别名。

注意填写到 XML tag 中的自定义 view 的名字。
它是自定义 view 的完全限定名。
如果你的 class 是一个内部类，你必须进一步指明它。
例如， `PieChart` 类有一个内部类 `PieView` 。
要使用这个类中的自定义属性，你需要使用 `tag` 为 `com.example.customviews.charting.PieChart$PieView`。

## 0x03、 应用自定义属性
当一个 view 被从一个 XML 布局文件创建，在 XML 文件中对应 tag 先面的所有属性也被读取到 bundle 中并作为 AttributeSet 被发送到 view 的构造函数了。
因此可以直接从 AttributeSet 中读取值，这么做有一些劣势：
-   属性值所引用的资源未被解析
-   风格未被应用

相反，将 AttributeSet 传入 obtainStyledAttribute()。
这个方法可以传回一个 TypedArray 其中包含了已经被解析的引用和样式。

为了开发者能便利的调用 obtainStyledAttributes() Android 资源编译器做了大量的工作。
资源文件中的每一个 `<declare-styleable>` 资源，R.java 中都定义了属性 ID 数组和内容集合，这些定义为数组中的每一个属性提供索引。
你是用这些预定义的内容读取 TypedArray 中的属性。
这里是 `PieChart` 类读取它的属性的代码：
```java
public PieChart(Context context, AttributeSet attrs) {
   super(context, attrs);
   TypedArray a = context.getTheme().obtainStyledAttributes(
        attrs,
        R.styleable.PieChart,
        0, 0);

   try {
       mShowText = a.getBoolean(R.styleable.PieChart_showText, false);
       mTextPos = a.getInteger(R.styleable.PieChart_labelPosition, 0);
   } finally {
       a.recycle();
   }
}
```

注意 TypedArray 对象是共享资源，在使用完成后必须释放。

## 0x04、 增加属性和事件
属性是控制 view 外观和行为的有力的方法，但是他们只能在 view 初始化的时候被读取。
要提供动态的行为，需要为每一个属性提供 getter 和 setter 方法。
下面代码片段展示了 `PieChart` 如何对外暴露自己的 `showText` 属性：
```java
public boolean isShowText() {
   return mShowText;
}

public void setShowText(boolean showText) {
   mShowText = showText;
   invalidate();
   requestLayout();
}
```

注意 setShowText 方法中的 `invalidate()` 和 `requestLayout()` 方法。
这些调用是保证 view 行为的关键。
针对 view 做过可能修改外观的设置后，你不得不废止当前 view 的，只有这样系统才能知道 view 需要重绘。
同样，如果对 view 的修改可能影响到 view 的尺寸和形状，你需要请求一个新的布局。
忘记调用这些方法将会造成南移发现你的 BUG 。

自定义 view 也应该支持事件监听。
例如 `PieChart` 暴露一个名为 `OnCurrentItemChanged` 自定义事件用于通知监听者用户已经旋转饼图到一个新的饼片。

对外暴露属性和事件是一件很容易被忽略的事情，特别是只有你一个人作为该自定义 view 的使用者的时候。
花一些时间考虑和定义你 view 的接口减少未来维护成本吧。

## 0x05、 Design For Accessibility
你自定义的 view 应该支持更广泛的用户。
其中也包含无法看到或者无法触摸屏幕的残疾人。
要支持残疾人你需要：
-   通过 android:contentDescription 属性标识你的输入属性。
-   当适当的时候通过 sendAccessibilityEvent() 方法发送无障碍事件。
-   支持备用控制器，例如 D-Pad 和 触摸球。

关于无障碍 view 的更多信息请查看 Android 开发者指导中的 [`Making Applications Accessible`](https://developer.android.com/guide/topics/ui/accessibility/apps.html#custom-views)