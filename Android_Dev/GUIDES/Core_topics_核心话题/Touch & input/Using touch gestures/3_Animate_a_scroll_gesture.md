# 为滚动手势添加动画

> 原文:https://developer.android.com/training/gestures/scroll

在 Android 中提到“滚动”，具有代表性的是 ScrollView 类。任何一个可能延伸超过包含其 layout 的 view 都应该被嵌入一个 ScrollView，以便提供通过 framework 管理的滚动视图。仅在特殊的情况下才有必要定制滚动视图。这篇文字描述了一种特殊情况：使用滚动控件展示滚动手势的效果。

你可以使用 scrollers(Scroller 或者 OverScroller) 收集数据你需要制作一个用于相应触摸事件的滚动动画。他们是相似的，但是 OverScroller 包含用于通知用户在手势事件之后已经到达内容边缘的方法。`InteractiveChart` 例子使用 [EdgeEffect](https://developer.android.com/reference/android/widget/EdgeEffect.html) 类(实际使用 EdgeEffectCompat 类)展示了当用户到达内容边缘时候的“发光”效果。

> ☆ 注意：我们认为制作滚动动画用 OverScroller 比用 Scroller 更好。OverScroller 为旧设备提供了更优秀的向前兼容。
> 另外，通常在实现自滚动时你需要 scroller 。 在你的布局中嵌套 ScrollView 或者 HorzontalScrollView 都可以为你服务。

一个滚动器随着时间推移而滚动的效果，使用平台标准的物理特性(摩擦力，速度，等)。滚动器自身不绘制任何内容。Scrovvers 追踪你在一定时间内的滚动偏移，但是它不自动应用这些位置到 view 中。通过一定的频率获取并使用新的坐标使动画看起来更平滑是你的责任。

参考如下资源：
- [Input Events API Guide](https://developer.android.com/guide/topics/ui/ui-events.html)
- [Sensors Overview](https://developer.android.com/guide/topics/sensors/sensors_overview.html)
- [Making the View Interactive](https://developer.android.com/training/custom-views/making-interactive.html)

## 一、 了解滚动(Scrolling)术语

根据不同上下文 Scrolling(滚动) 这个词语在 Android 系统中可以得到不同的解释。

滚动(Scrolling)是一个移动视图位置的普通程序(这是你在界面中看到的)。当 X 和 Y 轴同时发生滚动事称之为“平移”(panning)。例程序 `InterctiveChat` 中提供了简单的例子用于说明两种“滚动”(scrolling)：“拖”(dragging)和“抛”(flinging)的不同：

- **拖(Dragging)** 当用户在屏幕上拖动手指的时候发生。`GestureDetector.OnGestureListener.onScroll()` 经常实现拖(dragging)。关于此更多的细节请查看[Dragging and Scaling](https://developer.android.com/training/gestures/scale)。

- **抛(Flinging)** 当用户的手势在屏幕上快速滑动后离开发生这种滚动。当用户快速松开起手指，你通常希望控件继续保持滚动(strolling),直到逐渐减速至停止滚动。通过使用一个 scroller 对象实现 `GestureDetector.OnGestureListener.onFling()` 来实现这种滚动。这是本文的主题。

scroller 对象经常和 fling 手势结合使用，事实上，在任何 UI 需要根据手势展示漂亮的滚动响应的上下文都可以使用 scroller 。例如你可以覆写 onTouchEvent() 去执行触摸事件，并且制造一个滚动效果或者一个"捕捉页面(snapping to page)" 用于响应触摸事件。

## 二、 实现基于触摸的滚动

本节描述了如何使用 scroller 。下文展示的代码片段来自 `InteractiveChart` 提供了一个简单的类。使用了 `GestureDetector` ，并且覆写了 `GestureDetector.SimpleOnGestureListener.onFiling()` 。它通过 `OverScroller` 追踪 fling 手势。如果用户在 fling 手势之后超出了上下文页面，app 展示一个“发光”效果。

> ☆ 注意： **InteractiveChart** 例子展示了一个可以执行 放大、平移、滚动 等操作的图表。下文的代码片段中 `mContentRect` 代表 view 中的矩形坐标，图表将绘制于该矩形区域。在任意时间内总表的子域都会被绘制在这个矩形区域。 `mCurrentViewport` 代表图表当前展示在屏幕中的 图表的一部分。因为像素偏移通常是整数， `mContentRect` 是 [`Rect`](https://developer.android.com/reference/android/graphics/Rect.html) 类型。 因为图表范围是 decimal/float 值， `mCurrentViewport` 是 [`RectF`](https://developer.android.com/reference/android/graphics/RectF.html) 类型。

第一个片段展示了对 onFling() 的实现：
```java
// The current viewport. This rectangle represents the currently visible
// chart domain and range. The viewport is the part of the app that the
// user manipulates via touch gestures.
private RectF mCurrentViewport =
        new RectF(AXIS_X_MIN, AXIS_Y_MIN, AXIS_X_MAX, AXIS_Y_MAX);

// The current destination rectangle (in pixel coordinates) into which the
// chart data should be drawn.
private Rect mContentRect;

private OverScroller mScroller;
private RectF mScrollerStartViewport;
...
private final GestureDetector.SimpleOnGestureListener mGestureListener
        = new GestureDetector.SimpleOnGestureListener() {
    @Override
    public boolean onDown(MotionEvent e) {
        // Initiates the decay phase of any active edge effects.
        releaseEdgeEffects();
        mScrollerStartViewport.set(mCurrentViewport);
        // Aborts any active scroll animations and invalidates.
        mScroller.forceFinished(true);
        ViewCompat.postInvalidateOnAnimation(InteractiveLineGraphView.this);
        return true;
    }
    ...
    @Override
    public boolean onFling(MotionEvent e1, MotionEvent e2,
            float velocityX, float velocityY) {
        fling((int) -velocityX, (int) -velocityY);
        return true;
    }
};

private void fling(int velocityX, int velocityY) {
    // Initiates the decay phase of any active edge effects.
    releaseEdgeEffects();
    // Flings use math in pixels (as opposed to math based on the viewport).
    Point surfaceSize = computeScrollSurfaceSize();
    mScrollerStartViewport.set(mCurrentViewport);
    int startX = (int) (surfaceSize.x * (mScrollerStartViewport.left -
            AXIS_X_MIN) / (
            AXIS_X_MAX - AXIS_X_MIN));
    int startY = (int) (surfaceSize.y * (AXIS_Y_MAX -
            mScrollerStartViewport.bottom) / (
            AXIS_Y_MAX - AXIS_Y_MIN));
    // Before flinging, aborts the current animation.
    mScroller.forceFinished(true);
    // Begins the animation
    mScroller.fling(
            // Current scroll position
            startX,
            startY,
            velocityX,
            velocityY,
            /*
             * Minimum and maximum scroll positions. The minimum scroll
             * position is generally zero and the maximum scroll position
             * is generally the content size less the screen size. So if the
             * content width is 1000 pixels and the screen width is 200
             * pixels, the maximum scroll offset should be 800 pixels.
             */
            0, surfaceSize.x - mContentRect.width(),
            0, surfaceSize.y - mContentRect.height(),
            // The edges of the content. This comes into play when using
            // the EdgeEffect class to draw "glow" overlays.
            mContentRect.width() / 2,
            mContentRect.height() / 2);
    // Invalidates to trigger computeScroll()
    ViewCompat.postInvalidateOnAnimation(this);
}
```

当 onFling() 调用 postInvalidateOnAnimation() 时，computeScroll() 方法被触发用于更新 X 和 y 的值。这通常是使用 scroller 对象做滚动动画时完成的，如本例所示。

许多控件直接将 scroller 对象的 X 和 Y 坐标传递给 scrollTo() 方法。下面的代码使用了不同的实现方式——调用 computeScrollOffset() 方法获取当前位置的 X 和 Y 坐标。当满足显示过度滚动“发光”边缘效果(显示放大，X 或者 Y 超过范围，并且 app 尚未显示过多滚动)这里的代码设置滚动放光效果并且调用 postInvalidateOnAnimation() 触发一个无效的 view：
```java
// Edge effect / overscroll tracking objects.
private EdgeEffectCompat mEdgeEffectTop;
private EdgeEffectCompat mEdgeEffectBottom;
private EdgeEffectCompat mEdgeEffectLeft;
private EdgeEffectCompat mEdgeEffectRight;

private boolean mEdgeEffectTopActive;
private boolean mEdgeEffectBottomActive;
private boolean mEdgeEffectLeftActive;
private boolean mEdgeEffectRightActive;

@Override
public void computeScroll() {
    super.computeScroll();

    boolean needsInvalidate = false;

    // The scroller isn't finished, meaning a fling or programmatic pan
    // operation is currently active.
    if (mScroller.computeScrollOffset()) {
        Point surfaceSize = computeScrollSurfaceSize();
        int currX = mScroller.getCurrX();
        int currY = mScroller.getCurrY();

        boolean canScrollX = (mCurrentViewport.left > AXIS_X_MIN
                || mCurrentViewport.right < AXIS_X_MAX);
        boolean canScrollY = (mCurrentViewport.top > AXIS_Y_MIN
                || mCurrentViewport.bottom < AXIS_Y_MAX);

        /*
         * If you are zoomed in and currX or currY is
         * outside of bounds and you are not already
         * showing overscroll, then render the overscroll
         * glow edge effect.
         */
        if (canScrollX
                && currX < 0
                && mEdgeEffectLeft.isFinished()
                && !mEdgeEffectLeftActive) {
            mEdgeEffectLeft.onAbsorb((int)
                    OverScrollerCompat.getCurrVelocity(mScroller));
            mEdgeEffectLeftActive = true;
            needsInvalidate = true;
        } else if (canScrollX
                && currX > (surfaceSize.x - mContentRect.width())
                && mEdgeEffectRight.isFinished()
                && !mEdgeEffectRightActive) {
            mEdgeEffectRight.onAbsorb((int)
                    OverScrollerCompat.getCurrVelocity(mScroller));
            mEdgeEffectRightActive = true;
            needsInvalidate = true;
        }

        if (canScrollY
                && currY < 0
                && mEdgeEffectTop.isFinished()
                && !mEdgeEffectTopActive) {
            mEdgeEffectTop.onAbsorb((int)
                    OverScrollerCompat.getCurrVelocity(mScroller));
            mEdgeEffectTopActive = true;
            needsInvalidate = true;
        } else if (canScrollY
                && currY > (surfaceSize.y - mContentRect.height())
                && mEdgeEffectBottom.isFinished()
                && !mEdgeEffectBottomActive) {
            mEdgeEffectBottom.onAbsorb((int)
                    OverScrollerCompat.getCurrVelocity(mScroller));
            mEdgeEffectBottomActive = true;
            needsInvalidate = true;
        }
        ...
    }
```

以下代码是实际执行缩放的：

```java
// Custom object that is functionally similar to Scroller
Zoomer mZoomer;
private PointF mZoomFocalPoint = new PointF();
...

// If a zoom is in progress (either programmatically or via double
// touch), performs the zoom.
if (mZoomer.computeZoom()) {
    float newWidth = (1f - mZoomer.getCurrZoom()) *
            mScrollerStartViewport.width();
    float newHeight = (1f - mZoomer.getCurrZoom()) *
            mScrollerStartViewport.height();
    float pointWithinViewportX = (mZoomFocalPoint.x -
            mScrollerStartViewport.left)
            / mScrollerStartViewport.width();
    float pointWithinViewportY = (mZoomFocalPoint.y -
            mScrollerStartViewport.top)
            / mScrollerStartViewport.height();
    mCurrentViewport.set(
            mZoomFocalPoint.x - newWidth * pointWithinViewportX,
            mZoomFocalPoint.y - newHeight * pointWithinViewportY,
            mZoomFocalPoint.x + newWidth * (1 - pointWithinViewportX),
            mZoomFocalPoint.y + newHeight * (1 - pointWithinViewportY));
    constrainViewport();
    needsInvalidate = true;
}
if (needsInvalidate) {
    ViewCompat.postInvalidateOnAnimation(this);
}
```

`computeScrollSurfaceSize()` 在上文代码中被调用。用于计算当前可滚动的视图的像素尺寸。例如，如果整个图表区域都可见，则这只是当前的大小mContentRect。如果图表在两个方向都放大了200％，则返回的尺寸将是水平和垂直两倍。

```java
private Point computeScrollSurfaceSize() {
    return new Point(
            (int) (mContentRect.width() * (AXIS_X_MAX - AXIS_X_MIN)
                    / mCurrentViewport.width()),
            (int) (mContentRect.height() * (AXIS_Y_MAX - AXIS_Y_MIN)
                    / mCurrentViewport.height()));
}
```

对于滚动的另一个用例请参考 [ViewPager](https://developer.android.com/reference/android/support/v4/view/ViewPager.html) 的[源代码](http://github.com/android/platform_frameworks_support/blob/master/v4/java/android/support/v4/view/ViewPager.java)。它随着手指做出滚动动画，并且使用滚动实现"贴合到页面"的动画。