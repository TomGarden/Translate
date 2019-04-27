## 0x00、 [管理 ViewGroup 中的触摸事件](https://developer.android.com/training/gestures/viewgroup)

ViewGroup 特别关心当前持有的触摸事件，因为 ViewGroup 通常持有用于响应不同触摸事件的孩子 view 。要确保每一个 View 正确的接收了对应的触摸事件请覆写 [onInterceptTouchEvent()](https://developer.android.com/reference/android/view/ViewGroup.html#onInterceptTouchEvent(android.view.MotionEvent))

参考如下内容：
- [Input Events API Guide](https://developer.android.com/guide/topics/ui/ui-events.html)
- [Sensors Overview](https://developer.android.com/guide/topics/sensors/sensors_overview.html)
- [Making the View Interactive](https://developer.android.com/training/custom-views/making-interactive.html)

## 一、 拦截 ViewGroup 中的触摸事件

每当 ViewGroup 或者其子 View 发生触摸事件的时候 `onInterceptTouchEvent()` 就会被调用。 如果 `onInterceptTouchEvent()` return true，代表 MotionEvent 被拦截了，未传递给子 View 而是交由父控件的 onTouchEvent() 处理。

父控件通过 `onInterceptTouchEvent()` 方法可以在子控件之前检查任何触摸事件。 如果本方法 return true 表示之前处理事件的子 view 将收到一个 ACTION_CANCEL 信号，并且从该时间点开始之后的事件会被发送到父 view 的 oTouchEvent() 方法进行常规处理。 onInterceptTouchEvent() 也能return false ，事件经过简单的检查被传送到他们通常的目标处，在哪里这些事件将会在各自的 onTouchEvent() 中被处理。

在下文的代码片段中， `MyViewGroup` 集成 ViewGroup 。 `MyViewGroup` 包含多个子 view。如果你在屏幕上通过手指水平拖动一个子控件，子控件将不在接收触摸事件，MyViewGroup 应该通过滚动其中的内容处理该事件。然而，如果你按子视图中的按钮，或者垂直滚动子视图，父视图将不会拦截触摸事件，因为子视图是预定目标。在这种情况下 onInterceptTouchEvent() 应该 return false 。并且 MyViewGroup 的 onTouchEvent() 不会被调用。
```java
public class MyViewGroup extends ViewGroup {

    private int mTouchSlop;

    ...

    ViewConfiguration vc = ViewConfiguration.get(view.getContext());
    mTouchSlop = vc.getScaledTouchSlop();

    ...

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        /*
         * This method JUST determines whether we want to intercept the motion.
         * If we return true, onTouchEvent will be called and we do the actual
         * scrolling there.
         */


        final int action = MotionEventCompat.getActionMasked(ev);

        // Always handle the case of the touch gesture being complete.
        if (action == MotionEvent.ACTION_CANCEL || action == MotionEvent.ACTION_UP) {
            // Release the scroll.
            mIsScrolling = false;
            return false; // Do not intercept touch event, let the child handle it
        }

        switch (action) {
            case MotionEvent.ACTION_MOVE: {
                if (mIsScrolling) {
                    // We're currently scrolling, so yes, intercept the
                    // touch event!
                    return true;
                }

                // If the user has dragged her finger horizontally more than
                // the touch slop, start the scroll

                // left as an exercise for the reader
                final int xDiff = calculateDistanceX(ev);

                // Touch slop should be calculated using ViewConfiguration
                // constants.
                if (xDiff > mTouchSlop) {
                    // Start scrolling!
                    mIsScrolling = true;
                    return true;
                }
                break;
            }
            ...
        }

        // In general, we don't want to intercept touch events. They should be
        // handled by the child view.
        return false;
    }

    @Override
    public boolean onTouchEvent(MotionEvent ev) {
        // Here we actually handle the touch event (e.g. if the action is ACTION_MOVE,
        // scroll this container).
        // This method will only be called if the touch event was intercepted in
        // onInterceptTouchEvent
        ...
    }
}
```

注意 ViewGroup 提供了 `requestDisallowInterceptTouchEvent()` 方法。 当子控件不希望父控件 ViewGroup 通过 onInterceptTouchEvent() 拦截触摸事件的时候调用。

### 1.1、 处理 ACTION_OUTSIDE 事件

如果 ViewGroup 接收一个携带 ACTION_OUtSIDE 的 MotionEvent ，这个时间默认情况下不会被分配到子 view 。去处理一个携带 ACTION_OUTSIDE 的 MotionEvent ，要么覆写 dispatchTouchEvent(MotionEvent event) 将事件分发给合适的 View ，要么在相关的 `Windows.Callback` 中消耗它(例如 Activity)。

## 二、 使用 ViewConfiguration 上下文

在上文的代码片段中使用 ViewConfiguration 初始化变量 mTouchSlop。你可以使用 ViewConfiguration 类访问 Android 系统常用的 手势，速度和时间。 

“触摸斜率”代表一个用户触摸事件在被解释为滚动之前所能偏移的像素数。触摸斜率的特点用于当用户向执行其他的触摸操作(例如触摸屏幕上的元素)的时候防止意外的滚动。

另外两个常用的 ViewConfiguration 方法是 getScaledMinimumFlingVelocity() 和 getScaledMaximumFlingVelocity() 。这两个方法分别 return 初始化 fling 的最小和最大速率，单位是 像素/秒。例如：
```java
ViewConfiguration vc = ViewConfiguration.get(view.getContext());
private int mSlop = vc.getScaledTouchSlop();
private int mMinFlingVelocity = vc.getScaledMinimumFlingVelocity();
private int mMaxFlingVelocity = vc.getScaledMaximumFlingVelocity();

...

case MotionEvent.ACTION_MOVE: {
    ...
    float deltaX = motionEvent.getRawX() - mDownX;
    if (Math.abs(deltaX) > mSlop) {
        // A swipe occurred, do something
    }

...

case MotionEvent.ACTION_UP: {
    ...
    } if (mMinFlingVelocity <= velocityX && velocityX <= mMaxFlingVelocity
            && velocityY < velocityX) {
        // The criteria have been satisfied, do something
    }
}
```

## 三、 延伸子视图的触摸区域

Android 提供了 TouchDelegate 类使父 view 扩展子 view 的可触摸边界成为可能。当孩子 view 必须很小或者需要有一个大的触摸范围的之后这很有用。如果有必要你也可以利用这种方法缩小控件的触摸区域。

在下面的例子中， ImageButton 是 "代理 view"(就是触摸区域将被扩展的对象)，这是布局文件：
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
     android:id="@+id/parent_layout"
     android:layout_width="match_parent"
     android:layout_height="match_parent"
     tools:context=".MainActivity" >

     <ImageButton android:id="@+id/button"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:background="@null"
          android:src="@drawable/icon" />
</RelativeLayout>
```

下面的代码做了这些事情：
- 获取父 view 并且在 UI 线程上 post 一个 Runable 。这确保在调用 getHitRect() 之前显示孩子 view。getHitRect() 获取了孩子 view 相对于父 view 坐标系的隐藏的矩形触摸范围。
- 绑定 ImageButton 并且调用 getHicRect() 获取孩子 view 的触摸区域范围。
- 扩展 ImageButton 的隐藏矩形边界。
- 以孩子 View ImageButton 和其隐藏矩触摸形边界为参数实例化 TouchDelegate 。
- 将 TouchDelegate 设置到父 view，之后在触摸代理上的触摸事件都会交给孩子 view 。

作为 ImageButton 的代理的容器，父 view 会接收所有触摸事件，如果触摸事件出现在孩子隐藏触摸区域矩形内，父控件会将触摸事件交给孩子 view。
```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        // Get the parent view
        View parentView = findViewById(R.id.parent_layout);

        parentView.post(new Runnable() {
            // Post in the parent's message queue to make sure the parent
            // lays out its children before you call getHitRect()
            @Override
            public void run() {
                // The bounds for the delegate view (an ImageButton
                // in this example)
                Rect delegateArea = new Rect();
                ImageButton myButton = (ImageButton) findViewById(R.id.button);
                myButton.setEnabled(true);
                myButton.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View view) {
                        Toast.makeText(MainActivity.this,
                                "Touch occurred within ImageButton touch region.",
                                Toast.LENGTH_SHORT).show();
                    }
                });

                // The hit rectangle for the ImageButton
                myButton.getHitRect(delegateArea);

                // Extend the touch area of the ImageButton beyond its bounds
                // on the right and bottom.
                delegateArea.right += 100;
                delegateArea.bottom += 100;

                // Instantiate a TouchDelegate.
                // "delegateArea" is the bounds in local coordinates of
                // the containing view to be mapped to the delegate view.
                // "myButton" is the child view that should receive motion
                // events.
                TouchDelegate touchDelegate = new TouchDelegate(delegateArea,
                        myButton);

                // Sets the TouchDelegate on the parent view, such that touches
                // within the touch delegate bounds are routed to the child.
                if (View.class.isInstance(myButton.getParent())) {
                    ((View) myButton.getParent()).setTouchDelegate(touchDelegate);
                }
            }
        });
    }
}
```