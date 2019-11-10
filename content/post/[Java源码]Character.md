---
title: "[Java源码]Character"
date: 2019-10-16T17:04:16+08:00
lastmod: 2019-11-10T17:04:16+08:00
draft: false
keywords: ["Java源码", "Charecter"]
description: "Java源码 - Character"
tags: ["Java源码", "Character"]
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

这次来看看`Character`的源代码，基于 *jdk1.8.0_181.jdk* 版本，如有错误，欢迎联系指出。这也是Java的8个基础数据类型的最后一篇解析了，如果你还没看过其他的，可以去文章列表查看下其他的源码分析文章。

<!--more-->

## 前言

`Character`是`char`基础数据类型的包装类，提供了大小写转换、字符类型检测等方法。`char`数据类型基于原始的`Unicode`规范，将字符定义为**固定宽度**的16位实体。(Unicode规范也在发展更新，目前已经允许超过16位表示，具体的介绍可以参考[Unicode - 维基百科](https://zh.wikipedia.org/wiki/Unicode)。因为16位范围的限定，所以`char`只支持`U+0000 to U+FFFF`的`Basic Multilingual Plane(BMP)`，具体的介绍可以参考[Unicode字符平面映射](https://zh.wikipedia.org/wiki/Unicode字符平面映射)，另外这篇博客有些概念讲的也还不错 [编码简介：utf8,utf16以及其它](https://github.com/creeperyang/blog/issues/4)。因为源码中存在大量的常量定义字符集之类的逻辑，后续可能会省略部分介绍。

## 类定义

```java
public final class Character implements java.io.Serializable, Comparable<Character>
```

带有`final`标识，也就是说是**不可继承的**。另外实现了`Serializable`接口，所以是可以序列化的；实现了`Comparable`接口，可以进行比较。

## 属性

```java
public static final int MIN_RADIX = 2;

public static final int MIN_RADIX = 36;
```

* `MIN_RADIX`定义了最小进制数，为2
* `MIN_RADIX`定义了最大进制数，为36

```java
public static final char MIN_VALUE = '\u0000';

public static final char MAX_VALUE = '\uFFFF';
```

* `MIN_VALUE`定义了最小字符，为`'\u0000'`
* `MAX_VALUE`定义了最大字符，为`'\uFFFF'`

```java
public static final Class<Character> TYPE = (Class<Character>) Class.getPrimitiveClass("char");
```

获取类信息，`Character.TYPE == char.class`两者是等价的

```java
public static final int SIZE = 16;
```

表示了`Character`的bit数，为16位

```java
public static final int BYTES = SIZE / Byte.SIZE;
```

表示了`Character`的字节数，计算值固定为2

```java
public static final byte UNASSIGNED = 0;

......

public static final byte FINAL_QUOTE_PUNCTUATION = 30;
```

定义各个字符类型的常量值，如为分配字符、大写字符、小写字符等。

```java
static final int ERROR = 0xFFFFFFFF;
```

错误标志，使用`int`(code point)以避免与`U+FFFF`混淆。

任何文字在`Unicode`中都对应一个数字编号，这个编号就称为`code point`。`code point`的值写作U+XXXX的格式。

```java
/**
* 未定义的双向字符类型，未定义的char值在Unicode规范中具有不确定的方向性。
*/
public static final byte DIRECTIONALITY_UNDEFINED = -1;

/**
* Unicode规范中的强双向字符类型“ L”。
* LRM，大多数字母，音节，汉字表意文字，非欧洲或非阿拉伯数字，......
*/
public static final byte DIRECTIONALITY_LEFT_TO_RIGHT = 0;

......
  
/**
* Unicode规范中的强双向字符类型“ RLO”。
*/
public static final byte DIRECTIONALITY_RIGHT_TO_LEFT_OVERRIDE = 17;

/**
* Unicode规范中的弱双向字符类型“ PDF”。
*/
public static final byte DIRECTIONALITY_POP_DIRECTIONAL_FORMAT = 18;
```

定义了一些双向字符类型的常量值。相关介绍可以参考[双向文稿](https://zh.wikipedia.org/wiki/雙向文稿)。

```java
/**
* UTF-16的高位代理(前导代理)，表示的最小值
*/
public static final char MIN_HIGH_SURROGATE = '\uD800';

/**
* UTF-16的高位代理(前导代理)，表示的最大值
*/
public static final char MAX_HIGH_SURROGATE = '\uDBFF';

/**
* UTF-16的低位代理(后尾代理)，表示的最小值
*/
public static final char MIN_LOW_SURROGATE  = '\uDC00';

/**
* UTF-16的低位代理(后尾代理)，表示的最大值
*/
public static final char MAX_LOW_SURROGATE  = '\uDFFF';

/**
* UTF-16编码增补码表示的最小char值
*/
public static final char MIN_SURROGATE = MIN_HIGH_SURROGATE;

/**
* UTF-16编码增补码表示的最大char值
*/
public static final char MAX_SURROGATE = MAX_LOW_SURROGATE;

/**
* 辅助平面的最小的code point
*/
public static final int MIN_SUPPLEMENTARY_CODE_POINT = 0x010000;

/**
* 最小的code point
*/
public static final int MIN_CODE_POINT = 0x000000;

/**
* 最大的code point
*/
public static final int MAX_CODE_POINT = 0X10FFFF;
```

## 内部类

```java
public static class Subset  {

    private String name;

    /**
    * 创建新实例
    */
    protected Subset(String name) {
        if (name == null) {
          	throw new NullPointerException("name");
        }
        this.name = name;
    }

    /**
    * equals
    * 这个方法只是为了确保所有的Subset类中存在 equals() 方法
    */
    public final boolean equals(Object obj) {
      	return (this == obj);
    }

    /**
    * 调用 Object#hashCode，返回对应的值
    * 这个方法只是为了确保所有的Subset类中存在 hashCode() 方法
    */
    public final int hashCode() {
      	return super.hashCode();
    }

    public final String toString() {
      	return name;
    }
}
```

这个类的实例表示Unicode字符集的特定子集。Character类定义的子集集合是`UnicodeBlock`，Java API的其他部分可能会为了自身的使用定义其他的子集。

```java
public static final class UnicodeBlock extends Subset {
    /**
    * 定义初始大小为256的HashMap
    */
    private static Map<String, UnicodeBlock> map = new HashMap<>(256);

    /**
    * 根据指定的标识符创建UnicodeBlock，这个标识符必须与块标识符一致
    */
    private UnicodeBlock(String idName) {
        super(idName);
        map.put(idName, this);
    }

    /**
    * 根据指定的标识符和别名创建UnicodeBlock
    */
    private UnicodeBlock(String idName, String alias) {
        this(idName);
        map.put(alias, this);
    }

    /**
    * 根据指定的标识符和一系列别名创建UnicodeBlock
    */
    private UnicodeBlock(String idName, String... aliases) {
        this(idName);
        for (String alias : aliases)
          map.put(alias, this);
    }

    /**
    * 定义各类字符块，中间略过，有兴趣的可以自行查看
    */
    public static final UnicodeBlock  BASIC_LATIN =
        new UnicodeBlock("BASIC_LATIN",
                         "BASIC LATIN",
                         "BASICLATIN");
    // ......
    public static final UnicodeBlock ARABIC_MATHEMATICAL_ALPHABETIC_SYMBOLS =
        new UnicodeBlock("ARABIC_MATHEMATICAL_ALPHABETIC_SYMBOLS",
                         "ARABIC MATHEMATICAL ALPHABETIC SYMBOLS",
                         "ARABICMATHEMATICALALPHABETICSYMBOLS");

    /**
    * 对应的各个Unicode块的起始位置
    */
    private static final int blockStarts[] = {
        0x0000,   // 0000..007F; Basic Latin
        0x0080,   // 0080..00FF; Latin-1 Supplement
        0x0100,   // 0100..017F; Latin Extended-A
        // ......
        0xE01F0,  //               unassigned
        0xF0000,  // F0000..FFFFF; Supplementary Private Use Area-A
        0x100000  // 100000..10FFFF; Supplementary Private Use Area-B
    }

    /**
    * UnicodeBlock集合，部分位置为空，未使用
    */
    private static final UnicodeBlock[] blocks = {
        BASIC_LATIN,
        LATIN_1_SUPPLEMENT,
        // ......
        null,
        SUPPLEMENTARY_PRIVATE_USE_AREA_A,
        SUPPLEMENTARY_PRIVATE_USE_AREA_B
    }

    /**
    * 返回包含指定字符的Unicode block，存在null的可能性
    * 此方法不能处理增补码字符
    */
    public static UnicodeBlock of(char c) {
	      return of((int)c);
    }

    /**
    * 返回包含指定字符的Unicode block，存在null的可能性
    * 支持所有的Unicode字符
    */
    public static UnicodeBlock of(int codePoint) {
        if (!isValidCodePoint(codePoint)) {
          	throw new IllegalArgumentException();
        }

        int top, bottom, current;
        bottom = 0;
        top = blockStarts.length;
        current = top/2;

        // 二分查找法
        // invariant: top > current >= bottom && codePoint >= unicodeBlockStarts[bottom]
        while (top - bottom > 1) {
            if (codePoint >= blockStarts[current]) {
              	bottom = current;
            } else {
              	top = current;
            }
            current = (top + bottom) / 2;
        }
        return blocks[current];
    }

    /**
    * 返回具有给定名称的UnicodeBlock。块名称由Unicode标准确定。
    */
    public static final UnicodeBlock forName(String blockName) {
        UnicodeBlock block = map.get(blockName.toUpperCase(Locale.US));
        if (block == null) {
          	throw new IllegalArgumentException();
        }
        return block;
    }
}
```

这个类继承了`Subset`，代表了Unicode规范中字符块的字符子集集合。字符块通常定义用于特定脚本或者目的的字符。一个字符最多只会在一个字符块中出现存在。

```java
private static class CharacterCache {
    private CharacterCache(){}

    static final Character cache[] = new Character[127 + 1];

    static {
      for (int i = 0; i < cache.length; i++)
        cache[i] = new Character((char)i);
    }
}
```

定义了CharacterCache缓存类，缓存了从0～127的128个字符实例。

## Enum

```java
/**
* UnicodeScript枚举值，一系列表示字符脚本的字符子集。
*/
public static enum UnicodeScript {
    COMMON,
    // ......
    MIAO,
    UNKNOWN;
  
  	/**
    * UnicodeScript对应的起始位数组
    */
    private static final int[] scriptStarts = {
      0x0000,   // 0000..0040; COMMON
      0x0041,   // 0041..005A; LATIN
      // ......
      0xE0100,  // E0100..E01EF; INHERITED
      0xE01F0   // E01F0..10FFFF; UNKNOWN
    }

    /**
    * UnicodeScript的数组
    */
    private static final UnicodeScript[] scripts = {
      COMMON,
      LATIN,
      // ......
      INHERITED,
      UNKNOWN
    }

    /**
    * UnicodeScript对应的别名map
    */
    private static HashMap<String, Character.UnicodeScript> aliases;
    static {
      aliases = new HashMap<>(128);
      aliases.put("ARAB", ARABIC);
      aliases.put("ARMI", IMPERIAL_ARAMAIC);
      // ......
      aliases.put("ZYYY", COMMON);
      aliases.put("ZZZZ", UNKNOWN);
    }
  
  	/**
  	* 返回给定字符所属的Unicode script的枚举常量
  	*/
    public static UnicodeScript of(int codePoint) {
        if (!isValidCodePoint(codePoint))
          	throw new IllegalArgumentException();
        int type = getType(codePoint);
        // leave SURROGATE and PRIVATE_USE for table lookup
        if (type == UNASSIGNED)
	          return UNKNOWN;
        int index = Arrays.binarySearch(scriptStarts, codePoint);
        if (index < 0)
  	        index = -index - 2;
        return scripts[index];
    }
  
  	/**
  	* 通过给定的 Unicode script名称或者别名返回对应的UnicodeScript枚举常量
  	*/
    public static final UnicodeScript forName(String scriptName) {
        scriptName = scriptName.toUpperCase(Locale.ENGLISH);
        //.replace(' ', '_'));
        UnicodeScript sc = aliases.get(scriptName);
        if (sc != null)
          	return sc;
        return valueOf(scriptName);
    }
}
```

## 方法

### valueOf 方法

```java
public static Character valueOf(char c) {
  if (c <= 127) { // must cache
    return CharacterCache.cache[(int)c];
  }
  return new Character(c);
}
```

字符小于等于127时，从`CharacterCache`中获取缓存结果，避免重新实例化；大于127时，实例化返回结果。

### charValue 方法

```java
public char charValue() {
  return value;
}
```

返回对应字符的值。

### hashCode 方法

```java
@Override
public int hashCode() {
  	return Character.hashCode(value);
}

public static int hashCode(char value) {
  	return (int)value;
}
```

将字符转换成对应的int值作为hashCode结果返回。

### equals 方法

```java
public boolean equals(Object obj) {
  if (obj instanceof Character) {
    return value == ((Character)obj).charValue();
  }
  return false;
}
```

判断输入参数是否为`Character`的实例，然后判断其对应的值是否相等。

### toString 方法

```java
public String toString() {
  char buf[] = {value};
  return String.valueOf(buf);
}

public static String toString(char c) {
  return String.valueOf(c);
}
```

两个方法，一个为静态方法，调用String的方法转换成字符格式，比较简单。

### isValidCodePoint 方法

```java
public static boolean isValidCodePoint(int codePoint) {
  // Optimized form of:
  //     codePoint >= MIN_CODE_POINT && codePoint <= MAX_CODE_POINT
  //     codePoint >= 0x000000 && codePoint <= 0X10FFFF
  // 左边16为从0000～FFFF，必定在范围内，比较无意义，直接忽略
  int plane = codePoint >>> 16;
  return plane < ((MAX_CODE_POINT + 1) >>> 16); // plane < 0X11
}
```

判断字符的codePoint是否有效。

### isBmpCodePoint 方法

```java
/**
* 确定指定的代码点是否是一个BMP代码点
* 这样的代码点能够以一个单独的char表示
*/
public static boolean isBmpCodePoint(int codePoint) {
  // 范围为0x0000～0xFFFF，优化后的判断逻辑
  return codePoint >>> 16 == 0;
  // Optimized form of:
  //     codePoint >= MIN_VALUE && codePoint <= MAX_VALUE
  // We consistently use logical shift (>>>) to facilitate
  // additional runtime optimizations.
}
```

### isSupplementaryCodePoint 方法

```java
public static boolean isSupplementaryCodePoint(int codePoint) {
  // 0x10000~0x10FFFF范围
  return codePoint >= MIN_SUPPLEMENTARY_CODE_POINT
    && codePoint <  MAX_CODE_POINT + 1;
}
```

判断指定的代码点是否是补充代码点（即不是BMP代码点）。

### isXXXSurrogate 方法

```java
/**
* 输入参数是否是一个Unicode高位代理代码单元
* 这样的值不代表他们自己的字符，被用来表示UTF-16的补充字符
*/
public static boolean isHighSurrogate(char ch) {
  // Help VM constant-fold; MAX_HIGH_SURROGATE + 1 == MIN_LOW_SURROGATE
  return ch >= MIN_HIGH_SURROGATE && ch < (MAX_HIGH_SURROGATE + 1);
}

/**
* 输入参数是否是一个Unicode低位代理代码单元
* 这样的值不代表他们自己的字符，被用来表示UTF-16的补充字符
*/
public static boolean isLowSurrogate(char ch) {
  return ch >= MIN_LOW_SURROGATE && ch < (MAX_LOW_SURROGATE + 1);
}

/**
* 输入参数是否是一个Unicode代理代码单元
* 这样的值不代表他们自己的字符，被用来代表UTF-16的补充字符
*/
public static boolean isSurrogate(char ch) {
  return ch >= MIN_SURROGATE && ch < (MAX_SURROGATE + 1);
}

/**
* 输入的两个字符参数是否是有效的Unicode代理对
*/
public static boolean isSurrogatePair(char high, char low) {
  return isHighSurrogate(high) && isLowSurrogate(low);
}
```

### codePointXXX 方法

```java
/**
* 返回seq中指定下标的char的codePoint
* 如果c1在高位代理范围内，并且随后的字符在低位代理的范围内，返回增补代码点
* 否则直接返回对应index的值
*/
public static int codePointAt(CharSequence seq, int index) {
  char c1 = seq.charAt(index);
  if (isHighSurrogate(c1) && ++index < seq.length()) {
    char c2 = seq.charAt(index);
    if (isLowSurrogate(c2)) {
      return toCodePoint(c1, c2);
    }
  }
  return c1;
}

/**
* 内部调用codePointAtImpl实现
* 参数改为字符数组和下标
*/
public static int codePointAt(char[] a, int index) {
  return codePointAtImpl(a, index, a.length);
}

/**
* 内部调用codePointAtImpl实现
* 参数为字符数组、index和limit限制
*/
public static int codePointAt(char[] a, int index, int limit) {
	// 对index增加了limit参数限制
  if (index >= limit || limit < 0 || limit > a.length) {
    throw new IndexOutOfBoundsException();
  }
  return codePointAtImpl(a, index, limit);
}

/**
* 与codePointAt(CharSequence seq, int index)实现逻辑一致
* 返回字符数组中指定下标的char的codePoint
* 如果c1在高位代理范围内，并且随后的字符在低位代理的范围内，返回增补代码点
* 否则直接返回对应index的值
*/
static int codePointAtImpl(char[] a, int index, int limit) {
  char c1 = a[index];
  if (isHighSurrogate(c1) && ++index < limit) {
    char c2 = a[index];
    if (isLowSurrogate(c2)) {
      return toCodePoint(c1, c2);
    }
  }
  return c1;
}

/**
* 与codePointAt逻辑类似，返回指定坐标的前一个字符；如果前一个是增补代码点，则返回完整的字符codePoint
* 如果index-1的c2在低位代理范围内，并且index-2的c1字符在高位代理的范围内，返回增补代码点
* 否则直接返回对应index前一位的值
*/
public static int codePointBefore(CharSequence seq, int index) {
  char c2 = seq.charAt(--index);
  if (isLowSurrogate(c2) && index > 0) {
    char c1 = seq.charAt(--index);
    if (isHighSurrogate(c1)) {
      return toCodePoint(c1, c2);
    }
  }
  return c2;
}

/**
* 内部调用codePointBeforeImpl实现
* 参数为字符数组和定位
*/
public static int codePointBefore(char[] a, int index) {
  return codePointBeforeImpl(a, index, 0);
}

/**
* 内部调用codePointBeforeImpl实现
* 参数为字符数组，定位和起始位置，增加了对于index的限制判断
*/
public static int codePointBefore(char[] a, int index, int start) {
  if (index <= start || start < 0 || start >= a.length) {
    throw new IndexOutOfBoundsException();
  }
  return codePointBeforeImpl(a, index, start);
}

/**
* 与codePointBefore(CharSequence seq, int index)实现逻辑一致
* 如果index-1的c2在低位代理范围内，并且index-2的c1字符在高位代理的范围内，返回增补代码点
* 否则直接返回对应index前一位的值
*/
static int codePointBeforeImpl(char[] a, int index, int start) {
  char c2 = a[--index];
  if (isLowSurrogate(c2) && index > start) {
    char c1 = a[--index];
    if (isHighSurrogate(c1)) {
      return toCodePoint(c1, c2);
    }
  }
  return c2;
}
```

### charCount 方法

```java
/**
* 获取代码点的字符数；如果是增补代码返回2，否则返回1
*/
public static int charCount(int codePoint) {
  return codePoint >= MIN_SUPPLEMENTARY_CODE_POINT ? 2 : 1;
}
```

### toCodePoint 方法

```java
/**
* 将输入的代理对转换成对应的补充代码点
* 这个方法不会校验输入参数的有效性，需要自己调用 isSurrogatePair(char, char) 进行判断
*/
public static int toCodePoint(char high, char low) {
  // Optimized form of:
  // return ((high - MIN_HIGH_SURROGATE) << 10)
  //         + (low - MIN_LOW_SURROGATE)
  //         + MIN_SUPPLEMENTARY_CODE_POINT;
  return ((high << 10) + low) + (MIN_SUPPLEMENTARY_CODE_POINT
                                 - (MIN_HIGH_SURROGATE << 10)
                                 - MIN_LOW_SURROGATE);
}
```

### XXXSurrogate 方法

```java
/**
* 返回指定字符在UTF-16编码中的高位代理
* 如果输入参数不是一个增补代码，则返回一个未定义的char
*/
public static char highSurrogate(int codePoint) {
  return (char) ((codePoint >>> 10)
                 + (MIN_HIGH_SURROGATE - (MIN_SUPPLEMENTARY_CODE_POINT >>> 10)));
}

/**
* 返回指定字符在UTF-16编码中的低位代理
* 如果输入参数不是一个增补代码，则返回一个未定义的char
*/
public static char lowSurrogate(int codePoint) {
  return (char) ((codePoint & 0x3ff) + MIN_LOW_SURROGATE);
}
```

### toChars 方法

```java
/**
* 将指定的unicode code point转换成对应的UTF-16表示形式
* 如果是输入参数BMP，则将结果放在指定的dstIndex处，结果返回1
* 如果是增补代码，则结果放在dstIndex和dstIndex+1，结果返回2
*/
public static int toChars(int codePoint, char[] dst, int dstIndex) {
  if (isBmpCodePoint(codePoint)) {
    dst[dstIndex] = (char) codePoint;
    return 1;
  } else if (isValidCodePoint(codePoint)) {
    toSurrogates(codePoint, dst, dstIndex);
    return 2;
  } else {
    throw new IllegalArgumentException();
  }
}

/**
* 逻辑与 toChars(int codePoint, char[] dst, int dstIndex) 一致
*/
public static char[] toChars(int codePoint) {
  if (isBmpCodePoint(codePoint)) {
    return new char[] { (char) codePoint };
  } else if (isValidCodePoint(codePoint)) {
    char[] result = new char[2];
    toSurrogates(codePoint, result, 0);
    return result;
  } else {
    throw new IllegalArgumentException();
  }
}

/**
* 处理增补代码逻辑，高位代理放在index，低位代理放在index+1
*/
static void toSurrogates(int codePoint, char[] dst, int index) {
  // We write elements "backwards" to guarantee all-or-nothing
  dst[index+1] = lowSurrogate(codePoint);
  dst[index] = highSurrogate(codePoint);
}
```

### codePointCount 方法

```java
/**
* 计算指定范围内的字符序列的长度
* 此方法与String.length存在一定的差别，由于增补代码，String.length返回2，而此方法返回1
* 例如：𠮷，这个字符使用String.length结果为2，而对于此方法结果为1
*/
public static int codePointCount(CharSequence seq, int beginIndex, int endIndex) {
  int length = seq.length();
  if (beginIndex < 0 || endIndex > length || beginIndex > endIndex) {
    throw new IndexOutOfBoundsException();
  }
  int n = endIndex - beginIndex;
  for (int i = beginIndex; i < endIndex; ) {
    if (isHighSurrogate(seq.charAt(i++)) && i < endIndex &&
        isLowSurrogate(seq.charAt(i))) {
      n--;
      i++;
    }
  }
  return n;
}

/**
* 内部调用codePointCountImpl实现
*/
public static int codePointCount(char[] a, int offset, int count) {
  if (count > a.length - offset || offset < 0 || count < 0) {
    throw new IndexOutOfBoundsException();
  }
  return codePointCountImpl(a, offset, count);
}

/**
* 实现逻辑与 codePointCount(CharSequence seq, int beginIndex, int endIndex) 一致
* 注意点也一致
*/
static int codePointCountImpl(char[] a, int offset, int count) {
  int endIndex = offset + count;
  int n = count;
  for (int i = offset; i < endIndex; ) {
    if (isHighSurrogate(a[i++]) && i < endIndex &&
        isLowSurrogate(a[i])) {
      n--;
      i++;
    }
  }
  return n;
}
```

### offsetByCodePoints 方法

```java
/**
* 返回字符序列按照输入index点进行一定偏移量操作后的索引位置
* 主要考虑增补代码的原因，增补代码会出现2位的偏移量，导致结果可能不是你所预期的那样
*/
public static int offsetByCodePoints(CharSequence seq, int index,                                     
                                     int codePointOffset) {
  int length = seq.length();
  if (index < 0 || index > length) {
    throw new IndexOutOfBoundsException();
  }

  int x = index;
  if (codePointOffset >= 0) {
    int i;
    for (i = 0; x < length && i < codePointOffset; i++) {
      if (isHighSurrogate(seq.charAt(x++)) && x < length &&
          isLowSurrogate(seq.charAt(x))) {
        x++;
      }
    }
    if (i < codePointOffset) {
      throw new IndexOutOfBoundsException();
    }
  } else {
    // 负偏移量处理，向前偏移
    int i;
    for (i = codePointOffset; x > 0 && i < 0; i++) {
      if (isLowSurrogate(seq.charAt(--x)) && x > 0 &&
          isHighSurrogate(seq.charAt(x-1))) {
        x--;
      }
    }
    if (i < 0) {
      throw new IndexOutOfBoundsException();
    }
  }
  return x;
}

/**
* 内部调用offsetByCodePointsImpl实现逻辑
*/
public static int offsetByCodePoints(char[] a, int start, int count,
                                     int index, int codePointOffset) {
  if (count > a.length-start || start < 0 || count < 0
      || index < start || index > start+count) {
    throw new IndexOutOfBoundsException();
  }
  return offsetByCodePointsImpl(a, start, count, index, codePointOffset);
}

/**
* 具体实现和offsetByCodePoints一致
* 返回字符数组按照输入index点进行一定偏移量操作后的索引位置
* 主要考虑增补代码的原因，增补代码会出现2位的偏移量，导致结果可能不是你所预期的那样
*/
static int offsetByCodePointsImpl(char[]a, int start, int count,
                                  int index, int codePointOffset) {
  int x = index;
  if (codePointOffset >= 0) {
    int limit = start + count;
    int i;
    for (i = 0; x < limit && i < codePointOffset; i++) {
      if (isHighSurrogate(a[x++]) && x < limit &&
          isLowSurrogate(a[x])) {
        x++;
      }
    }
    if (i < codePointOffset) {
      throw new IndexOutOfBoundsException();
    }
  } else {
    int i;
    for (i = codePointOffset; x > start && i < 0; i++) {
      if (isLowSurrogate(a[--x]) && x > start &&
          isHighSurrogate(a[x-1])) {
        x--;
      }
    }
    if (i < 0) {
      throw new IndexOutOfBoundsException();
    }
  }
  return x;
}
```

### isXXX 方法

```java
/**
* 是否是小写字符（不能处理增补代码情况）
*/
public static boolean isLowerCase(char ch) {
	return isLowerCase((int)ch);
}

/**
* 是否是小写字符（可以处理增补代码情况）
*/
public static boolean isLowerCase(int codePoint) {
  return getType(codePoint) == Character.LOWERCASE_LETTER ||
    CharacterData.of(codePoint).isOtherLowercase(codePoint);
}

/**
* 是否是大写字符（不能处理增补代码情况）
*/
public static boolean isUpperCase(char ch) {
  return isUpperCase((int)ch);
}

/**
* 是否是大写字符（可以处理增补代码情况）
*/
public static boolean isUpperCase(int codePoint) {
  return getType(codePoint) == Character.UPPERCASE_LETTER ||
    CharacterData.of(codePoint).isOtherUppercase(codePoint);
}

/**
* 判断是不是标题字符（主要针对一些特殊字符，不能处理增补代码）
*/
public static boolean isTitleCase(char ch) {
  return isTitleCase((int)ch);
}

/**
* 判断是不是标题字符（可以处理增补代码）
*/
public static boolean isTitleCase(int codePoint) {
  return getType(codePoint) == Character.TITLECASE_LETTER;
}

/**
* 判断是不是数字（不可以处理增补代码）
*/
public static boolean isDigit(char ch) {
  return isDigit((int)ch);
}

/**
* 判断是不是数字（可以处理增补代码）
*/
public static boolean isDigit(int codePoint) {
  return getType(codePoint) == Character.DECIMAL_DIGIT_NUMBER;
}

/**
* 同上，可以处理增补代码
*/
public static boolean isDefined(char ch) {
  return isDefined((int)ch);
}

/**
* 判断字符是不是在Unicode中定义（不能处理增补代码）
*/
public static boolean isDefined(char ch) {
  return isDefined((int)ch);
}

/**
* 同上，可以处理增补代码
*/
public static boolean isDefined(int codePoint) {
  return getType(codePoint) != Character.UNASSIGNED;
}

/**
* 判断是不是字母，包含大写字符、小写字母、标题字母、变换字母、其他字母
* 不能处理增补代码
*/
public static boolean isLetter(char ch) {
  return isLetter((int)ch);
}

/**
* 同上，可以处理增补代码
*/
public static boolean isLetter(int codePoint) {
  return ((((1 << Character.UPPERCASE_LETTER) |
            (1 << Character.LOWERCASE_LETTER) |
            (1 << Character.TITLECASE_LETTER) |
            (1 << Character.MODIFIER_LETTER) |
            (1 << Character.OTHER_LETTER)) >> getType(codePoint)) & 1)
    != 0;
}

/**
* 判断是不是字母或者数字（不能处理增补代码情况）
*/
public static boolean isLetterOrDigit(char ch) {
  return isLetterOrDigit((int)ch);
}

/**
* 同上，可以处理增补代码
*/
public static boolean isLetterOrDigit(int codePoint) {
  return ((((1 << Character.UPPERCASE_LETTER) |
            (1 << Character.LOWERCASE_LETTER) |
            (1 << Character.TITLECASE_LETTER) |
            (1 << Character.MODIFIER_LETTER) |
            (1 << Character.OTHER_LETTER) |
            (1 << Character.DECIMAL_DIGIT_NUMBER)) >> getType(codePoint)) & 1)
    != 0;
}

/**
* 判断指定的字符是否允许作为Java标识符中的第一个字符
*/
@Deprecated
public static boolean isJavaLetter(char ch) {
  return isJavaIdentifierStart(ch);
}

/**
* 判断指定的字符是否可能是Java标识符的一部分，而不是第一个字符
* （这个命名感觉很不符合含义啊！问号脸）
*/
@Deprecated
public static boolean isJavaLetterOrDigit(char ch) {
  return isJavaIdentifierPart(ch);
}

/**
* 确定指定的字符（Unicode code point）是否为字母。
* 包含大写字符、小写字符、标题字符、变化字符、其他字符、字母数字
*/
public static boolean isAlphabetic(int codePoint) {
  return (((((1 << Character.UPPERCASE_LETTER) |
             (1 << Character.LOWERCASE_LETTER) |
             (1 << Character.TITLECASE_LETTER) |
             (1 << Character.MODIFIER_LETTER) |
             (1 << Character.OTHER_LETTER) |
             (1 << Character.LETTER_NUMBER)) >> getType(codePoint)) & 1) != 0) ||
    CharacterData.of(codePoint).isOtherAlphabetic(codePoint);
}

/**
* 判断字母（Unicode code point）是否为中文、日文、韩文或者越南文
*/
public static boolean isIdeographic(int codePoint) {
  return CharacterData.of(codePoint).isIdeographic(codePoint);
}

/**
* 判断指定的字符是否允许作为Java标识符中的第一个字符（不能处理增补代码）
* 代替 isJavaLetter 方法
*/
public static boolean isJavaIdentifierStart(char ch) {
  return isJavaIdentifierStart((int)ch);
}

/**
* 同上，可以处理增补代码
*/
public static boolean isJavaIdentifierStart(int codePoint) {
  return CharacterData.of(codePoint).isJavaIdentifierStart(codePoint);
}

/**
* 确定指定的字符是否可能是Java标识符的一部分，而不是第一个字符（不能处理增补代码）
* 代替 isJavaLetterOrDigit 方法
*/
public static boolean isJavaIdentifierPart(char ch) {
  return isJavaIdentifierPart((int)ch);
}

/**
* 同上，可以处理增补代码
*/
public static boolean isJavaIdentifierPart(int codePoint) {
  return CharacterData.of(codePoint).isJavaIdentifierPart(codePoint);
}

/**
* 输入的字符是否允许作为Unicode标识符中的第一个字符（不能处理增补代码）
*/
public static boolean isUnicodeIdentifierStart(char ch) {
  return isUnicodeIdentifierStart((int)ch);
}

/**
* 同上，可以处理增补代码
*/
public static boolean isUnicodeIdentifierStart(int codePoint) {
  return CharacterData.of(codePoint).isUnicodeIdentifierStart(codePoint);
}

/**
* 输入的字符是否可能是Unicode标识符的一部分，而不是第一个字符（不能处理增补代码）
*/
public static boolean isUnicodeIdentifierPart(char ch) {
  return isUnicodeIdentifierPart((int)ch);
}

/**
* 同上，可以处理增补代码
*/
public static boolean isUnicodeIdentifierPart(int codePoint) {
  return CharacterData.of(codePoint).isUnicodeIdentifierPart(codePoint);
}

/**
* 判断输入的字符是不是在Java标识符或者Unicode标识符中可以视为可忽略符号（不能处理增补代码）
*/
public static boolean isIdentifierIgnorable(char ch) {
  return isIdentifierIgnorable((int)ch);
}

/**
* 同上，可以处理增补代码
*/
public static boolean isIdentifierIgnorable(int codePoint) {
  return CharacterData.of(codePoint).isIdentifierIgnorable(codePoint);
}

/**
* 判断输入字符是不是属于 ISO-LATIN-1下的空包字符
* 只有为'\t', '\n', '\f', '\r', ' '中的一个时，返回true
*/
@Deprecated
public static boolean isSpace(char ch) {
  return (ch <= 0x0020) &&
    (((((1L << 0x0009) |
        (1L << 0x000A) |
        (1L << 0x000C) |
        (1L << 0x000D) |
        (1L << 0x0020)) >> ch) & 1L) != 0);
}

/**
* 判断字符是否为Unicode空格字符
* 当且仅当Unicode标准将其指定为空格字符时，才将其视为空格字符。
* SPACE_SEPARATOR、LINE_SEPARATOR、PARAGRAPH_SEPARATOR其中一个时才为true
* 不能处理增补代码
*/
public static boolean isSpaceChar(char ch) {
  return isSpaceChar((int)ch);
}

/**
* 同上，可以处理增补代码
*/
public static boolean isSpaceChar(int codePoint) {
  return ((((1 << Character.SPACE_SEPARATOR) |
            (1 << Character.LINE_SEPARATOR) |
            (1 << Character.PARAGRAPH_SEPARATOR)) >> getType(codePoint)) & 1)
    != 0;
}

/**
* 判断字符是否是java中的空白字符，只有以下的情况才会返回ture
* <li> It is a Unicode space character ({@code SPACE_SEPARATOR},
*      {@code LINE_SEPARATOR}, or {@code PARAGRAPH_SEPARATOR})
*      but is not also a non-breaking space ({@code '\u005Cu00A0'},
*      {@code '\u005Cu2007'}, {@code '\u005Cu202F'}).
* <li> It is {@code '\u005Ct'}, U+0009 HORIZONTAL TABULATION.
* <li> It is {@code '\u005Cn'}, U+000A LINE FEED.
* <li> It is {@code '\u005Cu000B'}, U+000B VERTICAL TABULATION.
* <li> It is {@code '\u005Cf'}, U+000C FORM FEED.
* <li> It is {@code '\u005Cr'}, U+000D CARRIAGE RETURN.
* <li> It is {@code '\u005Cu001C'}, U+001C FILE SEPARATOR.
* <li> It is {@code '\u005Cu001D'}, U+001D GROUP SEPARATOR.
* <li> It is {@code '\u005Cu001E'}, U+001E RECORD SEPARATOR.
* <li> It is {@code '\u005Cu001F'}, U+001F UNIT SEPARATOR.
* 不能处理增补代码
*/
public static boolean isWhitespace(char ch) {
  return isWhitespace((int)ch);
}

/**
* 同上，可以处理增补代码
*/
public static boolean isWhitespace(int codePoint) {
  return CharacterData.of(codePoint).isWhitespace(codePoint);
}

/**
* 判断输入字符是不是ISO控制字符
* 在'\u005Cu0000' ～ '\u005Cu001F'或者'\u005Cu007F' ～ '\u005Cu009F'之间的字符为ISO控制字符
* 不能处理增补代码
*/
public static boolean isISOControl(char ch) {
  return isISOControl((int)ch);
}

/**
* 同上，可以处理增补代码
*/
public static boolean isISOControl(int codePoint) {
  // Optimized form of:
  //     (codePoint >= 0x00 && codePoint <= 0x1F) ||
  //     (codePoint >= 0x7F && codePoint <= 0x9F);
  return codePoint <= 0x9F &&
    (codePoint >= 0x7F || (codePoint >>> 5 == 0));
}

/**
* 判断根据Unicode标准，输入字符是不是镜像的
* 当以从右到左的文本显示时，镜像字符的字形应水平镜像。
* 不能处理增补代码
*/
public static boolean isMirrored(char ch) {
  return isMirrored((int)ch);
}

/**
* 同上，可以处理增补代码
*/
public static boolean isMirrored(int codePoint) {
  return CharacterData.of(codePoint).isMirrored(codePoint);
}
```

### toXXX 方法

```java
/**
* 转换成小写字符（不能处理增补代码）
*/
public static char toLowerCase(char ch) {
  return (char)toLowerCase((int)ch);
}

/**
* 同上，可以处理增补代码
*/
public static int toLowerCase(int codePoint) {
  return CharacterData.of(codePoint).toLowerCase(codePoint);
}

/**
* 转换成大写字母（不能处理增补代码）
*/
public static char toUpperCase(char ch) {
  return (char)toUpperCase((int)ch);
}

/**
* 同上，可以处理增补代码
*/
public static int toUpperCase(int codePoint) {
  return CharacterData.of(codePoint).toUpperCase(codePoint);
}

/**
* 转换成标题字符；如果对应的字符不存在，会返回大写字符作为对应的结果；
* 如果输入参数已经是标题字母，则返回其本身作为结果
* 不能处理增补代码
*/
public static char toTitleCase(char ch) {
  return (char)toTitleCase((int)ch);
}

/**
* 同上，可以处理增补代码
*/
public static int toTitleCase(int codePoint) {
  return CharacterData.of(codePoint).toTitleCase(codePoint);
}

/**
* 使用UnicodeData文件中的信息，将字符转换成对应的大写字符
*/
static int toUpperCaseEx(int codePoint) {
  assert isValidCodePoint(codePoint);
  return CharacterData.of(codePoint).toUpperCaseEx(codePoint);
}

/**
* 使用Unicode规范中SpecialCasing文件的信息，将字符转换成大写字符数组返回
*/
static char[] toUpperCaseCharArray(int codePoint) {
  // As of Unicode 6.0, 1:M uppercasings only happen in the BMP.
  assert isBmpCodePoint(codePoint);
  return CharacterData.of(codePoint).toUpperCaseCharArray(codePoint);
}
```

### digit 方法

```java
/**
* 返回在对应的进制下字符对应的数值
* 如果进制不在返回内或者参数字符对应的值不属于对应进制内的有效值，结果返回-1
* 不能处理增补代码
*/
public static int digit(char ch, int radix) {
  return digit((int)ch, radix);
}

/**
* 同上，可以处理增补代码
*/
public static int digit(int codePoint, int radix) {
  return CharacterData.of(codePoint).digit(codePoint, radix);
}
```

### getXXXX 方法

```java
/**
* 返回字符对应的数值
* 如果输入字符无对应的数值，则返回-1
* 如果输入字符无法使用一个非负数进行表示，则返回-2
* 不能处理增补代码
*/
public static int getNumericValue(char ch) {
  return getNumericValue((int)ch);
}

/**
* 同上，可以处理增补代码
*/
public static int getNumericValue(int codePoint) {
  return CharacterData.of(codePoint).getNumericValue(codePoint);
}

/**
* 返回表示字符常规类别的值（不能处理增补代码）
*/
public static int getType(char ch) {
  return getType((int)ch);
}

/**
* 同上，可以处理增补代码
*/
public static int getType(int codePoint) {
  return CharacterData.of(codePoint).getType(codePoint);
}

/**
* 返回给定字符的Unicode方向性属性
* 字符的方向性用于计算文本的视觉顺序
* 不能处理增补代码
*/
public static byte getDirectionality(char ch) {
  return getDirectionality((int)ch);
}

/**
* 同上，可以处理增补代码
*/
public static byte getDirectionality(int codePoint) {
  return CharacterData.of(codePoint).getDirectionality(codePoint);
}

/**
* 返回指定字符的Unicode名称，如果codePoint是未分配的，返回null
*/
public static String getName(int codePoint) {
  if (!isValidCodePoint(codePoint)) {
    throw new IllegalArgumentException();
  }
  String name = CharacterName.get(codePoint);
  if (name != null)
    return name;
  if (getType(codePoint) == UNASSIGNED)
    return null;
  UnicodeBlock block = UnicodeBlock.of(codePoint);
  if (block != null)
    return block.toString().replace('_', ' ') + " "
    + Integer.toHexString(codePoint).toUpperCase(Locale.ENGLISH);
  // should never come here
  return Integer.toHexString(codePoint).toUpperCase(Locale.ENGLISH);
}
```

### forDigit 方法

```java
/**
* 判断指定数值和进制参数的字符表示形式。如果不存在，则返回null字符（‘\u005Cu0000'）
*/
public static char forDigit(int digit, int radix) {
  if ((digit >= radix) || (digit < 0)) {
    return '\0';
  }
  if ((radix < Character.MIN_RADIX) || (radix > Character.MAX_RADIX)) {
    return '\0';
  }
  if (digit < 10) {
    return (char)('0' + digit);
  }
  return (char)('a' - 10 + digit);
}
```

### compare 方法

```java
/**
* 比较两个字符大小，内部调用compare方法实现
*/
public int compareTo(Character anotherCharacter) {
  return compare(this.value, anotherCharacter.value);
}

/**
* 直接进行相减，返回对应的结果
*/
public static int compare(char x, char y) {
  return x - y;
}
```

### reverseBytes 方法

```java
public static char reverseBytes(char ch) {
  return (char) (((ch & 0xFF00) >> 8) | (ch << 8));
}
```

按字节进行逆序，因为只有两个字节，所以比较简单；高8位和低8位进行位置调换即可。

## 总结

Character的源码非常的长，中间存在了许多的常量定义以及一些相关的方法。从自己的角度来说，因为主要是Unicode标准的实现，如果日常工作中不涉及这部分，实际上用的还是很少的，所以日常有个了解即可，待涉及相关内容再详细的学习。