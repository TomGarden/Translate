# 拖动和缩放

> 原文： https://developer.android.com/training/gestures/scale

这篇文字描述了如何通过 onTouchEvnet() 方法解释触摸事件从而来操控屏幕中对象的拖动和缩放。

参考如下内容：
- [Input Events API Guide](https://developer.android.com/guide/topics/ui/ui-events.html)
- [Sensors Overview](https://developer.android.com/guide/topics/sensors/sensors_overview.html)
- [Making the View Interactive](https://developer.android.com/training/custom-views/making-interactive.html)

## 一、 拖动一个对象

> ☆ 如果你的目标兼容版本是 Android 3.0 及更高版本，你可以通过 View.OnDragListener 使用 build-in drag-and-drag 事件监听细节描述参照： [Drag and Drag](https://developer.android.com/guide/topics/ui/drag-drop.html)

针对一个触摸手势常用的操作是在屏幕上拖动一个对象。随后的代码片段使用户在屏幕上拖动一个图片。注意如下几个点：

- 在一个拖动(或者滑动)操作中。app 必须始终追踪原始的指针(或者手指)，甚至当新增加了手指编程多指触控的时候也是也不能放弃对原始指针的追踪。例如，当用户正在拖动一张图片的时候在屏幕上放置了第二个手指，然后第一根手指离开屏幕。如果你的 app 仅仅追踪一根手指，它会把第二根手指作为默认的数据发生点，当第一根手指离开后，图片就会移动到第二根手指的位置处。

- 为了防止上面的事情发生，你的 app 需要辨认在原始指针出现后出现的指针。要做到这件事情，需要在 onTouchEvnet() 方法中关注 ACTION_POINTER_DOWN 和 ACTION_POINTER_UP ，就像  [Handling Multi-Touch Gestures](https://developer.android.com/training/gestures/multi.html) 中描述的那样， 在第二个指针出现和离开的时候做出合理反馈。

- 在 ACTION_POINTER_UP 事件，例程中提取了原始指针的 index 确保活动指针的 ID 不是不触摸屏幕的指针。如果它是 app 选择另一个指针活动并且保存其当前的 X Y 坐标。因为当前保存的指针在 ACTION_MOVE 被用于计算在屏幕上的移动距离，app 将总是使用正确的指针计算移动距离。

下述代码片段允许用户在屏幕上拖动一个对象。它记录初始指针的位置，计算该指针移动的距离然后移动对象到新位置。它按照上述原理正确的管理增加指针的可能性。

注意下述代码使用 getActionMasked() 方法。你应该总是使用这个方法(或者使用兼容性更优的 MotionEventCompat.getActionMasked())去获取 MotionEvent 。不像 getAction() 方法，getActionMasked() 是专门为多指触控而设计的。它 return 正在执行的被屏蔽的动作，不包括指针索引位。

```java
// The ‘active pointer’ is the one currently moving our object.
private int mActivePointerId = INVALID_POINTER_ID;

@Override
public boolean onTouchEvent(MotionEvent ev) {
    // Let the ScaleGestureDetector inspect all events.
    mScaleDetector.onTouchEvent(ev);

    final int action = MotionEventCompat.getActionMasked(ev);

    switch (action) {
    case MotionEvent.ACTION_DOWN: {
        final int pointerIndex = MotionEventCompat.getActionIndex(ev);
        final float x = MotionEventCompat.getX(ev, pointerIndex);
        final float y = MotionEventCompat.getY(ev, pointerIndex);

        // Remember where we started (for dragging)
        mLastTouchX = x;
        mLastTouchY = y;
        // Save the ID of this pointer (for dragging)
        mActivePointerId = MotionEventCompat.getPointerId(ev, 0);
        break;
    }

    case MotionEvent.ACTION_MOVE: {
        // Find the index of the active pointer and fetch its position
        final int pointerIndex =
                MotionEventCompat.findPointerIndex(ev, mActivePointerId);

        final float x = MotionEventCompat.getX(ev, pointerIndex);
        final float y = MotionEventCompat.getY(ev, pointerIndex);

        // Calculate the distance moved
        final float dx = x - mLastTouchX;
        final float dy = y - mLastTouchY;

        mPosX += dx;
        mPosY += dy;

        invalidate();

        // Remember this touch position for the next move event
        mLastTouchX = x;
        mLastTouchY = y;

        break;
    }

    case MotionEvent.ACTION_UP: {
        mActivePointerId = INVALID_POINTER_ID;
        break;
    }

    case MotionEvent.ACTION_CANCEL: {
        mActivePointerId = INVALID_POINTER_ID;
        break;
    }

    case MotionEvent.ACTION_POINTER_UP: {

        final int pointerIndex = MotionEventCompat.getActionIndex(ev);
        final int pointerId = MotionEventCompat.getPointerId(ev, pointerIndex);

        if (pointerId == mActivePointerId) {
            // This was our active pointer going up. Choose a new
            // active pointer and adjust accordingly.
            final int newPointerIndex = pointerIndex == 0 ? 1 : 0;
            mLastTouchX = MotionEventCompat.getX(ev, newPointerIndex);
            mLastTouchY = MotionEventCompat.getY(ev, newPointerIndex);
            mActivePointerId = MotionEventCompat.getPointerId(ev, newPointerIndex);
        }
        break;
    }
    }
    return true;
}
```

## 二、 拖动平移

上一节展示了一个在屏幕上拖动对象的例子。另一种常见的情境是平移，也就是当用户拖动事件发生的时候对象在 X Y 轴方向均发生滚动。上面的代码片段中直接拦截 MotionEvent 活动实现拖动。本节代码片段得益于平台对常见手势的支持。覆写了 `GestureDetector.SimpleOnGestureListener.onScroll()`。

为了提供更多的上下文，当用户使用手指拖动内容平移的时候 onScroll() 被调用。onScroll() 仅当手指按着的时候被调用；只要手指离开屏幕手势事件结束，或者 fling 手势被调用(如果手指在它离开前加速移动了)。关于移动滚动的更多讨论：[Animating a Scroll Gesture](https://developer.android.com/training/gestures/scroll.html)。

代码片段摘自 onScroll():
```java
// The current viewport. This rectangle represents the currently visible
// chart domain and range.
private RectF mCurrentViewport =
        new RectF(AXIS_X_MIN, AXIS_Y_MIN, AXIS_X_MAX, AXIS_Y_MAX);

// The current destination rectangle (in pixel coordinates) into which the
// chart data should be drawn.
private Rect mContentRect;

private final GestureDetector.SimpleOnGestureListener mGestureListener
            = new GestureDetector.SimpleOnGestureListener() {
...

@Override
public boolean onScroll(MotionEvent e1, MotionEvent e2,
            float distanceX, float distanceY) {
    // Scrolling uses math based on the viewport (as opposed to math using pixels).

    // Pixel offset is the offset in screen pixels, while viewport offset is the
    // offset within the current viewport.
    float viewportOffsetX = distanceX * mCurrentViewport.width()
            / mContentRect.width();
    float viewportOffsetY = -distanceY * mCurrentViewport.height()
            / mContentRect.height();
    ...
    // Updates the viewport, refreshes the display.
    setViewportBottomLeft(
            mCurrentViewport.left + viewportOffsetX,
            mCurrentViewport.bottom + viewportOffsetY);
    ...
    return true;
}
```

实现 onScroll() 滚动视图，响应触摸手势：
```java
/**
 * Sets the current viewport (defined by mCurrentViewport) to the given
 * X and Y positions. Note that the Y value represents the topmost pixel position,
 * and thus the bottom of the mCurrentViewport rectangle.
 */
private void setViewportBottomLeft(float x, float y) {
    /*
     * Constrains within the scroll range. The scroll range is simply the viewport
     * extremes (AXIS_X_MAX, etc.) minus the viewport size. For example, if the
     * extremes were 0 and 10, and the viewport size was 2, the scroll range would
     * be 0 to 8.
     */

    float curWidth = mCurrentViewport.width();
    float curHeight = mCurrentViewport.height();
    x = Math.max(AXIS_X_MIN, Math.min(x, AXIS_X_MAX - curWidth));
    y = Math.max(AXIS_Y_MIN + curHeight, Math.min(y, AXIS_Y_MAX));

    mCurrentViewport.set(x, y - curHeight, x + curWidth, y);

    // Invalidates the View to update the display.
    ViewCompat.postInvalidateOnAnimation(this);
}
```

## 三、 通过触摸执行滚动

和 [Detecting Common Gestures](https://developer.android.com/training/gestures/detector.html) 中讨论的一样，GestureDetector 帮助你探测 Android 常用的手势，例如 滚动，投掷，长按。对于滚动，Android 提供了 ` ScaleGestureDetector. GestureDetector` 和 `ScaleGestureDetector` 相结合的方式探测额外手势。

手势探测器通过传递给其构造方法对象报告手势事件。`ScaleGestureDetector` 使用 `ScaleGestureDetector.OnScaleGestureListener` 。Android 提供了 `ScaleGestureDetector.SimpleOnScaleGestureListener` 作为辅助类，如果你不关心所有的时间报告你可以继承它。

### 3.1、 基于缩放的例子

这个代码片阐明了缩放所涉及的基本成分。

```java
private ScaleGestureDetector mScaleDetector;
private float mScaleFactor = 1.f;

public MyCustomView(Context mContext){
    ...
    // View code goes here
    ...
    mScaleDetector = new ScaleGestureDetector(context, new ScaleListener());
}

@Override
public boolean onTouchEvent(MotionEvent ev) {
    // Let the ScaleGestureDetector inspect all events.
    mScaleDetector.onTouchEvent(ev);
    return true;
}

@Override
public void onDraw(Canvas canvas) {
    super.onDraw(canvas);

    canvas.save();
    canvas.scale(mScaleFactor, mScaleFactor);
    ...
    // onDraw() code goes here
    ...
    canvas.restore();
}

private class ScaleListener
        extends ScaleGestureDetector.SimpleOnScaleGestureListener {
    @Override
    public boolean onScale(ScaleGestureDetector detector) {
        mScaleFactor *= detector.getScaleFactor();

        // Don't let the object get too small or too large.
        mScaleFactor = Math.max(0.1f, Math.min(mScaleFactor, 5.0f));

        invalidate();
        return true;
    }
}
```

### 3.2、 更复杂的缩放例子

这有来自 `InteractiveChart` 的更复杂的例子。`InteractiveChart` 支持滚动、平移、多指缩放，使用 ScaleGestureDetector "span" (getCurrentSpanX/Y) 和 "focus"(getFocusX/Y) ：

```java
@Override
private RectF mCurrentViewport =
        new RectF(AXIS_X_MIN, AXIS_Y_MIN, AXIS_X_MAX, AXIS_Y_MAX);
private Rect mContentRect;
private ScaleGestureDetector mScaleGestureDetector;
...
public boolean onTouchEvent(MotionEvent event) {
    boolean retVal = mScaleGestureDetector.onTouchEvent(event);
    retVal = mGestureDetector.onTouchEvent(event) || retVal;
    return retVal || super.onTouchEvent(event);
}

/**
 * The scale listener, used for handling multi-finger scale gestures.
 */
private final ScaleGestureDetector.OnScaleGestureListener mScaleGestureListener
        = new ScaleGestureDetector.SimpleOnScaleGestureListener() {
    /**
     * This is the active focal point in terms of the viewport. Could be a local
     * variable but kept here to minimize per-frame allocations.
     */
    private PointF viewportFocus = new PointF();
    private float lastSpanX;
    private float lastSpanY;

    // Detects that new pointers are going down.
    @Override
    public boolean onScaleBegin(ScaleGestureDetector scaleGestureDetector) {
        lastSpanX = ScaleGestureDetectorCompat.
                getCurrentSpanX(scaleGestureDetector);
        lastSpanY = ScaleGestureDetectorCompat.
                getCurrentSpanY(scaleGestureDetector);
        return true;
    }

    @Override
    public boolean onScale(ScaleGestureDetector scaleGestureDetector) {

        float spanX = ScaleGestureDetectorCompat.
                getCurrentSpanX(scaleGestureDetector);
        float spanY = ScaleGestureDetectorCompat.
                getCurrentSpanY(scaleGestureDetector);

        float newWidth = lastSpanX / spanX * mCurrentViewport.width();
        float newHeight = lastSpanY / spanY * mCurrentViewport.height();

        float focusX = scaleGestureDetector.getFocusX();
        float focusY = scaleGestureDetector.getFocusY();
        // Makes sure that the chart point is within the chart region.
        // See the sample for the implementation of hitTest().
        hitTest(scaleGestureDetector.getFocusX(),
                scaleGestureDetector.getFocusY(),
                viewportFocus);

        mCurrentViewport.set(
                viewportFocus.x
                        - newWidth * (focusX - mContentRect.left)
                        / mContentRect.width(),
                viewportFocus.y
                        - newHeight * (mContentRect.bottom - focusY)
                        / mContentRect.height(),
                0,
                0);
        mCurrentViewport.right = mCurrentViewport.left + newWidth;
        mCurrentViewport.bottom = mCurrentViewport.top + newHeight;
        ...
        // Invalidates the View to update the display.
        ViewCompat.postInvalidateOnAnimation(InteractiveLineGraphView.this);

        lastSpanX = spanX;
        lastSpanY = spanY;
        return true;
    }
};
```