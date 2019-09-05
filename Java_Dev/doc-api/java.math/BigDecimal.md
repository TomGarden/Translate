## 0x00、 java.math.BigDecimal

不可变的，任意精度的有符号十进制数。
一个 BigDecimal 由一个任意精度的 unscaled 整数和一个 32 位的 scale 整数组成。
如果是 0 或者正数， scale 是小数点后的位数。
If negative, the unscaled value of the number is multiplied by ten to the power of the negation of the scale.
因此，BigDecimal 表示的数值是 (unscaledValue × 10-scale)。

BigDecimal 类提供的操作包括 算数运算，scale 操作，舍入，比较，哈希，格式化转换。
toString() 方法提供了 BigDecimal 的典型表示形式。

BigDecimal 类允许使用者完全控制舍入行为。
如果没有指定舍入模式，并且无法准确表示结果，就会抛出异常；
另外，计算能被按照指定的精度执行，舍入模式下的操作需要提供一个恰当的 MathhContext 对象。
在任何一种情况下， 8 种舍入模式都可用于控制舍入计算。
在这个类中使用 整数 属性(例如 ROUND_HALF_UP)代表舍入模式是过时的；应该使用枚举 RoundingMode(例如 RoundingMode.HALF_UP) 替代它。

当精度设置为 0 的 MathContext 对象(例如 MathContext.UNLIMITED)被提供的时候,算数运算是准确的，(这是在 java5 之前被支持的唯一的行为)
作为准确计算结果的推论，不使用精度为零的舍入模式对象(MathContext)，因此于此无关。
在这种情况下：精确的商有无限长的十进制扩展；例如 1÷3 。
如果 商 有无限的十进制扩展并且操作被指定返回精确结果， 会抛出 ArithmeticException 异常。
其他情况下(和其他操作一样)，精确结果被返回。

当精度设置不是 0 的时候， BigDecimal 的算数运算规则广泛兼容指定的操作模式，在指定模式下的算数操作的细节定义在 ANSI X3.274-1996 and ANSI X3.274-1996/AM 1-2000 (section 7.4)。
不像这些标准， BigDecimal 包含多种舍入模式，在 java5 之前的版本中对于除法是强制的。
任何 ANSI 标准与 BigDecimal 规则相冲突的地方以 BigDecimal 为准。

因为相同的数字可以有不同的表示方式(因为 scales 不同)，四则运算和舍入必须支持所有的数字结果(不论使用的是那种 scales 表示)。

一般的，舍入模式和精度设置决定当一个操作的准确的结果有很多数字时(当除法的时候有可能是无限长度的精确结果)如何返回受限制的数字。
首先，可以通过 MathContext 的精度指定整个的数字数量(数字中包含几个数字)；这决定了结果的精度。
数字数量的计算是以从精确结果从左数第一个非零数开始计算的。
舍入模式决定了如何舍弃末尾的数字。

对于所有的算数运算操作，操作从一个准确的中间结果展开，如果有必要的话会对这个结果根据指定舍入模式对这个结果进行舍入操作。
如果精确结果没有被返回，精确结果的数字位置将会被丢弃。
当增加对结果的舍入幅度，可能因为前面的数字是 9 从而导致一个新的数字位置被创建(就是进一导致数字位数 +1 )。
例如，要舍入的值是 999.9 要舍入得到一个三位数字，将会得到一个等于 1000 的数，表示方式是 100 X 10^1 。
在这种情况下，数字 1 是返回结果的前导数字位置。

除了逻辑上的精确结果，每一个四则运算操作有一个用于表示结果的优先考虑的 scale 。
优先考虑的 scale 对于每种操作来说如下表所示：

- 对于四则运算来说的偏好 scale

|Operation | Preferred Scale of Result                |
|-|-|
|Add       | max(addend.scale(), augend.scale())      |
|Subtract  | max(minuend.scale(), subtrahend.scale()) |
|Multiply  | multiplier.scale() + multiplicand.scale()|
|Divide    | dividend.scale() - divisor.scale()       |

这些 scale 用于被方法返回精确的四则运算结果；
另外一个精确的界限不得不使用一个更大的 scale ，因为精确的结果或许有许多位数，例如 1/32 = 0.03125  。

在舍入之前，精确的中间值 scale 是操作的首选 scale 。
如果准确的的数字结果不能代表精确的位数，舍入选择
