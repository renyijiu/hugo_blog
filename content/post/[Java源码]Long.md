---
title: "[Java源码]Long"
date: 2019-09-30T15:06:52+08:00
lastmod: 2019-09-30T15:06:52+08:00
draft: false
keywords: ["Java", "Java源码", "Long"]
description: "Java源码 - Long"
tags: ["Java源码", "Long"]
categories: ["Java源码"]
author: "renyijiu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: false
autoCollapseToc: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: true
mathjax: false
---

这次来看看`Long`的源代码，基于 *jdk1.8.0_181.jdk* 版本，如有错误，欢迎联系指出。

<!--more-->

## 前言

`Long`是`long`基础数据类型的包装类，从源码的角度来看，整体的思路和`Integer`是极其相似的，如果没有看过`Integer`的源代码，建议可以看看 [ Java源码 - Integer ](https://blog.renyijiu.com/post/java源码integer/) 这篇文章，相关的思路已经说明清楚。这篇文章相似逻辑会引用原有的逻辑说明，不会再重复介绍了。

## 类定义

```java
public final class Long extends Number implements Comparable<Long>
```

带有`final`标识，也就是说**不可继承的**。另外继承了`Number`类，而`Number`类实现了`Serializable`接口，所以`Long`也是可以序列化的；实现了`Comparable`接口。

## 属性

```java
@Native public static final long MIN_VALUE = 0x8000000000000000L;

@Native public static final long MAX_VALUE = 0x7fffffffffffffffL;
```

* `MIN_VALUE` 表示了`Long`最小值，对应为`-2^63`
* `MAX_VALUE` 表示了`Lone`最大值，对应为`2^63 - 1`

这里的两个常量都带有`@Native`注解，表示这两个常量值字段可以被native代码引用。当native代码和Java代码都需要维护相同的变量时，如果Java代码使用了`@Native`标记常量字段时，编译时可以生成对应的native代码的头文件。

```java
public static final Class<Long> TYPE = (Class<Long>) Class.getPrimitiveClass("long");
```

获取类信息，`Long.TYPE == long.class`两者是等价的

```java
@Native public static final int SIZE = 64;
```

表示了`Long`的bit数，64位。

```java
public static final int BYTES = SIZE / Byte.SIZE;
```

表示了`Long`的字节数，计算值固定为8

```java
private final long value;
```

`Long`是`long`的包装类，这里存放了`long`类型对应的数据值信息

```java
@Native private static final long serialVersionUID = 4290774380558885855L;
```

## 内部类

```java
private static class LongCache {
  private LongCache(){}

  static final Long cache[] = new Long[-(-128) + 127 + 1];

  static {
    for(int i = 0; i < cache.length; i++)
      cache[i] = new Long(i - 128);
  }
}
```

`LongCache`是`Long`的静态内部类，内部定义了一个长度为`128+127+1=256`的数组，用于缓存常用的数字范围，避免后续使用时重新进行实例化，提升性能。

## 方法

### 构造方法

```java
public Long(long value) {
  this.value = value;
}

public Long(String s) throws NumberFormatException {
  this.value = parseLong(s, 10);
}
```

存在两个构造函数，支持传入`long`和`String`类型参数。但参数类型为`String`时，内部以十进制格式调用了`parseLong`方法进行处理实现。

### parseLong 方法

```java
public static long parseLong(String s) throws NumberFormatException

public static long parseLong(String s, int radix) throws NumberFormatException
```

存在两个`parseLong`方法，具体逻辑和`Integer`的`parseInt`类似，可以参考相关的文章介绍[parseInt 方法](https://blog.renyijiu.com/post/java源码integer/#parseint-方法)

### parseUnsignedLong 方法

```java
public static long parseUnsignedLong(String s) throws NumberFormatException {
  return parseUnsignedLong(s, 10);
}

public static long parseUnsignedLong(String s, int radix) throws NumberFormatException
```

存在两个`parseUnsignedLong`方法，第一个方法内部默认以十进制调用了第二个方法进行实现，方法的整体逻辑与`Integer`的`parseUnsignedInt`基本一致，可以参考 [parseUnsignedInt 方法](https://blog.renyijiu.com/post/java源码integer/#parseunsignedint-方法)

### getChars 方法

```java
static void getChars(long i, int index, char[] buf)
```

该方法主要的逻辑就是将输入`long`类型数据转换成字符形式放入char数组中，**不支持 Long.MIN_VALUE**。具体逻辑说明可以参考`Integer`的[getChars 方法](https://blog.renyijiu.com/post/java源码integer/#getchars-方法)

### stringSize 方法

```java
static int stringSize(long x) {
  long p = 10;
  for (int i=1; i<19; i++) {
    if (x < p)
      return i;
    p = 10*p;
  }
  return 19;
}
```

获取对应数据的字符串长度，设定最大长度19进行循环判断（因为`Long.MAX_VALUE`转换成十进制数，最大长度为19位）；从逻辑可以看出，**只支持正数**，负数的结果是错误的。

### toString 方法

```java
public String toString() {
  return toString(value);
}

public static String toString(long i)

public static String toString(long i, int radix)
```

存在三个`toString`方法，其中两个为静态方法，与`Integer`一致，可以参考[toString 方法](https://blog.renyijiu.com/post/java源码integer/#tostring-方法)

### toUnsignedString 方法

`Integer`的`toUnsignedString`方法内部实现调用了`Long`的`toUnsignedString `方法进行实现，所以我们来看看这个方法的实现。

```java
public static String toUnsignedString(long i, int radix) {
  if (i >= 0)
    return toString(i, radix);
  else {
    switch (radix) {
      case 2:
        return toBinaryString(i);

      case 4:
        return toUnsignedString0(i, 2);

      case 8:
        return toOctalString(i);

      case 10:
        /*
         * We can get the effect of an unsigned division by 10
         * on a long value by first shifting right, yielding a
         * positive value, and then dividing by 5.  This
         * allows the last digit and preceding digits to be
         * isolated more quickly than by an initial conversion
         * to BigInteger.
         */
        long quot = (i >>> 1) / 5;
        long rem = i - quot * 10;
        return toString(quot) + rem;

      case 16:
        return toHexString(i);

      case 32:
        return toUnsignedString0(i, 5);

      default:
        return toUnsignedBigInteger(i).toString(radix);
    }
  }
}
```

把`long`类型数值转换成对应的无符号字符串。因为`long`的最大值为`2^63 - 1`，正数的值转换成无符号数时依旧在这个范围内，所以直接调用`toString`方法实现。对于负数而言，根据进制的不一样，调用不同的方法进行处理。

```java
public static String toHexString(long i) {
  return toUnsignedString0(i, 4);
}

public static String toOctalString(long i) {
  return toUnsignedString0(i, 3);
}

public static String toBinaryString(long i) {
  return toUnsignedString0(i, 1);
}
```

这三个方法的内部实现均是调用了`toUnsignedString0`方法，我们来看下`toUnsignedString0`的代码实现

```java
static String toUnsignedString0(long val, int shift) {
  // assert shift > 0 && shift <=5 : "Illegal shift value";
  int mag = Long.SIZE - Long.numberOfLeadingZeros(val);
  int chars = Math.max(((mag + (shift - 1)) / shift), 1);
  char[] buf = new char[chars];

  formatUnsignedLong(val, shift, buf, 0, chars);
  return new String(buf, true);
}
```

我们来分析一下这几行代码：

```java
int mag = Long.SIZE - Long.numberOfLeadingZeros(val);
```

`mag`表示了该数字表示为二进制实际需要的位数（去除高位的0）。`numberOfLeadingZeros`方法就是获取该数字二进制格式下最高位1前面的0的个数。

```java
int chars = Math.max(((mag + (shift - 1)) / shift), 1);
```

根据`mag`和`shift`获取转换成对应进制的字符串所需的字符数（`shift`代表了进制数）。

```java
char[] buf = new char[chars];

formatUnsignedLong(val, shift, buf, 0, chars);
```

将转换的字符写入到`buf`数组中，来看下`formatUnsignedLong`方法实现。

```java
static int formatUnsignedLong(long val, int shift, char[] buf, int offset, int len) {
  int charPos = len;
  // 获取进制
  int radix = 1 << shift; 
  // 掩码，二进制都为1
  int mask = radix - 1;
  do {
    // 去除对应进制范围内的最后一位数，获取对应的字符
    buf[offset + --charPos] = Integer.digits[((int) val) & mask];
    // 无符号右移，去除上一步已经处理的值
    val >>>= shift;
  } while (val != 0 && charPos > 0);

  return charPos;
}
```

整体思路就是这样，这样就获取到了对应的字符数组，然后转换成字符串输出结果。

### toUnsignedBigInteger 方法

```java
private static BigInteger toUnsignedBigInteger(long i) {
  if (i >= 0L)
    return BigInteger.valueOf(i);
  else {
    int upper = (int) (i >>> 32);
    int lower = (int) i;

    // return (upper << 32) + lower
    return (BigInteger.valueOf(Integer.toUnsignedLong(upper))).shiftLeft(32).
      add(BigInteger.valueOf(Integer.toUnsignedLong(lower)));
  }
}
```

将`long`类型值转换成`BigInteger`类型值，当参数`>=0`时，使用`BigInteger.valueOf(i)`方法处理返回结果；如果小于0，则先拆分按照高位4字节和低位4字节进行处理，然后获取结果。

### valueOf 方法

```java
public static Long valueOf(String s, int radix) throws NumberFormatException {
  return Long.valueOf(parseLong(s, radix));
}

public static Long valueOf(String s) throws NumberFormatException {
  return Long.valueOf(parseLong(s, 10));
}

public static Long valueOf(long l) {
  final int offset = 128;
  if (l >= -128 && l <= 127) { // will cache
    return LongCache.cache[(int)l + offset];
  }
  return new Long(l);
}
```

存在三个`valueOf`方法，前两个内部实现均是调用了第三个方法。当参数在`-128~127`之间时，从`LongCache`的缓存数组中获取，否则重新实例化对象返回。所以可以看看下面的示例：

```java
Long a = Long.valueOf(108);
Long b = Long.valueOf(108);

Long c = Long.valueOf(1108);
Long d = Long.valueOf(1108);

System.out.println(a == b); // true
System.out.println(c == d); // false
```

### decode 方法

```java
public static Long decode(String nm) throws NumberFormatException
```

将对应的字符串转换成整数，支持十进制，`0x, 0X, #`开头的十六进制数，`0`开头的八进制数。代码逻辑解释可以参考 [Integer 的 decode 方法](https://blog.renyijiu.com/post/java源码integer/#decode-方法)

### getLong 方法

```java
public static Long getLong(String nm) {
  return getLong(nm, null);
}

public static Long getLong(String nm, long val) {
  Long result = Long.getLong(nm, null);
  return (result == null) ? Long.valueOf(val) : result;
}

public static Long getLong(String nm, Long val) {
  String v = null;
  try {
    v = System.getProperty(nm);
  } catch (IllegalArgumentException | NullPointerException e) {
  }
  if (v != null) {
    try {
      return Long.decode(v);
    } catch (NumberFormatException e) {
    }
  }
  return val;
}
```

三个`getLong`方法，主要是最后一个方法。传入一个配置的key以及默认值，获取对应的系统配置值，若为空或者为null，返回对应的默认值

### xxxValue 方法

```java
public byte byteValue() {
  return (byte)value;
}

public short shortValue() {
  return (short)value;
}

public int intValue() {
  return (int)value;
}

public long longValue() {
  return value;
}

public float floatValue() {
  return (float)value;
}

public double doubleValue() {
  return (double)value;
}
```

直接进行强制类型转换，返回对应的结果

### hashCode 方法

```java
public int hashCode() {
  return Long.hashCode(value);
}

public static int hashCode(long value) {
  return (int)(value ^ (value >>> 32));
}
```

`hashCode`返回`int`类型结果，首先将输入参数无符号右移32位，然后与原来的值进行异或运算，强制转换成`int`类型返回对应的结果。

### equals 方法

```java
public boolean equals(Object obj) {
  if (obj instanceof Long) {
    return value == ((Long)obj).longValue();
  }
  return false;
}
```

首先判断是不是`Long`对象的实例，然后比较两者的值是否相等；这里可以看出我们无需担心输入参数为`null`的时候会出现报错的情况。

### compare 方法

```java
public static int compare(long x, long y) {
  return (x < y) ? -1 : ((x == y) ? 0 : 1);
}
```

比较两个参数的值，三元表达式，`x < y` 时返回`-1`，`x == y` 时返回`0`，`x > y` 时返回`1`

### compareTo 方法

```java
public int compareTo(Long anotherLong) {
  return compare(this.value, anotherLong.value);
}
```

调用`compare`方法比较两个值，返回结果与`compare`方法一致

### compareUnsigned 方法

```java
public static int compareUnsigned(long x, long y) {
  return compare(x + MIN_VALUE, y + MIN_VALUE);
}
```

两个输入参数当作无符号整数进行比较，这里通过加上`MIN_VALUE`确保在范围内。有个小技巧，-1加上`MIN_VALUE`后会变成最大正数。

### divideUnsigned 方法

```java
public static long divideUnsigned(long dividend, long divisor) {
  if (divisor < 0L) { // signed comparison
    // Answer must be 0 or 1 depending on relative magnitude
    // of dividend and divisor.
    return (compareUnsigned(dividend, divisor)) < 0 ? 0L :1L;
  }

  if (dividend > 0) //  Both inputs non-negative
    return dividend/divisor;
  else {
    /*
    * For simple code, leveraging BigInteger.  Longer and faster
    * code written directly in terms of operations on longs is
    * possible; see "Hacker's Delight" for divide and remainder
    * algorithms.
    */
    return toUnsignedBigInteger(dividend).
      divide(toUnsignedBigInteger(divisor)).longValue();
  }
}
```

无符号除法，输入参数转换成无符号数相除获取结果

* 当除数小于0时，调用`compareUnsigned`比较除数和被除数，被除数 < 除数时返回0，否则返回1（取决于被除数的正负性）
* 当除数和被除数均为正数值，直接使用除法运算获取结果
* 当被除数小于0，除数大于0时，使用`toUnsignedBigInteger`转换成无符号`BigInteger`进行运算获取对应的结果

### remainderUnsigned 方法

```java
public static long remainderUnsigned(long dividend, long divisor) {
  if (dividend > 0 && divisor > 0) { // signed comparisons
    return dividend % divisor;
  } else {
    if (compareUnsigned(dividend, divisor) < 0) // Avoid explicit check for 0 divisor
      return dividend;
    else
      return toUnsignedBigInteger(dividend).
      remainder(toUnsignedBigInteger(divisor)).longValue();
  }
}
```

无符号取余，输入参数转换成无符号数进行取余获取结果。

* 当除数和被除数均大于0时，直接进行取余运算获取结果
* 当转换成无符号数，除数小于被除数时，直接返回除数作为结果
* 否则使用`toUnsignedBigInteger`方法，两者均转换成无符号`BigInteger`进行运算获取对应的结果

### highestOneBit 方法

```java
public static long highestOneBit(long i) {
  // HD, Figure 3-1
  i |= (i >>  1);
  i |= (i >>  2);
  i |= (i >>  4);
  i |= (i >>  8);
  i |= (i >> 16);
  i |= (i >> 32);
  return i - (i >>> 1);
}
```

获取最高位为1，其余为0的`long`类型值。通过位移或逻辑，将最高位右边1位设为1，然后2倍增长左移或操作，1的位数不断增加，最后1+2+4+8+16+32=63，可以确保覆盖所有可能性。然后使用`i - (i >>> 1)`保留最高位1返回结果。

### lowestOneBit 方法

```java
public static long lowestOneBit(long i) {
  // HD, Section 2-1
  return i & -i;
}
```

获取最低位1，其余位为0的值。负数以正数的补码表示，对整数的二进制进行取反码然后加1，得到的结果与输入二进制进行与操作，结果就是最低位1保留，其他位为0。

### numberOfLeadingZeros 方法

```java
public static int numberOfLeadingZeros(long i) {
  // HD, Figure 5-6
  if (i == 0)
    return 64;
  int n = 1;
  int x = (int)(i >>> 32);
  if (x == 0) { n += 32; x = (int)i; }
  if (x >>> 16 == 0) { n += 16; x <<= 16; }
  if (x >>> 24 == 0) { n +=  8; x <<=  8; }
  if (x >>> 28 == 0) { n +=  4; x <<=  4; }
  if (x >>> 30 == 0) { n +=  2; x <<=  2; }
  n -= x >>> 31;
  return n;
}
```

判断二进制格式下，最高位的1左边存在多少个0。这里使用了类似二分查找的思想，通过左右移位的操作一步步缩小1所在的bit位置范围，最后通过简单计算获取0的个数。开始增加了对特殊值0的判断。可以查看[numberOfLeadingZeros 方法](https://blog.renyijiu.com/post/java源码integer/#numberofleadingzeros-方法)中的例子加深了解

### numberOfTrailingZeros 方法

```java
public static int numberOfTrailingZeros(long i) {
  // HD, Figure 5-14
  int x, y;
  if (i == 0) return 64;
  int n = 63;
  y = (int)i; if (y != 0) { n = n -32; x = y; } else x = (int)(i>>>32);
  y = x <<16; if (y != 0) { n = n -16; x = y; }
  y = x << 8; if (y != 0) { n = n - 8; x = y; }
  y = x << 4; if (y != 0) { n = n - 4; x = y; }
  y = x << 2; if (y != 0) { n = n - 2; x = y; }
  return n - ((x << 1) >>> 31);
}
```

与上面的`numberOfLeadingZeros`方法对应，获取二进制格式下尾部的0的个数。

### bitCount 方法

```java
public static int bitCount(long i) {
  // HD, Figure 5-14
  i = i - ((i >>> 1) & 0x5555555555555555L);
  i = (i & 0x3333333333333333L) + ((i >>> 2) & 0x3333333333333333L);
  i = (i + (i >>> 4)) & 0x0f0f0f0f0f0f0f0fL;
  i = i + (i >>> 8);
  i = i + (i >>> 16);
  i = i + (i >>> 32);
  return (int)i & 0x7f;
}
```

统计二进制格式下1的数量。代码第一眼看着是懵的，都是位运算，实际里面实现的算法逻辑还是很巧妙的，着实佩服，这里就不介绍了，感兴趣的可以看下[Integer 的 bitCount 方法](https://blog.renyijiu.com/post/java源码integer/#bitcount-方法)介绍，了解其中的运算逻辑

### rotateXXX 方法

```java
public static long rotateLeft(long i, int distance) {
  return (i << distance) | (i >>> -distance);
}

public static long rotateRight(long i, int distance) {
  return (i >>> distance) | (i << -distance);
}
```

旋转二进制，`rotateLeft`将特定位数的高位bit放置低位，返回对应的数值；`rotateRight`将特定位数的低位bit放置高位，返回对应的数值。当`distance`是负数的时候，`rotateLeft(val, -distance) == rotateRight(val, distance)`以及`rotateRight(val, -distance) == rotateLeft(val, distance)`。另外，当`distance`是64的任意倍数时，实际是没有效果的 ，相当于无操作。

这里需要说一下`(i >>> -distance)`，`diatance`为正数时，右移一个负数逻辑相当于`i >>> 64+(-distance)`

### reverse 方法

```java
public static long reverse(long i) {
  // HD, Figure 7-1
  i = (i & 0x5555555555555555L) << 1 | (i >>> 1) & 0x5555555555555555L;
  i = (i & 0x3333333333333333L) << 2 | (i >>> 2) & 0x3333333333333333L;
  i = (i & 0x0f0f0f0f0f0f0f0fL) << 4 | (i >>> 4) & 0x0f0f0f0f0f0f0f0fL;
  i = (i & 0x00ff00ff00ff00ffL) << 8 | (i >>> 8) & 0x00ff00ff00ff00ffL;
  i = (i << 48) | ((i & 0xffff0000L) << 16) |
    ((i >>> 16) & 0xffff0000L) | (i >>> 48);
  return i;
}
```

看着是不是和`bitCount`有点类似，其实核心逻辑相似。先是交换相邻1位bit的顺序，再交换相邻2位bit顺序，再继续交换相邻4位bit顺序，然后交换相邻8位bit顺序，再交换中间32位bit的顺序，最后最高16位再和低16位进行交换就完成了整个过程，全程位运算甚是奇妙。

### signum 方法

```java
public static int signum(long i) {
  // HD, Section 2-7
  return (int) ((i >> 63) | (-i >>> 63));
}
```

获取符号数，若为负数，返回-1；若为0则返回0；为正数，则返回1。

### reverseBytes 方法

```java
public static long reverseBytes(long i) {
  i = (i & 0x00ff00ff00ff00ffL) << 8 | (i >>> 8) & 0x00ff00ff00ff00ffL;
  return (i << 48) | ((i & 0xffff0000L) << 16) |
    ((i >>> 16) & 0xffff0000L) | (i >>> 48);
}
```

按照输入参数二进制格式，以字节为单位进行翻转，返回对应的结果数值。

* `(i & 0x00ff00ff00ff00ffL) << 8 | (i >>> 8) & 0x00ff00ff00ff00ffL`交换相邻字节顺序
*  ` (i >>> 48) | (i << 48)`交换最高16位和最低16位bit位置。
* `((i >>> 16) & 0xffff0000L) | ((i & 0xffff0000L) << 16)`交换中间32位bit位置

### sum、max、min 方法

```java
public static long sum(long a, long b) {
  return a + b;
}

public static long max(long a, long b) {
  return Math.max(a, b);
}

public static long min(long a, long b) {
  return Math.min(a, b);
}
```

这个就不说了，很简单的方法。

## 特别说明

因为`long`是64bit，需要注意下`long`的原子性逻辑，这里是官方文档的具体说明[Non-Atomic Treatment of `double`and `long`](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.7)，相关解释说明可以参考 [double](https://blog.renyijiu.com/post/java源码double/#特别说明)文章中的详细说明。

## 总结

总的代码逻辑来看，`Long`和`Integer`的方法基本一致，方法内的实现也基本一致，如果了解了`Integer`的相关逻辑，`Long`相关代码可以快速的过一下即可；另外值得注意的是`long`的原子性问题。















