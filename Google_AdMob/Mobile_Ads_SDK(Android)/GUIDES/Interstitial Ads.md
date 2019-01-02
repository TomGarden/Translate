## 0x00、 [Interstitial Ads](https://developers.google.com/admob/android/interstitial)

插页式广告是完全覆盖 app 交互页面的全屏广告。
它通常展示在 app 流程中的自然过渡点，例如在 Activity 切换的时候或者游戏暂停的时候。
当一个 app 展示一个插页式广告的时候，用户可以选择点击广告并到达广告的目的地也可以关闭它回到 app 。

## 0x01、 前提条件
单独或作为 Firebase 的一部分导入 Google 移动广告 SDK。

## 0x02、 创建插页广告对象
插页式广告通过 InterstitialAd 对象请求并展示。
第一步，实例化 InterstitialAd 并且设置广告单元 ID 。
这些在 Activity 的 `onCreate()` 方法中完成。

```java
package ...

import com.google.android.gms.ads.InterstitialAd;

public class MainActivity extends Activity {

    private InterstitialAd mInterstitialAd;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        MobileAds.initialize(this,
            "ca-app-pub-3940256099942544~3347511713");

        mInterstitialAd = new InterstitialAd(this);
        mInterstitialAd.setAdUnitId("ca-app-pub-3940256099942544/1033173712");
    }
}
```

在 Activity 的整个生命周期中一个 InterstitialAd 对象能被用于请求和展示多个 插页广告，所以你仅需要控制它一次。

## 0x03、 应始终使用测试广告进行测试

在开发和测试应用时，请确保使用的是测试广告，而不是实际投放的广告。否则，可能会导致您的帐号被暂停。

最简便的测试广告加载方法就是使用以下 Android 横幅广告的专用测试广告单元 ID：

`ca-app-pub-3940256099942544/1033173712`

该测试广告单元 ID 已经过专门配置，可为每个请求返回测试广告，您可以在自己应用的编码、测试和调试过程中随意使用该测试广告单元 ID。只需确保您会在发布应用前用自己的广告单元 ID 替换该测试广告单元 ID 即可。

如需详细了解移动广告 SDK 的测试广告如何运作，请参阅测试广告。

## 0x04、 加载一个广告
>**☆NOTE:**确保所有的对 Mobile Ads SDK 的调用都是在主线程中完成。

通过调用 InterstitialAd 对象的 loadAd() 方法加载一个插页式广告。
这个方法接收一个 AdRequest 对象作为唯一的参数。

```java
package ...

import com.google.android.gms.ads.AdRequest;
import com.google.android.gms.ads.InterstitialAd;

public class MainActivity extends Activity {

    private InterstitialAd mInterstitialAd;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        MobileAds.initialize(this,
            "ca-app-pub-3940256099942544~3347511713");

        mInterstitialAd = new InterstitialAd(this);
        mInterstitialAd.setAdUnitId("ca-app-pub-3940256099942544/1033173712");
        mInterstitialAd.loadAd(new AdRequest.Builder().build());
    }
}
```

## 0x05、 展示广告
插页式广告应该在一个 app 正常流程中的暂停期间展示。
在游戏关卡之间就是一个很好的例子，或者在用户完成任务之后。
要展示插页式广告使用 isLoaded() 方法验证是否正在加载中，然后使用 show() 进行展示。
下面展示了在按钮点击后展示广告的例子：

```java
mMyButton.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        if (mInterstitialAd.isLoaded()) {
            mInterstitialAd.show();
        } else {
            Log.d("TAG", "The interstitial wasn't loaded yet.");
        }
    }
});
```

## 0x06、 [广告事件](https://developers.google.com/admob/android/banner#ad_events)

## 0x07、 使用 AdListener 进行重复加载

AdListener 类的 onAdClosed() 方法可以在一个广告关闭之后再次加载一个：

```java
...
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

        MobileAds.initialize(this,
            "ca-app-pub-3940256099942544~3347511713");

    mInterstitialAd = new InterstitialAd(this);
    mInterstitialAd.setAdUnitId("ca-app-pub-3940256099942544/1033173712");
    mInterstitialAd.loadAd(new AdRequest.Builder().build());

    mInterstitialAd.setAdListener(new AdListener() {
        @Override
        public void onAdClosed() {
            // Load the next interstitial.
            mInterstitialAd.loadAd(new AdRequest.Builder().build());
        }

    });
}
...
```

## 0x08、 一些最佳实践
### 8.1、 考虑插页式广告对你的 app 来讲是否是最佳的广告形式。

插页式广告在 app 的自然过渡点能良好的运行。
进程会在分享图片的时候或者游戏切换关卡的时候创建广告。
因为用户期望在这一过程得到恰当的休息，因此在这种时候插入广告不会影响用户的体验。
确保你对在自己应用流程中的哪一个节点插入广告这件事是考虑过用户体验的。

### 8.2、 当显示广告的时候记得暂停应用正在进行的动作
### 8.3、 允许足够的加载时间
确保广告在恰当的时间进行展示是必要的，避免用户等待广告的加载也是必要的。
广告在 loadAd() 调用后以及 show() 调用前被加载，确保你的 app 在展示之前有充足的时间加载好广告。

### 8.4、 不要铺天盖地的使用广告
虽然增加应用程序中的插播广告的频率似乎是增加收入的一个好方法，但是它也会降低用户体验并降低点击率。确保用户不会被频繁地打扰，以至于他们不能再享受你的应用程序的使用了。