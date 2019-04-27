> Java教程是为JDK 8编写的。本页描述的示例和实践没有利用后续版本中引入的改进。

## 0x00、 [Lambda 表达式](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)

[原文](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)

匿名类有一个问题，那就是如果你的匿名类在实现上非常简单，例如一个仅包含一个方法的接口，那么这个匿名类的语法看起来不够简洁。
在这种情况下，你通需要将方法作为一个参数传递给另一个方法，例如当某一个按钮被点击的时候应该触发什么相应的动作。
Lambda 表达式使你能够通过比以往更简洁的方式来完成这个需求，可以将方法作为其他方法的参数，或者将代码做作为数据(处理之后得到的数据)。

[匿名类](https://docs.oracle.com/javase/tutorial/java/javaOO/anonymousclasses.html)介绍了如何实现一个匿名类。
虽然匿名类比非匿名类更简洁，但只有一个方法的类甚至是一个匿名类看起来还是有一点繁琐。
Lambda 表达式使你表达单方法类实例更简洁。

## 0x01、 Lambda 表达式的理想用例
假设你正在创建一个网络应用。
你希望创建一个允许管理员执行任何操作(例如发送一个消息)的功能，应用的使用者要满足一定的标准。
下面描述了使用情况是细节：
|属性|描述|
|---|----|
|名字|对一个选定的成员执行动作|
|主要人员|管理员|
|先决条件|管理员已经登录了系统|
|后置条件|只有一个成员满足指定的标准动作才会被执行|
|主要动作|1. 管理员制定对什么成员执行什么动作的标准。<br/>2. 管理员指定对选定成员执行的动作。<br/>3. 管理员选择 **Submit** 按钮。<br/>4. 系统查找所有符合标准的成员。<br/>5. 系统对符合标准的成员执行指定动作。<br/>|
|另外|1a. 管理员有一个操作预览其中包含以前点击 **submit** 按钮后匹配哪些标准的成员曾经执行了哪些动作|
|发生频率|一天多次|

假设这个应用中的成员抽象为 `Person` 类：

```java
public class Person {

    public enum Sex {
        MALE, FEMALE
    }

    String name;
    LocalDate birthday;
    Sex gender;
    String emailAddress;

    public int getAge() {
        // ...
    }

    public void printPerson() {
        // ...
    }
}
```

假设应用中的成员们存储在一个 `List<Person>` 实例中。

本节首先介绍这种用例的简单方法。
它使用了本地和匿名类改进了上面的实现，然后通过 lambda 表达式用高效的方式做了进一步的改进。
例子中的代码:[RoterTest](https://docs.oracle.com/javase/tutorial/java/javaOO/examples/RosterTest.java)

### 1.1、 Approach 1: 创建一个根据特征搜索成员的 method
一个简单的方法是创建几个方法；
每一个方法搜索匹配一个特征的成员(例如性别和年龄)。
下面的方法打印大于指定年龄的成员：

```java
public static void printPersonsOlderThan(List<Person> roster, int age) {
    for (Person p : roster) {
        if (p.getAge() >= age) {
            p.printPerson();
        }
    }
}
```

**注意：** List 是一个 Collection 序列。
collection 是一个将多个元素放在同一单元管理的对象。
Collection 用于存储，取出，操作和汇总数据。
有关于此的更多信息请查看 [Collection](https://docs.oracle.com/javase/tutorial/collections/index.html)

这种方法使你的应用不够脆弱，可能由于引入更新(例如新的数据类型)而导致不能工作。
假设你升级你的应用并且修改 `Person` 的结构(例如包含不同的成员变量)；或许记录和计算成员年龄的时候使用了不同的数据类型。
你将不得不因为这个修改而重写大量 API 。
这种方法有了诸多不必要的限制；如果你希望打印比指定年龄更小的成员要怎么般呢？

### 1.2、 Approach 2: 创建更多通用搜索方法
下面的方法比 `printPersonsOlderThan` 更通用；它打印指定年龄范围的成员：

```java
public static void printPersonsWithinAgeRange(
    List<Person> roster, int low, int high) {
    for (Person p : roster) {
        if (low <= p.getAge() && p.getAge() < high) {
            p.printPerson();
        }
    }
}
```

如果你想打印指定性别的成员，或者将性别和年龄范围进行组合搜索该怎么办？
如果你决定修改 Person 类并且增加新属性(例如关系状态和地理位置)怎么办？
虽然这个方法比 `printPersonsOlderThan` 更通用，但我们仍需要尝试创建一个被分离的方法来应对每一种可能产生的新的查询来应对搜索需求的变更(降低代码的脆弱性)。
你能通过增加额外的类来分离搜索标准。

### 1.3、 Approach 3: 在本地类中指定搜索条件代码
下面的方法打印匹配搜索标准的成员：

```java
public static void printPersons(
    List<Person> roster, CheckPerson tester) {
    for (Person p : roster) {
        if (tester.test(p)) {
            p.printPerson();
        }
    }
}
```

这个方法通过调用 `CheckPerson` 的成员函数 `tester` 来校验 `List` 中的每一个 `Person` 从而筛选出符合搜索标准的 `Person` 成员。
每次 `tester.test` return true , `printPerson` 就会被调用一次。

通过实现 CheckPerson 实例来指定搜索标准：

```java
interface CheckPerson {
    boolean test(Person p);
}
```

下面的类通过指定的方式实现了 `test` 方法。
这个方法过滤符合美国兵役役制的成员：如果 `Person` 为年龄在 [18,25] 的男性 `test` return true 

```java
class CheckPersonEligibleForSelectiveService implements CheckPerson {
    public boolean test(Person p) {
        return p.gender == Person.Sex.MALE &&
            p.getAge() >= 18 &&
            p.getAge() <= 25;
    }
}
```

要使用这个类，你需要创建一个实例：

```java
printPersons(roster, new CheckPersonEligibleForSelectiveService());
```

虽然这种方法降低了脆弱性 —— 你不必在修改 `Person` 数据结构的时候重写方法 —— 但你依然需要新增代码(一个新的接口和一个本地类)。
因为 `CheckPersonEligibleForSelectiveService` 类实现了一个接口，你可以使用一个匿名类替代本地类(这绕过了为每次搜索增加一个新类的需要)。

### 1.4、 Approach 4: 在匿名类中指定搜索标准代码
下面 `printPersons` 方法的一个参数是一个内部类，该类用于过滤符合美国兵役役制的年龄在 [18,25] 的男性。

```java
printPersons(
    roster,
    new CheckPerson() {
        public boolean test(Person p) {
            return p.getGender() == Person.Sex.MALE
                && p.getAge() >= 18
                && p.getAge() <= 25;
        }
    }
);
```

这个方法降低了代码量因为你不必为每一个新的搜索创建一个新类。
然而考虑到 `CheckPerson` 内部类只有一个方法会觉得内部类的语法笨重。
在这种情况下，你能使用 lambda 表达式提到内部类，它的介绍在下一节。

### 1.5、 Approach 5: 使用 Lambda 表达式指定搜索标准

`CheckPersion` 是一个方法接口。
一个方法接口是一个只包含抽象方法的接口。
(一个方法接口可能包含一个或多个[默认方法](https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html)或者[静态方法](https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html#static)。)
因为一个方法接口只包含一个抽象方法，当你实现它的时候你就能忽略方法名。
要在使用内部类的时候做到这一点(忽略单一方法接口的方法名)你能使用 lambda 表达式(下面代码高亮处)：

```java
printPersons(
    roster,
    //lambda 表达式开始
    (Person p) -> p.getGender() == Person.Sex.MALE
        && p.getAge() >= 18
        && p.getAge() <= 25
    //lambda 表达式结束
);
```

查看本文的 [Lambda 表达式语法](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html#syntax) 有更多关于如何定义 lambda 表达式的细节。

您可以使用标准功能接口代替CheckPerson接口，从而进一步减少所需的代码量。

### 1.6、 Approach 6: 将标准功能结构与 Lambda 表达式结合使用

重新考虑 `CheckPerson` 接口：

```java
interface CheckPerson {
    boolean test(Person p);
}
```

这是一个非常简单的接口。
它是一个方法接口因为它仅包含一个抽象方法。
这个方法需要一个参数并且返回一个 `boolean` 值。
这个方法已经简单到不值得在应用中定义了。
其实 JDK 定义了一些标准方法接口，位于 `java.util.function` 包。

例如，你能使用 `Predicate<T>` 接口替换 `CheckPerson` 。
这个接口包含方法 `boolean test(T t)` :

```java
interface Predicate<T> {
    boolean test(T t);
}
```

`Predicate<T>` 是一个 generic 接口的例子(关于此的更多信息请查看 [Generics](https://docs.oracle.com/javase/tutorial/java/generics/index.html) )
Generic 类型(例如 generic 接口)用 `<>` 指定了一个或多个参数。
这个接口仅含一个参数 ， `T` 。
当你使用实际的参数声明或者实例化一个 generic 类型的时候，你有一个参数化类型。
例如 `Predicate<Person>` ：

```java
interface Predicate<Person> {
    boolean test(Person t);
}
```


This parameterized type contains a method that has the same return type and parameters as `CheckPerson.boolean test(Person p)`. Consequently, you can use `Predicate<T>` in place of `CheckPerson` as the following method demonstrates:
-   这个参数类型包含了一个方法(该方法具有相同的返回值类型和参数类型):`CheckPerson.boolean test(Person p)` 。*这一句话表达的是什么不是完全清晰*
    一般的，你能使用 `Predicate<T>` 替代 `CheckPerson` 如下所示：

```java
public static void printPersonsWithPredicate(
    List<Person> roster, Predicate<Person> tester) {
    for (Person p : roster) {
        if (tester.test(p)) {
            p.printPerson();
        }
    }
}
```

最终，下面的方法调用和调用 `Approach 3` 中的方法调用做了相同的事情：

```java
printPersonsWithPredicate(
    roster,
    p -> p.getGender() == Person.Sex.MALE
        && p.getAge() >= 18
        && p.getAge() <= 25
);
```

这不是此方法中唯一可以使用 lambda 表达式的地方。
下面的建议是关于 lambda 表达式的其他使用方式。


### 1.7、 Approach 7: 在你的应用中始终使用 Lambda 表达式

重新考虑 `printPersonsWithPredicate` 看看你其他可以使用 Lambda 表达式的地方：

```java
public static void printPersonsWithPredicate(
    List<Person> roster, Predicate<Person> tester) {
    for (Person p : roster) {
        if (tester.test(p)) {
            p.printPerson();
        }
    }
}
```

这个方法检查 `List` 中包含的每一个 `Person` 实例是否满足 `Predicate` 参数所指定的标准 `tester`。 
如果 `Person` 实例满足 `tester` 的过滤条件，`Person` 中的 `printPerson` 方法将会被调用。

替换调用方法 `printPerson` ,你能指定当 `Person` 满足 `tester` 后所执行的不同的动作。
你能通过 Lambda 表达式指定这些动作。
假设你想要用 Lambda 表达式做模仿 `printPerson` 的存在(持有一个 Person 类型参数返回值为 void)。
记住，要使用 lambda 表达式，你需要实现一个功能接口。
在这种情况下，你需要一个功能接口(这个接口持有一个具有一个 Person 返回值为 void 的抽象方法)。
`Consumer<T>` 接口包含具有上述特点的方法 `void accent(T t)` 。
下面的 method 用 `Consuer<Person>` 参数的 `accept` 替换了 `p.printPerson()` 。

```java
public static void processPersons(
    List<Person> roster,
    Predicate<Person> tester,
    Consumer<Person> block /*新参数*/) {
        for (Person p : roster) {
            if (tester.test(p)) {
                block.accept(p); /*原 p.printPerson()*/
            }
        }
}
```

下面的代码和 `Approach 3` 中代码的调用结果相同：

```java
processPersons(
     roster,
     p -> p.getGender() == Person.Sex.MALE
         && p.getAge() >= 18
         && p.getAge() <= 25,
     p -> p.printPerson()
);
```

如果你想对成员的个人资料做更多的操作(不仅仅是打印他们)。
假设你希望验证成员的个人资料或者你希望获取联系信息？
这种情况下，你需要一个功能接口(改接口中的抽象方法应该具有返回值)。
`Function<T,R>` 接口包含方法 `R apply(T t)` 。
下面的方法通过 `mapper` 参数获得指定数据，然后执行参数 `block` 指定的动作。

```java
public static void processPersonsWithFunction(
    List<Person> roster,
    Predicate<Person> tester,
    Function<Person, String> mapper,
    Consumer<String> block) {
    for (Person p : roster) {
        if (tester.test(p)) {
            String data = mapper.apply(p);
            block.accept(data);
        }
    }
}
```

下面的方法将 roster 中满足美国兵役役制的每一个成员 email 地址取出并打印。

```java
processPersonsWithFunction(
    roster,
    p -> p.getGender() == Person.Sex.MALE
        && p.getAge() >= 18
        && p.getAge() <= 25,
    p -> p.getEmailAddress(),
    email -> System.out.println(email)
);
```

### 1.8、 Approach 8: 更广泛的使用泛型

重新考虑 `processPersonsWithFuncthon` 方法。
下面是一个通用版本它接收任何数据类型作为元素的集合参数。

```java
public static <X, Y> void processElements(
    Iterable<X> source,
    Predicate<X> tester,
    Function <X, Y> mapper,
    Consumer<Y> block) {
    for (X p : source) {
        if (tester.test(p)) {
            Y data = mapper.apply(p);
            block.accept(data);
        }
    }
}
```

此时打印满足美国兵役役制的成员的邮箱调用 `processElements` 的调用方式如下：

```java
processElements(
    roster,
    p -> p.getGender() == Person.Sex.MALE
        && p.getAge() >= 18
        && p.getAge() <= 25,
    p -> p.getEmailAddress(),
    email -> System.out.println(email)
);
```

这个方法调用执行下面的动作：

1.  从集合 source 中获取一个源对象。
    在本例中，它从 `roster` 获取的源为 `Person` 对象。
    注意集合 `roster` ，的类型为 `List` ，也是一个 `Iterable` 对象。
2.  过滤符合 `Predicate.tester` 的对象。
    在这个例子中， `Predicate` 对象是一个 lambda 表达式，具体指定什么样的成员满足美国兵役役制。
3.  将过滤到的 `Person` 对象映射为一个 `Function` 对象 `mapper` 。
    在这个例子中， `Function` 对象是一个 lambda 表达式它 return 一个成员的 email 地址。
4.  在每一个 map 对象总执行一个动作(这个动作由 ` Consumer。block` 具体指定)。
    在这个例子中 `Consumer` 对象是一个输出字符串的 lambda 表达式，它输出的字符串是从 `Functon` 对象返回回来的 e-mail 地址。

你能用集合操作替换这里的每一个动作。

### 1.9、 Approach 9: 使用 Lambda 表达式作为参数的集合操作

下面的例子使用了集合操作打印集合 `roster` 中符合美国兵役役制的成员的 e-mail 地址。

```java
roster
    .stream()
    .filter(
        p -> p.getGender() == Person.Sex.MALE
            && p.getAge() >= 18
            && p.getAge() <= 25)
    .map(p -> p.getEmailAddress())
    .forEach(email -> System.out.println(email));
```

下表将方法 `processElements` 执行的每个操作映射到相应的聚合操作：

|processElements E动作|聚合操作|
|-|-|
|获取一个持有对象的资源|`Stream<E> stream()`|
|过滤匹配 `Predicate` 对象的对象|`Stream<T> filter(Predicate<? super T> predicate)`|
|将对象映射到 `Function` 对象指定的另一个值|`<R> Stream<R> map(Function<? super T,? extends R> mapper)`|
|通过 `Consumer` 对象执行一个指定动作|`void forEach(Consumer<? super T> action)`|

这些操作 filter,map，和 forEach 都是集合操作。
集合操处理流(Stream)中的每一个元素，而不是直接处理集合中的(这也是例子中首个调用方法为 `stream` 的原因)。
一个 stream 是一个元素队列。
不想集合，它不是一个用于存储元素的数据结构。
相反，流的当前值是通过管道传递的数据源的数据(例如集合)。
管道是一个流操作队列，在这个例子中是 `filter-map-forEach` 。
另外，集合操作类型通常 lambda 表达式作为参数，这使你能子定义行为方式。

有关于此的更多信息请查看 [集合操作](https://docs.oracle.com/javase/tutorial/collections/streams/index.html)

### 1.10、 关于上述描述中未通的点 
1. `p ->` 这半句怎么理解 ， `p` 是根据 `for` 循环中的变量命名指定的吗？还是语法上允许与 `for` 循环中指定的异名。
    - `p ->` 是 `approach 5` 中的 `(Person p) ->` 的简写形式吗？

2. approach 7 ～ 9 没看懂

## 0x02、 在 GUI 应用程序中的 Lambda 表达式

在图形用户界面(GUI)应用程序中执行事件(例如按键事件，鼠标事件，滚动事件)，你通常创建一个 事件 handler(句柄) ，这个 handler 通常调用一个特殊的接口。
通常事件 handler 接口是功能接口；它们往往只有一个方法。

在 JavaFX 例子中 [HelloWorld.java](https://docs.oracle.com/javase/8/javafx/get-started-tutorial/hello_world.htm) (前面讨论的匿名类)，在下面代码中你能用 lambda 代码替换上文中的高亮代码：

```java
btn.setOnAction(new EventHandler<ActionEvent>() {

    @Override
    public void handle(ActionEvent event) {
        System.out.println("Hello World!");
    }
});
```

方法调用 `btn.setOnAction` 指定当你选中代表按钮的 btn 对象的时候将会发生什么。
这个方法请求一个需要一个对象参数 : `EventHandler<ActionEvent>` 。
`EventHandler<ActionEvent>` 接口只包含一个方法， `void handle(T event)` 。
这个接口是一个功能接口，所以你能使用下面的 lambda 表它是替换它：

```java
btn.setOnAction(
  event -> System.out.println("Hello World!")
);
```

## 0x03、 Lambda 表达式语法

一个 Lambda 由下面的部分组成：

1.  括号中一个逗号分隔的形式化参数列表。
    `CheckPerson.test` 方法包含了一个参数 `,p,` 它代表一个 `Person` 实例。

    **注意：** 你可以在 lambda 表达式中省略数据类型。
    另外，在只有一个参数的情况下你还可以省略括号。
    例如，下面的 lambd 也是有效的：

    ```java
    p -> p.getGender() == Person.Sex.MALE 
        && p.getAge() >= 18
        && p.getAge() <= 25
    ```

2. 箭头标记 ， `->`

3.  一个包含单一表达式或者语句块的 body 。
    这个例子使用了下面的表达式：

    ```java
    p.getGender() == Person.Sex.MALE 
        && p.getAge() >= 18
        && p.getAge() <= 25
    ```

    如果你指定单一的表达式，Java 运行时评估这个表达式然后返回它的值。
    或者，你能使用 return 语句。

    ```java
    p -> {
    return p.getGender() == Person.Sex.MALE
        && p.getAge() >= 18
        && p.getAge() <= 25;
    }
    ```

    一个 return 语句不是一个表达式；在一个 lambda 表达式中，你必须用花括号 `{}` 括起来才行。
    然而你不必用括号包裹 void 方法调用。
    在这里例子中，下面是一个 void lambda 表达式：

    ```java
    email -> System.out.println(email)
    ```

注意 Lambda 表达式看起来非常像一个方法声明；你可以将 lambda 表达式理解为匿名方法(没有名字的方法)。

下面的例子中 [`Calculator`](https://docs.oracle.com/javase/tutorial/java/javaOO/examples/Calculator.java) ,是一个持有多个形式化参数的 lambda 表达式的例子：

```java
public class Calculator {
  
    interface IntegerMath {
        int operation(int a, int b);   
    }
  
    public int operateBinary(int a, int b, IntegerMath op) {
        return op.operation(a, b);
    }
 
    public static void main(String... args) {
    
        Calculator myApp = new Calculator();
        IntegerMath addition = (a, b) -> a + b;
        IntegerMath subtraction = (a, b) -> a - b;
        System.out.println("40 + 2 = " +myApp.operateBinary(40, 2, addition));
        System.out.println("20 - 10 = " +myApp.operateBinary(20, 10, subtraction));    
    }
}
```

方法 operateBinary 对两个 integer 变量执行一个数学操作。
这个操作通过 `IntegerMath` 的一个实例被指定。
这个例子用 lambda 表达式定义了两个操作， `addition` 和 `subtraction` 。
这个例子打印如下内容：

```java
40 + 2 = 42
20 - 10 = 10
```

## 3.1、 个人理解
Lambda 表达式用于单方法的功能接口中。例如：

```java
//普通 java 语法实现
btn.setOnAction(new EventHandler<ActionEvent>() {

    @Override
    public void handle(ActionEvent event) {
        System.out.println("Hello World!");
    }
});

//lambda 表达式实现
btn.setOnAction(
  event -> System.out.println("Hello World!")
);
```

`event -> System.out.println("Hello World!")` 这一句话我们可以理解为，对 `event` 参数做一个 `System.out.println("Hello World!")` 操作并返回 `void`。
当然这个操作有时候是有返回值的，有的时候对参数做的操作需要 `{}` 括起来。


## 0x04、 访问括号中的本地变量

像本地和匿名类， lambda 表达式能 [捕获变量](https://docs.oracle.com/javase/tutorial/java/javaOO/localclasses.html#accessing-members-of-an-enclosing-class);它们对括号本地变量可以做相同的访问。
然而不像本地和匿名类， lambda 表达式没有任何 [Shadowing](https://docs.oracle.com/javase/tutorial/java/javaOO/nested.html#shadowing) 问题。
Lambda 表达式是词法范围的。(*本句原文 ：Lambda expressions are lexically scoped.*)
Lambda 表达式中的声明与封闭环境中的声明一样被解释。
下面的例子对此做了展示：

```java
import java.util.function.Consumer;

public class LambdaScopeTest {

    public int x = 0;

    class FirstLevel {

        public int x = 1;

        void methodInFirstLevel(int x) {
            
            // The following statement causes the compiler to generate
            // the error "local variables referenced from a lambda expression
            // must be final or effectively final" in statement A:
            // x = 99;
            
            Consumer<Integer> myConsumer = (y) -> 
            {
                System.out.println("x = " + x); // Statement A
                System.out.println("y = " + y); // 我的疑问是这个 y 是什么
                System.out.println("this.x = " + this.x);
                System.out.println("LambdaScopeTest.this.x = " +
                    LambdaScopeTest.this.x);
            };

            myConsumer.accept(x);// 疑问解答 y 是从这里传入的

        }
    }

    public static void main(String... args) {
        LambdaScopeTest st = new LambdaScopeTest();
        LambdaScopeTest.FirstLevel fl = st.new FirstLevel();
        fl.methodInFirstLevel(23);
    }
}
```

输出如下：

```java
x = 23
y = 23
this.x = 1
LambdaScopeTest.this.x = 0
```

如果你在 Lambda 表达式 myConsumer 的声明中用 x 替代参数 y ，编译器会报错：

```java
Consumer<Integer> myConsumer = (x) -> {
    // ...
}
```

编译器报错 变量 `x` 在 `methodInFirstLevel(int)` 已经被定义” 因为 Lambda 表达式不会生成一个新的 scope。
所以，你能直接访问 属性，方法，以及 scope 中的本地变量。
例如， lambda 表达式直接访问 method `methodInFirstLevel`  中的参数 `x` 。
要访问封闭类中的变量，使用官架子 `this`。
在这个例子中 `this.x` 引用的是成员变量 `FirstLevel.x` 。

然而，像本地匿名类，一个 lambda 表达式只能访问闭合块中的 `final` 修饰或者实际上是 `final` 的本地变量和参数。
例如，假设你在 `methofInFirstLevel` 声明之后立刻增加下面的语句：

```java
void methodInFirstLevel(int x) {
    x = 99;
    // ...
}
```

因为这个赋值语句，变量 `FirstLevel.x` 在事实上就不再是 `final` 的了。
那么这么做的结果就是，当 lambda 表达式 myConsumer 尝试访问 `FirstLevel.x` 变量的时候 Java 编译器会报类似这样的错误 “local variables referenced from a lambda expression must be final or effectively final” ：

```java
System.out.println("x = " + x);
```

## 0x05、 目标类型

如何确定一个 lambda 表达式的类型呢？
再次调用`过滤在 [15，18] 岁之间男性`的 lambda 表达式:

```java
p -> p.getGender() == Person.Sex.MALE
    && p.getAge() >= 18
    && p.getAge() <= 25
```

这个 Lambda 表达式用到了下面的方法：

1. Approach 3 中的 `public static void printPersons(List<Person> roster, CheckPerson tester)`
2. Approach 6 中的 `public void printPersonsWithPredicate(List<Person> roster, Predicate<Person> tester)`

当 Java 运行时调用方法 `printPersons` 的时候，它期待数据类型 `CheckPerson` ，所以 lambda 表达式的类型就是它。
然而，当 Java 运行时调用方法 `pringPersonsWithPredicate` 的时候它期待的数据类型是 `Predicate<Person>`,这时候 Lambda 表达式的类型就是它。
lambda 表达式的数据类型就是他所在方法所期望的类型。
要 lambda 表达式的类型， Java 编译器将根据上下文或者情境来决定。
因此，您只能在Java编译器可以确定目标类型的情况下使用lambda表达式：

- Variable declarations
- Assignments
- Return statements
- Array initializers
- Method or constructor arguments
- Lambda expression bodies
- Conditional expressions, ?:
- Cast expressions

### 5.1、 目标类型和方法参数

对于方法参数， Java 编译器使用另外两种语言特性决定目标类型：
重载策略和参数类型推断：

考虑下面两个功能接口( `java.lang.Runnable` 和 `java.util.concurrent.Callable<V>` )：

```java
public interface Runnable {
    void run();
}

public interface Callable<V> {
    V call();
}
```

`Runable.run` 无返回值， `Callable<V>.call` 有。

假设你已经重载方法 `invoke` 如下([`Defining Methods`](https://docs.oracle.com/javase/tutorial/java/javaOO/methods.html) 有重载的更多信息):

```java
void invoke(Runnable r) {
    r.run();
}

<T> T invoke(Callable<T> c) {
    return c.call();
}
```

下面语句中会调用哪一个方法呢？

```java
String s = invoke(() -> "done");
```

回调用 `invoke(Callbal<T>)` ,因为它有返回值；二方法 `invoke(Runable)` 没有。
在这种情况下， lambda 表达式表示 `()->done` 是 `Callbale<T>` 类型的。

## 0x06、 Serialization —— 序列化
如果 Lambda 表达式的目标类型以及它的参数是可序列化的，你能序列化一个 lambda 表达式。
然而，像内部类，强烈建议不要对 lambda 表达式进行序列化。