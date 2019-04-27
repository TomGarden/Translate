## 0x00、 和 UI 线程交互

前面的教程讲述了如何 [Runcode on a thread pool thread](https://developer.android.com/training/multiple-threads/run-code.html),展示了你在 thread 管理的时候如何通过 ThreadPoolExecutor 开始一个任务。
最后一课展示了你如何任务中将数据发送到 UI 线程的对象中。
这些特征允许你的任务运行于后台并且他们能将运行结果发送到 UI 元素中(例如 bitmap)。

每一个 app 都有它自己的特殊运行于 UI 对象的线程例如 View 对象；这些线程被 UI 线程调用。
只有运行与 UI 线程的对象才能访问 UI 线程中的其他对象。
因为在线程池中的线程不能在 UI 线程中运行，他们不能访问 UI 对象。
将后台线程中的数据发送到 UI 线程使用运行于 UI 线程 Handler。

## 0x01、 在 UI 线程定义一个 handler

Handler 是 Android 系统中 framework 层中线程管理工具的一部分。
一个 Handler 对象接收 message 并且执行相关代码处理接收到的 message 。
通常你要为新线程创建一个 Handler，但是你也能创建一个 Handler 用于链接一个已经存在的线程。
当你将一个 Handler 与 UI 线程连接起来的时候，Handler message 所触发的代码将在 UI 线程中。

