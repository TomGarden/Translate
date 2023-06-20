## 0x00、 [IBinder](https://developer.android.com/reference/android/os/IBinder)

远程对象基本接口，为高效进程内或者扩进程通信效用设计的轻量级远程程序调用机制的核心部分。
这个接口描述了与远程对象交互的抽象协议。
不要直接实现本接口，如有需要可以继承 Binder 。

IBinder 中关键的 API 是 transact() 方法(和 Binder.onTransact() 相对应)。
这个方法分别允许你向 IBinder 对象发送一个调用，以及接收一个来自 Binder 对象的调用。
这个发送 API 是同步的，即，当你调用 transact() 方法后，直到目标完成 Binder.onTransact() 方法调用你才会等到 return ；这是调用本地进程时的预期行文，并且底层的进程间通信机制(IPC)确保本 API 在跨进程使用的时候会有相同的语义。

通过 transact() 发送的数据是一个 Parcel 对象(一个通用的数据缓存器)，其中存储了一些元数据。
缓存中的元数据被用于管理 IBinder 对象引用，所以这些引用可以被缓存并且跨进程传递。
这种机制确保了当一个 IBinder 被写入 Parcel 并且发送到另一个进程中的时候，如果后者将接收到的 IBinder 引用发送回原进程，则原进程将接收到相同的 IBinder 对象。
这些语义允许 IBinder/Binder 对象被作为唯一身份标识(存储token或者用作其它目的)被跨进程管理。

系统中每一个进程都持有一个事务线程池。
这些线程被用于调度所有来自其它进程的 IPCs 。
例如，当一个 IPC 从进程 A 发送到进程 B，调用线程被阻塞在方法 transact() ,因为它将事务发送到进程 B 了。
接下来进程 B 中的线程池总可用的线程接收到到来的事务，并且调用 Binder.onTransact() 方法处理目标对象，然后通过 Parcel 将处理结果返回。
进程 A 从 transact() 方法的返回值中接收到结果，进程 A 的阻塞状态被解除，进程 A 继续执行。
事实上，在其他进程中使用了额外的线程做了某些操作，而非在当前进程中。

Binder 体系也支持跨进程递归。
例如，如果进程 A 执行一个 transaction 到进程 B ，然后进程 B 获取到消息后调用了一个在进程 A 中实现的 IBinder， 此时进程 A 正在等待原来的 transaction 结果的状态被终结进而去调用 Binder.onTransact() 处理来自进程 B 的调用。
这确保了调用远程 binder 对象的递归语义与调用本地对象的递归语义相同。

当使用远程对象工作的时候，你经常希望了解它是否还是有效的对象。
这里有三个方法帮助你确定：
1.  如果你尝试在 IBinder 对象中调用一个已经不进程的对象 transact() 方法将会抛出 RemoteException 异常
2.  如果远程对象不存了调用 pingBinder() 将会 return false 。
3.  linkToDeath() 方法能携带一个 IBinder 对象注册与 IBinder.DeathRecipient ，当远程进程不存在的时候，这个这个方法将会被回调。