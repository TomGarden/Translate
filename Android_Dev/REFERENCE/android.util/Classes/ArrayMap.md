# ArrayMap

## 概述

ArrayMap 是一个一般的 key->value 映射数据结构，它设计用于提升传统 HashMap 的内存效率。
它将它的数据映射在一个数组中——integer数组保存每一个 item 的 hashcode ，一个 Object 数组保存 key/value 对。
这使你能够避免为每一个put到map中的对象创建一个 entry ，同时它也尝试积极控制数组的大小(因为增长只需要复制数组中的 entry，而不需要重新构建一个 map)。

