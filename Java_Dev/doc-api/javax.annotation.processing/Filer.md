## [Filer](https://docs.oracle.com/javase/8/docs/api/javax/annotation/processing/Filer.html)

这个接口用于通过注解程序创建一个新的文件。
通过这个接口创建的文件对 annotation processing tool(APT) 工具来说是可见的，从而更方便 APT 对创建文件的管理和后续操作。
通过 Writer 或 OutputStream 向文件写入内容并调用 close 方法后， APT 将对创建的文件做后续的处理。
文件被区分成三种类型：源文件(java 文件),字节码文件(class 文件),辅助类资源文件。

