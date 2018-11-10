# 一、 追踪触摸和轨迹移动

> 原文： https://developer.android.com/training/gestures/movement

这篇文字讲述了如何跟踪触摸事件中的移动。

一个触摸事件中 位置，方向，触摸面积都在不停变化的时候，系统会携带着 ACTION_MOVE 调用 onTouchEvent()。就像 [Detecting Common Gestures](https://developer.android.com/training/gestures/detector.html) 一文中描述的，所有的这些变化都记录于 onTouchEvent() 方法的 MotionEvent 参数中。

因为交互过程中基于手指的触目不总是精确的，探测触摸手势经常更多基于移动而非简单的接触。为了帮助 app 判断是基于移动的手势(例如滑动)还是基于非移动的手势(例如点按)，android 提供了“触摸斜率”的概念。“触摸斜率”指的是在手势被系统理解为基于移动的手势之前可以偏移的像素数。关于“触摸斜率”的更多讨论请参考：[Managing Touch Events in a ViewGroup](https://developer.android.com/training/gestures/viewgroup.html#vc)

根据你程序的需要，这里提供了几种不同的追踪移动手势的方式。例如：
- 移动开始和结束的位置(例如，在屏幕上从 A 点移动到 B 点)。
- 探测触摸点的 X，Y 坐标确定触摸点的移动方向。
- 历史。你可以通过调用 `MotionEvent.getHistorySize()` 获取手势历史记录数据。然后你可以通过 `MotionEvent.getHestorical<Value>` 获取每一个相关的历史事件的位置，size(这个属性碰到几次了，一直不了解具体是什么)，时间和压力。当解释一个正在追踪的手势的是偶历史是有作用的，比如触摸绘制手势。更多关于 [MotionEvent](https://developer.android.com/reference/android/view/MotionEvent.html) 的信息。
- 指针在屏幕移动的速度。

请参阅以下相关资源：
- [Input Events API Guide](https://developer.android.com/guide/topics/ui/ui-events.html)
- [Sensors Overview](https://developer.android.com/guide/topics/sensors/sensors_overview.html)
- [Making the View Interactive](https://developer.android.com/training/custom-views/making-interactive.html)

## 追踪速度

根据触摸点的位置和距离你可以获得一个基于移动的手势。速度常常是手势一个探测的一个特征因素，甚至用于判断手势事件是否发生。为了更方便的计算速出，Android 提供了 `VelocityTracker` 类。VelocityTracker 类帮助你追踪触摸事件的速度。对于手势来讲这是有用的，这个参数是一些手势的必要条件，例如 fling(抛,掷) 。

以下简单的例子说明了 [VelocityTracker](https://developer.android.com/training/gestures/movement) API 的用途:
```java
public class MainActivity extends Activity {
    private static final String DEBUG_TAG = "Velocity";
        ...
    private VelocityTracker mVelocityTracker = null;
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int index = event.getActionIndex();
        int action = event.getActionMasked();
        int pointerId = event.getPointerId(index);

        switch(action) {
            case MotionEvent.ACTION_DOWN:
                if(mVelocityTracker == null) {
                    // Retrieve a new VelocityTracker object to watch the
                    // velocity of a motion.
                    mVelocityTracker = VelocityTracker.obtain();
                }
                else {
                    // Reset the velocity tracker back to its initial state.
                    mVelocityTracker.clear();
                }
                // Add a user's movement to the tracker.
                mVelocityTracker.addMovement(event);
                break;
            case MotionEvent.ACTION_MOVE:
                mVelocityTracker.addMovement(event);
                // When you want to determine the velocity, call
                // computeCurrentVelocity(). Then call getXVelocity()
                // and getYVelocity() to retrieve the velocity for each pointer ID.
                mVelocityTracker.computeCurrentVelocity(1000);
                // Log velocity of pixels per second
                // Best practice to use VelocityTrackerCompat where possible.
                Log.d("", "X velocity: " +
                        VelocityTrackerCompat.getXVelocity(mVelocityTracker,
                        pointerId));
                Log.d("", "Y velocity: " +
                        VelocityTrackerCompat.getYVelocity(mVelocityTracker,
                        pointerId));
                break;
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_CANCEL:
                // Return a VelocityTracker object back to be re-used by others.
                mVelocityTracker.recycle();
                break;
        }
        return true;
    }
}
```

> **注意:** 你应该在 ACTION_MOVE 事件计算速度，而不是在 ACTION_UP 事件。在 ACTION_UP 之后 (X,Y) 坐标都是 0 。


## 二、 捕获指针

一些应用(例如，游戏，远程桌面，和虚拟机)，通过鼠标控制得到了许多好处。指针捕获是 Android 8.0(API26) 的特性，从此以后将所有的鼠标事件提供给app中的焦点 view 从而完成对 app 的控制。

### 2.1、 请求指针捕获

当 app 中某个视图层级获取到焦点的时候你才能申请该 View(位于视图层级中) 的指针捕获。因此，你应该在用户发生活动的 view 上申请指针捕获，例如 Activity 中的 onClick() 事件或者 onWindowFocusChanged() 事件。

通过调用 View.requestPointerCapture() 方法请求指针捕获。
```java
@Override
public void onClick(View view) {
    view.requestPointerCapture();
}
```

一旦请求指针捕获成功，Android 就调用 onPointerCaptureChange(true)。只要和请求指针捕获的 View 位于同一层级，系统将鼠标事件交给app中的焦点 view 。其他 app 在本 app 停止接收鼠标事件之前不在接收鼠标事件，其中包含 ACTION_OUTSIDE 事件。Android 通常从鼠标之外的其他来源传递鼠标事件，但是鼠标指针不可见。

### 2.2、 持有捕获的指针事件

一旦 View 成功申请指针捕获，Android 就开始向其传送鼠标事件。一旦焦点 view 可以通过执行下面的任务持有鼠标事件：
1. 如果使用自定义 view ，`override onCapturedPointerEvent(MotionEvent)` 。
2. 其他情况注册 `OnCapturedPointerListener` 。

以下展示了如何实现 `onCapturedPointerEvent(MotionEvent)`
```java
@Override
public boolean onCapturedPointerEvent(MotionEvent motionEvent) {
  // Get the coordinates required by your app
  float verticalOffset = motionEvent.getY();
  // Use the coordinates to update your view and return true if the event was
  // successfully processed
  return true;
}
```

下文展示了如何注册 `OnCapturedPointerListener`
```java
myView.setOnCapturedPointerListener(new View.OnCapturedPointerListener() {
  @Override
  public boolean onCapturedPointer (View view, MotionEvent motionEvent) {
    // Get the coordinates required by your app
    float horizontalOffset = motionEvent.getX();
    // Use the coordinates to update your view and return true if the event was
    // successfully processed
    return true;
  }
});
```

无论通过那种方式，你的 view 将会接收到 MotionEvent 其中携带指针坐标，用于指示相对位移，例如 X/Y 增量，类似于通过指示球传送坐标。你可以通过 getX(),getY() 获得指针坐标。

### 2.3、 释放指针捕获

你的 app 可以通过 releasePointerCapture() 释放指针捕获：
```java
@Override
public void onClick(View view) {
    view.releasePointerCapture();
}
```

系统可以再没有明确调用 `releasePointerCapture()` 情况下移除指针捕获，这通常是因为包含 view 的视图层级失去了焦点。