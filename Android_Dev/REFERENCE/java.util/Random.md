
## 0x00、 Override
本类的实例被用于生成伪随机数。
这个类使用一个 48bit 的种子，这个种子被线性同余公式进行修改(See Donald Knuth, The Art of Computer Programming, Volume 2, Section 3.2.1.)

如果两个 Random 实例通过相同的种子被创建，并且创建出的实例执行相同的方法序列相同，他们将返回相同的数字序列。(也就是说所产生的随机数在 **某种情况** 下是可预测的)
为了保持这种属性，为 Random 指定了特定的算法。
为了保证 Java 代码的绝对可移植性，Java 实现必须为 Random 使用这里展示的算法。
然而 Random 的子类允许使用其他算法，只要他们遵守所有方法的规约。

算法实现通过类 Random 的 protected 方法，
Random 类的 protected 方法实现算法，每次调用可以生成多达 32 个伪随机数的位。
>The algorithms implemented by class Random use a protected utility method that on each invocation can supply up to 32 pseudorandomly generated bits.

很多应用通过 Math.random() 进行简单应用。

java.util.Random 是线程安全的。
然而，跨线程使用相同的 Random 实例或许会造成资源竞争。
在多线程环境下考虑使用 [ThreadLocalRandom](https://developer.android.com/reference/java/util/concurrent/ThreadLocalRandom.html)

java.util.Random 不是密码安全的。
对安全性敏感的应用请考虑使用 [SecureRandom](https://developer.android.com/reference/java/security/SecureRandom.html),他能获取密码安全的伪随机数发生成生器。

## public int nextInt (int bound)
随机返回 [0,bound) 范围内的一个数，每一个数被选中的概率均相同。
