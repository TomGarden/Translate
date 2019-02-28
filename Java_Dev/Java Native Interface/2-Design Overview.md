## 0x00、 概要设计
[原文](https://docs.oracle.com/javase/8/docs/technotes/guides/jni/spec/design.html)

本章主要描述 JNI 的设计问题。
本节大多数的设计问题和本地方法相关。
关于 API 的调用设计请查阅[第五章](https://docs.oracle.com/javase/8/docs/technotes/guides/jni/spec/invocation.html)

## 0x01、 JNI 接口方法和指针
native 代码通过调用 JNI 方法访问 Java VM 功能。
JNI 方法通过一个接口指针使用。
接口指针是一个指向指针的指针。
这个指针指向一个存放指针的数组，数组中的每一个指针指向一个接口方法。
每一个接口方法在数组中都有预定义的偏移量。
下图说明了接口指针的组织样式：
![2018-10-18_JNI_designa.gif](/Java_Dev/Image/2018-10-18_JNI_designa.gif)

JNI 接口被组织的像 C++ 虚方法表或者一个 COM 接口。
不将函数项生硬地连接而是使用接口表的优点在于 JNI 命名空间与本地方法相分离。
一个虚拟机能方便的提供多个版本的 JNI 方法表。
例如，假设虚拟机支持两个 JNI 方法表：
-   一种彻底地执行非法属性检查，适用于调试；
-   另一种根据 JNI 规范执行尽可能小范围的请求检查，因此它效率更高。

JNI 接口指针仅仅在当前线程有效。
因此，禁止本地方法在线程间传递接口指针。
一个实现 JNI 的虚拟机有可能在 JNI 接口指针指向的区域分配和存储线程本地数据。

本地方法将 JNI 接口指针作为一个参数接收过来。
虚拟机保证在相同线程中多次调用本地方法的情况下每个方法得到的接口指针都是相同的。
然而一个本地方法能被不同的 Java 线程调用，因此可能接收不同的 JNI 接口指针。

## 0x02、 编译、加载和链接本地方法
因为 Java 虚拟机是多线程的，因此本地库也应该被多线程感知本地编译器所编译和链接。
例如在使用 Sun Sutido 编译器的时候 `-mt` 标记应该被用于编译 C++ 代码。
对于使用 GNU gcc 编译器编译的代码， `-D_REENTRANT` 和 `-D_POSIX_C_SOURCE` 标记应该被使用。

本地方法通过 `System.loadLibrary` 方法被加载。
在下面的例子中类初始化方法加载一个特定平台的本地库，在本地库中定义了 `f` 方法：
```java
package pkg; 

class Cls {
    native double f(int i, String s);
    static {
        System.loadLibrary(“pkg_Cls”);
    }
}
```

`System.loadLibrary` 的参数代表库的名字，这个名字可以按照开发这的意图填写。
系统遵循标准，但是特定平台的方法将库名称转换为本地库名称。
例如，Solaris 系统将 `pkg_Cls` 转换为 `libpkg_Cls.so`，而 Win32 系统将 `pkg_Cls` 库名转换为 `pkg_Cls.dll` 。

开发者或许希望使用单一的库来存储所有类所需要的所有的本地方法,只要这些类使用相同的类加载器即可。
虚拟机内部持有一个为每一个类加载器加载本地库的列表。
虚拟机开发商应该以避免命名冲突为原则选择本地库名字。

一个本地库可能被虚拟机静态链接。
库和 VM 镜像的实现方式依赖于实现。
`System.loadLibrary` 或者等价的 API 必须考虑加载库。

当且仅当暴露的库名称为 `JNI_OnLoad_L` 的时候才意味着库 L 已经与 VM 组合并被定义为静态链接了。

如果静态链接库 L 暴露的方法包含 `JNI_OnLoad_L` 和 `JNI_OnLoad`，的时候， 后者将被忽略。

If a library L is statically linked, then upon the first invocation of System.loadLibrary("L") or equivalent API, a JNI_OnLoad_L function will be invoked with the same arguments and expected return value as specified for the JNI_OnLoad function.

### 2.1、 解析本地方法名
### 2.2、 Native Method Arguments
## 0x03、 Referencing Java Objects
### 3.1、 Global and Local References
### 3.2、 Implementing Local References
## 0x04、 Accessing Java Objects
### 4.1、 Accessing Primitive Arrays
### 4.2、 Accessing Fields and Methods
## 0x05、 Java Exceptions
### 5.1、 Exceptions and Error Codes
### 5.2、 Asynchronous Exceptions
### 5.3、 Exception Handling
