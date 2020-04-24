## 0x00、 安装 Gradel

> 原文：https://gradle.org/install/

[版本列表](https://gradle.org/releases)

## 0x01、 先决条件
Gradle 能在所有主流操作系统运行，它所需要的仅仅是 8 以上版本的 Java JDK 或者 JRE 。
校验自己的设备是否已经安装了 Java ：
```terminate
$ java -version
java version "1.8.0_121"
```

## 0x02、 更多资源
- [Gradle 用户可以免费获取在线教程](https://gradle.com/training/?_ga=2.147246455.1104539424.1568698834-1164022602.1568698834)
- [自定进度的教程是使用各种语言尝试 Gradle 的不错方式](https://guides.gradle.org/)
- Gradle 新版的视觉 bulid 检查工具称为 [build scans](https://scans.gradle.com/?_ga=2.150449398.1104539424.1568698834-1164022602.1568698834)
- 最后，[Gradle Newsletter](https://newsletter.gradle.com/?_ga=2.146769396.1104539424.1568698834-1164022602.1568698834)是一个不错的保持最新版本的方式，每月都会进行恰当的修复。

## 0x03、 使用包管理器进行安装
[SDKMAIN!](http://sdkman.io/)是 类 Unix 系统上管理软件开发工具的并行版本工具。

```terminate
$ sdk install gradle 5.6.2
```

其他软件包管理器的安装方式略……

## 0x04、 手动安装
### 步骤一、下载最新版本的 Gradle
- [仅下载二进制文件](https://gradle.org/next-steps/?version=5.6.2&format=bin)
- [全部下载，包含文档和源码](https://gradle.org/next-steps/?version=5.6.2&format=all)

如果有疑问可以选择仅下载二进制文件，然后在线浏览[文档](https://docs.gradle.org/current)和[源码](https://github.com/gradle/gradle)。

### 步骤二、 解包
Linux & MacOS

```terminate
$ mkdir /opt/gradle
$ unzip -d /opt/gradle gradle-5.6.2-bin.zip
$ ls /opt/gradle/gradle-5.6.2
LICENSE  NOTICE  bin  getting-started.html  init.d  lib  media
```

### 步骤三、 配置环境变量
Linux & MacOS

```terminate
$ export PATH=$PATH:/opt/gradle/gradle-5.6.2/bin
```

### 验证安装结果
```terminate
$ gradle -v

------------------------------------------------------------
Gradle 5.6.2
------------------------------------------------------------
```

## 0x05、 更新 Gradle Wrapper

如果你有基于 Gradle (使用 Gradle Wrapper)的构建内容，可以通过运行 wrapper 任务轻松完成指定 Gradle 版本的升级。

```terminate
$ ./gradlew wrapper --gradle-version=5.6.2 --distribution-type=bin
```

如果你不确定 Gradle 是否是通过 Gradle warpper 安装的。
下面的调用将会下载和缓存指定版本的 Gradle 。
```terminate
$ ./gradlew tasks
Downloading https://services.gradle.org/distributions/gradle-5.6.2-bin.zip
...
```