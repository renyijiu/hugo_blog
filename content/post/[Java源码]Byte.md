---
title: "[Java源码]Byte"
date: 2019-08-22T16:48:19+08:00
lastmod: 2019-08-22T16:48:19+08:00
draft: false
keywords: ["Java源码", "Byte"]
description: "Java源码 - Byte"
tags: ["Java源码", "Java"]
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

这次我们来看看`Byte`类的源代码，基于 *jdk1.8.0_181.jdk* 版本 。

<!--more-->

## 概要

Java的`Byte`类主要是对`byte`基本数据类型的封装，有着一个字段存放着对应的`byte`数据值，另外提供了一些方法方便对`byte`进行相关的操作。

## 类定义

```java
public final class Byte extends Number implements Comparable<Byte>
```

`Byte`类带有关键字`final`，也就是**不可以继承的**，另外继承了`Number`类，实现了`Comparable`接口，也就是可以进行比较。

## 属性

```java
public static final byte   MIN_VALUE = -128;

public static final byte   MAX_VALUE = 127;
```

定义了`Byte`的范围大小，由于补码的关系，负数会比正数多一个值，`MIN_VALUE`能取的值为 `-2**8`，而`MAX_VALUE`能取的值为`2**8 - 1`，具体原码和补码的逻辑，可以参考下[原码、反码、补码的产生、应用以及优缺点有哪些？ - 知乎用户的回答 - 知乎](https://www.zhihu.com/question/20159860/answer/71256667)，个人觉得讲解的不错。

```java
public static final int SIZE = 8;
```

定义了`byte`值的二进制补码格式的`bit`位数，固定8位。

```java
public static final int BYTES = SIZE / Byte.SIZE;
```

定义了`byte`值的二进制补码格式的字节数，计算值固定为1。

```java
@SuppressWarnings("unchecked")
public static final Class<Byte>     TYPE = (Class<Byte>) Class.getPrimitiveClass("byte");
```

获取`Byte`的类信息，`Byte.TYPE == byte.class`，两者是等价的。

```java
private final byte value;
```

`Byte`因为是`byte`的包装类，所以这里包含了对应的`byte`基本类型数据的变量。

```java
private static final long serialVersionUID = -7183698231559129828L;
```

## 内部类

```java
private static class ByteCache {
  private ByteCache(){}

  static final Byte cache[] = new Byte[-(-128) + 127 + 1];

  static {
    for(int i = 0; i < cache.length; i++)
      cache[i] = new Byte((byte)(i - 128));
  }
}
```

这里定义了一个静态的嵌套类，内部定义了一个数组，大小为`-(-128)+127+1=256`，包含了-128～127之间的值。静态块中初始化了数组的值。这里补充一个知识点，Java类的加载顺序，`静态变量/代码块 -> 非静态变量/代码块 -> 构造方法`，相同部分里面是按照代码顺序进行加载，详细的内容可以自行搜索学习。

关于内部类的相关说明，可以参考官方的文档 [Nested Classes](https://docs.oracle.com/javase/tutorial/java/javaOO/nested.html) 进行进一步的了解学习。

## 方法

### 构造方法

```java
public Byte(byte value) {
  this.value = value;
}

public Byte(String s) throws NumberFormatException {
  this.value = parseByte(s, 10);
}
```

存在两个对应的构造方法，一个传入`byte`值，一个是传入字符串解析，转换成10进制整数处理（注意的是这个方法可能会抛出异常，需要注意处理），对应的解析方法下面介绍。

### parseByte 方法

```java
public static byte parseByte(String s, int radix)
  throws NumberFormatException {
  int i = Integer.parseInt(s, radix);
  if (i < MIN_VALUE || i > MAX_VALUE)
    throw new NumberFormatException(
    "Value out of range. Value:\"" + s + "\" Radix:" + radix);
  return (byte)i;
}
```

传入字符串和进制数，调用了`Integer.parseInt`方法获取转换后的整数，判断整数范围是否符合`Byte`的范围。如果是，强制转换`byte`类型返回；否，抛出`NumberFormatException`异常。

```java
public static byte parseByte(String s) throws NumberFormatException {
  return parseByte(s, 10);
}
```

存在一个单参数的`parseByte`方法，通过调用上面的方法实现，默认10进制数。

### toString 方法

```java
public static String toString(byte b) {
  return Integer.toString((int)b, 10);
}

public String toString() {
  return Integer.toString((int)value);
}
```

直接将`byte`类型值强制转换成`int`，然后调用了`Integer.toString`方法进行操作。

### valueOf 方法

```java
public static Byte valueOf(byte b) {
  final int offset = 128;
  return ByteCache.cache[(int)b + offset];
}
```

对于`byte`类型参数，直接通过计算偏移量，读取`ByteCache`内部类的静态变量数组对应值来操作，提升了对应的空间和时间性能。

```java
public static Byte valueOf(String s, int radix)
  throws NumberFormatException {
  return valueOf(parseByte(s, radix));
}

public static Byte valueOf(String s) throws NumberFormatException {
  return valueOf(s, 10);
}
```

这两个方法是针对`string`类型参数的，只是进制设定的问题。使用`parseByte`方法将字符串转换成`byte`类型值，然后直接使用第一个`valueOf`方法操作。

### decode 方法

```java
public static Byte decode(String nm) throws NumberFormatException {
  int i = Integer.decode(nm);
  if (i < MIN_VALUE || i > MAX_VALUE)
    throw new NumberFormatException(
    "Value " + i + " out of range from input " + nm);
  return valueOf((byte)i);
}
```

针对字符串解码，将字符串转换成`Byte`类型值。主要逻辑是调用了`Integer.decode`方法获取解码后的数字，然后判断对应返回是否符合`byte`要求，调用`valueOf`返回最终结果。

除去`+/-`符号，会根据实际情况进行特定的解码，默认会处理成十进制。`0x,0X,#`开头的会处理成16进制，`0`开头的会处理成8进制。

### xxxValue 方法

```java
public byte byteValue() {
  return value;
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
  return (double)value;
}
```

通过强制类型转换，扩大了原有值类型范围，返回结果。

### hashCode 方法

```java
@Override
public int hashCode() {
  return Byte.hashCode(value);
}

public static int hashCode(byte value) {
  return (int)value;
}
```

直接返回了对应的`int`类型值，作为其hashCode。

### equals 方法

```java
public boolean equals(Object obj) {
  if (obj instanceof Byte) {
    return value == ((Byte)obj).byteValue();
  }
  return false;
}
```

首先判断传入参数obj是不是`Byte`的实例，是的话比较两者的值是否相等；否则直接返回`false`。从这里可以看出来，我们在使用时也不需要对传入参数进行`null == obj`的判断，可以忽略。

### compare 方法

```java
public static int compare(byte x, byte y) {
  return x - y;
}
```

比较两者的值，x小于y时返回负数，x等于y时返回0，x大于y时返回正数。

### compareTo 方法

```java
public int compareTo(Byte anotherByte) {
  return compare(this.value, anotherByte.value);
}
```

内部调用`compare`方法。

### toUnsignedInt 方法

```java
public static int toUnsignedInt(byte x) {
  return ((int) x) & 0xff;
}
```

将`byte`转换成无符号整数。x强制转换成`int`类型值，通过与`0xff`进行位运算，保留低8位值，高24位设置成0。整体结果，0和正数保持不变，负数等于原来的值 + 2**8。

### toUnsignedLong 方法

```java
public static long toUnsignedLong(byte x) {
  return ((long) x) & 0xffL;
}
```

过程同上，只是`long`为64位，低8位保持原值，高56位设置成0。

## 总结

通过阅读源码发现，`Byte`类大量使用了`Integer`的方法，另外使用了内部类、位运算等方式进行了一定的逻辑优化。自己在看的过程对于内部类，补码，类的加载顺序也加深了了解，继续加油！