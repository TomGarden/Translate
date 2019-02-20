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
使用 RxJava 创建被观察者(它发出具体的数据项)，被观察者通过变量的方式精准的传送数据。