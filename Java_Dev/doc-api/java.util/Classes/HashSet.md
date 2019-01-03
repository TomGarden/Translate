## 0x00、 [HashSet](https://docs.oracle.com/javase/8/docs/api/)
本类实现 Set 接口，由哈希表支持(实际上是一个 HashMap 实例)。
它不保证集合的迭代顺序；
特别的，它不保证集合顺序不会随着时间的变化二保持不变。
这个类允许 null 元素。

假设散列函数在序列中正确的分配元素，这个类的操作(add, remove, contains 和 size)不会随着时间的变化而变化。

未完

```java
    public void test() {
        Set<Point> pointSet = new HashSet<>();
        Point point = new Point(1, 2);
        pointSet.add(new Point(point));
        pointSet.add(new Point(point));

        for (Point p : pointSet) {
            LogUtil.i(p.toString());
        }
    }

    //输出只有一个元素
```