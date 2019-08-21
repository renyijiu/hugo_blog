---
title: "[Java源码]Boolean"
date: 2019-08-20T16:37:12+08:00
lastmod: 2019-08-20T16:37:12+08:00
draft: false
keywords: ["Java源码", "Boolean"]
description: "Java源码分析 - Boolean"
tags: ["Java源码分析", "Boolean"]
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

因为业务发展关系，前段时间将一个风控平台从Ruby语言迁移到了Java平台，基于Spring进行了重构，写了一段时间Java后，自己发现竟然还挺喜欢的（可能因为那段时间真的深深体会了 **动态语言一时爽，重构火葬场** 的玩笑）。所以后续决定阅读部分源码进行深入提升Java水平，网上搜索了一番，看到了很多的推荐，例如`Spring`框架的源码之类的，还有一些其他的开源组件项目，但是后面想想还是先从Java基础入手吧，于是就有了这篇文章，从最简单的Boolean开始。

<!--more-->

源代码基于 *jdk1.8.0_181.jdk* 版本。水平有限，第一次看相关代码，也第一次写😂如有建议和意见，欢迎联系指出。

## 概要

Java的`Boolean`对象是对`boolean`基本数据类型的封装，有着一个字段存放对应的`boolean`数据值，提供了许多方法方便对`boolean`进行操作。

## 类定义

```java
public final class Boolean implements java.io.Serializable, Comparable<Boolean>
```

因为带有`final`关键字，也就是**不可继承**，另外实现了`Serializable`和`Comparable`接口，也就是可以进行序列化和比较的。

## 属性

```java
public static final Boolean TRUE = new Boolean(true);

public static final Boolean FALSE = new Boolean(false);
```

`TRUE`和`FALSE`定义了两个静态常量，代表着`boolean`中的`true`和`false`。

```java
@SuppressWarnings("unchecked")
public static final Class<Boolean> TYPE = (Class<Boolean>) Class.getPrimitiveClass("boolean");
```

获取类信息，`Boolean.TYPE == boolean.class`两者是等价的。

```java
private final boolean value;
```

`Boolean`因为是`boolean`的包装类，所以这里就是包含了对应的boolean基本类型的变量

```java
private static final long serialVersionUID = -3665804199014368530L;
```

## 方法

### 构造方法

```java
 public Boolean(boolean value) {
    this.value = value;
 }

public Boolean(String s) {
    this(parseBoolean(s));
}
```

存在两个构造方法，一个是传入boolean类型值，一个是传入字符串解析，而对应的解析方法如下：

```java
public static boolean parseBoolean(String s) {
    return ((s != null) && s.equalsIgnoreCase("true"));
}
```

这里就很直白的告诉了你，只有`true`（忽略大小写）格式的字符串才会返回`true`，其余的都为`false`。这里自己因此还踩过坑，一直以为字符串存在时就是真值，还是和Ruby不一样的。

### valueOf 方法

```java
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}

public static Boolean valueOf(String s) {
    return parseBoolean(s) ? TRUE : FALSE;
}
```

我们通过`Boolean b = new Boolean(true);`时会获得一个新的实例对象，当你并不需要这个新实例而只要对应的值时，就应该采用这里的静态方法，直接返回了上面定义的静态变量，节省了一点内存又避免了创建一个新实例的时间开销。

### booleanValue 方法

```java
public boolean booleanValue() {
    return value;
}
```

获取对应的boolean类型值。

### toString 方法

```java
public static String toString(boolean b) {
    return b ? "true" : "false";
}

public String toString() {
    return value ? "true" : "false";
}
```

返回对应布尔值的String对象。如果为`true`则返回"true"字符串，否则就是"false"。

### hashcode 方法

```java
@Override
public int hashCode() {
  	return Boolean.hashCode(value);
}

public static int hashCode(boolean value) {
  	return value ? 1231 : 1237;
}
```

其中重写了`hashcode`方法，返回值调用了静态方法`hashcode`，而静态方法返回值：当`true`时返回1231，`false`返回1237。这里很有意思的是这两个固定的数字，很好奇为什么是这两个值，所以网上搜索了下，有两篇回答可以参考一下。  

