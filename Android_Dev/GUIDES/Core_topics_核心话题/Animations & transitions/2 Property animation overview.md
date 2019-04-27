## 0x00、 [Property Animation Overview](https://developer.android.com/guide/topics/graphics/prop-animation)

### 0.1、 注
时间分数就是一个小数 = 动画已经执行的时间(已经流逝的时间)/动画持续时间

### 0.2、 Overview
property animation system 是一个强大的(robuts)框架,它几乎允许你为任何内容添加动画。
你能定义一个动画在一个时间段内修改任何对象的属性，不论它时候绘制在屏幕上。
属性动画会在指定的时间段内修改属性(对象中的属性变量)。
要为任何一个对象添加动画效果，你需要指定
- 你希望修改的属性(例如对象在屏幕中的位置坐标)
- 你希望动画持续多长时间
- 你希望目标属性值在一个什么区间内发生变化

属性动画系统允许你为一个动画定义下面几个字段：
1.  Duration : 你可以指定动画持续时间，默认为 300ms 
2.  Time interpolation (事件插值): 你能指定动画消耗时间的计算方式
3.  Repeat count and behavior (重复次数和行为): 你能指定当一个动画结束的时候是否重复执行动画已经重复的次数。
    你也能指定是否是否要动画反向播放。
    设置反向播放动画已经正向播放动画不断轮替知道达到指定的动画执行次数。
4.  Animator sets (动画设定): 你能指定你的动画的帧刷新频率。
    默认的帧刷新频率是 10ms ，但是你 app 中的这个刷新速率最终依赖于系统的繁忙程度以及系统为底层计时器提供的服务速度(service the underlying timer)。

