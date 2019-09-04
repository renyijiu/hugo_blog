---
title: "[Java源码]Integer"
date: 2019-09-02T11:06:51+08:00
lastmod: 2019-09-02T11:06:51+08:00
draft: false
keywords: ["Java源码", "Java", "Integer"]
description: "Java源码 - Integer"
tags: ["Java源码", "Integer"]
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

这次我们来看看`Integer`的源代码，基于  *jdk1.8.0_181.jdk* 版本，如有错误，欢迎联系指出。

<!--more-->

## 类定义

```java
public final class Integer extends Number implements Comparable<Integer>
```

带有`final`标识，也就是说**不可继承的**。另外继承了`Number`类，而`Number`类实现了`Serializable`接口，所以`Integer`也是可以序列化的；实现了`Comparable`接口。

## 属性

```java
@Native public static final int   MIN_VALUE = 0x80000000;

@Native public static final int   MAX_VALUE = 0x7fffffff;
```

* `MIN_VALUE` 表示了`Integer`最小值，对应为`-2^31`
* `MAX_VALUE` 表示了`Integer`最大值，对应为`2^31 - 1`

这里的两个常量都带有`@Native`注解，表示这两个常量值字段可以被native代码引用。当native代码和Java代码都需要维护相同的变量时，如果Java代码使用了`@Native`标记常量字段时，编译时可以生成对应的native代码的头文件。

```java
@Native public static final int SIZE = 32;
```

