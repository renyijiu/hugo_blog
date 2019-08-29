---
title: "[Java源码]Float"
date: 2019-08-27T11:35:06+08:00
lastmod: 2019-08-27T11:35:06+08:00
draft: false
keywords: ["Java", "Java源码", "Float"]
description: "Java源码-Float"
tags: ["Java源码", "Float"]
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

这次我们来看看`Float`类的源代码，基于 *jdk1.8.0_181.jdk* 版本 。

<!--more-->

## 前言

因为`Float`是`float`数据类型的包装类，所以先介绍一下`float`的一些基础知识点，我们先看下官方文档的基本介绍。

> The `float` data type is a single-precision 32-bit IEEE 754 floating point. Its range of values is beyond the scope of this discussion, but is specified in the [Floating-Point Types, Formats, and Values](https://docs.oracle.com/javase/specs/jls/se7/html/jls-4.html#jls-4.2.3) section of the Java Language Specification. As with the recommendations for `byte` and `short`, use a `float` (instead of `double`) if you need to save memory in large arrays of floating point numbers. This data type should never be used for precise values, such as currency. For that, you will need to use the [java.math.BigDecimal](https://docs.oracle.com/javase/8/docs/api/java/math/BigDecimal.html) class instead. [Numbers and Strings](https://docs.oracle.com/javase/tutorial/java/data/index.html) covers `BigDecimal` and other useful classes provided by the Java platform.

Java中的`float`数据类型是个**单精度 32bit** 的 *IEEE 754* 标准的浮点数。而IEEE 754是目前最广泛使用的浮点数运算标准，定义了表示浮点数的格式（包括-0）与反常值，一些特殊数值（+/-∞与NaN），以及这些数值的"浮点数运算符"。另外规定了四种表示浮点数值的方式：单精度（32位）、双精度（64位）、延伸单精度（43位以上）与延伸双精度（79位以上）。具体更加详细的标准说明可以参考 [IEEE 754 维基百科](https://zh.wikipedia.org/zh-cn/IEEE_754)。

### 单精度（32bit）

借用维基百科的图进行说明，![](../../img/float.png)

* sign: 符号位，`1`位。0表示正数，1表示负数。
* exponent: 指数位，`8`位。单精度的指数部分是−126～+127加上偏移值127，指数值的大小从1～254（0和255是特殊值）
* fraction: 尾数位，`23位`。

举个简单的例子，现在有个"01000001001100010100011110101110"字符串进行简单的分析：

1. 符号位为`0`，表示正数
2. 指数位为`10000010`，结果为130，减去偏移量127后为3
3. 尾数位为`01100010100011110101110`，对应的值为`1.0110001010001111010111`
4. 于是得到浮点数为`1011.0001010001111010111`，转成十进制为`11.07999992370605469`，约等于`11.08`

更加详细的说明可以自行搜索，或者查看官方文档 [Floating-Point Types, Formats, and Values](https://docs.oracle.com/javase/specs/jls/se7/html/jls-4.html#jls-4.2.3)

### 规约形式的浮点数

如果浮点数中指数域的编码值在`0 < exponent <= 2^e - 2`，且在科学表示法的表示方式下，尾数部分的最高有效位为1，那么这个浮点数称为`规约形式的浮点数`。在这种情况下，尾数有一位隐含的二进制有效数字1。

### 非规约形式的浮点数

如果浮点数的指数部分的编码值为0，尾数部分不为0，那么这个浮点数被称为`非规约形式的浮点数`。一般是某个数字相当接近于0时才会使用非规约形式来表示。

## 类定义

```java
public final class Float extends Number implements Comparable<Float>
```

定义中带有`final`标识，表示是**不可继承的**，另外继承了`Number`类，实现了`Comparable`接口

## 属性

```java
public static final float POSITIVE_INFINITY = 1.0f / 0.0f;

public static final float NEGATIVE_INFINITY = -1.0f / 0.0f;

public static final float NaN = 0.0f / 0.0f;
```

* `POSITIVE_INFINITY` 表示正无穷大，正无穷的值为`0 1111 1111 000 0000 0000 0000 0000 0000`(为了方便看，中间保留了空格)；**标准定义为指数域全为1，尾数域全为0**
* `NEGATIVE_INFINITY` 表示负无穷大，对应的值为`1 1111 1111 000 0000 0000 0000 0000 0000`；**标准定义为指数域全为1，尾数域全为0**
* `NaN` 英文缩写，not a number，用来表示错误的情况，例如 `0 / 0`的问题；**标准定义为指数域全为1，尾数域不全为0**，如`0 1111 1111 000 0000 0000 0000 0000 0001`

```java
public static final float MAX_VALUE = 0x1.fffffeP+127f; // 3.4028235e+38f

public static final float MIN_NORMAL = 0x1.0p-126f; // 1.17549435E-38f

public static final float MIN_VALUE = 0x0.000002P-126f; // 1.4e-45f
```

* `MAX_VALUE` 表示了`float`类型的最大规约数为`0x1.fffffeP+127f`，这里是十六进制的表示方式，即`(2 - Math.pow(2, -23))*Math.pow(2, 127)`，结果即为`3.4028235e+38f`
* `MIN_NORMAL`表示了最小的规约数，为`0x1.0p-126f`，即`Math.pow(2, -126)`，结果为`1.17549435E-38f`
* `MIN_VALUE`表示了最小的非规约数，为`0x0.000002P-126f`，即`Math.pow(2, -149)`，结果为`1.4e-45f`

```java
public static final int MAX_EXPONENT = 127;

public static final int MIN_EXPONENT = -126;
```

* `MAX_EXPONENT` 表示了最大的指数值，为`127`
* `MIN_EXPONENT` 表示了最小的指数值，为`-126`

```java
public static final int SIZE = 32;
```

定义了`Float`的 `bit`位数

```java
public static final int BYTES = SIZE / Byte.SIZE;
```

定义了`Float`的字节数，计算值固定为4

```java
@SuppressWarnings("unchecked")
public static final Class<Float> TYPE = (Class<Float>) Class.getPrimitiveClass("float");
```

获取类信息，`Float.TYPE == float.class`两者是等价的

```java
private final float value
```

因为`Float`是`float`的包装类，所以这里就存放了对应的`float`类型的值

```java
private static final long serialVersionUID = -2671257302660747028L;
```

## 方法

### 构造方法

```java
public Float(float value) {
  this.value = value;
}

public Float(double value) {
  this.value = (float)value;
}

public Float(String s) throws NumberFormatException {
  value = parseFloat(s);
}
```

提供三种构造函数，可以传入String、float和double类型数据，其中String类型调用`parseFloat`方法完成，double类型则直接强制类型转换（可能会出现精度丢失的问题）。

### parseFloat 方法

```java
public static float parseFloat(String s) throws NumberFormatException {
  return FloatingDecimal.parseFloat(s);
}
```

通过调用`FloatingDecimal.parseFloat`方法对字符串进行转换，`FloatingDecimal`类来自于`sun.misc.FloatingDecimal`，并不在 src.zip 的源码中包含，另外代码较长，主要实现了IEEE 754标准的一些计算，这里就不复制了，如果有兴趣可以去 [FloatingDecimal](https://github.com/frohoff/jdk8u-jdk/blob/master/src/share/classes/sun/misc/FloatingDecimal.java) 查看源码，介绍一下大致逻辑：

1. 去除两边空格，判断是否为空字符串或者null，是则抛出异常
2. 判断是否`-/+`开头，确定正负号
3. 判断是否符合`NaN`字符串，是则返回
4. 判断是否符合`Infinity`字符串，是则根据正负性，返回正无穷或者负无穷
5. 判断是否`0x / 0X`开头的16进制数，是的话调用`parseHexString`处理。然后正则匹配是否符合格式要求，是的话按照规范进行转换返回
6. 处理中间的数字字符串
7. 判断是否存在对应的`e`或者`E`的科学计数，中间还需要考虑处理溢出的问题。
8. `f`或`F`结束标志判断，判断是否还有剩余字符，否返回处理结果

返回处理结果的过程比较有意思，当非0数字小于7位时，会直接进行结果处理；否则当符合一定要求时，会采用`double`类型处理，然后强制转换成 `float`类型返回结果；最复杂的就是非0数字加上指数过大，超过一个可以一步完成操作的界限时，会通过一个近似正确的答案，然后按10的幂进行缩放来递进缩小误差，当误差小于一个认可值时就返回结果，这个过程中间使用`double`类型来避免产生上溢/下溢的问题。

*这个过程的代码比较复杂，多是数学计算，不是很好理解，本文的叙述可能会有错误，如果有错误或者你有好的源代码分析相关资料，欢迎联系指出。*

### toString 方法

```java
public static String toString(float f) {
  return FloatingDecimal.toJavaFormatString(f);
}

public String toString() {
  return Float.toString(value);
}
```

两个`toString`方法，下面的方法内部实现调用了第一个方法，而第一个的实现通过`FloatingDecimal`类的方法去实现。这里的代码依旧是比较复杂的🤦‍♂️，将浮点数转换成二进制形式，然后判断是否是正负无穷以及NaN逻辑（这比较简单），然后开始处理逻辑（这里过于复杂了，有兴趣的可以自行查看对应的源码）。`NaN`会返回对应的字符串"NaN"，`Infinity`会返回对应的"Infinity"或者"-Infinity"，当输入在`10^-3 ~ 10^7`之间，会返回正常的十进制数，而不在这个范围时，会采用科学计数法表示。

### toHexString 方法

```java
public static String toHexString(float f) {
  if (Math.abs(f) < FloatConsts.MIN_NORMAL
      &&  f != 0.0f ) {// float subnormal
            // Adjust exponent to create subnormal double, then
            // replace subnormal double exponent with subnormal float
            // exponent
    String s = Double.toHexString(Math.scalb((double)f,
                                             /* -1022+126 */
                                             DoubleConsts.MIN_EXPONENT-
                                             FloatConsts.MIN_EXPONENT));
    return s.replaceFirst("p-1022$", "p-126");
  }
  else // double string will be the same as float string
    return Double.toHexString(f);
}
```

返回十六进制格式字符串，内部主要使用了`Double.toHexString`方法实现

### valueOf 方法

```java
public static Float valueOf(String s) throws NumberFormatException {
  return new Float(parseFloat(s));
}

public static Float valueOf(float f) {
  return new Float(f);
}
```

存在两个`valueOf`方法，当参数为`float`类型时，直接`new Float(f)`然后返回；对于字符串参数，调用`parseFloat`方法转换成`float`，然后new一个新的对象返回。

### isNaN 方法

```java
public boolean isNaN() {
  return isNaN(value);
}

public static boolean isNaN(float v) {
  return (v != v);
}
```

两个`isNaN`的方法，用于判断传入的`float`是否是`NaN`，第一个方法内部调用了第二个方法实现。而第二个方法内部直接使用了`(v != v)`的逻辑。这是由`NaN`相关标准决定的，`NaN`是**无序的**，所以

1. 当一个或者两个操作数为`NaN`时，`<`, `<=`,`>`, and `>=`均返回`false`
2. 如果任一操作数为`NaN`时，`==`返回`false`。特别是，如果x或者y是`NaN`，那么`(x<y) == !(x>=y)`为`false`
3. 如果任一操作数为`NaN`时，`!=` 返回`true`。特别是，当且仅当x为`NaN`时，`x != x`为`true`

具体也可以参考官方资料 [oracel](https://docs.oracle.com/javase/specs/jls/se6/html/expressions.html#5198) 的详细说明。

### isInfinite 方法

```java
public boolean isInfinite() {
  return isInfinite(value);
}

public static boolean isInfinite(float v) {
  return (v == POSITIVE_INFINITY) || (v == NEGATIVE_INFINITY);
}
```

判断一个数是不是无穷数，包括正无穷和负无穷。

### isFinite 方法

```java
public static boolean isFinite(float f) {
  return Math.abs(f) <= FloatConsts.MAX_VALUE;
}
```

判断输入的数是不是有限浮点数，通过判断输入参数绝对值是否小于最大浮点数。

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
  return value;
}

public double doubleValue() {
  return (double)value;
}
```

获取各种类型的值，内部直接强制转换成对应类型的值返回。

### hashCode 方法

```java
@Override
public int hashCode() {
  return Float.hashCode(value);
}

public static int hashCode(float value) {
  return floatToIntBits(value);
}
```

调用`floatToIntBits`方法，也就是直接将对应浮点数转换成整数作为其hashCode

### floatToRawIntBits 方法

```java
public static native int floatToRawIntBits(float value);

/*
 * Find the bit pattern corresponding to a given float, NOT collapsing NaNs
 */
JNIEXPORT jint JNICALL
Java_java_lang_Float_floatToRawIntBits(JNIEnv *env, jclass unused, jfloat v)
{
    union {
        int i;
        float f;
    } u;
    u.f = (float)v;
    return (jint)u.i;
}
```

`floatToRawIntBits`是个`native`方法，具体实现由上面提供的c语言代码实现。`union`是个数据结构，能在同一个内存空间储存不同的数据类型，也就是说同一块内存，可以表示`float`，也可以表示`int`。

结果会保留`NaN`值。正无穷结果为`0x7f800000`，负无穷结果为`0xff800000`；当参数为`NaN`时，结果会是实际的`NaN`整数值，该方法不会像`floatToIntBits`一样，对NaN进行统一值处理。

### intBitsToFloat 方法

```java
public static native float intBitsToFloat(int bits);

/*
 * Find the float corresponding to a given bit pattern
 */
JNIEXPORT jfloat JNICALL
Java_java_lang_Float_intBitsToFloat(JNIEnv *env, jclass unused, jint v)
{
    union {
        int i;
        float f;
    } u;
    u.i = (long)v;
    return (jfloat)u.f;
}
```

`intBitsToFloat`也是个`native`方法，由上方对应的c语言代码实现，具体就不赘述了。参数为`0x7f800000`时，结果为正无穷；参数为`0xff800000`，结果为负无穷；当参数为`0x7f800001 ~ 0x7fffffff`或者`0xff800001 ~ 0xffffffff`之间时，结果为`NaN`。因为对于Java而言，相同类型而不同bit模式组成的NaN数据是不可分辨的。如果你需要区分，可以使用上面的`floatToRawIntBits`方法。

值得注意的是，这个方法可能无法返回与参数数据 *bit pattern* 一致的`NaN`结果。`IEEE 754`标准区分了两种类型的`NaN`数据（`quiet NaNs and signaling NaNs`），但是在Java中这两种的处理是不可见的，所以对于某些值`floatToRawIntBits(intBitsToFloat(start)) != start`可能是会存在的。不过对于上述给出的范围已经包含了所有可能的`NaN`数据的位信息。

### floatToIntBits 方法

```java
public static int floatToIntBits(float value) {
  int result = floatToRawIntBits(value);
  // Check for NaN based on values of bit fields, maximum
  // exponent and nonzero significand.
  if ( ((result & FloatConsts.EXP_BIT_MASK) ==
        FloatConsts.EXP_BIT_MASK) &&
      (result & FloatConsts.SIGNIF_BIT_MASK) != 0)
    result = 0x7fc00000;
  return result;
}
```

基本与`floatToRawIntBits`方法一致，只是增加了对`NaN`的判断，若是`NaN`则直接返回`0x7fc00000`（**这里对`NaN`做了统一处理，所有的都返回`0x7fc00000`**）；正无穷为`0x7f800000`；负无穷为`0xff800000`；其他的值返回对应的结果。

具体看下两个判断条件：

```java
(result & FloatConsts.EXP_BIT_MASK) == FloatConsts.EXP_BIT_MASK
```

`FloatConsts.EXP_BIT_MASK` 值为 `0x7F800000`, 二进制为 `0 11111111 00000000000000000000000`，这里就是对指数域做了判断，**指数域全为1**

```java
result & FloatConsts.SIGNIF_BIT_MASK) != 0
```

`FloatConsts.SIGNIF_BIT_MASK`值为 `0x007FFFFF`，二进制为 `0 00000000 11111111111111111111111`，这里就是对尾数域进行判断，**尾数域不全为0**

两者的结合符合我们上面所说的`NaN`的标准定义。

### equals 方法

```java
public boolean equals(Object obj) {
  return (obj instanceof Float)
    && (floatToIntBits(((Float)obj).value) == floatToIntBits(value));
}
```

首先判断obj是不是`Float`对象的实例，然后通过`floatToIntBits`获取两个整数值，进行比较判断是否一致；这里注意的是当误差小于精度范围时，结果是可能返回`true`的，例如

```java
Float a = 100000f;
Float b = 100000.001f;

System.out.println(Float.floatToIntBits(a)); // 1203982336
System.out.println(Float.floatToIntBits(b)); // 1203982336
System.out.println(a.equals(b)); // true

System.out.println(new Float(0.0f).equals(new Float(-0.0f))); // false
```

### compare 方法

```java
public int compareTo(Float anotherFloat) {
  return Float.compare(value, anotherFloat.value);
}

public static int compare(float f1, float f2) {
  if (f1 < f2)
    return -1;           // Neither val is NaN, thisVal is smaller
  if (f1 > f2)
    return 1;            // Neither val is NaN, thisVal is larger

  // Cannot use floatToRawIntBits because of possibility of NaNs.
  int thisBits    = Float.floatToIntBits(f1);
  int anotherBits = Float.floatToIntBits(f2);

  return (thisBits == anotherBits ?  0 : // Values are equal
          (thisBits < anotherBits ? -1 : // (-0.0, 0.0) or (!NaN, NaN)
           1));                          // (0.0, -0.0) or (NaN, !NaN)
}
```

`compareTo`方法内部调用了`compare`实现，所以看下`compare`的具体实现。首先判断`< / >`操作，如果成立直接返回；若不成立，则表示数据存在`NaN`情况，对于`NaN`来说，`<, <=, >, and >= `判断结果都是`false`，剩下的注释已经解释的很清楚了，不赘述了。不过与`==`还是存在区别的，值得注意下。

```java
System.out.println(-0.0f == 0.0f ? true : false); // true
System.out.println(-0.0f < 0.0f ? true : false);  // false
System.out.println(-0.0f <= 0.0f ? true : false); // true
System.out.println(Float.compare(-0.0f, 0.0f));   // -1
System.out.println(Float.compare(0.0f, -0.0f));   // 1
System.out.println(Float.compare(-Float.NaN, Float.NaN)); // 0
System.out.println(Float.compare(Float.NaN, -Float.NaN)); // 0
System.out.println(Float.compare(1.0f, Float.NaN));  // -1
System.out.println(Float.compare(1.0f, -Float.NaN)); // -1
```

### sum、min、max 方法

```java
public static float sum(float a, float b) {
  return a + b;
}

public static float max(float a, float b) {
  return Math.max(a, b);
}

public static float min(float a, float b) {
  return Math.min(a, b);
}
```

这个很好理解，不过还是要注意一些特殊边界值。

```java
System.out.println(Float.max(0.0f, -0.0f)); // 0.0
System.out.println(Float.min(0.0f, -0.0f)); // -0.0
System.out.println(Float.max(Float.NaN, 1.0f));  // NaN
System.out.println(Float.min(-Float.NaN, 1.0f)); // NaN
```

## 总结

从文章各种说明也可以看出来，`Float`代码比前两次相对来说复杂多了。其中比较重要的是`IEEE 754`标准，因为实际代码中很多都是根据标准而来，如果对标准有所了解整体思路理解起来就会简单很多。另外其中一些方法的计算（如`compare`，`equals`）等也是比较有意思的，看了代码你就了解了为什么`new Float(0.0f).equals(new Float(-0.0f))`是不成立的。

另外中间因为考虑浮点数的有效位数这个问题，网上搜索了很久的资料，五花八门的，各种答案都有，不过还是看英文资料叙述的详细，有兴趣的可以看看下面提供的参考资料，**强烈推荐！！！**

## 参考资料

1. [维基百科](https://zh.wikipedia.org/wiki/IEEE_754)
2. [Single-precision_floating-point_format](https://en.wikipedia.org/wiki/Single-precision_floating-point_format)
3. [Is the most significant decimal digits precision that can be converted to binary and back to decimal without loss of significance 6 or 7.225?](https://stackoverflow.com/questions/30688422/is-the-most-significant-decimal-digits-precision-that-can-be-converted-to-binary)
4. [What's the reason why “text-float-text” guarantee 6 digit but “float-text-float” does 9?](https://stackoverflow.com/questions/47123510/whats-the-reason-why-text-float-text-guarantee-6-digit-but-float-text-float)
5. [Decimal Precision of Binary Floating-Point Numbers](https://www.exploringbinary.com/decimal-precision-of-binary-floating-point-numbers/)