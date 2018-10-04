
## 0x00、 绘制概览
原文：https://developer.android.com/guide/topics/graphics/drawables

当你需要在你的 app 中展示静态图片的时候，你能使用 Drawable 类和它的子类绘制形状和图片。
Drawable 是可绘制内容的抽象表示。
它的子类帮助指定图片场景，你能继承它去定义你自己的具有独特行为方式的 drawable 对象。

除了使用构造方法外还有两种方式定义和实例化 Drawable ：
- 加载项目中保存的 image 资源。
- 加载可以被绘制的 XML 资源。

> **☆NOTE：** 
    你可能更倾向于通过定义一系列的点、线、曲线、颜色来替代向量图像。
    这种方式在图像尺寸变化的时候不会失真。
    关于此的更多信息请查阅：[Vector drawables overvies](https://developer.android.com/guide/topics/graphics/vector-drawable-resources.html)

## 0x01、 从资源文件中创建图像
你能通过对项目中 image 资源的引用在 app 中添加图像。
这种方式支持的图片类型包括 PNG(首选)，JPG(次选)，和 GIF(不推荐)。
App 的图标，logo 和其他图片，例如游戏中使用的图片，使用这中技术是合适的。

要使用一个图片资源首先需要将其添加到你项目的 `res/drawable` 文件中。
一旦添加到你的项目中，你就能在 .java 和 .xml 文件中使用它了。
使用的方式就是通过携带文件类型的文件名字 ID 。
例如，`my_image.png` 在 ID 中的名字为 `my_image` 。

>**☆NOTE：** 
    位于 `res/drawable` 文件夹中的图片资源在编译阶段可能会被 **aapt** 工具进行无损压缩。
    例如，a true-color PNG that doesn't require more than 256 colors may be converted to an 8-bit PNG with a color palette。这种处理方式能在获取同等质量图片的前提下减少图片的内存占用量。
    正因为这种操作和处理，项目中的图片文件在编译阶段会被替换。
    如果你打算通过二进制六的方式从问佳佳中读取图片，你需要将这些待读取的资源放到 `res/raw/` 文件夹下，aapt 不会触及这里的文件。

下面的代码片段展示了如何通过 drawable 资源构建一个 ImageView 并且将该控件添加到布局中：
```java
LinearLayout mLinearLayout;

protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);

  // Create a LinearLayout in which to add the ImageView
  mLinearLayout = new LinearLayout(this);

  // Instantiate an ImageView and define its properties
  ImageView i = new ImageView(this);
  i.setImageResource(R.drawable.my_image);

  // set the ImageView bounds to match the Drawable's dimensions
  i.setAdjustViewBounds(true);
  i.setLayoutParams(new Gallery.LayoutParams(LayoutParams.WRAP_CONTENT,
      LayoutParams.WRAP_CONTENT));

  // Add the ImageView to the layout and set the layout as the content view
  mLinearLayout.addView(i);
  setContentView(mLinearLayout);
}
```

在一些情况下，你可能需要通过一个 Drawable 对象持有你的图像资源：
```java
Resources res = mContext.getResources();
Drawable myImage = ResourcesCompat.getDrawable(res, R.drawable.my_image, null);
```

>**！NOTE：** 
    不论在你的项目中针对一个资源有创建了多少的不同的实例，他们最终都是指向相同的内存控件的。
    例如，如果你使用同一个 image resources 实例化了两个 Drawable ，修改其中一个对象的数据(例如透明度)，另一个对象也会受到相同的作用效果。
    当处理一个 image 资源的多个对象的时候，应该通过 [tween animation](https://developer.android.com/guide/topics/graphics/view-animation.html#tween-animation) 代替对图片的直接操作。

下面代码片段展示了，如何 在 XML 布局文件中向 ImageView 添加一个 drawable 资源：
```xml
<ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@drawable/my_image" />
```
有关于使用项目中的资源的更多信息请查阅：[Resources and assets](https://developer.android.com/guide/topics/resources/index.html)

>**☆NOTE：**
    当你在项目中使用 image 资源的时候请注意为不同的像素密度准备不同尺寸的 image 资源。
    如果没有匹配当前像素密度的资源，将会缩放其他可用的资源，这可能造成较差的效果。
    有关于此的更多信息请参阅：[Support defferent pixel densities](https://developer.android.com/training/multiscreen/screendensities.html)。

## 0x02、 在 XML 中创建 drawable
如果你想创建一个 Drawable 对象，并且不希望通过代码在用户交互过程中操作，在 XML 中定义 `Drawable` 是一个不错的选择。
即使你希望在用户交互过程中改变 Drawable 的属性，你也可以考虑通过 XML 定义 Drawable 对象。

当你在 XML 中定义了 Drawable 后，将该定义文件保存在 `res/drawable/` 中。
下面的例子展示了定义一个 TransitionDrawable(Drawable子类) 资源：
```xml
<!-- res/drawable/expand_collapse.xml -->
<transition xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@drawable/image_expand">
    <item android:drawable="@drawable/image_collapse">
</transition>
```

在定义之后，通过调用 Resources.getDrawable() 的方式解析资源 ID 所指定的 XML 文件。
任何 Drawable 子类都支持 inflate() 方法，都可以通过 XML 定义和实例化。
每一个支持利用 XML 的 Drawable 类都支持特定的 XML 属性，帮助定义对象属性。
下面的代码实例化了 TransitionDrawable 并且设置它作为 ImageView 的内容：
```java
Resources res = mContext.getResources();
TransitionDrawable transition =
    (TransitionDrawable) ResourcesCompat.getDrawable(res, R.drawable.expand_collapse, null);

ImageView image = (ImageView) findViewById(R.id.toggle_image);
image.setImageDrawable(transition);

// Then you can call the TransitionDrawable object's methods
transition.startTransition(1000);
```

## 0x03、 形状绘制
当你想绘制二维图像的时候 ShapeDrawable 是一个不错的选择。
你能通过编程的方式在 ShapeDrawable 绘制原始的二维图像，并且能为其应用你的 APP 特有的风格。

ShapeDrawable 是 Drawable 的子类。
因此在任何可以要求 Drawable 的地方，你都可以时候用 ShareDrawable 。
例如，你能通过 View 的 setBackGroundDrawable() 方法为 View 设置 ShapeDrawable 对象。
你也能在自定义空间中绘制自己的形状，并且将其添加到自己的 APP 。

因为 ShapeDrawable 有它自己的 draw() 方法，你能创建一个 View 的子类，并且在 onDraw() 方法中调用它，看一个例子：
```java
public class CustomDrawableView extends View {
  private ShapeDrawable mDrawable;

  public CustomDrawableView(Context context) {
    super(context);

    int x = 10;
    int y = 10;
    int width = 300;
    int height = 50;

    mDrawable = new ShapeDrawable(new OvalShape());
    // If the color isn't set, the shape uses black as the default.
    mDrawable.getPaint().setColor(0xff74AC23);
    // If the bounds aren't set, the shape can't be drawn.
    mDrawable.setBounds(x, y, x + width, y + height);
  }

  protected void onDraw(Canvas canvas) {
    mDrawable.draw(canvas);
  }
}
```

你能在任何能够使用自定义控件的地方使用上面 CustomDrawableView 类。
例如你能以编程的方式通过将其添加到你的 Activity 中。
```java
CustomDrawableView mCustomDrawableView;

protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);
  mCustomDrawableView = new CustomDrawableView(this);

  setContentView(mCustomDrawableView);
}
```

如果你希望通过布局文件使用上述自定义view ，你必须在 CustomDrawableView 中定义双参数构造方法 `View(Context,AttributeSet)`，供 inflate 调用。
下面展示了如何在 XML 文件中声明 CustomDrawableView ：
```xml
<com.example.shapedrawable.CustomDrawableView
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        />
```

ShapeDrawable 类和其他位于 `android.graphics.drawable` 包的  Drawable 类一样。允许你通过公共方法定义属性。
包括那些你可能会调整到属性如：alpha，transparency，color filter,dither,opacity,color。

你也能通过 XML 资源定义个人的形状。
有关与此得到更多信息请阅读 [Drawable resource type](https://developer.android.com/guide/topics/resources/drawable-resource.html)中的[Shape Drawable](https://developer.android.com/guide/topics/resources/drawable-resource.html#Shape)


## 0x04、 NinePatch Drawables
NinePatchDrawable 图像是一个可伸展位图，你可以用它作为 view 的背景。
Android 根据容纳它的 View 自动调整其大小。
一个使用 NinePatch 图像的例子是 Android 标准 Button，它就根据 Button 中的文字长度进行伸展。
NinePathc 图像是一个标准的 PNG 图像，它包含一个1像素长度的边。
它必须以 `9.png` 为后缀保存在你项目的 `res/drawalbe/` 文件夹中。

使用这个一像素的边定义图片中可伸展和不可伸展的区域。
你通过在左边和上边绘制一像素(或者更长)的黑边(绘制在那一像素长度的边上)表名你希望拉伸的区域(未被绘制黑色的其他部分应该保持透明或者白色)。
你能按照自己的心意设定多个可伸展的区域。
伸展区域的相对大小是相同的，所以大地方还是大小的还是小。

在 image 上有一个可选的部分(有效填充线)，通过在右边和底边绘制黑线标示它。
如果 View 设置了 NinePathic 图像作为背景并且指定了 view 的 text，图片将扩展它自己确保文字内容仅在右边和底边的线所重叠的区域。
如果 padding line 没有被定义，Android 使用左边和上边线定义图像区域。

澄清：
    左边和上边的线，定义 image 中那些像素可以被复制，从而完成图像拉伸。
    底边和右边的线，定义 view 中的内容可以占据 image 的相对区域。

图一展示了用于定义 button 的 NinePatch 图像：
-   ![Figure 1: Example of a NinePatch graphic that defines a button](/Android_Dev/GUIDES/Images/2018-10-01-ninepatch_raw.png)

这个 NinePatch 图像通过左边和上边线定义了一个可拉伸的区域，并且绘制区域受限于下边和右边线。
在上面的图像中，灰色虚线部分指定了被用于复制从而拉伸图片的区域。
粉色长方形定义了内容所将占据的区域。
如果内容和这个区域不匹配，会通过拉伸图片修正它。

[Draw 9-patch](https://developer.android.com/tools/help/draw9patch.html)工具通通使用 WYSIWYG 图像编辑器提供了一个非常方便的方式去创建 NinePatch image。
他甚至提供了恰当的警告，如果你定义的可拉伸区域处于风险区域，当像素复制的时候会发出警告。

下面的代码展示了如何向 Button 添加 NitePatch 图片。
图片保存在 `/res/drawable/my_button_background.9.png`:
```xml
<Button id="@+id/tiny"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentTop="true"
        android:layout_centerInParent="true"
        android:text="Tiny"
        android:textSize="8sp"
        android:background="@drawable/my_button_background"/>

<Button id="@+id/big"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_centerInParent="true"
        android:text="Biiiiiiig text!"
        android:textSize="30sp"
        android:background="@drawable/my_button_background"/>
```

注意 `layout_width` 和 `layout_height` 属性被设置的值为 `wrap_content` ,确保按钮包裹文本。

图二展示了上面两个 XML 节点所设置的 Button。
注意 button 的 width 和 height ，以及背景图片是如何通过拉伸来容纳文字内容的。
-   ![Figure 2: Buttons rendered using an XML resource and a NinePatch graphic](/Android_Dev/GUIDES/Images/2018-10-01ninepatch_examples.png)

## 0x05、 自定义 Drawable
当你想创建自定义图像的时候，你能通过继承 Drawable(或任何它的子类) 完成它。

最重要的是要实现 draw(Canvas) 方法，因为它提供了 Canvas 对象，你必须通过它绘制你的内容。

下面展示了绘制圆形的 Drawable：
```java
public class MyDrawable extends Drawable {
    private final Paint mRedPaint;

    public MyDrawable() {
        // Set up color and text size
        mRedPaint = new Paint();
        mRedPaint.setARGB(255, 255, 0, 0);
    }

    @Override
    public void draw(Canvas canvas) {
        // Get the drawable's bounds
        int width = getBounds().width();
        int height = getBounds().height();
        float radius = Math.min(width, height) / 2;

        // Draw a red circle in the center
        canvas.drawCircle(width/2, height/2, radius, mRedPaint);
    }

    @Override
    public void setAlpha(int alpha) {
        // This method is required
    }

    @Override
    public void setColorFilter(ColorFilter colorFilter) {
        // This method is required
    }

    @Override
    public int getOpacity() {
        // Must be PixelFormat.UNKNOWN, TRANSLUCENT, TRANSPARENT, or OPAQUE
        return PixelFormat.OPAQUE;
    }
}
```

然后你能将 drawable 添加到 ImageView 进行展示。
```java
MyDrawable mydrawing = new MyDrawable();
ImageView image = findViewById(R.id.imageView);
image.setImageDrawable(mydrawing);
```

在 Android 7.0(API level 24) 和更高的版本上，你也能通过使用 XML 文件定义自定义 drawable 实例：

- 使用类的完全限定名作为元素的名字。
    通过这种方式，自定义 drawable 必须是 public 的顶级类(没有子类)：
    ```xml
    <com.myapp.MyDrawable xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="#ffff0000" />
    ```

-   使用 drawable 作为 XML 标签名字并且在属性中指定类的完全限定名。
    这种方式可用于 public 顶级类和 public static 内部类。
    ```xml
    <drawable xmlns:android="http://schemas.android.com/apk/res/android"
    class="com.myapp.MyTopLevelClass$MyDrawable"
    android:color="#ffff0000" />
    ```

## 0x06、 为 Drawable 着色
在 Android 5.0(API levle 21) 和更高版本上，你能使用 ALPHA 标记为 bitmaps 和 nine-patches 着色。
你能使用 color 资源或者包含颜色资源的主题属性(例如，`?android:attr/colorPrimary`)为 Drawable 着色。
通常你只需要创建这些资源一次他们的颜色就会自动匹配你的主题。

你能为 BitmapDrawable，NinePatchDrawable或者 VectorDrawable 对象通过 `setTint()` 应用一个 tint 。
你也能在 layout 文件中 通过 `android:tint` 和 `android:tintMode` 设置 tint 颜色和模式。

## 0x07、 从图像中提取突出的颜色
Android Support Library 包含 Plattte 类，它能从图像中提取突出的颜色。
你能将图像资源加载到 Bitmap 中然后通过 Palette 解析资源中的颜色。
有关于此的更多信息请阅读：`Selecting color with the Palette API`。