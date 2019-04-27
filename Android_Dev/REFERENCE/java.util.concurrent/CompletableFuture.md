## [java.util.concurrent.CompletableFuture<T>](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)

**Since: 1.8**

一个 [Future](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Future.html) 已经明确的被完成了(设置值或者状态)，并且可用作 [CompletionState](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletionStage.html)，支持在完成时触发功能和动作。

当两个或更多的线程尝试 complete，completeExceptionally 或者 cancel 一个 CompletableFuture ,只有一个能成功。

除了这些直接操作状态和结果的相关方法外， CompletableFuture 用下面的策略实现了接口 CompletionState ：

1.  为非异步方法的依赖完成提供的操作，可以被完成当前 CompletableFuture 的线程执行，也可以被其他已完成的方法调用。
2.  所有没有明确执行参数的异步函数都是使用 [ForkJoinPool.commonPool()](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ForkJoinPool.html#commonPool--) (除非它不支持至少两个并行，在这种情况下，会创建新线程来执行任务)执行的。
    要简化 监控、调试、追踪，所有异步任务都是 `CompletableFuture.AsynchronousCompletionTask` 的实例。
3.  所有的 CompletionStage 函数都被其他 public 函数独立实现，所以每一个函数的行为都不会被其他子类覆盖或影响。

CompletableFuture 通过下面的策略实现了 `Future` ：

1.  因为这个类(不同于 FutureTask)不能直接控制导致它完成的计算,取消被视为异常完成的另一种形式。
    函数 `cancel` 和 `completeExceptionally(newCancellationException())` 有相同的效果。
    函数 `isCompletedExceptionally() ` 能被用于确定 `CompletableFuture` 是否以异常的方式完成 。
2.  在异常完成的情况下会携带一个 CompletionException ，函数 `get()` 和 `get(long, TimeUnit)` 会抛出一个 `ExecutionException` 这个异常和 `CompletionException` 中的异常相同。
    要简化在多数上下文中的用法，这个类也定义了 `join()` 和 `getNow(T)` 函数，而不是直接抛出 `CompletionException ` 。

## [Interface CompletionStage<T>](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletionStage.html)

- **java.util.concurrent**  
- Since: 1.8

异步计算的一个阶段， 当其他 CompletionStage 完成的时候执行一个动作或者计算一个值。
一个阶段完全完成并终止它的计算，但是这可能触发其他依赖阶段。
定义在此接口中的功能仅使用几种基本的形式，这些形式扩展为更大的函数集以捕获一系列使用方式：

1.   由 stage(阶段) 执行的计算可以表示为一个 Function，Consumer(消费者) 或者 Runable (使用的函数名中分别包含  apply, accept, 或者 run) 这取决于其是否需要参数或者产生结果。
    例如 
    ```java
    stage.thenApply(x -> square(x))
         .thenAccept(x -> System.out.print(x))
         .thenRun(() -> System.out.println())
    ```
    另一种形式(组成)应用 stage 本身的功能，而不是它的结果。
2.   一个 stage 的执行可能通过另一个 stage 或者两个 stage 或者两个中任何一个的完成来触发。
     依赖一个单一的 stage 使用前缀为 *then* 的方法。
     被两个 stage 的完成所触发的 stage 或许会 combine(组合) 他们的结果或者影响，使用携带 combine 字段的函数。
     被两个 stage 或者其一个 stage 的完成所触发的 stage 无法保证依赖哪一个结果或影响，使用携带 either 关键字的函数。
3.   各 stage 之间的依赖控制计算的触发，但是不能保证相互间的特定顺序。
     另外，执行一个新的 stage 的计算有三种可用的手段：默认执行，默认异步执行(使用带有 async 前缀的函数，通过使用 stage 的默认异步设施执行)，自定义(通过实现一个 [Executor](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executor.html))。
     默认的执行模式和默认异步执行模式被 CompletionStage 的实现类指定。以 `Executor` 作为执行参数的方式可以以任意的默认执行，甚至不支持并发，但是以适应异步的方式进行处理
4.   两种函数形式支持处理对 stage 的触发是以默认的形式还是异常的形式：
     函数 [whenComplete](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletionStage.html#whenComplete-java.util.function.BiConsumer-) 允许不论结果如何都注入一个动作，其他的则是在完成时保留结果。
     函数 [handle](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletionStage.html#handle-java.util.function.BiFunction-) 允许 stage 替换结果，该结果可能被其他相关 stage 做进一步处理。
     在其他所有情况下，如果一个 stage 由于一个未捕获的异常而结束，所有依赖它的 stage 也会终结自身，并将 [`CompleteException`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletionException.html) 作为终结原因。
     如果一个 stage(称为 stageA) 通过 `either` 的方式依赖其他两个 stage ，并且被依赖的 stage 有一个执行结束了，无法保证 stageA 是以默认方式结束还是以异常方式结束。
     在 `whenComplete` 函数中注入的动作中发生了未捕获的异常，那么 stage 以这个异常作为结束异常。

所有的方法都遵循上述的 触发、执行、异常完成 规范(在具体的方法中不在重复叙述)。
另外，用于传递给完成结果的参数可能为 null (即，泛型参数 T )，在任何情况下传递 null 参数都会抛出 `NullPointerException` 异常。

这个接口没有定义用于 初始化、强制结束、异常、探测执行 stage 或结果、或者等待 stage 的函数。
在实现 CompletionStage 的时候可酌情提供类似函数。
[toCompletableFuture()](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletionStage.html#toCompletableFuture--)函数允许在提供转化类型的前提下完成此接口不同实现间的互操作。