要查看更多属性动画的例子请查看[实例工程 android-CustomTransition](https://github.com/googlesamples/android-CustomTransition) 中的 `ChangeColor` 类。

## 0x01、 How property animation works
首先让我们通过一个简单的例子看一下动画的整体工作流程。
Figure 1 描述了一个其 x 属性通过 animate 进行修改的假想对象， x 属性代表对象在屏幕上的位置。
动画的持续事件被设置为 40ms 路程的总长度为 40像素。
10ms/次 是默认的帧刷新速率，也是对象水平移动的速率是  10pix/10ms 。
在 40ms 的时候动画停止，对象水平移动到终点。
这个例子是一个线性插值动画，意味着这个对象在恒定的(constant)速度下移动。

![Figure 1. Example of a linear animation](/Android_Dev/GUIDES/Images/2018-11-13_animation-linear.png)

你也可以指定动画发生的时间插值是非线性的。
Figure 2 插图假想对象在动画开始的时候加速，在动画结束的时候减速。
对象依然使用 40ms 移动 40pix ，但这次是非线性的。
开始的时候，动画加速运行一半的距离(和时间)然后减速运行剩余的距离(和时间)。
如 Figure 2 所展示的，在动画开始时候或结束时候([0,10]~[30,40])运行的距离是没有中间([10,20])运行的距离远的。

![Figure 2. Example of a non-linear animation](/Android_Dev/GUIDES/Images/2018-11-13_animation-nonlinear.png)

让我们看看上面插图中的属性动画系统组件如何实现计算动画细节的。
Fiigure 3 展示了主要的类是如何和其他类一起工作的。

![Figure 3. How animations are calculated](/Android_Dev/GUIDES/Images/2018-11-13_valueanimator.png)

`ValueAnimator` 对象保持对动画执行的跟踪，其中包含例如动画已经执行的事件，当前属性值等数据。

`ValueAnimator` 封装(encapsulate)了：一个时间插值器(`TimeInterpolator`),它定义动画的插值时机；一个 类型评估器(`TypeEvaluator`) ,它定义了如何计算动画属性的值。
例如 Figure 2 ,使用的`AccelerateDecelerateInterpolator`是`TimeInterpolator` 使用的 `IntEvaluator` 是 `TypeEvaluator` 。

开始一个动画，创建一个 `ValueAnimator` 为关心的属性给定一个开始和一个结束值，以及动画持续的时间。
在整个动画执行期间，`ValueAnimator` 基于动画已经执行的事件计算已经逝去的时间分数(值区间[0,1])。
这个时间分数代表动画执行时间的百分比和画完成度，0 表示 0% ，1 表示 100% 。
例如，Figure 1 中在 t=10ms 的时候逝去时间分数为 0.25(25%) 。

当 `ValueAnimator` 完成逝去时间分数的计算后它会调用 `TimeInterpolator` ，后者已经设置好了，用于计算插值分数 。
插值分数在考虑到所有时间插值的情况后将自己的值映射为一个逝去时间分数。
例如，Figure 2 因为动画处于缓慢加速中，插值分数大概是 0.15，它是小于在 t=10 时候的 0.25 。
在 Figure 1 插值分数总是保持和已经逝去的时间分数相同。

当插值分数倍计算出来，`ValueAnimator` 调用适当的 `TypeEvluator` 基于插值分数，动画的起始值，动画的结束值，去计算当前动画的属性值。
例如，在 Figure 2 在 t=10 时插值分数为 0.15 所以在这时的属性值应该是 0.15x(40 - 0) 或者 6 。

## 0x02、 How property animation differs from view animation

视图动画系统(view animation system)只为 view 提供了动画能力(capability) ，所以如果你想在非 view 对象中应用动画，你必须通过自己的代码实现。
视图动画系统也的界限太过狭窄，例如它能为 缩放(scaling) 和旋转(rotation)提供支持却无法改变背景色。

视图动画系统(view animation system)的另一个劣势是它只能修改 view 绘制的位置，而不是控制 view 本身。
例如，如果你为一个 button 设置一个在屏幕中移动的动画(Button 已经绘制完成了)，这个动画只是改变了视觉上的 Button 的位置，如果想要点击该 Button 还是要点击动画开始之前的位置才能触发点击事件，所以你需要实现你自己的逻辑完善这种缺陷。

使用属性动画系统(property animate system)，这些限制(canstarints)完全被取消，并且能在实际控制对象的前提下，为任何对象(view 或则 non-view)的任何属性设置动画。
property animation system 在完成动画方面在不断的变强大(robust)。
在更高的级别，你可以为特定的属性(例如颜色，位置，或者尺寸)分配动画，并且可以定义动画的各个方便，比如插值(interpolation)，多动画同步。

同时较视图动画系统(view animation system)花费更少的时间和写更少的代码。
如果视图动画系统(view animation system)完成了你的需求，或者你现存的代码已经完成了你想做的事情，那你不需要使用属性动画系统。
如果情况允许也可以同时使用两种动画系统。 

## 0x03、 AIP overview
你能发现多数 property animation system 的 API 在 `android.animation` 。
因为 view animation system 已经在 `android.view.animation` 中定义了许多插值器，他们也能在 property animation system 很好运行。
下面的表格描述了 property animation system 的主要组成。

`Animator` 类提供了创建动画的基本结构。
你通常不使用这个类，因为它只提供了最精简的(需要继承并重写才能使用)方法。
下面的子类继承了 `Animator`：

**Table 1.** Animators

|Class|Description|
|-|-|
|[ValueAnimator](https://developer.android.com/reference/android/animation/ValueAnimator.html)|主要的定时引擎，也计算属性动画中的值。它持有所有计算动画值的核心方法并且包含每一个动画的定时细节，关于一个动画是否重复的信息，监听接收更新的事件，以及有能力设置自定义的 evaluate。两方面的动画属性：为正在执行动画的对象计算并设置其属性值。`ValueAnimator`不执行第二部分，所以你必须监听计算值的变化并且按照自己的逻辑修改自己的对象。在 [Animation with ValueAnimator](https://developer.android.com/guide/topics/graphics/prop-animation#value-animator)有更详细的信息|
|[ObjectAnimator](https://developer.android.com/reference/android/animation/ObjectAnimator.html)|`ValueAnimator`对象的子类，允许你为目标对象和对象的属性设置动画。当新的数据被计算出来后这个类会更新设置的属性值。多数情况下你会需要这个类，因为它设置程序的动画对象和属性很方便。然而，在某些情况下你可能需要 ValueAnimator ，直接原因是 ObjectAnimator 本身存在一定的局限性，例如要求目标对象上存在特定的访问方法。|
|[AnimatorSet](https://developer.android.com/reference/android/animation/AnimatorSet.html)|提供了将动画分组的机制以便他们互相运行。你能设置动画一起运行，顺序运行或延时运行。更多细节请查看 [Choreographing multiple animations with Animator Sets](https://developer.android.com/guide/topics/graphics/prop-animation#choreography)|

插值器告诉 paoperty animation system 如何为给定的属性计算值。
他们持有从 `Animator` 类得来的计时数据，动画开始和结束数据，基于这些计算属性的动画数据。
属性动画提供下面的求值器(evaluators):

**Table 2** Evaluators(求值器)

|Class/Interface| Description|
|-|-|
|[IntEvaluator](https://developer.android.com/reference/android/animation/IntEvaluator.html)   | The default evaluator to calculate values for int properties.|
|[FloatEvaluator](https://developer.android.com/reference/android/animation/FloatEvaluator.html) | The default evaluator to calculate values for float properties.|
|[ArgbEvaluator](https://developer.android.com/reference/android/animation/ArgbEvaluator.html)  | The default evaluator to calculate values for color properties that are represented as hexidecimal values.|
|[TypeEvaluator](https://developer.android.com/reference/android/animation/TypeEvaluator.html)  |一个允许创建自己求值器的接口。如果你正在操作的动画的属性不是以上三种类型，你必须实现本接口指定你对象的动画属性如何计算。你也可以为自己的 int, float, color重新自定义 TypeEvaluator 。关于此接口的更多信息： [Using a TypeEvaluator](https://developer.android.com/guide/topics/graphics/prop-animation#type-evaluator)|

一个时间插值器定义了在一个动画中特定值如何计算为时间的函数(_A time interpolator defines how specific values in an animation are calculated as a function of time._)。
例如你能指定整个动画以线性的方式发生，意味着在所有的时间都在执行动画；你也可以指定动画使用非线性的时间，例如在开始加速在结束减速。
表三描述了包含在 `android.view.animation` 中的插值器。
如果没有现存的插值器满足你的需求，实现 TimeInterpolator 接口并创建你自己的插值器吧。
关于如何创建自定义插值器，在 [Using interpolators](https://developer.android.com/guide/topics/graphics/prop-animation#interpolators) 你能找到更多细节。

**Table 3.** Interpolators(插值)

|Class/Interface                 | Description|
|-|-|
|AccelerateDecelerateInterpolator| An interpolator whose rate of change starts and ends slowly but accelerates through the middle.|
|AccelerateInterpolator          | An interpolator whose rate of change starts out slowly and then accelerates.|
|AnticipateInterpolator          | An interpolator whose change starts backward then flings forward.|
|AnticipateOvershootInterpolator | An interpolator whose change starts backward, flings forward and overshoots the target value, then finally goes back to the final value.|
|BounceInterpolator              | An interpolator whose change bounces at the end.|
|CycleInterpolator               | An interpolator whose animation repeats for a specified number of cycles.|
|DecelerateInterpolator          | An interpolator whose rate of change starts out quickly and then decelerates.|
|LinearInterpolator              | An interpolator whose rate of change is constant.|
|OvershootInterpolator           | An interpolator whose change flings forward and overshoots the last value then comes back.|
|TimeInterpolator                | An interface that allows you to implement your own interpolator.|


## 0x04、 Animate using ValueAnimator
`ValueAnimator` 类使你可以指定一系列的数据类型(int,float,color)的属性为其执行动画，并为动画设置某种类型。
你可以通过 `ofInt(), ofFloat(), ofObject()` 获取一个 `ValueAnimator` 类型的对象。

```java
ValueAnimator animation = ValueAnimator.ofFloat(0f, 100f);
animation.setDuration(1000);
animation.start();
```

上面的代码中当 `start()` 方法被调用后 `ValueAnimator` 开始在 1000ms 的时间段内计算从 0 到 100 的动画值。

你能指定自定义类型的动画：

```java
ValueAnimator animation = ValueAnimator.ofObject(new MyTypeEvaluator(), startPropertyValue, endPropertyValue);
animation.setDuration(1000);
animation.start();
```

在上面的代码中，当 `start()` 方法被调用后 `ValueAnimator` 类会在 100ms 的时间段内通过自己的动画逻辑计算从 `startPropertyValue` 到 `endPropertyValue` 的值。

你能通过添加 `AnimatorUpdateListener` 到 `ValueAnimator` 在动画值变化的是时候使用它：

```java
animation.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
    @Override
    public void onAnimationUpdate(ValueAnimator updatedAnimation) {
        // You can use the animated value in a property that uses the
        // same type as the animation. In this case, you can use the
        // float value in the translationX property.
        float animatedValue = (float)updatedAnimation.getAnimatedValue();
        textView.setTranslationX(animatedValue);
    }
});
```

在上面代码的 `onAnimationUapdate()` 方法你能访问并更新你所提供的 view 的动画属性的值。
关于这个 listener 的更多信息请移步 [Animation listeners](https://developer.android.com/guide/topics/graphics/prop-animation#listeners)


## 0x05、 Animate using ObjectAnimator
`ObjectAnimator` 类是上一节讨论的 `ValueAnimator` 类的子类，并且集成了计时引擎、`ValueAnimator` 的针对目标对象的动画属性值计算能力。
这使得为任何对象添加动画都变得更简单了，因为你不再需要实现 `ValueAnimator.AnimatorUpdateListener` 便可以完成动画属性的自动更新。

实例化一个 `ObjectAnimator` 比 `ValueAnimator` 更简便，但是你仍然需要指定动画对象、对象中的动画属性名称(通过 String)、以及动画属性值的变换区间：

```java
ObjectAnimator animation = ObjectAnimator.ofFloat(textView, "translationX", 100f);
animation.setDuration(1000);
animation.start();
```

要使 `ObjectAnimator` 得体地更新属性，你必须做下面的事情：

-   你所指定的动画属性必须有一个以驼峰命名法命名的 setter 方法 `set<PropertyName>()`。
    因为 `ObjectAnimator` 自动更新属性值的时候必须通过这个 setter 方法访问属性。
    例如，假设属性名为 `foo` 你就必须有一个 `setFoo()` 方法。
    如果这个 setter 方法不存在，你有三中选择：
    1.  如果你有权限的话到目标类中增加 setter 方法
    2.  将你无权限修改的类用自己的类包装起来，并且在自己的类中实现 setter 方法，最终间接达到修改目标类中目标属性的目的。
    3.  使用 `ValueAnimator` 替代

-   如果你在 `ObjectAnimator` 的工厂方法的 `value...` 参数只指定了一个值，它将被作为动画属性的结束边界被使用。
    因此动画对象的必须有动画属性的 getter 方法用于获取动画属性的当前值(作为动画属性的开始边界)。
    getter 方法必须符合 `get<PropertyName>()` 格式。
    例如，假设属性名为 `foo` 你的 getter 方法为 `getFoo()`。

-   getter(如果需要) 和 setter 方法所操作的动画起始值和结束值必须和你在 `ObjectAnimator` 中指定的数据类型相同。
    例如，假设你的 `ObjectAnimator` 是使用下面代码初始化的，那么你必须有 `targetObject.setPropName(float)` 和 `targetObject.getPropName(float)` 方法：
    ```java
    ObjectAnimator.ofFloat(targetObject, "propName", 1f)
    ```

-   这一点依赖于你的动画所设置的目标对象和属性，如果目标对象是一个 view 你可能需要在更新动画值之后强制重绘 view 。
    你应该在 `onAnimationUpdate()` 方法中做这件事。
    例如，假设动画属性是 Drawable 对象的 color ，那就需要在更新的时候重绘目标对象。
    View 中的所有 setter 方法(例如 `setAlpha()` 和 `setTranslationX()`) 会自动调用 invalidate ，所以你无需手动调用。
    有关于此的更多信息请查看 [Animation listeners](https://developer.android.com/guide/topics/graphics/prop-animation#listeners)

## 0x06、 Choreograph multiple animations using an AnimatorSet(使用 AnimatorSet 编排多个动画)
很多时候，你是否要运行一个动画的前提是另一个动画是否已经开始运行或结束运行了。
Android 允许你将多个动画一同打包进一个 Animator Set 从而你可以指定同时/顺序/延时运行动画。
AnimatorSet 可以相互叠加。

下面的代码片段以如下的几种方式运行 Animator 对象的动画：
1.  运行 `bounceAnim`
2.  同时运行 `squashAnim1`, `squashAnim2`, `stretchAnim1`, `stretchAnim2` 
3.  运行 `bounceBackAnim`
4.  运行 `fadeAnim`
```java
AnimatorSet bouncer = new AnimatorSet();
bouncer.play(bounceAnim).before(squashAnim1);
bouncer.play(squashAnim1).with(squashAnim2);
bouncer.play(squashAnim1).with(stretchAnim1);
bouncer.play(squashAnim1).with(stretchAnim2);
bouncer.play(bounceBackAnim).after(stretchAnim2);
ValueAnimator fadeAnim = ObjectAnimator.ofFloat(newBall, "alpha", 1f, 0f);
fadeAnim.setDuration(250);
AnimatorSet animatorSet = new AnimatorSet();
animatorSet.play(bouncer).before(fadeAnim);
animatorSet.start();
```

## 0x07、 Animation listeners
你能使用下面提到的监听者在动画执行期间监听并响应你关心的重要事件：

`Animator.AnimatorListener`
1.   `onAnimationStart()` - 动画启动时调用
2.   `onAnimationEnd()` - 动画结束时调用
3.   `onAnimationRepeat()` - 动画重复执行时调用
4.   `onAnimationCancel()` - 动画被取消时调用。 
    不论动画本身是如何结束的，动画被取消事件也会触发 `onAnimationEnd()`。

`ValueAnimator.AnimatorUpdateListener`

## 0x08、 Animate layout changes to ViewGroup objects(动画内容由布局更改为 ViewGroup 对象)
property animation system 具有将动画应用到 ViewGroup 对象也像将动画应用到 View 对象一样简单的能力。

你能通过 `LayoutTransition` 类为布局内的任何 ViewGroup 设置动画。
当你将 View 移入(出) ViewGroup 或者设置 View 的可见性(通过 `setVisibility()` 方法的 VISIBLE, INVISIBLE, GONE)的时候你可以使用淡入淡出动画。
要保持 View 在 ViewGroup 内部，你也能通过动画为其设置新位置和移动它。
你能通过在 `LayoutTransition` 添加如下动画，并且调用 `setAnimator()`  将 `LayoutTransition` 添加到 `Animator` 对象中：
*   `APPEARING` - 代表一个动画标签：一个条目出现在它的容器中
*   `CHANGE_APPEARING` - 代表一个动画标签：一个条目因为相同容器中新增了一个条目而发生的变化
*   `DISAPPEARING` - 代表一个动画标签：一个条目从它的容器中消失
*   `CHANGE_DISAPPEARING` - 代表一个动画标签：一个条目因为相同容器中减少了一个条目而发生的变化

你能为上述四种事件自定义动画从而达到你喜欢的视觉效果，或者告诉动画系统使用默认的动画。

`LayoutAnimations` API Demos 实例中展示了如何为 layout 变化定义动画，以及想 View 对象设置你喜欢的动画。

`LayoutAnimationsByDefault` 以及与其相关的 `layout_animations_by_default.xml` 布局资源文件像你展示了如何在 XML 文件中打开默认转换动画。
你只需要设置 `android:animateLayoutchanges` 属性为 `true` 。
```xml
<LinearLayout
    android:orientation="vertical"
    android:layout_width="wrap_content"
    android:layout_height="match_parent"
    android:id="@+id/verticalContainer"
    android:animateLayoutChanges="true" />
```

## 0x09、 Animate view state changes using StateListAnimator
`StateListAnimator` 类让你可以再 view 状态变化的时候定义动画。
这个对象的行为被包裹在 `Animator` 对象中，每当指定 view 状态(例如，persse 或 focuse)改变就会被调用。

`StateListAnimator` 能定义在 XML 资源文件中的 `<selector>` 根节点下，子节点 `<item>` 元素用于指定 view 在 `StateListAnimator` 中定义的不同的状态。
每一个 `<item>` 中包含了 [property animation set](https://developer.android.com/guide/topics/resources/animation-resource.html#Property)

例如，下面的文件创建了一个 state list animator ,用于在 view 被按下(perss)的时候修改 x 和 y 的 scale(比例/尺寸)。

res/xml/animate_scale.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- the pressed state; increase x and y size to 150% -->
    <item android:state_pressed="true">
        <set>
            <objectAnimator android:propertyName="scaleX"
                android:duration="@android:integer/config_shortAnimTime"
                android:valueTo="1.5"
                android:valueType="floatType"/>
            <objectAnimator android:propertyName="scaleY"
                android:duration="@android:integer/config_shortAnimTime"
                android:valueTo="1.5"
                android:valueType="floatType"/>
        </set>
    </item>
    <!-- the default, non-pressed state; set x and y size to 100% -->
    <item android:state_pressed="false">
        <set>
            <objectAnimator android:propertyName="scaleX"
                android:duration="@android:integer/config_shortAnimTime"
                android:valueTo="1"
                android:valueType="floatType"/>
            <objectAnimator android:propertyName="scaleY"
                android:duration="@android:integer/config_shortAnimTime"
                android:valueTo="1"
                android:valueType="floatType"/>
        </set>
    </item>
</selector>
```

要将这个 state list animator 附加到一个 view 上，需要为 view 设置 `android:stateListAnimator` 属性：
```xml
<Button android:stateListAnimator="@xml/animate_scale"
        ... />
```

当前动画定义在 `animate_scale.xml` 文件中，并且动画在 button 状态改变的时候触发。

也可以使用 java 代码(`AnimatorInflater.loadStateListAnimator()` 方法)声明一个 state list animator ,并且通过 `View.setStateListAnimator()` 将 `StateListAnimator` 附加到 View 上。

也可以使用 `AnimatedStateListDrawable` 为 view 定义一个 动画属性，你能在 view 状态发生变化的时候执行该动画。
在 Android 5.0 及更高版本上一些系统控件默认使用动画。
下面的代码展示了如何在 XML 资源中定义 `AnimateStateListDrawable` ：
```xml
<!-- res/drawable/myanimstatedrawable.xml -->
<animated-selector
    xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- provide a different drawable for each state-->
    <item android:id="@+id/pressed" android:drawable="@drawable/drawableP"
        android:state_pressed="true"/>
    <item android:id="@+id/focused" android:drawable="@drawable/drawableF"
        android:state_focused="true"/>
    <item android:id="@id/default"
        android:drawable="@drawable/drawableD"/>

    <!-- specify a transition -->
    <transition android:fromId="@+id/default" android:toId="@+id/pressed">
        <animation-list>
            <item android:duration="15" android:drawable="@drawable/dt1"/>
            <item android:duration="15" android:drawable="@drawable/dt2"/>
            ...
        </animation-list>
    </transition>
    ...
</animated-selector>
```


## 0x10、 Use a TypeEvaluator
如果你希望执行一种 Android 系统不知道的动画类型，你能通过实现 `TypeEvaluator` 接口创建自己的 evaluator 。
Android 系统已知的类型为 `int`，`float` 和 color ,分别对应的支持类为 `IntEvaluator` , `FloatEvaluator` 和 `ArgbEvaluator`。

`TypeEvaluator` 接口只有一个方法， `evaluate()` 。
这个方法使你正在使用的动画为当前属性返回一个合理的值。
下面演示了如何使用：
```java
public class FloatEvaluator implements TypeEvaluator {

    public Object evaluate(float fraction, Object startValue, Object endValue) {
        float startFloat = ((Number) startValue).floatValue();
        return startFloat + fraction * (((Number) endValue).floatValue() - startFloat);
    }
}
```

> **☆ Note：** 
    当 [ValueAnimator](https://developer.android.com/reference/android/animation/ValueAnimator.html)(或则 [ObjectAnimator](https://developer.android.com/reference/android/animation/ObjectAnimator.html)) 运行的时候，会计算一个已经流逝的动画分数(0~1之间的值)，并且使用你当前的插值器计算出在当前分数下的值。
    你的 `TypeEvaluator` 通过 fraction 参数接收插值分数，所以在计算动画值的时候你不必记录记录动画属性值。

## 0x11、 Use Interpolators
一个插值器定义了如何根据当前时间计算动画属性的当前值的功能。
例如你能指定整个动画呈线性状态发生这意味着动画在整个时间段内匀速移动，或者你能指定动画使用非线性时间，例如在开始或结束的时候加速或减速。

插值器在动画系统中，从 Animators 接收已经流逝的时间分数。
插值器根据当前的动画类型修改这个分数。
Android 系统在 `android.view.animation` 包中提供了一系列常见的插值器。
如果没有满足你需要的，你能实现 `TimeInterpolator` 接口创建自己的插值器。

一个关于如何定义插值器 `AccelerateDecelerateInterpolator` 以及 `LinearInterpolator` 计算插值分数的对比的例子。
`LinearInterpolator` 对流逝的时间分数没有影响。
`AccelerateDecelerateInterpolator` 在动画开始的时候加速结束的时候减速。
下面的方法定义了上述逻辑：

**AccelerateDecelerateInterpolator**
```java
@Override
public float getInterpolation(float input) {
    return (float)(Math.cos((input + 1) * Math.PI) / 2.0f) + 0.5f;
}
```

**LinearInterpolator**
```java
@Override
public float getInterpolation(float input) {
    return input;
}
```

下表表示这些插值器为持续1000ms的动画计算的近似值：

|ms elapsed	|Elapsed fraction/Interpolated fraction (Linear) |	Interpolated fraction (Accelerate/Decelerate)|
|-|-|-|
|0          | 0        |  0     |     
|200        | .2       |  1     |     
|400        | .4       |  345   |         
|600        | .6       |  8     |     
|800        | .8       |  9     |     
|1000       | 1        |  1     |      

如表所示， `LinearInterpolator` 列中的数值变化速度均匀。
`AccelerateDecelerateInterpolator` 列中的数据变化在 200ms～600ms 速度大于 `LinearInterpolator` 的值，在 600ms～1000ms 的速度小于 `LinearInterpolator` 。

## 0x12、 Specify keyframes(指定关键帧)
`Keyframe` 对象包含 time/value 对，这些成对的数据让你能在动画特定的事件指定动画的状态。
每一个 keyframe 都能有它自己的插值器，用于控制与上一个 keyframe 的间隔和本 keyframe 的时间。

要实例化一个 `Kyeframe` ，你必须使用一个工厂方法 `ofInt()`, `ofFloat()` 或者 `ofObject()` 去获取恰当类型的 Keyframe 。
然后调用 `ofKeyframe()` 工厂方法获取一个 `PropertyValuesHolder` 对象。
一旦你有了这个对象，你就能用上述两个对象获得一个 `Animator` 。
```java
Keyframe kf0 = Keyframe.ofFloat(0f, 0f);
Keyframe kf1 = Keyframe.ofFloat(.5f, 360f);
Keyframe kf2 = Keyframe.ofFloat(1f, 0f);
PropertyValuesHolder pvhRotation = PropertyValuesHolder.ofKeyframe("rotation", kf0, kf1, kf2);
ObjectAnimator rotationAnim = ObjectAnimator.ofPropertyValuesHolder(target, pvhRotation);
rotationAnim.setDuration(5000);
```

## 0x13、 Animate views
属性动画系统允许 View 应用对象流线动画，并且提供了一些优于 view 动画系统的优势。
视图动画系统通过更改它们的绘制方式来转换View对象。 
这是在每个View的容器中处理的，因为View本身没有可操作的属性。
这么做会使 View 发生动画，但是 View 本身没有任何变化。
这导致行为:例如，对象依然存在于原来的位置,甚至即使在屏幕其他位置绘制也无法改变 View 位置属性。
在 Android 3.0 ，你的属性和坐标 getter 、setter 方法被增加用于消除(eliminate)这些缺点。

属性动画系统能在使 View 执行动画的时候真正的改变 View 的属性值。
此外，每当属性被更新的时候 View 也自动调用 `invalidate()` 方法更新在屏幕上的显示。
View 中这些新的属性对属性动画起到促进作用：
-   **translationX and translationY**: 这写属性作为 view 在父容器中的 X/Y 坐标的增量用于设置 View 在父容器中的位置。
-   **rotation, rotationX, and rotationY**: 这些属性用于控 2D/3D 动画中发生动画的中心点。
-   **scaleX and scaleY**: 这些动画用于控制发生 2D 缩放动画时候的动画中心点。
-   **pivotX and pivotY**: 这些属性用于控制中心点包括 2D 缩放动画或者 2D/3D 旋转动画。默认的中心点是控件的中心。
-   **x and y**: 这些属性的作用(utility)是描述 view 在其容器中的坐标，这个坐标是 left / top 的值与 translationX / translationY 的和
-   **alpha**: 代表 view 的 alpha 透明度。这个值默认是 1 (不透明)，如果是 0 表示完全透明(不可见)。

View 的动画属性(例如 color 或者 rotation 的值),或者你希望执行动画的属性，如下例：
```java
ObjectAnimator.ofFloat(myView, "rotation", 0f, 360f);
```

关于此的更多信息请查阅 [ValueAnimator](https://developer.android.com/guide/topics/graphics/prop-animation#value-animator) 和 [ObjectAnimator](https://developer.android.com/guide/topics/graphics/prop-animation#object-animator)


### 13.1、 Animate using ViewPropertyAnimator
[`ViewPropertyAnimator`](https://developer.android.com/reference/android/view/ViewPropertyAnimator.html) 提供了一种简单的方式在只使用一个 `Animator` 对象的前提下为同一个 View 同时执行多个动画。
它的行为和 `ObjectAnimator` 非常像，因为它真的会修改 View 的动画属性，但是它更多用在在 view 有多个动画属性的场景下。
另外，它的代码更简洁、明了、易读。
下面的代码皮阿奴单展示了二者的不同：

**Multiple ObjectAnimator objects**
```java
ObjectAnimator animX = ObjectAnimator.ofFloat(myView, "x", 50f);
ObjectAnimator animY = ObjectAnimator.ofFloat(myView, "y", 100f);
AnimatorSet animSetXY = new AnimatorSet();
animSetXY.playTogether(animX, animY);
animSetXY.start();
```

**One ObjectAnimator**
```java
PropertyValuesHolder pvhX = PropertyValuesHolder.ofFloat("x", 50f);
PropertyValuesHolder pvhY = PropertyValuesHolder.ofFloat("y", 100f);
ObjectAnimator.ofPropertyValuesHolder(myView, pvhX, pvhY).start();
```

**ViewPropertyAnimator**
```java
myView.animate().x(50f).y(100f);
```

要了解关于 `ViewPropertyAnimator` 的更多信息可以查看 [blog post](http://android-developers.blogspot.com/2011/05/introducing-viewpropertyanimator.html)


## 0x14、 Declare animations in XML
property animation system 允许你使用编程的方式和 XML 资源文件的方式声明动画。
在 XML 定义的动画，能更容易的复用和修改。

为了在旧的 [view animation framework](https://developer.android.com/guide/topics/graphics/view-animation.html) 存在的情况下区分出正在动画文件是新的属性动画的 API ，从 Android 3.1 开始，应该将 property animations 的 XML 文件保存到 `res/animator/` 文件夹 。

下面的属性动画类分别声明了受支持的 XML 标签：
-   `ValueAnimator` - ```<animator>```
-   `ObjectAnimator` - ```<objectAnimator>```
-   `AnimatorSet` - ```<set>```

要了解在 XML 文件中的所有支持的属性，请查阅 [Aniamtion resources](https://developer.android.com/guide/topics/resources/animation-resource.html#Property)。
下面的例子展示了两个动画对象(set)的顺序,将两个对象嵌套在同一个 set 节点内。
```xml
<set android:ordering="sequentially">
    <set>
        <objectAnimator
            android:propertyName="x"
            android:duration="500"
            android:valueTo="400"
            android:valueType="intType"/>
        <objectAnimator
            android:propertyName="y"
            android:duration="500"
            android:valueTo="300"
            android:valueType="intType"/>
    </set>
    <objectAnimator
        android:propertyName="alpha"
        android:duration="500"
        android:valueTo="1f"/>
</set>
```

为了运行这个动画，你必须加载这个 XML 资源并通过它生成一个 `AnimatorSet` 对象，然后为动画设置对象。
调用 `setTraget()` 设置唯一的一个动画对象与 `AnimatorSet`(和其中的所有 `Animator` 结合)) 方便的结合。
```java
AnimatorSet set = (AnimatorSet) AnimatorInflater.loadAnimator(myContext,
    R.animator.property_animator);
set.setTarget(myObject);
set.start();
```

你也可以在 XML 文件中声明 `valueAnimator` ：
```xml
<animator xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="1000"
    android:valueType="floatType"
    android:valueFrom="0f"
    android:valueTo="-100f" />
```

要在你的代码中使用这个 `ValueAnimator` ，你必须加载这个对象并且添加一个 `AnimatorUpdateListener` 用于更新动画值，定将新值应用与你的 view ：
```java
ValueAnimator xmlAnimator = (ValueAnimator) AnimatorInflater.loadAnimator(this,
        R.animator.animator);
xmlAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
    @Override
    public void onAnimationUpdate(ValueAnimator updatedAnimation) {
        float animatedValue = (float)updatedAnimation.getAnimatedValue();
        textView.setTranslationX(animatedValue);
    }
});

xmlAnimator.start();
```

有关于此的更多信息请查阅： [Animation resources](https://developer.android.com/guide/topics/resources/animation-resource.html#Property)


## 0x15、 Potential effects on UI performance(影响 UI 效果的潜在因素)
正在执行的动画的每一帧都需要额外的渲染。
因为这个原因，太过精细的动画会影响你 App 的性能。

动画UI所需的工作将添加到渲染管道的[动画阶段](https://developer.android.com/topic/performance/rendering/profile-gpu.html#at)。
通过打开 **Profile GPU Rendering** 和监控动画阶段，你能了解到动画是否影响到你 APP 的性能。
更多信息请查阅： [Profile GPU rendering walkthrough](https://developer.android.com/studio/profile/dev-options-rendering.html) 。