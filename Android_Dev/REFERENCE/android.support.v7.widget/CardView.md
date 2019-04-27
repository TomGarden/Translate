## 0x00、 	android.support.v7.widget.CardView

一个具有背景阴影和圆角的 FrameLayout 。

CardView 使用 Lollipop 平台的 `elevation` 属性来显示阴影兼容旧平台的自定义阴影模拟实现。

由于在 Lollipop 平台之前圆角的剪裁是很消耗资源的，CardView 没有取消子控件和圆角重合这件事。
它增加了 padding 来避免这种重合。(查看 `setPreventCornerOverlap(boolean)` 函数对行为的改变。)

在 Lollipop 之前， CardView 增加它内容的 padding 距离并且在空出来的位置绘制隐隐。
这个 padding 距离左右两边的值为 `maxCardElevation + (1 - cos45) * cornerRadius` ，距离上下两边的值为 `maxCardElevation * 1.5 + (1 - cos45) * cornerRadius` 。

因为 padding 为了阴影对内容做出偏移，所你不能在 CardView 使用 padding 。
你可以使用 content padding 属性(XML 或者 Java`setContentPadding(...)` 都行)来设置 CardView 的子 view 相对于 CardView 的边距。

注意，你为 CardView 指定明确尺寸的时候，由于 阴影的存在，在不同的平台你指定的值会有不同的表现(分界平台为 Lollipop)。
通过使用 api 版本指定资源的值，你能避免这种差异。
通常，如果你希望 CardView 增加内部 paddding 你可以调用 `setUseCompatPadding(true)` 。

要改变 CardView 的 elevation 对背景的兼容方式，使用 `setCardElevation(float)` 。
CardView 将使用在 Lollipop 和之前的平台使用 elevation API ，这能改变阴影尺寸。
要避免当 阴影 尺寸变化的时候 view 产生移动，可以通过 `getMaxCardElevation()` 限制 阴影的尺寸。
如果你希望动态修改阴影尺寸，你应该在 CardView 初始化的时候调用 `getMaxCardElevation()` 。