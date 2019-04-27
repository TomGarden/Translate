# Detect common gesture（检测常见手势）
> 原文：https://developer.android.com/training/gestures/detector


当用户将一个或多个手指放到触摸屏上，一个触摸手势就产生了，你的 app 应该将该触摸动作翻译为特定的手势操作。这是手势检测相对应的两个阶段。
1. 收集关于触摸事件的数据。
2. 分析上述数据查看这些数据是否与你app所支持的手势相匹配。

参考下述资源：
- [Input Events API Guide](http://developer.android.com/guide/topics/ui/ui-events.html)
- [Sensors Overview](https://developer.android.com/guide/topics/sensors/sensors_overview.html)
- [Making the View Interactive](https://developer.android.com/training/custom-views/making-interactive.html)


**Support Library classes**

本教程中的例子使用了 [GestureDetectorCompat](https://developer.android.com/reference/android/support/v4/view/GestureDetectorCompat.html) 和 [MotionEventCompat](https://developer.android.com/reference/android/support/v4/view/MotionEventCompat.html) classes。这些类在 [Support Library](https://developer.android.com/tools/support-library/index.html)中。你应该是用 Support Library classes 从而为应用获得 Android1.6 及更高版本的兼容性。注意， [MotionEventCompat](https://developer.android.com/reference/android/support/v4/view/MotionEventCompat.html) 并非 [MotionEvent](https://developer.android.com/reference/android/view/MotionEvent.html) 的替代品。相反 MotionEventCompat 提供了静态方法(通过这些方法)访问 MotionEvent 从而可以接收和事件相关连的操作。


## 第一阶段__收集数据

当一个用户将一个或多个手指放到屏幕上 View 中用于接受触摸事件的回调方法 [onTouchEvent()](https://developer.android.com/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent)) 就会被触发。将所有的触摸事件(位置，压力，size(可能是面积也可能是压力大小)，增加手指，等事件)排序，这些触摸事件最终会被处理并解释成一个手势，在这过程中 [onTouchEvent()](https://developer.android.com/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent)) 会被多次调用。

一个触摸手势开始于手指首次和屏幕产生接触，在手指移动的过程中系统会追踪手势所产生的位置数据，手势数据截止于手指离开屏幕事件数据被系统捕捉到。在交互的整个过程中，MotionEvent 向 onTouchEvent() 提供的每一个交互的西IE数据。你的 app 可以使用这些由 MotionEvent 提供的数据确定当前手势是否 app 关心的手势。

### 1.1 在 Activity 或者 View 中捕获触摸事件

在 Activity 或 View 中通过 onTouchEvent() 拦截触摸事件。

下面的代码片段通过 [getActionMasked()](https://developer.android.com/reference/android/support/v4/view/MotionEventCompat.html#getActionMasked(android.view.MotionEvent)) 方法提取用户执行的动作。这里将原始数据交给你用来判断当前手势是否你关心的手势。

```java
public class MainActivity extends Activity {
...
// This example shows an Activity, but you would use the same approach if
// you were subclassing a View.
@Override
public boolean onTouchEvent(MotionEvent event){

    int action = MotionEventCompat.getActionMasked(event);

    switch(action) {
        case (MotionEvent.ACTION_DOWN) :
            Log.d(DEBUG_TAG,"Action was DOWN");
            return true;
        case (MotionEvent.ACTION_MOVE) :
            Log.d(DEBUG_TAG,"Action was MOVE");
            return true;
        case (MotionEvent.ACTION_UP) :
            Log.d(DEBUG_TAG,"Action was UP");
            return true;
        case (MotionEvent.ACTION_CANCEL) :
            Log.d(DEBUG_TAG,"Action was CANCEL");
            return true;
        case (MotionEvent.ACTION_OUTSIDE) :
            Log.d(DEBUG_TAG,"Movement occurred outside bounds " +
                    "of current screen element");
            return true;
        default :
            return super.onTouchEvent(event);
    }
}
```

你可以用自己的逻辑去判断你关心的手势是否发生了。这是一种编程判断手势是否发生的方法。如果你关心的手势是 双击，长按，速滑，等你可以通过使用 [GestureDetctor](https://developer.android.com/reference/android/view/GestureDetector.html) 类 GestureDector 可以帮你在无需更多编码的情况下简便的判断大量常用手势。关于其更多的讨论在下文：[Detect Gestures](https://developer.android.com/training/gestures/detector#detect) 

### 1.2 针对单独的 view 捕捉触摸事件

调用 onTouchEvent() 方法的一个替代方法是为 View 通过 [setOnTouchListener()](https://developer.android.com/reference/android/view/View.html#setOnTouchListener(android.view.View.OnTouchListener)) 附加 [View.OnTouchLinstener](https://developer.android.com/reference/android/view/View.OnTouchListener.html) 对象。这使你可以再不创建 View 子类的前提下监听 View 的触摸事件。例如：
```java
View myView = findViewById(R.id.my_view);
myView.setOnTouchListener(new OnTouchListener() {
    public boolean onTouch(View v, MotionEvent event) {
        // ... Respond to touch events
        return true;
    }
});
```

值的注意的是，如果本方法 return false ，将只能拦截 ACTION_DOWN 事件。如果你 return false 这个 listener 将不会调用子队列中的 ACTION_MOVE 和 ACTION_UP 事件。这是因为 ACTION_DOWN 是所有事件的开始。

如果你自定义一个 View ，你可以像上文的描述一样 Overried onTouchEvent() 。

## 第二阶段__检测手势

Android 提供了 [GestureDetector](https://developer.android.com/reference/android/view/GestureDetector.html) 类用于检测常用手势。一些手势被包含的方法支持，如 onDown(),onLongPress(),onFling(),等等。你可以将 GestureDetector 与上文描述的 oTouchEvent() 结合。

### 2.1 检测所有支持的手势

当你实例化一个 [GestureDetectorCompat](https://developer.android.com/reference/android/support/v4/view/GestureDetectorCompat.html) 对象时，其中一个参数就是 [GestureDetector.OnGestureListener](https://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html) 接口的实现类。当一个特定的触摸事件发生的时候 [GestureDetector.OnGestureListener](https://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html) 向用户发出通知。要使 GestureDetector 对对象 接受事件，你应该覆写 View 或者 Activity 的 onTouchEvent() 方法，并且将所有的被观察事件交给 detector 实例。

在下面的代码片段中， _on`<TouchEvent>`_ return true 表示你已经处理了触摸事件。return false 表示将触摸事件传递给 View 栈直到事件被成功处理。

运行下面代码片段去体验一下当你和屏幕交互的时候后手势是如何触发的，以及 MotionEvent 如何处理每一个手势事件。你将意识到如此简单的交互生成了多大的数据。

```java
public class MainActivity extends Activity implements
        GestureDetector.OnGestureListener,
        GestureDetector.OnDoubleTapListener{

    private static final String DEBUG_TAG = "Gestures";
    private GestureDetectorCompat mDetector;

    // Called when the activity is first created.
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        // Instantiate the gesture detector with the
        // application context and an implementation of
        // GestureDetector.OnGestureListener
        mDetector = new GestureDetectorCompat(this,this);
        // Set the gesture detector as the double tap
        // listener.
        mDetector.setOnDoubleTapListener(this);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event){
        if (this.mDetector.onTouchEvent(event)) {
            return true;
        }
        return super.onTouchEvent(event);
    }

    @Override
    public boolean onDown(MotionEvent event) {
        Log.d(DEBUG_TAG,"onDown: " + event.toString());
        return true;
    }

    @Override
    public boolean onFling(MotionEvent event1, MotionEvent event2,
            float velocityX, float velocityY) {
        Log.d(DEBUG_TAG, "onFling: " + event1.toString() + event2.toString());
        return true;
    }

    @Override
    public void onLongPress(MotionEvent event) {
        Log.d(DEBUG_TAG, "onLongPress: " + event.toString());
    }

    @Override
    public boolean onScroll(MotionEvent event1, MotionEvent event2, float distanceX,
            float distanceY) {
        Log.d(DEBUG_TAG, "onScroll: " + event1.toString() + event2.toString());
        return true;
    }

    @Override
    public void onShowPress(MotionEvent event) {
        Log.d(DEBUG_TAG, "onShowPress: " + event.toString());
    }

    @Override
    public boolean onSingleTapUp(MotionEvent event) {
        Log.d(DEBUG_TAG, "onSingleTapUp: " + event.toString());
        return true;
    }

    @Override
    public boolean onDoubleTap(MotionEvent event) {
        Log.d(DEBUG_TAG, "onDoubleTap: " + event.toString());
        return true;
    }

    @Override
    public boolean onDoubleTapEvent(MotionEvent event) {
        Log.d(DEBUG_TAG, "onDoubleTapEvent: " + event.toString());
        return true;
    }

    @Override
    public boolean onSingleTapConfirmed(MotionEvent event) {
        Log.d(DEBUG_TAG, "onSingleTapConfirmed: " + event.toString());
        return true;
    }
}
```

### 2.2 检测支持的手势子集

如果你仅关心一小部分手势，可以继承 [GestureDetector.SimpleOnGestureListener](https://developer.android.com/reference/android/view/GestureDetector.SimpleOnGestureListener.html) 取代实现 [GestureDetector.OnGestureListener ](https://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html) 的做法。

[GestureDetector.SimpleOnGestureListener](https://developer.android.com/reference/android/view/GestureDetector.SimpleOnGestureListener.html) 通过 _on`<TouchEvent>`_ return false 的方式实现了所有方法。因此你可以通过覆写指定方法达到处理你关心的手势的目的。例如，下文代码片段创建了一个 实现了 GestureDetector.SimpleOnGestureListener 的类，并且覆写了 onFline() 和 onDown() 方法。

GestureDetector.OnGestureListener 有一个开关可以用来决定是否启用该接口，那就是实现 onDown() 方法，并且在方法中 return ture(打开该接口)。这是因为所有的手势都开始于 onDown() 方法。 如果 return false 系统会假设你不关心这些手势动作，从而 GestureDetector.OnGestureListener 的其他方法不会被调用。这可能在你的app中造成意向不到的事情发生。如果你确实不关心用户手势只需 onDown() 方法中 return fasle。
```java
public class MainActivity extends Activity {

    private GestureDetectorCompat mDetector;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mDetector = new GestureDetectorCompat(this, new MyGestureListener());
    }

    @Override
    public boolean onTouchEvent(MotionEvent event){
        this.mDetector.onTouchEvent(event);
        return super.onTouchEvent(event);
    }

    class MyGestureListener extends GestureDetector.SimpleOnGestureListener {
        private static final String DEBUG_TAG = "Gestures";

        @Override
        public boolean onDown(MotionEvent event) {
            Log.d(DEBUG_TAG,"onDown: " + event.toString());
            return true;
        }

        @Override
        public boolean onFling(MotionEvent event1, MotionEvent event2,
                float velocityX, float velocityY) {
            Log.d(DEBUG_TAG, "onFling: " + event1.toString() + event2.toString());
            return true;
        }
    }
}
```