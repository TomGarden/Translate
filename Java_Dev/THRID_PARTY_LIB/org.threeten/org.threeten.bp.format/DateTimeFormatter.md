
## 0x00、 DateTimeFormatter

用于输出和解析格式化的 日期-时间 对象。

这个类提供了打印和解析的入口点。
常见的实例化方式：
-   使用格式化字符串，例如 `yyy-MMM-dd`
-   使用本地化的风格，例如 long 或者 medium
-   使用预定义内容，例如 [ISO_LOCAL_DATE](https://www.threeten.org/threetenbp/apidocs/org/threeten/bp/format/DateTimeFormatter.html#ISO_LOCAL_DATE)

更完全格式化功能通过 [DateTimeFormatterBuilder](https://www.threeten.org/threetenbp/apidocs/org/threeten/bp/format/DateTimeFormatterBuilder.html) 提供。

在需要格式化的多数情况系没必要直接使用这个类。
主要的 日期-时间 类都提供了两个方法，一个用于格式化 `format(DateTimeFormatter formatter)`， 另一个用于解析，例如 ：

    ```java
    String text = date.format(formatter);
    LocalDate date = LocalDate.parse(text, formatter);
    ```

一些格式化和解析操作依赖于 locale 。
locale 数据可以通过 `withLocale(Locale)` 函数修改，它 return 一个参数 locale 类型的 fomatter 对象。

一些应用或许需要使用旧的 Format 类进行格式化。
`toFromat()` 函数 return 实现 旧  API 的 Format 。

此类是线程安全的。