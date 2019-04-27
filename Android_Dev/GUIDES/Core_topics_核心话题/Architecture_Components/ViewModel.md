## 0x00、 ViewModel 概览

ViewModel 被设计用于在一个 声明周期 存储和管理 UI 相关的数据 。
ViewModel 类允许保留配置变化(例如屏幕旋转)。

> **☆注意 :** 要在你的项目中实现 ViewModel ，请查看在[生命周期发布日志](https://developer.android.com/jetpack/androidx/releases/lifecycle#declaring_dependencies)中的声明的指南(重要)。

Android framework 管理 UI 控制器(例如 Activity 和 Fragment)的生命周期。
fragment 可能会决定销毁或者重建 UI 控制器从而响应用户事件(这些事件可能超出你的控制范围)。

如果系统销毁或者重建 UI 控制器，任何瞬态的 UI 相关的数据就会丢失。
例如，你的 app 可能在一个 Activity 中包含一个用户列表。
当 activity 因为配置变化被重建，新的 activity 必须重新获取用户数据列表。。
对于简单的数据， activity 使用 `onSaveInstanceState()`  将数据存取在 Bundle 中，并且在 onCreate() 方法重新获取这些数据，但是这仅适用于较小的可序列化的数据，对于较大的shuju(例如列表或者位图)就不适用了。

另一个问题是 UI 控制器通常需要需要同步调用，这可能需要一些时间才能返回结果。
UI controller 需要管理这些调用并确保系统在恰当的时候清除他们，从而避免造成内存泄露。
这种管理需要大量的维护，并且在为配置更改而重新创建对象的情况下，这是浪费资源的，因为对象可能必须重新发出它已经发出的调用。

UI 控制器 (例如 Activity 和 Fragment) 主要用于展示 UI 数据，想用用户事件，或者与系统通信，例如获取权限。
要求 UI 控制器完成从数据库或者网络加载数据的请求是不合理的。
将太多的责任放到 UI 控制器身上，会造成一个单一的类负责过多的工作(相对于为不同的责任寻找不同的代理而言)。
以这种方式为UI控制器分配过多的责任也会使测试变得更加困难。

将视图数据所有权与UI控制器逻辑分离起来将使事情更容易，更高效。

## 0x02、 实现一个 ViewModel

架构组件 为 UI controller 提供 ViewModel 助手类，它的责任是为 UI 准备数据。
ViewModel  对象在 配置改变的时候自动保存数据示例，从而它保存的数据能被下一个 activity 或者 fragment 实例立即重用。
例如，如果你需要展示一个 user 列表，确保将责任分配给 ViewModel(用于保存 user list)，实例化一个 Activity 或者 fragment。如下所示：

```java
public class MyViewModel extends ViewModel {
    private MutableLiveData<List<User>> users;
    public LiveData<List<User>> getUsers() {
        if (users == null) {
            users = new MutableLiveData<List<User>>();
            loadUsers();
        }
        return users;
    }

    private void loadUsers() {
        // Do an asynchronous operation to fetch users.
    }
}
```

然后你可以在 Activity 或者 Fragment 中访问这个列表：

```java
public class MyActivity extends AppCompatActivity {
    public void onCreate(Bundle savedInstanceState) {
        // Create a ViewModel the first time the system calls an activity's onCreate() method.
        // Re-created activities receive the same MyViewModel instance created by the first activity.

        MyViewModel model = ViewModelProviders.of(this).get(MyViewModel.class);
        model.getUsers().observe(this, users -> {
            // update UI
        });
    }
}
```

如果 Activity 重新创建，它会重新获得 Activity 关闭前其持有的 MyViewModel 实例。
当 MyViewModel 的所有者(Activity) 被 finish 的时候，framework 调用 ViewModel 对象的 `onCleared()` 方法清除这些资源。

> **重中之重-警告：** ViewModel绝不能引用视图，生命周期或任何可能包含对活动上下文的引用的类。

ViewModel 对象被设作为超出视图或者生命周期的特定实例。
此设计还意味着您可以更轻松地编写测试以覆盖ViewModel，因为它不了解视图和Lifecycle对象。
ViewModel 对象能包含 `LifecycleObservers` 例如 `LiveData` 对象。
但是，ViewModel对象绝不能观察到生命周期感知的可观察对象（例如LiveData对象）的更改。 如果ViewModel需要Application上下文，例如查找系统服务，它可以扩展AndroidViewModel类并具有在构造函数中接收Application的构造函数，因为Application类扩展了Context。




## 0x04、 用 ViewModel 替换 Loaders

像 CursorLoader 这样的 Loader 类通常用于持有和 UI 相同步的数据。
你能使用 ViewModel (涉及更少的类) 替换 loader 。
使用 ViewModel 将从数据加载操作中将你的的 UI 控制器分离出来，这意味着你在类之间有更少的强引用。

通常使用 loader(可能是 CursorLoader) 的时候,它会观察 Database 。
当 database 中的数据发生变化的时候， loader 自动触发一个重新加载数据并更新 UI 的动作。

![2019-04-24-viewmodel-loader.png](/Android_Dev/GUIDES/Images/2019-04-24-viewmodel-loader.png)


ViewModel 和 Room 、 LiveData 一起工作，他们可以替换 loader 。
ViewModel 确保在设配配置更改后数据仍然存在。
Room 会在 database 变化的时候通知你 LiveData，LiveData 用修改后的数据更新你的 UI 。

![2019-04-24-viewmodel-replace-loader.png](/Android_Dev/GUIDES/Images/2019-04-24-viewmodel-replace-loader.png)

### 4.1、 功能信息

当你的数据变的越来与复杂，你可能会选择用一个分离的类专门负责加载数据。
ViewModel 的目标是为一个 UI 控制器封装这些数据，使得当配置更改的时候数据依然存在。
有关如何跨配置更改加载、保持和管理数据的信息，请参阅 [Saving UI States](https://developer.android.com/topic/libraries/architecture/saving-states.html)。

[Android应用程序架构指南](https://developer.android.com/topic/libraries/architecture/guide.html#fetching_data)建议构建一个存储库类来处理这些功能。



## 略