## 0x00、 [Animations Overview](https://developer.android.com/training/animation/overview)

动画能增加视觉提示(Visual cues),告知用户你的 app 正在进行某种操作。
当 UI 变换的时候它显得特别有用，例如当加载新内容的时候或者新的动作正在执行的时候。
动画让你的 app 有更高质量的视觉体验，使你的 app 看起来更加优雅(polished)。

Android 包含根据不同的动画类型所依赖的不同的动画 API，所以这篇文字提供了一个关于你以不同方式移动 UI 部件的概览。

对动画最好的理解是使用它，你也可以从 [material design guide to motion](https://material.io/guidelines/motion/material-motion.html) 了解到更多信息。

## 0x01、 Animate bitmaps
![Figure 1. An animated drawable](/Android_Dev/GUIDES/Images/2018-11-12_drawable-animation.gif)

当你想为一个 Bitmap(例如一个 icon 或者 插图(illustration)) 添加动画效果，你应该使用 drawable 动画 API 。
通常动画被定义与静态资源文件(/drawable),当然你也可以在运行时定义动画行为。

例如，当用户点击的时候，将一个 play 按钮转换为一个 pause 按钮(按下一个是另一个可见)，对用户来讲就是一种状态变化的交流。

有关于此的更多信息请参照 [Animate Drawable Graphics](https://developer.android.com/guide/topics/graphics/drawable-animation.html)

## 0x02、 Animate UI visibility and motion
![Figure 2. A subtle animation when a dialog appears and disappears makes the UI change less jarring](/Android_Dev/GUIDES/Images/2018-11-12_view-animation-dialog.gif)

当你需要改变 view 的可见性和位置的时候，你应该使用细微(subtle)的动画帮助用户理解 UI 正在发生的变化。

移动，显示，或者隐藏当前布局的 view，你可以使用系统从 Android3.0(API level 11)开始提供的 `android.animation` 包内的**属性动画(property animation)**。
这些 API 在一段时间内持续更新你 View 对象的属性，并且在 view 属性发生变化的时候持续进行重绘。
例如，当你修改位置属性的时候，view 会在屏幕上移动，或者当你修改透明度(alpha)属性的时候 view 会逐渐(fades)出现或者消失。

创建动画会花费大量精力(To create these animations with the least amount of effort)，你可以在你的布局中打开动画开关，这样在你做简单变化的时候，动画将会被自动应用。
有关于此的更多信息请查阅 [Auto Animate Layout Updates](https://developer.android.com/training/animation/layout.html)

要学习如何通过属性动画系统构建动画，请查阅 [Property Animation Overview](https://developer.android.com/guide/topics/graphics/prop-animation.html)。
或者查看下面的页面创建一般动画：
-   使用交叉渐变(crossfade)修改 view 的可见性
-   通过 circular reveal 修改 view 可见性
-   使用卡片翻转(card fllip)交换 view
-   通过放大动画修改 view 的尺寸

### 2.1、 Physics-base motion
|||
|-|-|
|![Figure 3. Animation built with ObjectAnimator](/Android_Dev/GUIDES/Images/2018-11-12_targetchange_oa.gif)|![Figure 4. Animation built with physics-based APIs](/Android_Dev/GUIDES/Images/2018-11-12_targetchange_pba.gif)| 
|Figure 3. Animation built with ObjectAnimator|Figure 4. Animation built with physics-based APIs|

不论什么时候只要有可能，你的动画就应该符合物理世界的真实情况，这样看起来才真实。
例如，当他们改变的时候应该持有动量，也应该平滑。

要提供这些行为， Android Support library 包含基于物理的动画 API ，他们控制你的动画依赖物理(定律))发生。

两种常用的基于物理的动画为：
-   [Spring Animation](https://developer.android.com/guide/topics/graphics/spring-animation.html)
-   [Fling Animation](https://developer.android.com/guide/topics/graphics/fling-animation.html)

