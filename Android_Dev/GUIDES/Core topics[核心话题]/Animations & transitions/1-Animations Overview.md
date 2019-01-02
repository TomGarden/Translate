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

不基于物理的动画(例如构建 ObjectAnimatior 的 API )，相当于是在一定的时间段内是静态的。
如果目标值发生变化，你需要在值变化的时候取消动画，并且重新配置动画的起始值和目标值。
视觉上，动画有一个突兀(abrupt)的停止和一个后续不连贯的动作，如图 Figure 3。

然而，基于物理动画 API 构建的动画(例如 DynamicAnimation)是由力驱动的。
目标值的变化仿佛是在一种变化的外力下发生的。
新的力施加于已经存在的速度，造成了目标值的持续变化。
这个程序的结果看起来更自然，如图 Figure 4。

## 0x03、 Animate layout changes

![Figure 5. 展示了不论是布局更改还是启动新的 activity 都可以通过动画来完成](/Android_Dev/GUIDES/Images/2018-11-13_layout-transition.gif)

在 Android 4.4 (API level 19) 或和更高的版本，当你从当前 Activity 或者 Fragment 变换布局的时候，你能使用转框架创建动画。
有关于此你所需要做的仅仅是指定开始和目标布局，以及指定所适用的动画类型。
你能使用这种方式变换整个 UI 或者移动/替换某个 view 。

例如，用户点击一个 item 要查看更多信息的时候，你能通过类似 figure 5 的变化完成布局的替换。

开始和目标布局存储在一个 `Scene` 中，`Scene` 中的开始布局通常自动设置为当前布局。
然后你创建一个 `Transition` 告诉系统你希望的动画类型，最后调用 `TransitionManager.go()` ,系统就会使用指定动画的方式完成指定布局间的切换。

有关于此的更多信息可以阅读 [Animate Between Layout Using a Transition](https://developer.android.com/training/transitions/index.html)。
如果要查看示例代码： [BasicTransition](https://github.com/googlesamples/android-BasicTransition)

## 0x04、 Animate between activities
在 Android 5.0(API level 21) 以及更高的版本，你也能创建 Activity 的切换动画。
这是基于与[animate layout changes]相同的变换(transition:变换;过渡)框架，但是它允许你在不相关的 Activity 之间添加动画完成切换。

你能使用简单的动画(例如从一边开始滑动或者淡入淡出)，也可以通过所有 Activity 的共享视图完成动画创建。
例如，当用户点击一个 item 要查看更多信息，你能通过一个类似于 item 在不断长大(到全屏)的过渡动画完成一个 view 到一个 Activity 的切换，例如 figure 5 。

通常你会调用 startActivity() ,但是要使用动画需要通过 ActivityOptions.makeSceneTransitionAnimation() 传递一个携带动画参数的 bundle 。
这个 bundle 包含和目标 activity 共享的 view ，通过这个信息变换(transition:变换;过渡)框架能在动画中将 view 和 activity 连贯起来。

有关于此要查看更多的信息 [Strat an Activity with an Animation](https://developer.android.com/training/transitions/start-activity.html)
关于示例代码：[ActivitySceneTransitionBasic](https://github.com/googlesamples/android-ActivitySceneTransitionBasic)