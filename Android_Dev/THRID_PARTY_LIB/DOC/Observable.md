
## 0x00、 Observable

Obaservable 是非背压的，可选的多值基础反应类 ， 它提供工厂方法，中间操作符和消费同步/异步响应事件数据流的能力。

本类中很多操作接收 `ObservableSource(s)` ,这是非背压流的基本反应接口， `Observable` 本身也实现了这种接口。

`Observable` 的操作，默认的，在运行时操作一个持有 128 个元素的缓存空间(可查看`Flowable.bufferSize()`)，它能被系统参数 `rx2.buffer-size` 覆写。
多数操作，都具有显式设置其内部缓存空间的重载。

![legend.png](/Android_Dev/THRID_PARTY_LIB/DOC/Images/2019-02-25-legend.png)

本类的设计由 [Reactive-Streams design and specification](https://github.com/reactive-streams/reactive-streams-jvm) 驱动(通过移除与背压相关的基础设置和实现细节)，替换了 `org.reactivestreams.Subscription` , `Disposable` 作为处理流的主要手段。

Observable 满足协议：

`onSubscribe onNext* (onError | onComplete)?`

流通被 Disposable 实例通过 `Observable.onSunscribe` 提供给消费者。

```java
 Disposable d = Observable.just("Hello world!")
     .delay(1, TimeUnit.SECONDS)
     .subscribeWith(new DisposableObserver<String>() {
         @Override public void onStart() {
             System.out.println("Start!");
         }
         @Override public void onNext(String t) {
             System.out.println(t);
         }
         @Override public void onError(Throwable t) {
             t.printStackTrace();
         }
         @Override public void onComplete() {
             System.out.println("Done!");
         }
     });
 
 Thread.sleep(500);
 // the sequence can now be disposed via dispose()
 d.dispose();
```

## 0x01、 Method

### amb
在几个 ObservableSources 的 Iterable 中镜像一个，它首先发送一个条目或者一个终止通知。

## subscribeOn

在指定的 `Scheduler` 上向此 `ObservableSource` 异步订阅 `Observers` 。

Scheduler:
- 你可以指定这个操作将在哪一个 `Scheduler` 被使用。

Parameters:
- 指定订阅动作的调度者(Scheduler)

## observeOn


