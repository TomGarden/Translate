## 0x00、 介绍
### 0.1、 什么是 JProfiler ?
JProfiler 是分析运行在 JVM 中东西的专业的分析工具。
当你的系统出现问题，你可以将其用于开发、质量保证和消防。

下面是关于 JProfiler 的四个主要话题：

- **方法调用**              
这通常称之为 “CPU 分析” 。
方法调用可以通过不同的方式被度量并展示可视化度量结果。
对方法调用的分析帮助你理解你的应用程序正在做什么，从而你可以优化它们。

- **分配**              
分析对象在堆中的资源分配，链式引用和垃圾回收被归类为 “内存分析” 。
这能帮你修复内存泄露问题，分配更少的临时对象和使用更少的内存。

- **线程和锁**              
线程能持有锁，例如通过 `synchronize` 一个对象。
当多个线程都关心锁的时候，JProfiler 能将线程和锁的情况可视化的展示给你。
有时候锁会被争夺，这意味着某线程必须在其获得锁之前等待。
JProfiler 提供了各个线程中各个锁的状态的监控信息。

JProfiler 的用户界面以桌面应用的方式提供。
你能通过交互的方式分析一个存活着的 JVM ，也能不通过 UI 自动分析。
分析数据会以快照的形式持久化存储，并且展示在 JProfiler 界面。
另外，命令行工具和构建工具的交互帮你自动分析绘画。

### 0.2、 接下来做什么？
文档按照阅读顺序，后面的内容建立在前面内容的基础上。(需要有序阅读)

首先，关于应用架构的技术概览将帮助你理解分析工作。

关于安装和分析 JVM 的话题将帮助你安装和运行 JProfiler 。

跟随下面提到的数据记录和快照的讨论将使你明白你所需要(针对应用)继续深入探索的方向。

后续的子章节是关于 JProfiler 的其他专业功能的介绍。
这部分作为文档结尾可以选择性阅读那些你需要的功能。

我们感谢您的反馈。
如果你感觉文档在哪里有不足之处或者你发现文档的不准确的地方，请毫不犹豫地联系我们：support@ej-technologies.com。

## 0x01、 JProfiler 架构

![JProfiler_Architecture](/www.ej-technologies.com/JPROFILER/Images/JProfiler_Architecture.png)

### 1.1、 分析代理
“JVM 工具接口 (JVNTI)” 是一个本地接口执行分析的用户用来访问信息以及添加钩子并且插入自己的工具。
这意味着至少部分分析代理必须通过本地代码来实现，因此一个 JVM 分析不是平台无关的。
JProfiler 支持所有者其官网列举的平台。
一个 JVM 分析器被作为一个本地库实现并且在程序启动的时候或者程序启动稍后被启动。
要在程序启动的时候加载它需要将 VM 参数 `-agentpath:<path to native library>` 添加到命令行中。
在极少数情况下你需要手动添加自己的参数，因为 JProfilter 将为你添加，例如作为集成 IDE 的一部分，作为作为集成向导的一部分，或者它直接启动 JVM 。
然而知道如何开始分析仍然是重要的。

如果 JVM 成功加载本地库，它调用库中一个特殊的方法给分析代理一个初始化的机会。
JProfiler 随后将打印一系列的以 `JProfiler>` 开始的诊断信息，从而让你知道分析正在执行。
如果传递了 `-agentpath` 要么分析器加载成功，要么 JVM 启动失败。

一旦分析器被加载，分析代理会请求 JVMTI 杀死所有事件，例如线程创建或者类加载事件。
这些事件直接传送到分析数据。
使用类加载事件，配置代理工具类被加载并且插入它自己的二进制并执行其度量。

使用 JProfiler UI 或者命令行(`bin/jpenable`)，JProfiler 能在一个已经运行的 JVM 中加载代理。
在这种情况下大量已经被加载的类或许必须要更新自己从而适应检测器的需求。

### 1.1、 记录数据

JProfiler 仅收集探知到的数据。
JProfiler UI 被单独启动并且通过一个 socket 连接到配置文件代理。
这意味着被检测的 JVM 运行在本地或是远程是一件无关紧要的事 —— 配置代理和 JProfiler UI 之间的通信机制总是相同的。

