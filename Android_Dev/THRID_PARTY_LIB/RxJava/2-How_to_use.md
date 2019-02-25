## 0x01、 如何使用 RxJava
原文：https://github.com/ReactiveX/RxJava/wiki/How-To-Use-RxJava

## 0x01、 Hello World!

下面的例子通过 Java, Groovy, Clojure, 和 Scala 语言实现 “Hello World” 程序(本文只有 Java)。
具体实现细节包括：从一个 String 列表创建一个可观察者然后订阅这个可观察者，并且在被观察者发出每一条字符串后通过一个方法打印 “Hello String!” 。

你能在 `/src/examples` 目录中找到每一种语言的代码实现例如：
- [RxGroovy examples](https://github.com/ReactiveX/RxGroovy/tree/1.x/src/examples/groovy/rx/lang/groovy/examples)
- [RxClojure examples](https://github.com/ReactiveX/RxClojure/tree/0.x/src/examples/clojure/rx/lang/clojure/examples)
- [RxScala examples](https://github.com/ReactiveX/RxScala/tree/0.x/examples/src/main/scala)

Java实现
```java
public static void hello(String... args) {
  Flowable.fromArray(args).subscribe(new Consumer<String>() {
      @Override
      public void accept(String s) {
          System.out.println("Hello " + s + "!");
      }
  });
}

/*------------------------------打印日志
hello("Ben", "George");
Hello Ben!
Hello George!
*/
```

## 0x02、如何使用 RxJava 完成设计
要使用 RxJava 你需要创建 Observable (它发出具体的数据项)，通过各种方式转换 Observable 从而准确获取你关心的数据(使用 Observable 操作)，然后 观察者(observe) 响应你感兴趣的队列(通过实现 Observers 或者 Subscribers ，然后将他们注册到最终得到的 Observable)。

## 0x03、 创建 Observable
要创建 Observable ，你能通过方法覆写展示 Observable 行为的 `create()` 方法，或者你也可以将通过 Observable 的操作一个已经存在的数据结构((能达到此目的的数据结构)[https://github.com/ReactiveX/RxJava/wiki/Creating-Observables])。

### 3.1、 用一个已经存在的数据结构创建 Observable
你使用 `just()` 和 `from()` 方法转换 object，list，或者数组 到 Observable 并发送他们：

```java
Observable<String> o = Observable.from("a", "b", "c");

def list = [5, 6, 7, 8]
Observable<Integer> o = Observable.from(list);

Observable<String> o = Observable.just("one object");
```

Observable 的这些转换将同步调用 `onNext()` 方法，通知订阅者，并且在通知完成后回调 onCompleted() 方法。

### 3.2、 通过 `create()` 创建一个 Observable
你能设计自己的 Observable 并通过 `create()` 方法实现同步 i/o ,计算，甚至是无限数据流:

```java
/**
 * This example shows a custom Observable that blocks 
 * when subscribed to (does not spawn an extra thread).
 */
def customObservableBlocking() {
    return Observable.create { aSubscriber ->
        50.times { i ->
            if (!aSubscriber.unsubscribed) {
                aSubscriber.onNext("value_${i}")
            }
        }
        // after sending all values we complete the sequence
        if (!aSubscriber.unsubscribed) {
            aSubscriber.onCompleted()
        }
    }
}

// To see output:
customObservableBlocking().subscribe { println(it) }
```
