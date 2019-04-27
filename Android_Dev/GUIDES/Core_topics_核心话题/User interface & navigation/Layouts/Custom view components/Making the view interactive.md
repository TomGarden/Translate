## 0x00、 [使 View 可交互](https://developer.android.com/training/custom-views/making-interactive)

绘制 UI 只是创建自定义控件这件事的一部分。
你也需要使你的 view 响应用户的输入(这些输入应该尽可能模仿真实世界的动作)。
我们的对象的行为应该和真实世界中的真实对象的行为相仿。
例如，图像不应该突然弹到用户视野中或者其他什么地方，因为在真实世界中不会发生这样的事。
相反，图像应该从一个地方移动到另一个地方。

在界面中用户对细小的动作也是有感觉的，这细小的动作和真实世界相仿能使用户产生最微妙的反应。
例如用户抛一个 UI 对象的时候，它应该首先感觉到摩擦力，在这个动作的最后应该感觉到 UI 对象具有向抛方向的动量(惯性)。

这篇文字将介绍 Android framework 添加真实世界中的行为到自定义控件的特征。

在开始之前你可以在 [Input Events](https://developer.android.com/guide/topics/ui/ui-events.html) 和 [Property Animation](https://developer.android.com/guide/topics/graphics/prop-animation.html) 发现于此相关的额外信息。

## 0x01、 获取输入手势
和其他 UI 框架相同，Android 支持一个事件输入模型。
用户操作会转换为一个事件，该事件会触发方法回调，你能覆写回调方法自定义该操作在你应用中的响应。
在 Android 中最常见的输入事件就是触摸，这回触发 `onTouchEvent(android.view.MotionEvent)`。
覆写这个方法，获取事件：
```java
@Override
   public boolean onTouchEvent(MotionEvent event) {
    return super.onTouchEvent(event);
   }
```

触摸事件本身没有特别之处。
现在的 UI 定义了一系列手势例如点击，拉动，推动，投掷和缩放。
Android 为将原始的触摸事件转换为手势提供了 `GestureDetector`。

通过创建一个实现 `GestureDetector.onGestureListener` 的类构造一个 `GestureDetector` 。
如果你仅需要执行极少数的手势，你能继承 `GestureDetector.SimpleOnGestureListener` 从而创建一个 `GestureDetector` 实例。
在下面的例子继承了 `GestureDetector.SimpleOnGestureListener` 覆写了 `onDown(MotionEvent)`。
```java
class MyListener extends GestureDetector.SimpleOnGestureListener {
   @Override
   public boolean onDown(MotionEvent e) {
       return true;
   }
}
mDetector = new GestureDetector(PieChart.this.getContext(), new MyListener());
```

不论你是否使用 `GestureDetector.SimpleOnGestureListener` 你都必须实现 `onDown()` 方法并且 `return true` 。
这一步是有必要的，因为所有的手势都是开始于 `onDown()` 的。
假如在 `GestureDetector.SimpleOnGestureListener` 的实现类中 `onDown()` 方法 `return false`，系统会假设你希望忽略随之发生的手势，因此 `GestureDetector.SimpleOnGestureListener` 的其他方法将不会被调用。
只有当你真的希望忽略随之发生的手势事件的时候你才需要在 `onDonw` 方法 `return true` 。
一旦你实现了 `GestureDetector.OnGestureListener` 并且创建了 `GestureDetector` 对象，你就可以将 `onTouchEvent()` 接收到的手势事件交给 `GestureDetector` 处理了。
```java
@Override
public boolean onTouchEvent(MotionEvent event) {
   boolean result = mDetector.onTouchEvent(event);
   if (!result) {
       if (event.getAction() == MotionEvent.ACTION_UP) {
           stopScrolling();
           result = true;
       }
   }
   return result;
}
```

当你通过 `onTouchEvent()` 接收到了一个自己不认识的手势，你可以 `return false` 。
你能够定义自己的手势监测代码了。

## 0x02、 创建物理上合理的运动

手势是控制触摸设备的有效的方式，但是在没有与现实物理运行相仿的效果的情况下它是违反人类直觉的。
一个很好的例子是抛(fling)这个手势：用户的手指快速略过屏幕并且离开屏幕。
如果 UI 响应是在用户 fling 的方向快速移动然后听下来，这个手势所产生的感觉就好像用户推一个飞轮并让它旋转起来。

然而，模拟这个飞轮的感觉这件事并非无足轻重。
一个飞轮模型的正常工作需要大量的物理和数学知识。
幸好，Android 提供了工具类帮助模仿这些(还有其他的)行为。
`Scroller` 类是处理飞轮风格的 fling 手势的基础。

在 fling 开始的时候，调用携带起始速度以及 X Y 的边界值参数的 `fling()` 方法。
对于起始速度，你能使用 `GestureDetector` 计算。
```java
@Override
public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
   mScroller.fling(currentX, currentY, velocityX / SCALE, velocityY / SCALE, minX, minY, maxX, maxY);
   postInvalidate();
    return true;
}
```

>   **☆ NOTE：** 虽然速度通过 `GestureDetector` 进行计算在物理上是精确的，许多开发者感觉使用这个值作为 fling 动画太快了。
    通常会将速度在 X 和 Y 方向使用除以 4~8 之间的值。

`fling()` 方法设置了抛(fling)手势的物理模型。
以后你需要通过 `Scroller.computeScrollOffset()` 定期更新 `Scroller` 的。
`computeScrollOffset()` 通过读取当前时间和正在使用的物理模型计算 x 和 y 的值进而更新 `Scroller` 对象的内部状态。
调用 `getCurrX()` 和 `getCurrY()` 可以获取这些值。

多数视图通过 `Scroller` 对象的 x 和 y 位置直接调用 `scrollTo()`
饼图的例子有一点小小的不同：它使用当前滚动 y 的位置去设置饼旋转的角度。
```java
if (!mScroller.isFinished()) {
    mScroller.computeScrollOffset();
    setPieRotation(mScroller.getCurrY());
}
```

`Scroller` 类为你计算滚动的位置，但是不会自动将计算结果应用到你的视图上。
获取并且应用坐标确保滚动动画看起来流畅(平滑)是开发者的责任。
这里有两种方法可以做到这一点：
1.  在 `fling()` 之后调用 `postInvalidate()` ,从而强制重绘视图。
    这种技术需要你在 `onDraw()` 方法中计算滚动偏移量，并且每次滚动偏移两变化的时候调用 `postInvalidate()`。
2.  在抛(fling)期间为动画设置 `ValueAnimator` ，并且通过调用 `addUpdateListener()` 方法为执行更新的动画设置监听者。

在饼图(PieChart)的例子中使用第二种途径。
这种技术执行起来略显复杂，但是它能和系统动画更紧凑的工作在一起，并且不需要对 view 做额外的不必要的更新。 
缺点是 `ValueAnimator` 对 API level 11 以下不支持，所以这种技术不能在 Android 3.0 以下的版本使用。

>   **☆ NOTE：** 你能在较低级别的 API 中使用 `ValueAnimator` 。
    为此你需要在运行时检查 API level，并且忽略 API level > 11 时候的调用。
```java
mScroller = new Scroller(getContext(), null, true);
mScrollAnimator = ValueAnimator.ofFloat(0,1);
mScrollAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
    @Override
    public void onAnimationUpdate(ValueAnimator valueAnimator) {
        if (!mScroller.isFinished()) {
            mScroller.computeScrollOffset();
            setPieRotation(mScroller.getCurrY());
        } else {
            mScrollAnimator.cancel();
            onScrollFinished();
        }
    }
});
```

## 0x03、 使你 view 平滑过渡
用户期望 UI 在两种状态下的过渡是平滑的。
UI 元素应该逐渐褪色，而不是直接消失。
运动平滑的开始和结束而不是突然发生某种动作。
Android [proerty animation framework](https://developer.android.com/guide/topics/graphics/prop-animation.html)支持 Android 3.0 ，使用平滑的过渡动画是方便的。

要使用动画系统，不论什么时候一个属性的修改将会影响到你的 view 的外观，不要直接修改属性。
利用 `ValueAnimator` 修改属性。
在以下示例中，修改PieChart中当前选定的饼图切片会导致整个图表旋转，以使选择指针在所选切片中居中。
ValueAnimator 在几百毫秒内完成一次旋转，而不是立刻设置新的旋转值。
```java
mAutoCenterAnimator = ObjectAnimator.ofInt(PieChart.this, "PieRotation", 0);
mAutoCenterAnimator.setIntValues(targetAngle);
mAutoCenterAnimator.setDuration(AUTOCENTER_ANIM_DURATION);
mAutoCenterAnimator.start();
```

如果要更改的值是基本View属性之一，则执行动画会更加容易，因为Views具有内置的ViewPropertyAnimator，该ViewPropertyAnimator针对多个属性的同时动画进行了优化。
例如：
```java
animate().rotation(targetAngle).setDuration(ANIM_DURATION).start();
```