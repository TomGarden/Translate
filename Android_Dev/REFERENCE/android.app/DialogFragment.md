## 0x00、 [android.app.DialogFragment](https://developer.android.com/reference/android/app/DialogFragment.html)

### 0.1、 关于 android.support.v4.app.DialogFragment

对 DialogFragment 的静态库支持版本。
APP 的运行版本不受限制；
暂无更多描述。

### 02、 概览

> **注意** 这个类在 API level 28 被废弃。
    使用 [Support Library](https://developer.android.com/tools/extras/support-library.html) [DialogFragment](https://developer.android.com/reference/android/support/v4/app/DialogFragment.html) 做到统一的设备行为和生命周期管理。

一个展示 dialog 窗口的 fragment 浮于 activity 窗口上方。
这个 fragment 持有一个 Dialog 对象，它根据 fragment 的状态做出合适的展示。
控制这个 dialog (决定什么时候展示隐藏和关闭) 应该通过这个类中的 API 而不是直接调用 dialog 。

实现应该集成这个类并且实现 `Fragment.onCreateView(android.view.LayoutInflater, android.view.ViewGroup, android.os.Bundle)` 用于提供 dialog 内容。
或者，复写 `onCreateDialog(android.os.Bundle)` 去完全自定义一个 dialog ， 例如一个 AlertDialog 。

本文的四个话题：
- Lifecycle
- Basic Dialog
- Alert Dialog
- Selecting Between Dialog or Embedding

## 0x01、 声明周期

在一个 Dialog 实例中 DialogFragment 做各种各样的事来保持或者操作 fragment 的生命周期。
注意 dialog 是一个自治实体——它有自己的窗口，接收自己的 input 事件，自主决定什么时候消失(通过接收返回按钮点击事件，还是用户对某个 button 的点击事件)。

DialogFragment 需要确保知晓 Fragment 正在发生什么以及使 Dialog 的状态与其保持一至。
为此，它会监视从对话框中删除事件，并在发生事件时注意删除自己的状态。
这意味着你应该使用 `show(android.app.FragmentManager, java.lang.String) ` 或 ` show(android.app.FragmentTransaction, java.lang.String)` 在你的 UI 界面创建 DialogFragment 实例，以此来确定当 dialog 消失时机从而确定何时移除自身。

## 0x02、 基本 Dialog

DialogFragment 的最简单的实现是作为一个浮动 Fragment 视图结构的容器。如下例：

```java
public static class MyDialogFragment extends DialogFragment {
    int mNum;

    /**
     * Create a new instance of MyDialogFragment, providing "num"
     * as an argument.
     */
    static MyDialogFragment newInstance(int num) {
        MyDialogFragment f = new MyDialogFragment();

        // Supply num input as an argument.
        Bundle args = new Bundle();
        args.putInt("num", num);
        f.setArguments(args);

        return f;
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mNum = getArguments().getInt("num");

        // Pick a style based on the num.
        int style = DialogFragment.STYLE_NORMAL, theme = 0;
        switch ((mNum-1)%6) {
            case 1: style = DialogFragment.STYLE_NO_TITLE; break;
            case 2: style = DialogFragment.STYLE_NO_FRAME; break;
            case 3: style = DialogFragment.STYLE_NO_INPUT; break;
            case 4: style = DialogFragment.STYLE_NORMAL; break;
            case 5: style = DialogFragment.STYLE_NORMAL; break;
            case 6: style = DialogFragment.STYLE_NO_TITLE; break;
            case 7: style = DialogFragment.STYLE_NO_FRAME; break;
            case 8: style = DialogFragment.STYLE_NORMAL; break;
        }
        switch ((mNum-1)%6) {
            case 4: theme = android.R.style.Theme_Holo; break;
            case 5: theme = android.R.style.Theme_Holo_Light_Dialog; break;
            case 6: theme = android.R.style.Theme_Holo_Light; break;
            case 7: theme = android.R.style.Theme_Holo_Light_Panel; break;
            case 8: theme = android.R.style.Theme_Holo_Light; break;
        }
        setStyle(style, theme);
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
            Bundle savedInstanceState) {
        View v = inflater.inflate(R.layout.fragment_dialog, container, false);
        View tv = v.findViewById(R.id.text);
        ((TextView)tv).setText("Dialog #" + mNum + ": using style "
                + getNameForNum(mNum));

        // Watch for button clicks.
        Button button = (Button)v.findViewById(R.id.show);
        button.setOnClickListener(new OnClickListener() {
            public void onClick(View v) {
                // When button is clicked, call up to owning activity.
                ((FragmentDialog)getActivity()).showDialog();
            }
        });

        return v;
    }
}
```

在 Activity 中 调用 showDialog() 显示这个 DialogFragment :

```java
void showDialog() {
    mStackLevel++;

    // DialogFragment.show() will take care of adding the fragment
    // in a transaction.  We also want to remove any currently showing
    // dialog, so make our own transaction and take care of that here.
    FragmentTransaction ft = getFragmentManager().beginTransaction();
    Fragment prev = getFragmentManager().findFragmentByTag("dialog");
    if (prev != null) {
        ft.remove(prev);
    }
    ft.addToBackStack(null);

    // Create and show the dialog.
    DialogFragment newFragment = MyDialogFragment.newInstance(mStackLevel);
    newFragment.show(ft, "dialog");
}
```

上述操作是移除当前正在显示的 dialog ， 携带参数创建一个新的 DialogFragment 并且将其作为堆栈中的最新内容展示出来。
当 transaction 当前 DialogFragment 以及它的 dialog 将被销毁，并重新显示前一个（如果有）。
请注意，在这种情况下，DialogFragment将负责弹出Dialog的事务，并将其单独解除。

## 0x03、 Alert Dialog

实例化(或者另外)实现 `Fragment.onCreateView(LayoutInflater, ViewGroup, Bundle)` 去生成 dialog 示例的视图结构，也可以实现 `onCreateDialog(android.os.Bundle)`  去创建自定义 Dialog 对象。

这通常用于创建 AlertDialog，允许你通过 fragment 管理展示给用户的标准 alert 窗。
一个简单例子：

```java
public static class MyAlertDialogFragment extends DialogFragment {

    public static MyAlertDialogFragment newInstance(int title) {
        MyAlertDialogFragment frag = new MyAlertDialogFragment();
        Bundle args = new Bundle();
        args.putInt("title", title);
        frag.setArguments(args);
        return frag;
    }

    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        int title = getArguments().getInt("title");

        return new AlertDialog.Builder(getActivity())
                .setIcon(R.drawable.alert_dialog_icon)
                .setTitle(title)
                .setPositiveButton(R.string.alert_dialog_ok,
                    new DialogInterface.OnClickListener() {
                        public void onClick(DialogInterface dialog, int whichButton) {
                            ((FragmentAlertDialog)getActivity()).doPositiveClick();
                        }
                    }
                )
                .setNegativeButton(R.string.alert_dialog_cancel,
                    new DialogInterface.OnClickListener() {
                        public void onClick(DialogInterface dialog, int whichButton) {
                            ((FragmentAlertDialog)getActivity()).doNegativeClick();
                        }
                    }
                )
                .create();
    }
}
```

下面的 activity 创建上面的  fragment 可以通过下面的方法创建 dialog 以及接收交互事件。

```java
void showDialog() {
    DialogFragment newFragment = MyAlertDialogFragment.newInstance(
            R.string.alert_dialog_two_buttons_title);
    newFragment.show(getFragmentManager(), "dialog");
}

public void doPositiveClick() {
    // Do stuff here.
    Log.i("FragmentAlertDialog", "Positive click!");
}

public void doNegativeClick() {
    // Do stuff here.
    Log.i("FragmentAlertDialog", "Negative click!");
}
```

注意在这种情况下 fragment 不会放在堆栈上，仅仅是增加了一个无限期运行的 fragment 。
因为 dialog 通常是 modal 的，他将作为后台堆栈运行因为它将捕获用户输入知道它小时。
当他消失的时候，DialogFragment 将负责将其从自身删除。

## 0x04、 在上述二者之间做一个选择

DialogFragment 能像普通 Fragment 一样被持续操作(如果你期望这样)。
如果你有一个 fragment 你希望将其展示为 dialog 或者将其嵌入更大的 UI 这是有用的。
