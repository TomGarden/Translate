## 0x00、 [Handler](https://developer.android.com/reference/android/os/Handler)

一个 Handler 允许你通过 MessageQueue 发送和执行 Message 和 Runable 对象。
每一个 Handler 实例和一个单独的线程以及该线程的消息队列相关。
当你创建新的 Handler 的时候，与这个 Handler 绑定在一起的 thread(以及线程的 message queue)也就被同时创建了，它将传送消息到 message queue ，并且不断的从 message queue 中执行消息相关的动作。

Handler 有两个主要的用途：
1.  将消息放到 message queue，并在想来的某个事件执行该消息；
2.  在另一个线程中执行动作。

通过这些方法将 message 安排到日程中：`post(Runnable)`, `postAtTime(Runnable, long)`, `postDelayed(Runnable, Object, long)`, `sendEmptyMessage(int)`, `sendMessage(Message)`, `sendMessageAtTime(Message, long)`, 和 `sendMessageDelayed(Message, long)`。
post 方法允许你将 Runable 对象排到队列中，当 message queue 接收到之后会调用它；
sendMessage 允许你将 Message 对象排到队列中，Message 对象中包含一个 bundle 数据包，Handler 的 handleMessage(Message) 方法将会处理这些数据(需要你实现 Hamdler 的一个子类)。

当传递一个消息到 Handler 的时候，只要 message queue 已经被准备停当，你能选择立刻执行这个消息，或者指定延时执行。
后者允许你实现超时，计时，以及其他基于时间的行为。

当系统为你的 app 创建一个进程的时候，它的主线程专注于运行一个 message queue 去管理 app 顶级对象(activities,broacase receivers,等)以及它创建的 windows。
你能创建你自己的线程并且通过一个 Handler 和 app 的主线程交互。
给定的 Runable 或者 Message 将会被排到 Handler 的 message queue 并且在适当的时候被执行。