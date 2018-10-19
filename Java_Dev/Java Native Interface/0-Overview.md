原文：https://docs.oracle.com/javase/8/docs/technotes/guides/jni/index.html

## 0x00、 JNI

Java Native Interface(JNI) 是用于写 java 本地方法以及嵌入 JVM 中本地方法的标准编程接口。
设计它的主要目标是在特定平台实现不依赖 JVM 的本地方法库的二进制兼容(就是说在不同的平台下可以通过 JNI 技术在不依赖 JVM 的前提下调用该平台独有的 API)。

## 0x01、 [JNI 6.0 API 规范](https://docs.oracle.com/javase/8/docs/technotes/guides/jni/spec/jniTOC.html)
JNI 6.0 API 规范描述了在 Solaris,Linux,Windows 平台下 AWT(Abstract Window Toolkit) 包是如何被设计使用 JNI 机制展示对象的。

- AWT(Abstract Window Toolkit)是Java的平台獨立的視窗系統

## 0x02、 [JNI API 增强功能](https://docs.oracle.com/javase/8/docs/technotes/guides/jni/enhancements.html)
JNI 增强功能将介绍，JNI 技术的额外功能。

## 0x03、 [FAQ](http://www.oracle.com/technetwork/java/jni-j2sdk-faq-141732.html)