---
title: "[Java源码]Double"
date: 2019-08-30T11:02:48+08:00
lastmod: 2019-08-30T11:02:48+08:00
draft: false
keywords: ["Java源码", "Java", "Double"]
description: "Java源码 - Double"
tags: ["Java源码", "Double"]
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

这次来看看`Double`的源代码，基于  *jdk1.8.0_181.jdk* 版本，如有错误，欢迎联系指出。

<!--more-->

## 前言

`Double`是`double`基础数据类型的包装类，而`double`是`IEEE 754`标准的**双精度 64bit** 的浮点数，具体`IEEE 754`标准的一些信息这里就不再详细的介绍了，建议可以看看我的上一篇文章 [[Java源码\]Float](https://blog.renyijiu.com/post/java源码float/) 对此有个大致的了解，两者的逻辑基本是一致的。Java中对于十进制浮点数，`double`是默认的类型。

### 双精度（64 bit）

继续使用类似的图说明下，![](../../img/double.jpg)

* sign: 符号位，`1`位。0表示正数，1表示负数。
* exponent: 指数位，`11`位。双精度的指数部分是−1022～+1024加上偏移值1023，指数值的大小从1～2046（0和2047是特殊值）
* fraction: 尾数位，`52`位。

## 类信息

```java
public final class Double extends Number implements Comparable<Double>
```

定义中带有`final`标识，表示是`不可继承的`，另外继承了`Number`类，实现了`Comparable`接口

## 属性

```java
public static final double POSITIVE_INFINITY = 1.0 / 0.0;

public static final double NEGATIVE_INFINITY = -1.0 / 0.0;

public static final double NaN = 0.0d / 0.0;
```

* `POSITIVE_INFINITY` 表示正无穷，值为`0x7ff0000000000000L`；**标准定义指数域全为1，尾数域全为0**
* `NEGATIVE_INFINITY` 表示负无穷，值为`0xfff0000000000000L`；**标准定义指数域全为1，尾数域全为0**
* `NaN` 英文缩写，`Not-a-Number `，**标准定义为 指数域全为1，尾数域不全为0**

```java
public static final double MAX_VALUE = 0x1.fffffffffffffP+1023; // 1.7976931348623157e+308

public static final double MIN_NORMAL = 0x1.0p-1022; // 2.2250738585072014E-308

public static final double MIN_VALUE = 0x0.0000000000001P-1022; // 4.9e-324
```

* `MAX_VALUE` 最大规约数为`0x1.fffffffffffffP+1023`，这里是十六进制浮点数表示，也就是`0x7fefffffffffffffL`，也是`(2 - Math.pow(2, -52) * Math.pow(2, 1023))`，计算值为`1.7976931348623157e+308`
* `MIN_NORMAL` 最小的规约数为`0x1.0p-1022`，这里是十六进制浮点数表示，也就是`0x0010000000000000L`，也是`Math.pow(2, -1022)`，计算值为`2.2250738585072014E-308`
* `MIN_VALUE` 最小非规约数为`0x0.0000000000001P-1022`，这里是十六进制浮点数表示，也就是`0x1L`，也是`Math.pow(2, -1074)`，计算值为`4.9e-324`

```java
public static final int MAX_EXPONENT = 1023;

public static final int MIN_EXPONENT = -1022;
```

* `MAX_EXPONENT` 表示了最大的指数值，为1023
* `MIN_EXPONENT` 表示了最小的指数值，为`-1022`

```java
public static final int SIZE = 64;
```

定义了 `bit` 位数

```java
public static final int BYTES = SIZE / Byte.SIZE;
```

定义了`Double`对象的字节数，计算值固定为8

```java
@SuppressWarnings("unchecked")
public static final Class<Double>   TYPE = (Class<Double>) Class.getPrimitiveClass("double");
```

获取类信息，`Double.TYPE == double.class`两者是等价的

```java
private final double value;
```

`Double`是`double`的包装类，这里存放了对应的`double`数据值

```java
private static final long serialVersionUID = -9172774392245257468L;
```

## 方法

### 构造方法

```java
public Double(double value) {
  this.value = value;
}

public Double(String s) throws NumberFormatException {
  value = parseDouble(s);
}
```

可以传入`double`或者`String`类型参数，`String`参数的构造方法内部会调用`parseDouble`方法进行处理。

### parseDouble 方法

```java
public static double parseDouble(String s) throws NumberFormatException {
  return FloatingDecimal.parseDouble(s);
}
```

内部调用了`FloatingDecimal.parseDouble`实现具体逻辑，其中具体的处理过程和`Float`类似，可以查看 [parsefloat 方法](https://blog.renyijiu.com/post/java源码float/#parsefloat-方法) 了解，这里就不再重复叙述了。结果返回对应的`double`类型数据值。

### toString 方法

```java
public static String toString(double d) {
  return FloatingDecimal.toJavaFormatString(d);
}
```

依然是调用了`FloatingDecimal.toJavaFormatString`的方法，处理过程和`Float`也基本一致，结果返回对应的字符串格式。

### toHexString 方法

```java
public static String toHexString(double d) {
	// 判断是否是有限数值
  if (!isFinite(d) )
    // 对于 infinity 和 NaN, 直接调用 toString 返回
    return Double.toString(d);
  else {
    // 使用最大输出长度初始化StringBuilder容量
    StringBuilder answer = new StringBuilder(24);
		
    // 负数，增加符号标识
    if (Math.copySign(1.0, d) == -1.0)
      answer.append("-");            

    answer.append("0x");

    d = Math.abs(d);
		// 如果是0.0，直接输出返回
    if(d == 0.0) {
      answer.append("0.0p0");
    } else {
      // 判断是否为非规约数
      boolean subnormal = (d < DoubleConsts.MIN_NORMAL);

      // DoubleConsts.SIGNIF_BIT_MASK = 0x000FFFFFFFFFFFFFL
      // & 操作保留尾数位数据
      // | 操作是将最高位设为1，为了保留指数位的0，保留原来的长度，因为是个long类型整数
      long signifBits = (Double.doubleToLongBits(d)
                         & DoubleConsts.SIGNIF_BIT_MASK) |
        0x1000000000000000L;
	
      // 规约数为1.开头，非规约数为0.开头
      answer.append(subnormal ? "0." : "1.");

      // 使用Long.toHexString获取十六进制字符串，提取尾数位对应的字符串信息
      // 判断如果全为0，使用一个0替换
      // 如若不是，去除字符串尾部的所有0
      String signif = Long.toHexString(signifBits).substring(3,16);
      answer.append(signif.equals("0000000000000") ? // 13 zeros
                    "0":
                    signif.replaceFirst("0{1,12}$", ""));

      answer.append('p');
			
      // DoubleConsts.MIN_EXPONENT = -1022
      // 如果是非规约数，使用最小的指数位替换
      // 规约数，获取对应的指数值替代
      answer.append(subnormal ?
                    DoubleConsts.MIN_EXPONENT:
                    Math.getExponent(d));
    }
    return answer.toString();
  }
}
```

整体的逻辑在代码注释中进行了说明，清晰且简单，结果返回对应的十六进制字符串。

### valueOf 方法

```java
public static Double valueOf(double d) {
  return new Double(d);
}

public static Double valueOf(String s) throws NumberFormatException {
  return new Double(parseDouble(s));
}
```

存在两个`valueOf`方法，当参数为`double`类型时，直接`new Double(d)` 然后返回；对于字符串参数，调用`parseDouble`转换成`double`数据值，然后new一个新对象返回。

### isNaN 方法

```java
public static boolean isNaN(double v) {
  return (v != v);
}

public boolean isNaN() {
  return isNaN(value);
}
```

判断是否是`NaN`，使用`(v != v)`判断；具体`NaN`的规则描述可以参考 [isNaN 方法](https://blog.renyijiu.com/post/java%E6%BA%90%E7%A0%81float/#isnan-%E6%96%B9%E6%B3%95)

### isInfinite 方法

```java
public static boolean isInfinite(double v) {
  return (v == POSITIVE_INFINITY) || (v == NEGATIVE_INFINITY);
}

public boolean isInfinite() {
  return isInfinite(value);
}
```

判断是不是无穷数，包含正无穷和负无穷

### isFinite 方法

```java
public static boolean isFinite(double d) {
  return Math.abs(d) <= DoubleConsts.MAX_VALUE;
}
```

通过输入参数绝对值是否小于`double`类型的最大值，判断是不是有限数

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
  return (long)value;
}

public float floatValue() {
  return (float)value;
}

public double doubleValue() {
  return value;
}
```

返回对应类型的值，直接进行强制类型转换

### hashCode 方法

```java
@Override
public int hashCode() {
  return Double.hashCode(value);
}

public static int hashCode(double value) {
  long bits = doubleToLongBits(value);
  return (int)(bits ^ (bits >>> 32));
}
```

`>>>`为无符号右移，高位以0补齐。`(bits ^ (bits >>> 32))`逻辑为高32位与低32位异或计算返回`int`整数值作为hashCode

### longBitsToDouble 方法

```java
public static native double longBitsToDouble(long bits);
```

`longBitsToDouble`是个`native`方法，由c代码实现。返回对应`double`数据值

* 参数为`0x7ff0000000000000L`时，结果为正无穷

* 参数为`0xfff0000000000000L`时，结果为负无穷
* 参数在`0x7ff0000000000001L ~ 0x7fffffffffffffffL`或者`0xfff0000000000001L ~ 0xffffffffffffffffL`之间时，结果为`NaN`。

### doubleToRawLongBits 方法

```java
public static native long doubleToRawLongBits(double value);
```

`doubleToRawLongBits `是个`native`方法，由对应的c代码实现。

结果会保留`NaN`值，正无穷结果为`0x7ff0000000000000L`；负无穷结果为`0xfff0000000000000L`；当参数为`NaN`时，结果会是输入参数对应的实际整数值，该方法不会像`doubleToLongBits`，对`NaN`进行统一的返回值处理

### doubleToLongBits 方法

```java
public static long doubleToLongBits(double value) {
  long result = doubleToRawLongBits(value);
  if ( ((result & DoubleConsts.EXP_BIT_MASK) ==
        DoubleConsts.EXP_BIT_MASK) &&
      (result & DoubleConsts.SIGNIF_BIT_MASK) != 0L)
    result = 0x7ff8000000000000L;
  return result;
}
```

基本与`doubleToRawLongBits`方法一致，只是增加了对`NaN`的判断。若是`NaN`则直接返回`0x7ff8000000000000L`(**对所有的`NaN`值进行了统一返回值处理**)。这里识别`NaN`的逻辑符合**指数域全为1，尾数域不全为0**的标准规范，具体说明可以参考 [floatToIntBits 方法](https://blog.renyijiu.com/post/java源码float/#floattointbits-方法) 说明。

### equals 方法

```java
public boolean equals(Object obj) {
  return (obj instanceof Double)
    && (doubleToLongBits(((Double)obj).value) ==
        doubleToLongBits(value));
}
```

首先判断是不是`Double`对象实例，然后通过`doubleToLongBits`获取两个对应的长整型数，判断两者是否一致；值得注意的是一些特殊值的判断逻辑。

```java
System.out.println(new Double(0.0d).equals(new Double(-0.0d))); // false
System.out.println(new Double(Double.NaN).equals(new Double(-Double.NaN))); // true
```

### compare 方法

```java
public int compareTo(Double anotherDouble) {
  return Double.compare(value, anotherDouble.value);
}

public static int compare(double d1, double d2) {
  if (d1 < d2)
    return -1;           // Neither val is NaN, thisVal is smaller
  if (d1 > d2)
    return 1;            // Neither val is NaN, thisVal is larger

  // Cannot use doubleToRawLongBits because of possibility of NaNs.
  long thisBits    = Double.doubleToLongBits(d1);
  long anotherBits = Double.doubleToLongBits(d2);

  return (thisBits == anotherBits ?  0 : // Values are equal
          (thisBits < anotherBits ? -1 : // (-0.0, 0.0) or (!NaN, NaN)
           1));                          // (0.0, -0.0) or (NaN, !NaN)
}
```

与`Float`一致，可以参考 [compare方法](https://blog.renyijiu.com/post/java%E6%BA%90%E7%A0%81float/#compare-%E6%96%B9%E6%B3%95) 段落说明，需要注意的依然是`-0.0`，`0.0`，`Double.NaN`和`-Double.NaN`这类的特殊值，可以自行编写几个进行测试一下。

### sum、min、max方法

```java
public static double sum(double a, double b) {
  return a + b;
}

public static double max(double a, double b) {
  return Math.max(a, b);
}

public static double min(double a, double b) {
  return Math.min(a, b);
}
```

逻辑很简单，依旧需要注意的是`-0.0`，`0.0`，`Double.NaN`和`-Double.NaN`这类的特殊值，可以自行测试下结果，也许会出乎你的意料哦。

## 特别说明

因为`double`是64bit，需要注意下`double`的原子性逻辑，这里是官方文档的具体说明[Non-Atomic Treatment of `double`and `long`](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.7)，引用解释一下：

> For the purposes of the Java programming language memory model, a single write to a non-volatile `long`or`double`value is treated as two separate writes: one to each 32-bit half. This can result in a situation where a thread sees the first 32 bits of a 64-bit value from one write, and the second 32 bits from another write.
>
> Writes and reads of volatile`long`and `double`values are always atomic.
>
> Writes to and reads of references are always atomic, regardless of whether they are implemented as 32-bit or 64-bit values.
>
> Some implementations may find it convenient to divide a single write action on a 64-bit `long`or `double`value into two write actions on adjacent 32-bit values. For efficiency's sake, this behavior is implementation-specific; an implementation of the Java Virtual Machine is free to perform writes to `long`and `double`values atomically or in two parts.
>
> Implementations of the Java Virtual Machine are encouraged to avoid splitting 64-bit values where possible. Programmers are encouraged to declare shared 64-bit values as `volatile`or synchronize their programs correctly to avoid possible complications.

出于Java编程语言内存模型的原因，对于没有`volatile`修饰的`long`或者`double`的单个写操作，会被分成两次写操作，每次对32位操作。因此可能会导致线程会读到来自不同线程写入的32位数据组合的错误结果。

对于`volatile`修饰的`long`和`double`而言，写和读操作都是原子的。

对于引用的读写，不管是32位或者64的数据值，都是原子操作。

一些实现方案中也许会发现将64位数据的单次写操作分成两次相邻32位数据的写操作很方便。出于效率的缘故，这种是比较特殊的实现；JVM的实现可以自由的选择对`long`和`value`的写入采用原子逻辑或者分成两步。

鼓励JVM的实现在可能的情况下避免拆分64位的逻辑。对于程序员而言，鼓励在共享的64位值上添加`volatile`或者`synchronize`的声明修饰，避免复杂问题的出现。

从上面的描述可以看出来，原子性的问题是可能存在的。不过对于现在绝大部分的64位的机器以及使用64位的JVM时，这个问题一般是忽略的。但是当你使用的环境不符合要求时，**请注意这个问题的存在**。

## 总结

总的代码逻辑来看，`Double`和`Float`的逻辑基本一致，因为都是`IEEE 754`标准的浮点数，主要还是使用的bit数不同带来的一些差距。如果你已经了解了`float`，那再理解这个其实很简单。









