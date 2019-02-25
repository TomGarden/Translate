## 0x00、 关于 Lambda / Kotlin / Java / C++ / 闭包

## 0x01、 关于 java8 的 Lambda
[Java8 新特性](https://www.oracle.com/technetwork/java/javase/8-whats-new-2157071.html)一文中对于 Lambad 的描述为：

### 1.1、 [Java 编程语言](https://docs.oracle.com/javase/8/docs/technotes/guides/language/enhancements.html#javase8)
    
- Lambda 表达式，一个新语言特性，已经在这次发布中被介绍。
  它允许开发者将功能视为方法参数，或者将代码视为数据。
  Lambda 表达式允许您更紧凑地表达单方法接口（称为功能接口）的实例。

- Lambda 表达式允许你将一个行为封装成一个单元并传送给其他代码。
  当一个程序被执行的时候，或者一个程序出错的时候，如果你想确保一个集合中的每一个元素都被执行一个动作，你能使用 lambda 表达式。
  Lambda 表达式通过下面的功能被支持。

  - 紧凑的[方法引用](http://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html),已经有名字的方法具有易于理解的 lambda 表达式。
  - [默认方法](http://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html)使你能向你的库中的接口添加新功能并且确保对相同接口的旧版本二进制兼容。
    已经被实现的接口方法在方法签名之前有一个 `default` 关键字。
    你也能在接口中定义静态方法。
  - [在Java SE 8中利用Lambda表达式和流的新增和增强的API](https://docs.oracle.com/javase/8/docs/technotes/guides/language/lambda_api_jdk8.html)描述了利用lambda表达式和流的新增和增强类。

    - 方法引用为已经有名字的方法提供易读 lambda 表达式。

### 1.2、 [并发](http://docs.oracle.com/javase/8/docs/technotes/guides/concurrency/changes8.html)

    - 已将方法添加到 `java.util.concurrent.ConcurrentHashMap` 类中，以支持基于新添加的流工具和lambda表达式的聚合操作。