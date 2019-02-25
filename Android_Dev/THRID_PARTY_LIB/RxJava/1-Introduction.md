## 0x00、 Home-Introduction RxJava

原文:https://github.com/ReactiveX/RxJava/wiki

RxJava 是 ReactiveX(Reactive Extensions) 基于 Java 虚拟机的实现。
ReactiveX 通过观察者模式实现的基于事件的异步组件库。

要了解关于 ReactiveX 的更多相关信息请移步：[Introduction to ReactiveX](https://github.com/ReactiveX/RxJava/wiki)。

## 0x01、 RxJava 很轻
RxJava 努力更轻量级。
作为一个 JAR 它聚焦于实现抽象的观察者以及关心更高阶的函数。

## 0x02、 RxJava 有多语种的实现
RxJava 支持基于 Java 6 或更高 java 版本的虚拟机实现，例如 Groovy, Clojure, JRuby, Kotlin 和 Scala 。

RxJava 被用于比 Java/Scala 更多的语言环境，他被设计于尊重每一种基于 JVM 的语言。(这是 RxJava 仍在努力做的事)

## 0x03、 RxJava 库
下述外部库可以和 RxJava 一起使用：
- Hystrix 容错和批量标准库。
- Camel RX 提供一个简单的方式去重用任何 Apache camel 组件、协议、传输和数据格式。
- rxjava-http-tail 允许你查看 HTTP log，例如`tail -f`
- mod-rxvertx-Extension for VertX 使用 RxJava 库为 Reactive Extensions(RX) 提供支持。
- rxjava-jdbc 使用带有jdbc连接的RxJava来传输ResultSets并执行语句的功能组合
- rtree - immutable in-memory R-tree and R*-tree with RxJava api including backpressure

## 0x04、 关于如何集成
https://github.com/ReactiveX/RxJava/wiki/Getting-Started