1. [Boolean.hashCode()](https://stackoverflow.com/questions/3912303/boolean-hashcode)

2. [Why does Java's Boolean.hashCode() map the values 'true' and 'false' to 1237 and 1231 instead of 1 and 0?](https://www.quora.com/Why-does-Javas-Boolean-hashCode-map-the-values-true-and-false-to-1237-and-1231-instead-of-1-and-0)

大致意思是，主要这两个是较大的质数（实际上其他较大的质数也可以），至于较大质数的原因是可以有效减少Hash碰撞冲突的发生。而Boolean是一个很常用的对象，会经常在其他里面使用，这么做有助于提升效率。

### equals 方法

```java
public boolean equals(Object obj) {
	  if (obj instanceof Boolean) {
  	  	return value == ((Boolean)obj).booleanValue();
  	}
  	return false;
}
```

首先判断传入的`obj`是不是`Boolean`的实例，然后比较两者的值是否相等。从这里可以发现，如果我们习惯于在判断前增加`if (null == obj)`的逻辑，其实在这里是可以忽略的，因为`null`肯定不是`Boolean`的实例，会直接返回`false`。

### getBoolean 方法

```java
public static boolean getBoolean(String name) {
    boolean result = false;
    try {
      	result = parseBoolean(System.getProperty(name));
    } catch (IllegalArgumentException | NullPointerException e) {
    }
    return result;
}
```

从代码上看，这个方法**并不是**将输入字符串转换成`boolean`类型的方法。而是当且仅当 *系统属性* 中存在着对应名称的属性，且它的值为“true”（忽略大小写）时，返回`true`类型，否则返回`false`。

### compare 方法

```java
public int compareTo(Boolean b) {
  	return compare(this.value, b.value);
}

public static int compare(boolean x, boolean y) {
  	return (x == y) ? 0 : (x ? 1 : -1);
}
```

比较两个Boolean实例对象或者boolean类型值，当相等时，返回0；否则根据第一个值进行判断，第一个值为`true`时返回1，否则返回-1。

### logical 比较方法

```java
public static boolean logicalAnd(boolean a, boolean b) {
		return a && b;
}

public static boolean logicalOr(boolean a, boolean b) {
    return a || b;
}


public static boolean logicalXor(boolean a, boolean b) {
  return a ^ b;
}
```

提供了三个静态方法，对于两个`boolean`类型值进行与、或、异或操作，返回对应的判断结果值。

## boolean类型

在看代码的过程中，想知道boolean到底是个啥东西，然后去看了下 [oracle](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html) 的官方文档，突然学到了一点小知识，顺带记录一下。

在Java的八种基本数据类型中，除了`boolean`其它7种都有明确的字节数长度，只有`boolean`没有给出具体的字节数长度定义，详细的介绍如下：

> **boolean**: The `boolean` data type has only two possible values: `true` and `false`. Use this data type for simple flags that track true/false conditions. This data type represents one bit of information, but its "size" isn't something that's precisely defined.

`boolean`只存在`true`和`false`两个值，这个数据类型也是用于跟踪真/假条件的简单标志。这个数据类型代表了1bit的信息，但是它的大小并没有明确的定义。由此搜索了一下，发现具体的应该是由Java虚拟机规范定义的。官方的文档  [Java Virtual Machine Specification](https://docs.oracle.com/javase/specs/jvms/se8/html/index.html) 有这个相关的说明，`boolean`类型值会被编译使用`int`类型进行替换，也就是4字节的大小；而`boolean`类型数组，jvm并不是直接支持的，而是通过`byte[]`实现，其中也就是占用1字节。具体的说明可以查看提供的原文链接，也是非常有意思的信息。

## 总结

整体的源代码是很简单的，但是这次查看源码的过程中也学到了很多的东西，平时一些可能完全不会关注的点，在这次过程中也都学习到了，例如有些情况下应该使用`Boolean.valueOf(true)`替换`new Boolean(true)`，`hashCode`的值，`boolean`的相关文档定义等等，魔鬼藏在细节里，继续加油。







