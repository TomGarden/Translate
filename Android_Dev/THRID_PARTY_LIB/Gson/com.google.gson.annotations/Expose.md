## 0x00、 Expose

一个用于标记成员是否应该在序列化和反序列话过程中被公开的 annotation。

除非使用 `GsonBuilder` 构建 `Gson` 并且调用 `GsonBuilder.excludeFieldsWithoutExposeAnnotation()` 否则这个注解无效。

这里有一个例子关于 Expose 的具体意义：

```java
 public class User {
   @Expose private String firstName;
   @Expose(serialize = false) private String lastName;
   @Expose (serialize = false, deserialize = false) private String emailAddress;
   private String password;
 }
```

如果你通过 `new Gson()` 创建 Gson ， toJson() 和 fromJson() 函数将使用 `firstName,lastName,emailAddress,password` 都会参与序列化与反序列化。
然而如果你 `Gson gson = new GsonBuilder().excludeFieldsWithoutExposeAnnotation().create()` 创建 `Gson` ,`toJson()` 和 `fromJson()` **将会排除 `password` 。
这是因为 `password` 没有被 `@Expose` 标记**。
`Gson` 在序列化的时候也会排除 `lastName` 和 `emailAddress` 因为他们标记了 `serialize = false` 。
在反序列化的时候 `Gson` 会排除 `emailAddress` 因为它标记了 `deserialize = false` 。

注意，另一种具有相同效果的方式是使用 `transient` 标记 password ，这时 Gson 默认排除它。
`@Expose` 的效果体现在你希望显示指定序列化与反序列化的所有字段。