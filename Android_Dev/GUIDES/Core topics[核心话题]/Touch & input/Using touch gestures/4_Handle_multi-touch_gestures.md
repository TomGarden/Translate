# 多点触控手势

多个(>1)手指在同一时间触摸屏幕会发生多点称触控事件。这篇文字描述了如何探测多点触控手势。

参考如下内容：
- [Input Events API Guide](https://developer.android.com/guide/topics/ui/ui-events.html)
- [Sensors Overview](https://developer.android.com/guide/topics/sensors/sensors_overview.html)
- [Making the View Interactive](https://developer.android.com/training/custom-views/making-interactive.html)

## 一、 追踪多个触控点

当屏幕在同一时间有多个触摸点的时候，系统会生成如下触摸事件：

- [ACTION_DOWN](https://developer.android.com/reference/android/view/MotionEvent.html#ACTION_DOWN) —— 首个触控点触摸屏幕的时候发生。这时多点触控手势的开始。这一个触控点的数据会保存在索引为 0 的 MotionEvent 。
- [ACTION_POINTER_DOWN](https://developer.android.com/reference/android/support/v4/view/MotionEventCompat.html#ACTION_POINTER_DOWN) —— 当第若干个(>1)触控点触摸屏幕的时候发生。这个触控点的数据所在的索引可以通过 getActionIndex() 获得。
- [ACTION_MOVE](https://developer.android.com/reference/android/view/MotionEvent.html#ACTION_MOVE) —— 当触摸这的点发生变化的时候发生。
- [ACTION_POINTER_UP](https://developer.android.com/reference/android/support/v4/view/MotionEventCompat.html#ACTION_POINTER_UP) —— 当一个次要的触控点离开屏幕的时候发生。
- [ACTION_UP](https://developer.android.com/reference/android/view/MotionEvent.html#ACTION_UP) —— 当最后一个触控点离开屏幕发生。

你可以通过使用 MotionEvent(结合触控点的index和ID) 对一个单独的点保持追踪。

- **index :**  MotionEvent 将每一个有效触控点的信息都存储在一个数组中。触控点的 index 就是数组的 index。大量的 MotionEvent 通过使用 index 参数与触控点数据产生交互而不是使用 ID 。
- **ID :** 每一个触控点都有一个和触摸事件相对一的持久的 ID 。可以通过 ID 完成对一个手势的追踪。

在一个触控首示众单个触控点的出现顺序是未定义的。 因此，触控点的 index 可以从一个事件切换到另一个，但是，触控点的 ID 在触控点保持存活期间是不变的。 可以通过 getPointerId() 获取一个触控点的 ID 从而追踪该触控点所发生的手势。通过 ID 成功获取手势之后可以通过 findPointerIndex() 方法获取该 ID 对应触摸手势的 index 。例如：
```java
private int mActivePointerId;

public boolean onTouchEvent(MotionEvent event) {
    ....
    // Get the pointer ID
    mActivePointerId = event.getPointerId(0);

    // ... Many touch events later...

    // Use the pointer ID to find the index of the active pointer
    // and fetch its position
    int pointerIndex = event.findPointerIndex(mActivePointerId);
    // Get the pointer's current position
    float x = event.getX(pointerIndex);
    float y = event.getY(pointerIndex);
}
```

## 二、 获取 MotionEvnet 的活动。

你可以通过 getActionMasked()(或者兼容性更优的 MotionEventCompat.getActionMasked()) 获取 MotionEvent 中的活动。 和旧的 getAction() 方法不同，getActionMasked() 方法是为多点触控手势而生。它 return 正在执行的被屏蔽的操作，不包含指针 index 。你可以在后续使用 getActionIndex() 获取和活动相关的触控点 index 。在下文代码中进行了说明：

> ☆ 注意： 这个例子使用了 [MotionEventCompat](https://developer.android.com/reference/android/support/v4/view/MotionEventCompat.html) 类。这个类在 [SupportLibrary](https://developer.android.com/tools/support-library/index.html)。你应该使用 [MotionEventComat](https://developer.android.com/reference/android/support/v4/view/MotionEventCompat.html) 为应用提供更好的平台兼容性。注意 MotionEventCompat 不是 [MotionEvent](https://developer.android.com/reference/android/view/MotionEvent.html) 的替代品。相反，它提供的静态方法最终也是依赖 MotionEvent 对象的。

```java
int action = MotionEventCompat.getActionMasked(event);
// Get the index of the pointer associated with the action.
int index = MotionEventCompat.getActionIndex(event);
int xPos = -1;
int yPos = -1;

Log.d(DEBUG_TAG,"The action is " + actionToString(action));

if (event.getPointerCount() > 1) {
    Log.d(DEBUG_TAG,"Multitouch event");
    // The coordinates of the current screen contact, relative to
    // the responding View or Activity.
    xPos = (int)MotionEventCompat.getX(event, index);
    yPos = (int)MotionEventCompat.getY(event, index);

} else {
    // Single touch event
    Log.d(DEBUG_TAG,"Single touch event");
    xPos = (int)MotionEventCompat.getX(event, index);
    yPos = (int)MotionEventCompat.getY(event, index);
}
...

// Given an action int, returns a string description
public static String actionToString(int action) {
    switch (action) {

        case MotionEvent.ACTION_DOWN: return "Down";
        case MotionEvent.ACTION_MOVE: return "Move";
        case MotionEvent.ACTION_POINTER_DOWN: return "Pointer Down";
        case MotionEvent.ACTION_UP: return "Up";
        case MotionEvent.ACTION_POINTER_UP: return "Pointer Up";
        case MotionEvent.ACTION_OUTSIDE: return "Outside";
        case MotionEvent.ACTION_CANCEL: return "Cancel";
    }
    return "";
}
```

关于多点触控更多的介绍请查阅 [Drag and Scale](https://developer.android.com/training/gestures/scale.html)