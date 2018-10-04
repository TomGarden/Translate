## 0x00、 高效加载大图片

原文：https://developer.android.com/topic/performance/graphics/load-bitmap

>**☆NOTE:** 
    一些使用最佳实践方式加载图片的库。
    你能在你的 app 中使用这些库用最优的方式加载图片。
    我们推荐快速流畅加载图片的 [Glide](https://github.com/bumptech/glide) 库。
    其他流行的图像加载库还有来自 Facebook 的 Square 和 [Fresco](https://github.com/facebook/fresco) 的 [Picasso](http://square.github.io/picasso/)。
    这些库简化了 Android 平台上有关图像的复杂操作。

图像有多种形状和尺寸。
在多数情况下他们的尺寸大于展示它们的 UI 。
例如系统应用 Gallery(画廊) 展示照相机拍摄的照片的时候，照片解析度是普遍高于屏幕密度的。

在内存受限制的情况下，比较理想的做法是加载低解析度的图片到内存。
低解析度的版本的尺寸应该和展示它的 UI 的尺寸相匹配。
具有更高分辨率的图像不会提供任何明显的好处，但仍会占用宝贵的内存并因额外的动态扩展而产生额外的性能开销。

本篇文字介绍了：在不超过 app 内存限制的情况下通过在内存中加载大图片的缩略图，的方式展示图片。

## 0x01、 读取 Bitmap 的尺寸和类型。
BitmapFactory 为创建 Bitmap 对象提供了一系列解码方法(decodeByteArray(),decodeFile(),decodeResource()等)。
选择适当的方法解码你的图像资源。
这些方法尝试在构造位图的时候会尝试为其分配内存空间，这时候容易发阿生 OutOfMemory 异常。
每一种解码方式都能通过增加标记的方式指定解码选项变量 BitmapFactory.Options 类对象。
设置 inJestDecodeBounds 属性为 true 可以避免解码图片过程中的内存分配，这样解码方法将会 return null ，但是 BitmapFactory.Options 对象中的 outWidth， outHeight， 和 outMimeType 却被设置了值。

```java
BitmapFactory.Options options = new BitmapFactory.Options();
options.inJustDecodeBounds = true;
BitmapFactory.decodeResource(getResources(), R.id.myimage, options);
int imageHeight = options.outHeight;
int imageWidth = options.outWidth;
String imageType = options.outMimeType;
```

要避免 `java.java.OutOfMemory` 异常，可在真正解码图片之前检查 bitmap 的尺寸，除非你绝对信任提供的携带可预见尺寸的图像数据资源，否则就要修正资源使打到内存可接受的程度。

## 0x02、 加载所放过的图片到内存中
现在待加载图片的尺寸是已知的，可以通过它决定是应该将其全部加载到内存还是加载其缩略图。
这里是一些做这个决定所需要考虑的因素：
-   预计加载整个图像需要占用的内存空间。
-   其他通过你的 App 获取图片的时候，留意他们想要的图片所占的内存大小。
-   目标 ImageView 或者其他 UI 展示图片的控件的尺寸。
-   当前设备的屏幕大小和像素密度。

例如，为大小为 128×96pix 的 ImageView 加载 1024×768pix 的图片是没有价值的。

要将一个图片解码为更小的版本，需要将你的 BitmapFactory.Option 的 inSampleSize 设置为 true 。
例如，一个解析度为 2048×1536 的图片要将其解析为原图的 1/4 约为 512x384(假设位图的配置为 ARGB_8888) 。这时候内存的占用量将会是 0.75MB 而不是 2MB 。
下面是一个基于宽度和高度计算图片缩放比率的方法：
```java
public static int calculateInSampleSize(
            BitmapFactory.Options options, int reqWidth, int reqHeight) {
    // Raw height and width of image
    final int height = options.outHeight;
    final int width = options.outWidth;
    int inSampleSize = 1;

    if (height > reqHeight || width > reqWidth) {

        final int halfHeight = height / 2;
        final int halfWidth = width / 2;

        // Calculate the largest inSampleSize value that is a power of 2 and keeps both
        // height and width larger than the requested height and width.
        while ((halfHeight / inSampleSize) >= reqHeight
                && (halfWidth / inSampleSize) >= reqWidth) {
            inSampleSize *= 2;
        }
    }

    return inSampleSize;
}
```

>**☆NOTE：**
    使用 2 作为因子是因为解码器使用最终值通过舍入得到最接近的 2 的幂。

使用这个方法，首次解码的时候 inJustDecodeBounds 设置为 true ，获取 options 值，然后使用心得 inSampleSize 对图片进行解码的时候 inJustDecodeBound 设置为 false ：
```java
public static Bitmap decodeSampledBitmapFromResource(Resources res, int resId,
        int reqWidth, int reqHeight) {

    // First decode with inJustDecodeBounds=true to check dimensions
    final BitmapFactory.Options options = new BitmapFactory.Options();
    options.inJustDecodeBounds = true;
    BitmapFactory.decodeResource(res, resId, options);

    // Calculate inSampleSize
    options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);

    // Decode bitmap with inSampleSize set
    options.inJustDecodeBounds = false;
    return BitmapFactory.decodeResource(res, resId, options);
}
```

这个方法使得加载一个大图的缩略图到一个 100x100pix 的 ImageView 变的简单。
```java
mImageView.setImageBitmap(
    decodeSampledBitmapFromResource(getResources(), R.id.myimage, 100, 100));
```

你能按照上面的例子从不同的数据源解码图片。