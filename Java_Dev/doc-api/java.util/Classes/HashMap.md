Class HashMap<K,V>

java.lang.Object
    java.util.AbstractMap<K,V>
        java.util.HashMap<K,V>

Type Parameters:
- K - the type of keys maintained by this map
- V - the type of mapped values

All Implemented Interfaces:
    Serializable, Cloneable, Map<K,V>

Direct Known Subclasses:
    LinkedHashMap, PrinterStateReasons

HashMap 是对 Map 接口的实现。
这个实现提供了所有可选的 map 操作，包含键/值为 null 的情况。(在不考虑 HashMap 的非同步性和允许null的特性的情况下，和 HuasTable 是类似的)。
这个类不保证元素的顺序，特别是不能保证随着时间变化元素的顺序不变。

在假定 hush 方法在队列中均匀的放置元素，那么这个实现在基本操作(get 和 put)的操作上时间是恒定的。
迭代器收集按照 HashMap 实例的容量(队列长度)加上尺寸(元素个数)采集请求时间。
因此在考虑性能的前提下不要将容量设置的太大(或者太小)是很重要的。

一个 HashMap 实例有两个影响其性能的参数， 初始化容量(initial capacity) 和 加载因子(load factor) 。
capacity 是 hash table 的容量，在 hash table 被创建的时候指定。
load factor 是用于度量在何种情况下 hash table 的容量应该增加的。
当元素实体的个数超过 hash table 的容量和加载因子的时候，hash table 被重新构建从而使 hash table 的容量×2。

一般情况下 load factor 是 0.75 的时候时间和控件效率最高。
更高的加载因子会减少空间成本增加查询成本(这些影响在 HashMap 中的多数操作都被影响，包括 get 和 put)。
当设置初始容量的时候，加载因子和期望元素数是应该被考虑的因素，从而做到使需要重新计算 hash 的元素最少。
如果初始容量大于初始容量与加载因子的乘积，不需要重新计算 hash 值。

如果很多 map 存储在 HasMap 实例中，创建一个容量足够大的实例比随着容量的增大自动扩容的效率更高。
注意，许多具有相同的 hashCos() 方法的大量 key 是降低 hash table 性能的可靠方法。
为了改善影响，当键是Comparable时，此类可以使用键之间的比较顺序来帮助打破关系。

意见需要注意的重要的事是，HashMap 是非 synchronized 的。
如果多线程同时访问一个 hash map ，且至少有一个线程修改 map 的结构，那么必须在外部设置 synchronized(结构的修改相关操作有：增加/删除 map 元素。以 key 为线索修改一个已经存在于容器中的map 的 value 不算对结构的修改)。
这写操作通常完成在有一个包含目标 map 并且被同步的对象之内。 
如果没有这样一个对象存在， map 应该使用 Collections.synchronizedMap() 方法包裹。
其最好的实现实在创建对象的时候，就提供可用于异步访问的 map 。

```java
Map m = Collections.synchronizedMap(new HashMap(...));
```

如果 map 的结构在其 迭代器被创建之后 被修改了，那么迭代器返回这个类的所有集合方法都会失败，通过任何方式调用迭代器中被移除的方法都会抛出异常 ConcurrentModificationException。
因此，面对同步修改 迭代器会快速失败和被清除，而不是冒风险继续运行(非确定的行为即是风险)。

注意，迭代器的快速失败的方法不保证不变化，一般来说，