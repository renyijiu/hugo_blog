---
title: "[Java源码]Short"
date: 2019-09-10T18:50:50+08:00
lastmod: 2019-09-10T18:50:50+08:00
draft: false
keywords: ["Java源码", "Java", "short", "源码"]
description: "Java源码 - Short"
tags: ["Java源码", "Short"]
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

这次我们来看看`Short`的源代码，基于 *jdk1.8.0_181.jdk* 版本，如有错误，欢迎联系指出。

<!--more-->

## 类信息

```java
public final class Short extends Number implements Comparable<Short>
```

带有`final`标识，也就是说**不可继承的**。另外继承了`Number`类，而`Number`类实现了`Serializable`接口，所以`Integer`也是可以序列化的；实现了`Comparable`接口。

## 属性

```java
public static final int SIZE = 16;
```

表示了`Short`类型的bit数，16位

```java
public static final int BYTES = SIZE / Byte.SIZE;
```

表示了`Short`类型的字节数，计算固定值为2

```java
public static final short   MIN_VALUE = -32768;

public static final short   MAX_VALUE = 32767;
```

* `MIN_VALUE` 表示了最小值，为-32768，即`-2^15`
* `MAX_VALUE` 表示了最大值，为32767，即`2^15 - 1`

```java
private final short value;
```

因为`Short`是`short`基本数据类型的包装类，所以这里存放了对应的值

```java
public static final Class<Short>    TYPE = (Class<Short>) Class.getPrimitiveClass("short");
```

获取类信息，`Short.TYPE == short.class` 两者是等价的

```java
private static final long serialVersionUID = 7515723908773894738L;
```

## 内部类

```java
private static class ShortCache {
  private ShortCache(){}

  static final Short cache[] = new Short[-(-128) + 127 + 1];

  static {
    for(int i = 0; i < cache.length; i++)
      cache[i] = new Short((short)(i - 128));
  }
}
```

内部定义了一个长度`128+127+1=256`的数组，缓存了从`-128~127`的值。当值在这个范围中时，可以直接从数组中获取对应的Short对象，避免再次实例化，提升一定的性能。

## 方法

### 构造函数

```java
public Short(short value) {
  this.value = value;
}

public Short(String s) throws NumberFormatException {
  this.value = parseShort(s, 10);
}
```

存在两个构造函数，支持`short`和`String`类型参数。当参数为`String`类型时，内部实现调用了`parseShort`方法，下面来看看`parseShort`的具体实现逻辑。

### parseShort 方法

```java
public static short parseShort(String s) throws NumberFormatException {
  return parseShort(s, 10);
}

public static short parseShort(String s, int radix)
  throws NumberFormatException {
  int i = Integer.parseInt(s, radix);
  if (i < MIN_VALUE || i > MAX_VALUE)
    throw new NumberFormatException(
    "Value out of range. Value:\"" + s + "\" Radix:" + radix);
  return (short)i;
}
```

