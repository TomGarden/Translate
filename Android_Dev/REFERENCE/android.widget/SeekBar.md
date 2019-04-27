## 0x00、 SeekBar

SeekBar 是 ProgressBar 的扩展，增加了对拖动的支持。
用户可以触摸并左右拖动 thumb 或者使用方向键，去设置当前级别。
不鼓励将可获得焦点的小部件放置在 SeekBar 的左侧或者右侧。

可以通过增加 `SeekBar.OnSeekBarChangeListener` 去监听用户动作。

### 0.1、 自带 XML 属性

- `android:thumb`   绘制 SeekBar 的 thumb

### 0.2、 继承来的 XML 属性，继承自：From class android.widget.AbsSeekBar 

- `android:thumbTint`   应用到 drawable 上的色调。
    可能是一个颜色值，格式可以是 ：  `#rgb`, `#argb`, `#rrggbb`, or `#aarrggbb`.

- `android:thumbTintMode`   混合模式使用 thumb tint。
    必须是下表中的一个值

    |Constant	|Value	|Description|
    |-----------|-------|------------|
    |add        |10     |将色调和可绘制颜色以及alpha通道组合在一起，将结果限制为有效的颜色值。饱和（S+D）|
    |moutiply   |e      |将可绘制的颜色和alpha通道与淡色的颜色和alpha通道相乘。【`SA*DA、SC*DC`】|
    |screen     |	f	|[`Sa + Da - Sa * Da`, `Sc + Dc - Sc * Dc]`|
    |...|...|...|

- `android:thumbTintMode`   可绘制刻度线的着色

### 0.3、 继承自 class android.widget.ProgressBar

- `android:animationResolution`     动画每一帧之间的间隔毫秒数
- `android:indeterminate`   允许不确定模式
-   `android:indeterminateBehavior` 定义在不确定模式下到达进度最大值后的动作
-   `android:indeterminateDrawable` 在不确定模式绘制
-   `android:indeterminateDuration` 不定动画的持续时间
-   `android:indeterminateOnly`     限制为仅限于不确定模式（状态保持进度模式将不起作用）。
-   `android:indeterminateTint`     色彩适用于不确定的进度指标。
-   `android:indeterminateTintMode`      
-   `android:interpolator`  设置不定动画的加速曲线
-   ``