表示了`Integer`的bit数，32位。也使用了`@Native`注解，这里有个比较有意思的问题，[Why is the SIZE constant only @Native for Integer and Long?](https://stackoverflow.com/questions/28770822/why-is-the-size-constant-only-native-for-integer-and-long) 可以进行深入阅读了解。

```java
public static final int BYTES = SIZE / Byte.SIZE;
```

表示了`Integer`的字节数，计算值固定为4

```java
@SuppressWarnings("unchecked")
public static final Class<Integer>  TYPE = (Class<Integer>) Class.getPrimitiveClass("int");
```

获取类信息，`Integer.TYPE == int.class`，两者是等价的。

```java
final static char[] digits = {
  '0' , '1' , '2' , '3' , '4' , '5' ,
  '6' , '7' , '8' , '9' , 'a' , 'b' ,
  'c' , 'd' , 'e' , 'f' , 'g' , 'h' ,
  'i' , 'j' , 'k' , 'l' , 'm' , 'n' ,
  'o' , 'p' , 'q' , 'r' , 's' , 't' ,
  'u' , 'v' , 'w' , 'x' , 'y' , 'z'
};
```

代表了所有可能字符。因为允许二进制至36进制，所有必须有36个字符代表所有可能情况。

```java
final static char [] DigitTens = {
  '0', '0', '0', '0', '0', '0', '0', '0', '0', '0',
  '1', '1', '1', '1', '1', '1', '1', '1', '1', '1',
  '2', '2', '2', '2', '2', '2', '2', '2', '2', '2',
  '3', '3', '3', '3', '3', '3', '3', '3', '3', '3',
  '4', '4', '4', '4', '4', '4', '4', '4', '4', '4',
  '5', '5', '5', '5', '5', '5', '5', '5', '5', '5',
  '6', '6', '6', '6', '6', '6', '6', '6', '6', '6',
  '7', '7', '7', '7', '7', '7', '7', '7', '7', '7',
  '8', '8', '8', '8', '8', '8', '8', '8', '8', '8',
  '9', '9', '9', '9', '9', '9', '9', '9', '9', '9',
} ;

final static char [] DigitOnes = {
  '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
  '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
  '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
  '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
  '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
  '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
  '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
  '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
  '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
  '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
} ;
```

定义了两个数组，`DigitTens`存放了0~99之间的数字的十位数字符；`DigitOnes`存放了0~99之间的数字的个位数字符。

```java
final static int [] sizeTable = { 9, 99, 999, 9999, 99999, 999999, 9999999,
                                 99999999, 999999999, Integer.MAX_VALUE };
```

数组，存放了范围各位数整数的最大值。

```java
private final int value;
```

`Integer`是`int`的包装类，这里就是存放了`int`类型对应的数据值信息

```java
@Native private static final long serialVersionUID = 1360826667806852920L;
```

## 内部类

```java
private static class IntegerCache {
  static final int low = -128;
  static final int high;
  static final Integer cache[];

  static {
    // high value may be configured by property
    int h = 127;
    String integerCacheHighPropValue =
      sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
    if (integerCacheHighPropValue != null) {
      try {
        int i = parseInt(integerCacheHighPropValue);
        i = Math.max(i, 127);
        // Maximum array size is Integer.MAX_VALUE
        h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
      } catch( NumberFormatException nfe) {
        // If the property cannot be parsed into an int, ignore it.
      }
    }
    high = h;

    cache = new Integer[(high - low) + 1];
    int j = low;
    for(int k = 0; k < cache.length; k++)
      cache[k] = new Integer(j++);

    // range [-128, 127] must be interned (JLS7 5.1.7)
    assert IntegerCache.high >= 127;
  }

  private IntegerCache() {}
}
```

`IntegerCache`是`Integer`的静态内部类，内部定义了一个数组，用于缓存常用的数值范围，避免后续使用时重新实例化，提升性能。其默认缓存的范围是-128～127，我们可以通过`-XX:AutoBoxCacheMax=<size>`选项自行配置缓存最大值，但是必须要大于等于127。

## 方法

### 构造方法

```java
public Integer(int value) {
  this.value = value;
}

public Integer(String s) throws NumberFormatException {
  this.value = parseInt(s, 10);
}
```

存在两个构造方法，一个参数为`int`类型，一个为`String`类型；参数为`String`对象时，内部调用了`parseInt`方法使用十进制进行处理。

### parseInt 方法

```java
public static int parseInt(String s) throws NumberFormatException {
  return parseInt(s,10);
}

public static int parseInt(String s, int radix)
  throws NumberFormatException
    {
      /*
      * 注意：这个方法在VM初始化时可能会早于 IntegerCache 的初始化过程，所以值得注意的是不要使用 valueOf 方法 
      */
      
        // 判断空值
        if (s == null) {
            throw new NumberFormatException("null");
        }
				
        // Character.MIN_RADIX = 2
        // 判断最小进制
        if (radix < Character.MIN_RADIX) {
            throw new NumberFormatException("radix " + radix +
                                            " less than Character.MIN_RADIX");
        }

        // Character.MAX_RADIX = 36
        // 判断最大进制
        if (radix > Character.MAX_RADIX) {
            throw new NumberFormatException("radix " + radix +
                                            " greater than Character.MAX_RADIX");
        }

        int result = 0;
        boolean negative = false;
        int i = 0, len = s.length();
        int limit = -Integer.MAX_VALUE;
        int multmin;
        int digit;

        if (len > 0) {
            char firstChar = s.charAt(0);
            if (firstChar < '0') { // 第一个字符可能是"+" or "-"
                if (firstChar == '-') {
                    negative = true;
                    limit = Integer.MIN_VALUE;
                } else if (firstChar != '+')
                    throw NumberFormatException.forInputString(s);

                if (len == 1) // 不能单独只存在 "+" or "-"
                    throw NumberFormatException.forInputString(s);
                i++;
            }
            multmin = limit / radix;
            while (i < len) {
                // Character.digit 返回对应字符对应进制的数字值，如果输入进制不在范围内或者字符无效，返回-1                
                digit = Character.digit(s.charAt(i++),radix);
                if (digit < 0) {
                    throw NumberFormatException.forInputString(s);
                }
                // 判断结果是否溢出
                if (result < multmin) {
                    throw NumberFormatException.forInputString(s);
                }
                result *= radix;
                // 判断增加当前位后的计算结果是否溢出
                if (result < limit + digit) {
                    throw NumberFormatException.forInputString(s);
                }
                // 采用了负数的形式存放结果
                result -= digit;
            }
        } else {
            throw NumberFormatException.forInputString(s);
        }
        return negative ? result : -result;
    }
```

存在两个`parseInt`方法，第一个内部调用了第二个方法实现，所以具体来看看第二个方法的实现。相关的代码已经增加了注释，不重复介绍了。这里存在一个比较有意思的地方，代码逻辑上通过计算负数，然后结果判断符号位的逻辑进行运算，这样子可以避免正负逻辑分开处理。另外具体不采用正数逻辑应该是`Integer.MIN_VALUE`转换成正数时会产生溢出，需要单独处理。

### parseUnsignedInt 方法

```java
public static int parseUnsignedInt(String s) throws NumberFormatException {
  return parseUnsignedInt(s, 10);
}

public static int parseUnsignedInt(String s, int radix)
  throws NumberFormatException {
  // null空值判断
  if (s == null)  {
    throw new NumberFormatException("null");
  }

  int len = s.length();
  if (len > 0) {
    char firstChar = s.charAt(0);
    // 判断第一个字符，无符号字符串处理，出现了非法符号‘-’
    if (firstChar == '-') {
      throw new
        NumberFormatException(String.format("Illegal leading minus sign " +
                                            "on unsigned string %s.", s));
    } else {
      if (len <= 5 || // Integer.MAX_VALUE以36进制形式表示为6位字符
          (radix == 10 && len <= 9) ) { // Integer.MAX_VALUE以十进制表示为10位字符
        // 这个范围内，完全确保在int的数值范围
        return parseInt(s, radix);
      } else {
        long ell = Long.parseLong(s, radix);
        // 判断是不是超过32bit的范围
        if ((ell & 0xffff_ffff_0000_0000L) == 0) {
          return (int) ell;
        } else {
          throw new
            NumberFormatException(String.format("String value %s exceeds " +
                                                "range of unsigned int.", s));
        }
      }
    }
  } else {
    throw NumberFormatException.forInputString(s);
  }
}
```

存在两个`parseUnsignedInt`方法，第一个方法内部调用了第二个方法实现。我们来看看第二个方法，首先判断字符串是否符合格式要求；然后判断范围，在int范围内的使用`parseInt`处理，否则使用`Long.parseLong`处理，再强制转成int数据类型返回对应结果。

### getChars 方法

```java
static void getChars(int i, int index, char[] buf) {
  int q, r;
  int charPos = index;
  char sign = 0;

  // 不支持Integer.MIN_VALUE，转换成正数时会产生溢出
  if (i < 0) {
    sign = '-';
    i = -i;
  }

  // 每次循环，处理两位数字
  // 从低位开始，所以索引是在向前走，从后往前
  while (i >= 65536) {
    q = i / 100;
    // 相当于 r = i - (q * 100);
    r = i - ((q << 6) + (q << 5) + (q << 2));
    i = q;
    buf [--charPos] = DigitOnes[r];
    buf [--charPos] = DigitTens[r];
  }

  // 对于小于等于 65536 的数字，采用快速下降模式
  // assert(i <= 65536, i);
  for (;;) {
    q = (i * 52429) >>> (16+3);
    // 相当于 r = i - (q * 10)
    r = i - ((q << 3) + (q << 1));
    buf [--charPos] = digits [r];
    i = q;
    if (i == 0) break;
  }
  if (sign != 0) {
    buf [--charPos] = sign;
  }
}
```

该方法主要的逻辑就是将输入`int`类型数据转换成字符形式放入char数组中，**不支持 Integer.MIN_VALUE**。

当输入值大于65536时，每次处理两位数字，进行效率的提升；另外中间乘法操作也都使用位移运算来替代。

这里比较有意思的是当数字小于等于65536时的计算逻辑，中间计算使用了**52429**这个数字，那么为什么是它呢？这里说一下我的看法，仅个人观点，如有错误欢迎指出。先看下代码的注释信息：

> // I use the "invariant division by multiplication" trick to
>
> // accelerate Integer.toString.  In particular we want to
>
> // avoid division by 10.
>
> //
>
> // The "trick" has roughly the same performance characteristics
>
> // as the "classic" Integer.toString code on a non-JIT VM.
>
> // The trick avoids .rem and .div calls but has a longer code
>
> // path and is thus dominated by dispatch overhead.  In the
>
> // JIT case the dispatch overhead doesn't exist and the
>
> // "trick" is considerably faster than the classic code.
>
> //
>
> // TODO-FIXME: convert (x * 52429) into the equiv shift-add
>
> // sequence.
>
> //
>
> // RE:  Division by Invariant Integers using Multiplication
>
> //      T Gralund, P Montgomery
>
> //      ACM PLDI 1994
>
> //

1. `52429 / 2^(16+3) = 0.10000038146972656`，实际这里的操作就是除以10的逻辑。
2. 主要是使用乘法和移位操作来代替除法操作，提升性能。具体原理说明可以参考论文[ Division by Invariant Integers using Multiplication](https://gmplib.org/~tege/divcnst-pldi94.pdf)
3. 这里为什么选择`65536`作为界限说实话自己也没有看明白，各处都没有看到相关的说明。如果你了解，欢迎评论指出。
4. 我们要确保乘法操作之后不能溢出，因为上面逻辑中`i`已经进行负数判断，所有`i`必定为正数，我们可以认为乘积是个无符号整数，最大值为2^32（后续的无符号右移可以对应）；也就是对应的乘数必须小于 2^32 / 65536，即65536。
5. `(i * 52429) >>> (16+3);`其中`52429`为乘数，设为a，`16+3`为指数，设为b。也就是`a / (2^b) = 0.1`，即`a = 2^b / 10`。但是呢这样可能会**由于除数向下取整，导致结果错误**的问题，举个例子：

```java
System.out.println((1120 * 52428) >>> (16+3)); // 111
System.out.println((1120 * 52429) >>> (16+3)); // 112
```

所以进行修正，改成`a = 2^b / 10 + 1`，避免向下取整可能带来的问题

5. 然后进行多组数据测试

```reStructuredText
乘数: 2, 指数：4, 除法: 2 / 16.0 = 0.125
乘数: 4, 指数：5, 除法: 4 / 32.0 = 0.125
乘数: 7, 指数：6, 除法: 7 / 64.0 = 0.109375
乘数: 13, 指数：7, 除法: 13 / 128.0 = 0.1015625
乘数: 26, 指数：8, 除法: 26 / 256.0 = 0.1015625
乘数: 52, 指数：9, 除法: 52 / 512.0 = 0.1015625
乘数: 103, 指数：10, 除法: 103 / 1024.0 = 0.1005859375
乘数: 205, 指数：11, 除法: 205 / 2048.0 = 0.10009765625
乘数: 410, 指数：12, 除法: 410 / 4096.0 = 0.10009765625
乘数: 820, 指数：13, 除法: 820 / 8192.0 = 0.10009765625
乘数: 1639, 指数：14, 除法: 1639 / 16384.0 = 0.10003662109375
乘数: 3277, 指数：15, 除法: 3277 / 32768.0 = 0.100006103515625
乘数: 6554, 指数：16, 除法: 6554 / 65536.0 = 0.100006103515625
乘数: 13108, 指数：17, 除法: 13108 / 131072.0 = 0.100006103515625
乘数: 26215, 指数：18, 除法: 26215 / 262144.0 = 0.10000228881835938
乘数: 52429, 指数：19, 除法: 52429 / 524288.0 = 0.10000038146972656
乘数: 104858, 指数：20, 除法: 104858 / 1048576.0 = 0.10000038146972656
乘数: 209716, 指数：21, 除法: 209716 / 2097152.0 = 0.10000038146972656
乘数: 419431, 指数：22, 除法: 419431 / 4194304.0 = 0.10000014305114746
```

可以看出来，`52429`是这个范围内精度最大的值，所以选择这个值。另外从结果看当指数大于19时，精度的提升已经很小，所以猜测可能选定了这个指数后确定了`65536`的范围，当然也只是猜测。

上面的说明大致介绍了代码逻辑，但这也是出于时代的原因（当时的机器设备性能等），从[jdk9](http://hg.openjdk.java.net/jdk9/jdk9/jdk/file/65464a307408/src/java.base/share/classes/java/lang/Integer.java#l485)的源代码来看，这里的代码已经进行了重构，改成了简单易理解的除法操作。具体的这里有个对应的[issue](https://bugs.openjdk.java.net/browse/JDK-8136500)，从目前现状来看两者的性能差距几乎可以忽略不计吧。

### stringSize 方法

```java
static int stringSize(int x) {
  for (int i=0; ; i++)
    if (x <= sizeTable[i])
      return i+1;
}
```

获取对应数据的字符串长度，通过`sizeTable`去遍历查询；从逻辑可以看出，**只支持正数**，负数的结果是错误的。

### toString 方法

```java
public static String toString(int i) {
  if (i == Integer.MIN_VALUE)
    return "-2147483648";
  int size = (i < 0) ? stringSize(-i) + 1 : stringSize(i);
  char[] buf = new char[size];
  getChars(i, size, buf);
  return new String(buf, true);
}
```

输入`int`数据类型的参数，对于`Integer.MIN_VALUE`进行了特殊判断，相等就返回固定字符串`"-2147483648"`，这里的逻辑是因为后续的`getChars`方法不支持`Integer.MIN_VALUE`。判断正负数，负数的数组大小比正数多1，用于存放`-`符号。最后调用的`String`的构造方法，返回结果字符串。

```java
public String toString() {
  return toString(value);
}
```

直接调用了上面的`toString`方法进行处理

```java
public static String toString(int i, int radix) {
  // 判断进制范围，不在范围内，默认设为十进制
  if (radix < Character.MIN_RADIX || radix > Character.MAX_RADIX)
    radix = 10;

  // 如果是十进制，调用上面的toString方法返回结果
  if (radix == 10) {
    return toString(i);
  }

  char buf[] = new char[33];
  boolean negative = (i < 0);
  int charPos = 32;

  // 正数转换成负数，统一后续处理逻辑
  if (!negative) {
    i = -i;
  }

  while (i <= -radix) {
    buf[charPos--] = digits[-(i % radix)];
    i = i / radix;
  }
  buf[charPos] = digits[-i];

  if (negative) {
    buf[--charPos] = '-';
  }

  return new String(buf, charPos, (33 - charPos));
}
```

这个方法带了进制信息，若进制不在设定范围内，默认使用十进制进行处理。然后转换成对应的字符串，代码逻辑比较简单。

### toUnsignedLong 方法

```java
public static long toUnsignedLong(int x) {
  return ((long) x) & 0xffffffffL;
}
```

转换成无符号的`long`类型数据，保留低32位bit数据，高32位设为0。0和正数等于其自身，负数等于输入加上2^32。

### divideUnsigned 方法

```java
public static int divideUnsigned(int dividend, int divisor) {
  // In lieu of tricky code, for now just use long arithmetic.
  return (int)(toUnsignedLong(dividend) / toUnsignedLong(divisor));
}
```

转换成无符号`long`类型数据相除，返回无符号整数结果。

### remainderUnsigned 方法

```java
public static int remainderUnsigned(int dividend, int divisor) {
  // In lieu of tricky code, for now just use long arithmetic.
  return (int)(toUnsignedLong(dividend) % toUnsignedLong(divisor));
}
```

转换成无符号`long`类型数据，进行取余操作，返回无符号整数结果。

### numberOfLeadingZeros 方法

```java
public static int numberOfLeadingZeros(int i) {
  // HD, Figure 5-6
  if (i == 0)
    return 32;
  int n = 1;
  if (i >>> 16 == 0) { n += 16; i <<= 16; }
  if (i >>> 24 == 0) { n +=  8; i <<=  8; }
  if (i >>> 28 == 0) { n +=  4; i <<=  4; }
  if (i >>> 30 == 0) { n +=  2; i <<=  2; }
  n -= i >>> 31;
  return n;
}

// 类似二分查找的思想，通过左右移位缩小1所在bit位置的范围。举个例子看下
// 开始输入          i = 00000000 00000000 00000000 00000001      n = 1
// i >>> 16 == 0，  i = 00000000 00000001 00000000 00000000      n = 17
// i >>> 24 == 0，  i = 00000001 00000000 00000000 00000000      n = 25
// i >>> 28 == 0,   i = 00010000 00000000 00000000 00000000      n = 29
// i >>> 30 == 0,   i = 01000000 00000000 00000000 00000000      n = 31
// n = n - i >>> 31, i >>> 31 = 0, 所以n = 31
```

判断二进制格式下，最高位的1左边存在多少个0。这里使用了二分查找的思想，通过左右移位的操作一步步缩小1所在的bit位置范围，最后通过简单计算获取0的个数。开始增加了对特殊值0的判断。

### numberOfTrailingZeros 方法

```java
public static int numberOfTrailingZeros(int i) {
  // HD, Figure 5-14
  int y;
  if (i == 0) return 32;
  int n = 31;
  y = i <<16; if (y != 0) { n = n -16; i = y; }
  y = i << 8; if (y != 0) { n = n - 8; i = y; }
  y = i << 4; if (y != 0) { n = n - 4; i = y; }
  y = i << 2; if (y != 0) { n = n - 2; i = y; }
  return n - ((i << 1) >>> 31);
}

// 类似二分查找的思想，通过左移缩小0所在bit位置的范围。举个例子看下
// 开始输入          i = 11111111 11111111 11111111 11111111    n = 31
// i << 16 != 0，   i = 11111111 11111111 00000000 00000000    n = 15
// i << 8 != 0，    i = 11111111 00000000 00000000 00000000    n = 7
// i << 4 != 0,     i = 11110000 00000000 00000000 00000000    n = 3
// i << 2 != 0,     i = 11000000 00000000 00000000 00000000    n = 1
// (i << 1) >>> 31 => 00000000 00000000 00000000 00000001 => 1
// 结果为 1 - 1 = 0
```

与上面的`numberOfLeadingZeros `方法对应，获取二进制格式下尾部的0的个数。具体逻辑与上面类似，就不再赘述了。

### formatUnsignedInt 方法

```java
/**
* 将无符号整数转换成字符数组
* @param val 无符号整数
* @param shift 基于log2计算， (4 对应十六进制, 3 对应八进制, 1 对应二进制)
* @param buf 字符写入数组
* @param offset 数组起始位置偏移量
* @param len 字符长度
* @return 字符写入数组后对应的起始位置
*/
static int formatUnsignedInt(int val, int shift, char[] buf, int offset, int len) {
  int charPos = len;
  int radix = 1 << shift;
  int mask = radix - 1;
  do {
    // val & mask，获取最后一位数字值
    buf[offset + --charPos] = Integer.digits[val & mask];
		
    // 移位去除最后一位数字，类似于十进制除10逻辑
    val >>>= shift;
  } while (val != 0 && charPos > 0);

  return charPos;
}
```

将无符号整数转换成字符串写入对应的数组位置，返回写入数组后的起始位置。

### toUnsignedString 方法

```java
public static String toUnsignedString(int i, int radix) {
    return Long.toUnsignedString(toUnsignedLong(i), radix);
}
```

将输入整数按进制转换成无符号的字符串，内部调用了`toUnsignedLong`获取无符号的long类型数据，然后转换成对应的字符串。

### toXxxString 方法

```java
public static String toHexString(int i) {
  return toUnsignedString0(i, 4);
}

public static String toOctalString(int i) {
  return toUnsignedString0(i, 3);
}

public static String toBinaryString(int i) {
  return toUnsignedString0(i, 1);
}

private static String toUnsignedString0(int val, int shift) {
  // assert shift > 0 && shift <=5 : "Illegal shift value";
  int mag = Integer.SIZE - Integer.numberOfLeadingZeros(val);
  int chars = Math.max(((mag + (shift - 1)) / shift), 1);
  char[] buf = new char[chars];

  // 调用formatUnsignedInt，获取对应字符串数组信息
  formatUnsignedInt(val, shift, buf, 0, chars);

  // Use special constructor which takes over "buf".
  return new String(buf, true);
}
```

`toHexString, toOctalString, toBinaryString`三个方法如名字一样获取不同进制的字符串，内部调用了`toUnsignedString0`方法进行处理，传入了不用的进制数参数。

### valueOf 方法

```java
public static Integer valueOf(String s) throws NumberFormatException {
  return Integer.valueOf(parseInt(s, 10));
}

public static Integer valueOf(String s, int radix) throws NumberFormatException {
  return Integer.valueOf(parseInt(s,radix));
}

public static Integer valueOf(int i) {
  if (i >= IntegerCache.low && i <= IntegerCache.high)
    return IntegerCache.cache[i + (-IntegerCache.low)];
  return new Integer(i);
}
```

存在三个`valueOf`方法，主要看看第三个方法。当参数在缓存范围内时，直接从缓存数组中获取对应的`Integer`对象；超出范围时，实例化对应的参数对象返回结果。

### xxxValue 方法

```java
public byte byteValue() {
  return (byte)value;
}

public short shortValue() {
  return (short)value;
}

public int intValue() {
  return value;
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

直接进行强制类型转换，返回对应的结果

### hashCode

```java
@Override
public int hashCode() {
  return Integer.hashCode(value);
}

public static int hashCode(int value) {
  return value;
}
```

对应的值作为其hashCode。

### equals 方法

```java
public boolean equals(Object obj) {
  if (obj instanceof Integer) {
    return value == ((Integer)obj).intValue();
  }
  return false;
}
```

判断obj是不是`Integer`的实例对象，然后判断两者的值是否相等。这里可以看出来，我们可以不需要对obj进行null判断。

### decode 方法

```java
public static Integer decode(String nm) throws NumberFormatException {
  int radix = 10;
  int index = 0;
  boolean negative = false;
  Integer result;

  if (nm.length() == 0)
    throw new NumberFormatException("Zero length string");
  char firstChar = nm.charAt(0);
  // 判断负数
  if (firstChar == '-') {
    negative = true;
    index++;
  } else if (firstChar == '+')
    index++;

  // 判断十六进制
  if (nm.startsWith("0x", index) || nm.startsWith("0X", index)) {
    index += 2;
    radix = 16;
  }
  // 判断十六进制
  else if (nm.startsWith("#", index)) {
    index ++;
    radix = 16;
  }
  // 判断八进制
  else if (nm.startsWith("0", index) && nm.length() > 1 + index) {
    index ++;
    radix = 8;
  }
	
  // 字符串格式不合法
  if (nm.startsWith("-", index) || nm.startsWith("+", index))
    throw new NumberFormatException("Sign character in wrong position");

  try {
    // 如果是 Integer.MIN_VALUE，转换成正数会溢出抛出异常
    result = Integer.valueOf(nm.substring(index), radix);
    result = negative ? Integer.valueOf(-result.intValue()) : result;
  } catch (NumberFormatException e) {
    // 处理 数字是 Integer.MIN_VALUE的异常错误信息，如果存在错误，会继续抛出异常
    String constant = negative ? ("-" + nm.substring(index))
      : nm.substring(index);
    result = Integer.valueOf(constant, radix);
  }
  return result;
}
```

将对应的字符串转换成整数，支持十进制，`0x, 0X, #`开头的十六进制数，`0`开头的八进制数。

### getInteger 方法

```java
// 返回对应数值或者null
public static Integer getInteger(String nm) {
  return getInteger(nm, null);
}

// 返回对应的数值或者val默认值
public static Integer getInteger(String nm, int val) {
  Integer result = getInteger(nm, null);
  return (result == null) ? Integer.valueOf(val) : result;
}

public static Integer getInteger(String nm, Integer val) {
  String v = null;
  try {
    // 获取对应的系统配置信息
    v = System.getProperty(nm);
  } catch (IllegalArgumentException | NullPointerException e) {
  }
  if (v != null) {
    try {
      return Integer.decode(v);
    } catch (NumberFormatException e) {
    }
  }
  return val;
}
```

三个`getInteger`方法，主要是最后一个方法。传入一个配置的key以及默认值，获取对应的系统配置值，若为空或者为null，返回对应的默认值。

### compare 方法

```java
public static int compare(int x, int y) {
  return (x < y) ? -1 : ((x == y) ? 0 : 1);
}
```

比较两个整数，`x < y` 返回-1，`x == y` 返回0，`x > y` 返回1

### compareTo 方法

```java
public int compareTo(Integer anotherInteger) {
  return compare(this.value, anotherInteger.value);
}
```

内部调用`compare`方法实现具体逻辑

### compareUnsigned 方法

```java
public static int compareUnsigned(int x, int y) {
  return compare(x + MIN_VALUE, y + MIN_VALUE);
}
```

两个输入数当作无符号整数进行比较，这里通过加上`MIN_VALUE`确保在范围内。有个小技巧，-1加上`MIN_VALUE`后会变成最大正数。

```java
System.out.println((-1 + Integer.MIN_VALUE) == Integer.MAX_VALUE); // true
```

### highestOneBit 方法

```java
public static int highestOneBit(int i) {
  // HD, Figure 3-1
  i |= (i >>  1);
  i |= (i >>  2);
  i |= (i >>  4);
  i |= (i >>  8);
  i |= (i >> 16);
  return i - (i >>> 1);
}

// 移位或逻辑，保证1+2+4+8+16=31位最后都为1，相减最后保留最高位1
// 输入              00010000 00000000 00000000 00000001
// i |= (i >>  1)   00011000 00000000 00000000 00000000
// i |= (i >>  2)   00011110 00000000 00000000 00000000
// i |= (i >>  4)   00011111 11100000 00000000 00000000
// i |= (i >>  8)   00011111 11111111 11100000 00000000
// i |= (i >> 16)   00011111 11111111 11111111 11111111
// i - (i >>> 1)    00010000 00000000 00000000 00000000
```

获取最高位为1，其余为0的整数值。通过位移或逻辑，将最高位右边1位设为1，然后2倍增长左移或操作，1的位数不断增加，最后1+2+4+8+16=31，可以确保覆盖所有可能性。然后使用`i - (i >>> 1)`保留最高位1返回结果。

### lowestOneBit 方法

```java
public static int lowestOneBit(int i) {
  // HD, Section 2-1
  return i & -i;
}

// 输入 00000000 00000000 00000000 11101010
// -i  11111111 11111111 11111111 00010110
// 结果 00000000 00000000 00000000 00000010
```

获取最低位1，其余位为0的值。负数以正数的补码表示，对整数的二进制进行取反码然后加1，得到的结果与输入二进制进行与操作，结果就是最低位1保留，其他位为0。

### bitCount 方法

```java
public static int bitCount(int i) {
  // HD, Figure 5-2
  i = i - ((i >>> 1) & 0x55555555);
  i = (i & 0x33333333) + ((i >>> 2) & 0x33333333);
  i = (i + (i >>> 4)) & 0x0f0f0f0f;
  i = i + (i >>> 8);
  i = i + (i >>> 16);
  return i & 0x3f;
}
```

统计二进制格式下1的数量。代码第一眼看着是懵的，都是位运算，实际里面实现的算法逻辑还是很巧妙的，着实佩服，这里就不介绍了，感兴趣的可以看下别人的具体分析文章 [java源码Integer.bitCount算法解析，分析原理（统计二进制bit位）](https://segmentfault.com/a/1190000015763941)这篇文章，我觉得已经说得很清晰了。

### rotateXXX 方法

```java
public static int rotateLeft(int i, int distance) {
  return (i << distance) | (i >>> -distance);
}

public static int rotateRight(int i, int distance) {
  return (i >>> distance) | (i << -distance);
}
```

旋转二进制，`rotateLeft`将特定位数的高位bit放置低位，返回对应的数值；`rotateRight`将特定位数的低位bit放置高位，返回对应的数值。当`distance`是负数的时候，`rotateLeft(val, -distance) == rotateRight(val, distance)`以及`rotateRight(val, -distance) == rotateLeft(val, distance)`。另外，当`distance`是32的任意倍数时，实际是没有效果的 ，相当于无操作。

这里需要说一下`(i >>> -distance)`，`diatance`为正数时，右移一个负数逻辑相当于`i >>> 32+(-distance)`

### reverse 方法

```java
public static int reverse(int i) {
  // HD, Figure 7-1
  i = (i & 0x55555555) << 1 | (i >>> 1) & 0x55555555;
  i = (i & 0x33333333) << 2 | (i >>> 2) & 0x33333333;
  i = (i & 0x0f0f0f0f) << 4 | (i >>> 4) & 0x0f0f0f0f;
  i = (i << 24) | ((i & 0xff00) << 8) |
    ((i >>> 8) & 0xff00) | (i >>> 24);
  return i;
}
```

看着是不是和`bitCount`有点类似，其实核心逻辑相似。先是交换相邻1位bit的顺序，然后再交换相邻2位bit顺序，再继续交换相邻4位bit顺序，这样子每一个byte内的bit流顺序已经翻转过来了，然后采用和`reverseBytes`一样的逻辑，将对应bit位按字节为单位翻转，这样子就完成了所有bit的翻转操作，甚是绝妙。

### signum 方法

```java
public static int signum(int i) {
  // HD, Section 2-7
  return (i >> 31) | (-i >>> 31);
}

// 输入 1      00000000 00000000 00000000 00000001
// i >> 31    00000000 00000000 00000000 00000000
// -i         11111111 11111111 11111111 11111111
// -i >>> 31  00000000 00000000 00000000 00000001
// 结果返回1
```

获取符号数，若为负数，返回-1；若为0则返回0；为正数，则返回1。

### reverseBytes 方法

```java
public static int reverseBytes(int i) {
  return ((i >>> 24)           ) |
    ((i >>   8) &   0xFF00) |
    ((i <<   8) & 0xFF0000) |
    ((i << 24));
}
```

按照输入参数二进制格式，以字节为单位翻转，返回对应的结果数值。`(i >>> 24) | (i << 24)`交换最高8位和最低8位bit位置。`((i >> 8) & 0xFF00) | ((i << 8) & 0xFF0000) `为交换中间16位bit位置的逻辑。

### sum、max、min方法

```java
public static int sum(int a, int b) {
  return a + b;
}

public static int max(int a, int b) {
  return Math.max(a, b);
}

public static int min(int a, int b) {
  return Math.min(a, b);
}
```

这个就不说了，很简单的方法。

## 总结

由于存在很多的位运算逻辑，第一眼感觉代码逻辑是比较复杂的，但是当慢慢品味时，发现代码思路甚是奇妙，算法之神奇，只能拍手称赞👏。为了性能，所做的各种小优化也是体现其中，不过部分这个比较hack的值设定缺乏完整的注释理解也是比较困难。总而言之，只能感慨代码实现之奇妙。