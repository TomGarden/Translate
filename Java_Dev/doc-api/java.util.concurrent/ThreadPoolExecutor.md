# ThreadPoolExecutor

一个 ExecutorService ，它使用服务池中的线程执行每一次提交的任务，通常使用 Executors 的工厂方法配置它 。

线程池解决两个不同的问题：当执行大数量的异步任务的时候它通常具有更高的性能，因为它减少了调用每一个任务的开销；同时它提供了绑定和管理资源(包括执行任务集合时使用的线程)的手段。
每一个 ThreadPoolExecutor 也维护了基本的统计数据，例如已完成的任务数量。

为了能贯穿大量的上下文来使用，奔雷提供了许多课调整的参数和可扩展的钩子。
开发人员能更方便的使用 Executors 工厂方法：
 - `Executors.newCachedThreadPool()`(未绑定线程，线程会被自动创建)
 - `Executors.newFixedThreadPool(int) `(修正线程池中的线程数量)
 - `Executors.newSingleThreadExecutor()`(单一的后台线程)
这些预配置设置是为最常见的使用场景提供的。
其他的情况下根据下文的指导手动配置和调节类：

## 核心和最大线程数

ThreadPoolExecutor 将根据 coerPoolSize 和 maximumPoolSize 两个值自动调整线程数目(请看 `getPoolSize()`)。
当一个新的任务通过 execute 被提交，小于 corePoolSize 数目的线程将会运行，一个新的线程被创建用于持有这个请求，即使存在其他闲置的工作线程 。
如果正在运行的线程数超过了 corePoolSize 但是少于 maximumPoolSize ，只有队列是满的时候才会创建一个新线程。
通过将 corePoolSize 和 maximumPoolSize 设置为相同的值，你能创建一个修正数量的线程池。
通过设置 maximumPoolSize 为一个无边界的值(例如 Integer.MAX_VALUE),你能允许线程池容纳任意数量的并发任务。
多数情况下，核心和最大池数字在使用的时候才会设置，他们可以通过 `setCorePoolSize(int)` 和 `setMaximumPoolSize(int)` 进行设置 。

## 按需构造

默认的，只有当新任务到达的时候才会有一个核心线程被创建和初始化，但是这些动作可以通过 `prestartCoreThread()` 和 `prestartAllCoreThreads()` 动态覆盖。
如果你通过一个飞空队列构造一个线程池，你或许希望预先启动线程。

## 创建新线程

新线程通过 ThreadFactory 创建。
如果没有重写