存在两个`parseShort`方法，对应不同类型参数。第一个方法内部默认以十进制形式调用了第二个方法实现，所以具体来看看第二个方法的逻辑。首先调用了`Integer.parseInt(s, radix)`方法获取String参数对应的数值（具体此方法实现逻辑可以参考 [Integer的parseInt 方法](https://blog.renyijiu.com/post/java%E6%BA%90%E7%A0%81integer/#parseint-%E6%96%B9%E6%B3%95)），然后判断整数是否在`Short`的数值范围内，是则进行类型转换返回结果；否则抛出对应的异常。

### toString 方法

```java
public static String toString(short s) {
  return Integer.toString((int)s, 10);
}

public String toString() {
  return Integer.toString((int)value);
}
```

一个静态方法一个非静态方法，但是两者效果是一样的，都是以十进制形式转换获取对应的字符串结果。内部调用了`Integer.toString`方法实现具体逻辑，具体实现可以参考[Integer的toString 方法](https://blog.renyijiu.com/post/java%E6%BA%90%E7%A0%81integer/#tostring-%E6%96%B9%E6%B3%95)。

### valueOf 方法

```java
public static Short valueOf(String s) throws NumberFormatException {
  return valueOf(s, 10);
}

public static Short valueOf(String s, int radix)
  throws NumberFormatException {
  return valueOf(parseShort(s, radix));
}

public static Short valueOf(short s) {
  final int offset = 128;
  int sAsInt = s;
  if (sAsInt >= -128 && sAsInt <= 127) { // must cache
    return ShortCache.cache[sAsInt + offset];
  }
  return new Short(s);
}
```

存在三个`valueOf`方法，主要来看看第三个方法的实现。当参数范围在`-128~127`之间时，直接从内部类数组中换取对应对象返回；若超出缓存范围则重新实例化返回结果。所以可以看到这样的相等判断结果：

```java
Short a = Short.valueOf((short) 108);
Short b = Short.valueOf((short) 108);

Short c = Short.valueOf((short) 1108);
Short d = Short.valueOf((short) 1108);

System.out.println(a == b); // true
System.out.println(c == d); // false
```

### decode 方法

```java
public static Short decode(String nm) throws NumberFormatException {
  int i = Integer.decode(nm);
  if (i < MIN_VALUE || i > MAX_VALUE)
    throw new NumberFormatException(
    "Value " + i + " out of range from input " + nm);
  return valueOf((short)i);
}
```

这个方法主要是解码字符串转换成`Short`对象，支持十进制，`0x, 0X, #`开头的十六进制，`0`开头的八进制字符串。内部实现调用了`Integer.decode`方法获取对应的整数值（此方法可以参考[Integer的decode方法](https://blog.renyijiu.com/post/java%E6%BA%90%E7%A0%81integer/#decode-%E6%96%B9%E6%B3%95)），然后判断数值是否在范围内，是则调用`valueOf`返回对应的值；否则抛出对应的异常。

### xxxValue 方法

```java
public byte byteValue() {
  return (byte)value;
}

public short shortValue() {
  return value;
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
  return (double)value;
}
```

直接进行强制类型转换，返回对应类型的数据值。

### hashCode 方法

```java
@Override
public int hashCode() {
  return Short.hashCode(value);
}

public static int hashCode(short value) {
  return (int)value;
}
```

返回本身的数值作为hashCode

### equals 方法

```java
public boolean equals(Object obj) {
  if (obj instanceof Short) {
    return value == ((Short)obj).shortValue();
  }
  return false;
}
```

与输入对象进行比较，判断两者是否相等。首先判断输入对象是否为`Short`类型实例，然后判断两者的数值是否相等。代码逻辑上可以看出来，输入参数可以接受`null`值，不需要担心NPE错误。

### compare 方法

```java
public static int compare(short x, short y) {
  return x - y;
}
```

比较两个`short`基本数据类型值。采用了相减的判断逻辑，所以`x < y`时返回负数，`x == y`时返回0，而`x > y`时返回正数。

### compareTo 方法

```java
public int compareTo(Short anotherShort) {
  return compare(this.value, anotherShort.value);
}
```

比较两个`Short`对象值，内部调用了`compare`方法实现，返回值与`compare`方法一致。

### reverseBytes 方法

```java
public static short reverseBytes(short i) {
  return (short) (((i & 0xFF00) >> 8) | (i << 8));
}
```

以字节为单位，逆序输入参数的二进制格式。因为`Short`类型只有16位bit，所以逻辑比较简单，高8位和低8位采用位逻辑交换位置，然后返回对应的数值。

### toUnsignedInt 方法

```java
public static int toUnsignedInt(short x) {
  return ((int) x) & 0xffff;
}
```

通过无符号转换将参数转换成对应的`int`数据类型值。保留低16位的bit数据，高16位设为0。0和正数等于其自身，负数等于输入加上`2^16`。

### toUnsignedLong 方法

```java
public static long toUnsignedLong(short x) {
  return ((long) x) & 0xffffL;
}
```

通过无符号转换将参数转换成对应的`long`类型数据值。保留低16位数据，高48位设为0。0和正数等于其自身，负数等于输入加上`2^16`。

## 总结

总体来看，`Short`类型的代码是比较简单的，部分函数调用了`Integer`的方法实现，所以没有什么复杂逻辑。





































