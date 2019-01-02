## 0x00、 [AnimatedVectorDrawable](https://developer.android.com/reference/android/graphics/drawable/AnimatedVectorDrawable)

本类是使用 `ObjectAnimator` 或 `AnimatorSet` 为 `VectorDrawable` 执行属性动画。

从 API 25 开始 AnimatedVectorDrawable 运行在 RenderThread(与早起运行在 UI 线程不同)。
这意味着，当 UI 线程执行很艰巨的任务的时候，动画依然能够保持顺滑。
注意： 如果 UI 线程失去了响应， RednerThread 可能会一直执行动画直到 UI 线程能够切换新帧。
因此，不可能精确地协调启用 RenderThread 的 AmatitedVectorDrawable 与 UI 线程动画。
另外，当 AnimatedVectorDrawable 在 RenderThread 完成最后一帧的时候 `Animatable2.AnimationCallback.onAnimationEnd(Drawable)` 会被调用。

AnimatedVectorDrawable 能被定义在三个分散的 XML 文件，也能只定义在一个 XML 文件中。

### 0.1、 Define an AnimatedVectorDrawable in three separate XML files

**XML for the VectorDrawable containing properties to be animated**

动画在 VectorDrawable 中的动画属性执行。
这些属性的动画通过 `ObjectAnimator` 执行。
`ObjectAnimator` 的 target 可以是 root 元素， group 元素或者 path 元素。
VectorDrawable 中的目标元素必须有唯一的名字。
没有动画的元素可以不命名。

`VectorDrawable` 的所有属性如下：

|Element Name |     Animatable attribute name   |
|-|-|
|`<vector>`	  |     alpha                       |
|`<group>`	  |     rotation                    |
|             |     pivotX                      |
|             |     pivotY                      |
|             |     scaleX                      |
|             |     scaleY                      |
|             |     translateX                  |
|             |     translateY                  |
|`<path>`	  |     pathData                    |
|             |     fillColor                   |
|             |     strokeColor                 |
|             |     strokeWidth                 |
|             |     strokeAlpha                 |
|             |     fillAlpha                   |
|             |     trimPathStart               |
|             |     trimPathEnd                 |
|             |     trimPathOffset              |
|`<clip-path>`|     pathData                    |

下面的例子，一个 VectorDrawable 定义在 vectordrawable.xml 。
这个 VectorDrawable 通过文件名(不包含文件后缀)被下文(XML for AnimatedVectorDrawable)引用 。

```xml
 <vector xmlns:android="http://schemas.android.com/apk/res/android"
     android:height="64dp"
     android:width="64dp"
     android:viewportHeight="600"
     android:viewportWidth="600" >
     <group
         android:name="rotationGroup"
         android:pivotX="300.0"
         android:pivotY="300.0"
         android:rotation="45.0" >
         <path
             android:name="v"
             android:fillColor="#000000"
             android:pathData="M300,70 l 0,-70 70,70 0,0 -70,70z" />
     </group>
 </vector>
```

**XML for AnimatedVectorDrawable**

一个 `AnimatedVectorDrawable` 元素有一个 `VectorDrawable` 属性，以及一个或多个目标元素。
目标元素能通过 `android:name` 属性指定目标，还能通过 `android:animation` 属性链接恰当的 `ObjectAnimator` 或者 `AnimatorSet` 。

下面代码简单定义了一个 `AnimatedVectorDrawable` 。
注意 name 引用自上文的例子(XML for the VectorDrawable containing properties to be animated)。

```xml
 <animated-vector xmlns:android="http://schemas.android.com/apk/res/android"
     android:drawable="@drawable/vectordrawable" >
     <target
         android:name="rotationGroup"
         android:animation="@animator/rotation" />
     <target
         android:name="v"
         android:animation="@animator/path_morph" />
 </animated-vector>
```

**XML for Animations defined using ObjectAnimator or AnimatorSet**

`XML for AnimatedVectorDrawable` 一节中用到了两个动画 `rotation.xml` 和 `path_morph.xml` 。

`rotation.xml` 用 6000ms 的事件将目标 group 从 0° 旋转到 360° 。
```xml
 <objectAnimator
     android:duration="6000"
     android:propertyName="rotation"
     android:valueFrom="0"
     android:valueTo="360" />
```

`path_morph.xml` 将 path 从一个形状变换到另一个。
注意，path 必须兼容变化。
明确一下， path 必须有相同的命令，相同的顺序，并且每一个命令必须有相同数量的参数。
通常将 path 作为 string 资源以便重用。
```xml
 <set xmlns:android="http://schemas.android.com/apk/res/android">
     <objectAnimator
         android:duration="3000"
         android:propertyName="pathData"
         android:valueFrom="M300,70 l 0,-70 70,70 0,0 -70,70z"
         android:valueTo="M300,70 l 0,-70 70,0  0,140 -70,0 z"
         android:valueType="pathType"/>
 </set>
```

### 0.2、 Define an AnimatedVectorDrawable all in one XML file

因为 AAPT 工具支持将多个相关 XML 文件放在一个的新格式，我们能合并上面例子中的 XML 到一个 XML 文件中：
```xml
 <animated-vector xmlns:android="http://schemas.android.com/apk/res/android"
                  xmlns:aapt="http://schemas.android.com/aapt" >
     <aapt:attr name="android:drawable">
         <vector
             android:height="64dp"
             android:width="64dp"
             android:viewportHeight="600"
             android:viewportWidth="600" >
             <group
                 android:name="rotationGroup"
                 android:pivotX="300.0"
                 android:pivotY="300.0"
                 android:rotation="45.0" >
                 <path
                     android:name="v"
                     android:fillColor="#000000"
                     android:pathData="M300,70 l 0,-70 70,70 0,0 -70,70z" />
             </group>
         </vector>
     </aapt:attr>

     <target android:name="rotationGroup"> *
         <aapt:attr name="android:animation">
             <objectAnimator
             android:duration="6000"
             android:propertyName="rotation"
             android:valueFrom="0"
             android:valueTo="360" />
         </aapt:attr>
     </target>

     <target android:name="v" >
         <aapt:attr name="android:animation">
             <set>
                 <objectAnimator
                     android:duration="3000"
                     android:propertyName="pathData"
                     android:valueFrom="M300,70 l 0,-70 70,70 0,0 -70,70z"
                     android:valueTo="M300,70 l 0,-70 70,0  0,140 -70,0 z"
                     android:valueType="pathType"/>
             </set>
         </aapt:attr>
      </target>
 </animated-vector>
```