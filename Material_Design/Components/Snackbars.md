## 0x00、 [Snackbars](https://material.io/develop/android/components/snackbar/)

![2018-10-20-Snackbars-1.svg](/Material_Design/Image/2018-10-20-Snackbars-1.svg)


Snackbar 控件通过一个底部视图提供了一个可操作的简短的反馈。
Snackbar 会在出现一定时间后，或者用户和其他 UI 元素交互后，自动消失，也可以通过手动滑动使其从屏幕消失。

Snackbars 也有执行一个动作的能力，例如取消刚做的一个操作，或者重试刚才失败的操作。

## 0x01、 Design & API Documentation
[MATERIAL DESIGN GUIDELINES: SNACKBARS](https://material.io/go/design-snackbar)

[CLASS DEFINITION](https://github.com/material-components/material-components-android/tree/master/lib/java/com/google/android/material/snackbar/Snackbar.java)

[CLASS OVERVIEW](https://developer.android.com/reference/com/google/android/material/snackbar/Snackbar)

## 0x03、 Usage
**Snackbar** 类提供了 static 方法 **make** 用于制作一个符合期望的 snackbar 。
Snackbar 有一个 **View**(参数) ，该 view g是一个在 snackbar 中作为展示的内容的根 **ViewGroup** ；一个文本参数 String (也可以是 **CharSequence** 或者资源 ID)；一个时间参数表示控件自动消失之前存活的时间(预设值，也可以是毫秒值)。
A suitable ancestor ViewGroup will be either the nearest CoordinatorLayout to the View passed in, or the root DecorView if none could be found.

持续时间的可选值：
- 无限期 - [LENGTH_INDEFINITE](https://developer.android.com/reference/com/google/android/material/snackbar/Snackbar#LENGTH_INDEFINITE)
- 长时间 - [LENGTH_LONG](https://developer.android.com/reference/com/google/android/material/snackbar/Snackbar#LENGTH_LONG)
- 短时间 - [LENGTH_SHORT](https://developer.android.com/reference/com/google/android/material/snackbar/Snackbar#LENGTH_SHORT)

**注意：** 
    如果 Snackbars 显示在 [CoordinatorLayout](https://developer.android.com/reference/androidx/coordinatorlayout/widget/CoordinatorLayout) 内效果最佳。
    **CoordinatorLayout** 允许 snackbar 能够支持轻扫退出，就像自动移动控件的 [FloatingActionButton](https://material.io/components/android/catalog/floating-action-button/)


调用 **make** 仅仅创建了 snackbar，并没有将他展示到屏幕。
要展示 snackbar ，使用 **show** 方法。
同一时刻只能展示一个 snackbar 。
一个新的 snackbar 展示之前会将之前正在展示的 snackbar 取消掉。

展示一个只有文字没有动作的 action 只需如下：
```java
/*
  view 用于制作 snackbar。
  它应该包含在你打算展示 snackbar 的地方的层级结构中。
  通常，它是需要交互才能触发的(就像一个需要点击的 button ，或者一个需要滑动的卡片)。
*/

View contextView = findViewById(R.id.context_view);

Snackbar.make(contextView, R.string.item_removed_message, Snackbar.LENGTH_SHORT)
    .show();
```

### 3.1 添加动作
要添加动作，可以再 **make** 放回之后使用 setAction 方法。
动作也是以文本的方式展示的，同时持有一个 **OnClickListener** 点击事件。
Snackbar 在点击之后会自动消失。

展示一个同时带有消息和动作的 snackbar 如下：
```java
Snackbar.make(contextView, R.string.item_removed_message, Snackbar.LENGTH_LONG)
    .setAction(R.string.undo, new OnClickListener() {
      @Override
      public void onClick(View v) {
        /*
          响应点击，例如撤销导致此提示的修改操作。
        */
      })
    });
```

动作文本的颜色能通过 **setActionTextColor** 方法自定义(默认是你的主题色)。

## 0x04、 相关内容
具有其他内容的临时底部 bar 可以通过继承 [BaseTransientBottomBar](https://developer.android.com/reference/com/google/android/material/snackbar/BaseTransientBottomBar) 来实现。

Android 也提供了 API 相似的 [Toast](https://developer.android.com/reference/android/widget/Toast.html) 用于显示系统界别的通知。
通常 Snackbar 是展示用户回馈的首选机制，因为他们能在发生动作的 UI 的上下文中显示。
在某些无法使用 Snackbar 的情况下可以使用 **Toast** 。