在 JProfiler UI 中你能控制代理收集数据，展示配置数据以及将快照保存到磁盘。
作为替代品可以通过 [MBean](https://en.wikipedia.org/wiki/Java_Management_Extensions) 控制分析代理。
命令行工具(`bin/jpcontroller`)使用的就是 MBean 。

另一个控制性能分析代理的方法是预定义动作和触发器。
这种方式性能分析代理能在无人值守的情况下被操作。
这称作 离线性能分析” 在 JProfiler 的自动性能分析章节有介绍如何使用。

### 1.2、 快照
JProfiler UI 能展示存活的性能分析数据，将记录的性能分析数据保存起来通常也是必要的。
快照也是，技能手动保存也能预定义其触发。

快照能被 JProfiler UI 打开和对比。
对自动化程序来说，命令行工具 `bin/jpexport and bin/jpcompare` 能被用于从预先存储的快照中提取数据并创建 HTML 报告。

从正在运行的 JVM 中获取堆快照的一个低开销的方式是使用 `bin/jpdump` 工具。
它使用 JVM 内建的方式保存一个 HPROF 快照，该快照能被 JProfiler 打开并且不需要加载性能分析代理。

## 0x02、 安装 JProfiler

MacOS - 略

Windows - 略

在 Linux/Unix 平台，安装器在下载后不能执行，所以你不得不在命令前添加 `sh` 。
如果你输入了 `-c` 参数，安装程序通过命令行进行安装。
命令行中携带 `-q` 参数可以完成完成无人值守安装(适用于 Windows/Linux/Unix)。
在这种情况下你能添加参数 `-dir<direction>` 用于指定安装目录。

例如`$sh jprofiler_linux_x_x_x.sh -c`

当你运行完安装程序后，他将保存一个文件 `.install4j/response.varfile` 它包含了所有用户输入。
你能通过这个文件并且在命令行数添加参数 `-varfile <path to response.varfile>` 从而完成无人值守的自动安装。

在无人值守情况下安装要设置许可信息，增加参数 `-Vjprofiler.licenseKey==<license key> -Vjprofiler.licenseName=<user name> ` 并且选项 `-Vjprofiler.licenseCompany=<company name>` 也作为命令行参数。
如果你有一个浮动的许可，使用 `FLOAT:<server name or IP address> ` 取代许可代码。

档案也以压缩包的形式提供，对于 Linux 而言：
`tar xzvf filename.tar.gz` 档案解压到一个特定的文件夹。
要启动 Jprofiler 执行 `bin/jprofiler` 。
在 Linux/Unix 平台 文件 jprofiler.desktop 能被用于整合到窗口管理器中。
例如在 Ubuntu 中你能拖动 desktop 文件到启动器边栏，从而可以在启动器边栏创建一个启动器。

### 2.1、 将性能分析代理发送到远程。
JProfiler 有两部分：用于操作快照桌面 UI 和命令行程序，和用于性能分析代理的命令行程序(用于控制性能分析 JVM 的另一种方式)。
你从网站下载的安装程序档案包含了这两部分。

对于远程性能分析，你只需要性能分析代理被安装与远端。
当你能在远程机器上简单提取携带 JProfiler 描述的档案，你或许想限制需要的文件数量，特别是当自动化部署的时候。
性能分析代理允许自由再发行，所以你能将其包装在你的应用程序或者将其安装到自定义的设备上用于排除故障。

要得到携带性能分析代理的最小包，远程继承向导提供了用于创建支持任何平台档案的操作。
在 JProfiler GUI ： `Session-> Integration Wizards -> New Server/Remote Integration` ,选择 `Remote` 选项，然后dianji  `Create archive with profiling agent` 并自定义文件输出位置。

![Server_Remote_Integration](/www.ej-technologies.com/JPROFILER/Images/Server_Remote_Integration.png)

如果有必要，JProfiler 会与 `jpenable`,`jpdump` 和 `jpcontroller` 一起下载请求本地代理库并创建一个针对指定平台的 `.tar.gz` 或者 `.zip` 档案。
当性能分析代理运行在 Java 5 或更高版本的时候，上面的文档运行只需要 Java 6 或更低版本。

下面的子文件夹实在远程机器上提取出来的。
他们是 JProfiler 安装程序针对特定平台的的一个子集。

![JProfiler_subset.png](/www.ej-technologies.com/JPROFILER/Images/JProfiler_subset.png)

### 2.2、 分析 JVM
要分析 JVM JProfiler 的性能分析代理必须首先加载到 JVM 中。
要将性能分析代理加载到 JVM 中有两种方式可以实现：在启动脚本中指定一个 `-agentpath` VM 参数，或者使用附加 API 将代理加载到正在运行的 JVM 中。

JProfiler 支持两种模式。
增加 VM 参数是常见的被集成向导使用的方式，IDE 插件和会话配置会将 JVM 和 JProfiler 一同启动。
可以附加到本地程序或者通过 SSH 附加到远程程序。

**-agentpath VM parameter**

理解 VM 参数如何