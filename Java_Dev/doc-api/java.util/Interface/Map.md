## 0x00、 Interface Map<K,V>

https://docs.oracle.com/javase/8/docs/api/java/util/Map.html

## 0x01、 Override
一个将 key 映射到 value 的对象。
map 中 key 唯一；一个 key 最多映射一个 value。

这个接口取代了抽象类 `Dictionary` 。

Map 接口提供了三个集合视图，允许将 map 的所有的 key/value 视为一个 set ，或者将整个 map 视为一个 key-value 的集合。
Map 中内容的顺序就是 map 的集合视图的迭代器(iterators)所返回元素的顺序。
一些 map 的实现(例如 TreeMap)对元素的顺序进行了指定；另一些(例如 HashMap)没有指定元素顺序(无序)。

注意： 如果将易变的对象作为 key 需要格外小心。
