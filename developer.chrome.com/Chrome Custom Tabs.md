## 0x00、 [Chrome Custom Tabs](https://developer.chrome.com/multidevice/android/customtabs)

## 0x01、 What are Chrome Custom Tabs?
开发者面临的一个问题是：当需要访问一个网址的时候是跳转到浏览器还是应用内部维护一个 WebView 。

两个选项都面临着挑战——启动浏览器无法自定义，并且是一个繁重的上下文切换，WebView 不与浏览器共享状态，会增加维护开销。

Chrome Custom Tab 给了 APP 更对控制 web 体验的能力，并且在本地代码和 web 内容之间的切换更简单，不必使用 WebView 。

Chrome Custom Tab 允许 app 定制其外观。
app 能改变的细节包括：
- Toolbar 颜色
- 打开和退出动画
- 为 toolbar 增加自定义动作，允许 toolbar 按钮溢出

Chrome Custom Tab 也允许开发者为了更块的加载速度预启动 Chrome 与预提取内容。

![2018-12-18-performance.gif](/developer.chrome.com/Image/2018-12-18-performance.gif)

你可以在 [示例代码](https://github.com/GoogleChrome/custom-tabs-client) 中对细节进行测试。

## 0x02、 When should I use Chrome Custom Tabs vs WebView?
如果你在应用程序内部托管自己的 web 内容 WebView 是一个不错的解决方案。
如果你的 app 重定向到其他域名，我们建议你使用 Chrome Custom Tom 的几个原因如下：
1. 易于实现，无需自己管理请求，权限授予或 cookie 存储。
2. UI 自定义
    - Toolbar color
    - Action button
    - Custom menu items
    - Custom in/out animations
    - Bottom toolbar
3. 导航感知：the browser delivers a callback to the application upon an external navigation.
4. 安全：浏览器使用 Google's Safe Browsing 保证用户设备访问的是安全的站点。
5. 性能优化：
    - 在后台预热浏览器，同时避免从应用程序中窃取资源。
    - 提前向浏览器提供可能的URL，这可能会执行推测性工作，从而加快页面加载时间。
6. 生命周期管理：浏览器为了防止自己系统清理，会将自己的重要程度提高到前景级别。
7. 共享 cookie jar 和权限模型，以至于用户不必重复登录自己曾链接过的网站，或者不必重复授予自己曾经授予过的权限。
8. 如果用户已启用Data Saver，他们仍将从中受益。
9. 跨设备的同步可以很出色的完成。
10. 简单的自定义模式。
11. 只需要一次点击即可快速返回应用。
12. 您希望在Lollipop之前的设备上使用最新的浏览器实现（自动更新WebView）而不是旧的WebView。

## 0x03、 When will this be available?
从Chrome 45开始，Chrome自定义标签现在通常可供所有Chrome用户使用，支持所有Chrome支持的Android版本（Jellybean以后版本）。

……

## 0x04、 Implementation guide
一个完整的例子：https://github.com/GoogleChrome/custom-tabs-client。
它包含了可重用的用于自定义 UI 的类，后台链接服务，以及对 application 和 custom tab activity 生命周期的处理。

如果你跟随本页面的指南，你也能很好地整合这些内容。

第一步就是要将 [Custom Tabs Support Library](http://developer.android.com/tools/support-library/features.html#custom-tabs)整合到你的项目中。
打开 `build.gradle` 文件并且增加如下代码：
```
dependencies {
    ...
    compile 'com.android.support:customtabs:23.3.0'
}
```

一旦支持库添加完成后，你的项目就可以完成如下两个定制：
1. 通过自定义标签完成自定义 UI 组件。
2. 在保证 app 存活的前提下，使页面加载的更快。

UI 定制通过 CustomTabsIntent 和 CustomTabsIntent.Builder 类完成；
性能的提升通过使用 CustomTabsClient 连接到自定义的 Tab service 实现，热身 Chrome 让它知道即将启动的 url 。

### 4.1、 Opening a Chrome Custom Tab
```java
// Use a CustomTabsIntent.Builder to configure CustomTabsIntent.
// Once ready, call CustomTabsIntent.Builder.build() to create a CustomTabsIntent
// and launch the desired Url with CustomTabsIntent.launchUrl()

String url = ¨https://paul.kinlan.me/¨;
CustomTabsIntent.Builder builder = new CustomTabsIntent.Builder();
CustomTabsIntent customTabsIntent = builder.build();
customTabsIntent.launchUrl(this, Uri.parse(url));
```

### 4.2、 Configure the color of the address bar
Chrome Custom Tab 很重要的一点就是你能够根据应用主题修改地址栏的颜色。
```java
// Changes the background color for the omnibox. colorInt is an int
// that specifies a Color.

builder.setToolbarColor(colorInt);
```

### 4.3、 Configure a custom action button

![2018-12-18-tumblr_action.png](/developer.chrome.com/Image/2018-12-18-tumblr_action.png)

作为应用的开发者，你可以完全控制 Chrome tab 中展示给用的动作按钮。

多数情况下，我们的主要动作例如 分享，或者打开其他 activity。

动作按钮表示为一个 Bundle ，其中包含动作按钮的图标以及当用户点击动作按钮时Chrome将调用的pendingIntent。 该图标的高度为24dp，宽度为24-48 dp。
```java
// Adds an Action Button to the Toolbar.
// 'icon' is a Bitmap to be used as the image source for the
// action button.

// 'description' is a String be used as an accessible description for the button.

// 'pendingIntent is a PendingIntent to launch when the action button
// or menu item was tapped. Chrome will be calling PendingIntent#send() on
// taps after adding the url as data. The client app can call
// Intent#getDataString() to get the url.

// 'tint' is a boolean that defines if the Action Button should be tinted.

builder.setActionButton(icon, description, pendingIntent, tint);
```
