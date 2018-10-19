## 0x00、 Android Init Language

原文位置： `AOSP/system/core/init/readme.txt`

Android Init Language 包含 4 中类型的语法: Actions ，Commands，Services 和 Options 。

这些语法都是面向行(line-oriented)的，行中包含由空格分离的标记。
C 风格的反斜杠转义可用于将空格插入标记中。
双引号可用于防止空格将文本分成多个标记。
当反斜杠位于行末的时候，可能适用于折叠该行的。

行以 “#” 开头(允许前导空格)，是注释。

Actions 和 Service 隐式声明一个新的部分。
所有的注释或者选项属于最近声明的部分。
在第一部分之前的注释或选项会被忽略。

Actions 和 Services 有唯一的名字。
如果声明了名字冲服务的 Action 或者 Services 后声明的将被忽略并发生错误(???将被覆盖)


## 0x01、 Actions

## 0x02、 Services
Services 是程序启动语法(也可以设置当某个程序退出时是否重启它)。
Services 的使用形式如下：

```cmd
service <name> <pathname> [ <argument> ]* 
    <option>
    <option>
    ……
```

## 0x03、 Options
Options 是 Servicess 的调节器。
他影响着何时如何初始化运行 service 。

### 3.1、 `critical`
  This is a device-critical service. If it exits more than four times in
  four minutes, the device will reboot into recovery mode.

disabled
  This service will not automatically start with its class.
  It must be explicitly started by name.

setenv <name> <value>
  Set the environment variable <name> to <value> in the launched process.

### 3.4、 `socket <name> <type> <perm> [ <user> [ <group> [ <seclabel> ] ] ]`
  Create a unix domain socket named /dev/socket/<name> and passits fd to the launched process. 
   <type> must be "dgram", "stream" or "seqpacket".
  User and group default to 0.
  'seclabel' is the SELinux security context for the socket.
  It defaults to the service security context, as specified by seclabel or computed based on the service executable file security context.

  创建一个名为 `/dev/aocket/<name>` 的 unix 域名 socket   并且将它的 `fd` 传送给启动程序。
  `<type>` 必须是 “dgram” ,"stream" 或者 "seqpacket" 其中之一。
  `user` 和 `group` 默认是 0。
  ‘seclabel’ 是 socket 的 SELinux 安全上下文。
  它默认是 service 安全上下文，由 ‘seclabel’ 或者基于 service 执行文件安全上下文计算得到。

user <username>
  Change to username before exec'ing this service.
  Currently defaults to root.  (??? probably should default to nobody)
  Currently, if your process requires linux capabilities then you cannot use
  this command. You must instead request the capabilities in-process while
  still root, and then drop to your desired uid.

group <groupname> [ <groupname> ]*
  Change to groupname before exec'ing this service.  Additional
  groupnames beyond the (required) first one are used to set the
  supplemental groups of the process (via setgroups()).
  Currently defaults to root.  (??? probably should default to nobody)

seclabel <seclabel>
  Change to 'seclabel' before exec'ing this service.
  Primarily for use by services run from the rootfs, e.g. ueventd, adbd.
  Services on the system partition can instead use policy-defined transitions
  based on their file security context.
  If not specified and no transition is defined in policy, defaults to the init context.

oneshot
  Do not restart the service when it exits.

class <name>
  Specify a class name for the service.  All services in a
  named class may be started or stopped together.  A service
  is in the class "default" if one is not specified via the
  class option.

  为 service 指定一个 class name 。
  如果没有指定，默认的 name 值为 “default” 。

onrestart
  Execute a Command (see below) when service restarts.

writepid <file...>
  Write the child's pid to the given files when it forks. Meant for
  cgroup/cpuset usage.

## Reference
1. https://blog.csdn.net/cs0301lm/article/details/39646301