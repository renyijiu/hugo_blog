---
title: "Float的精度问题"
date: 2018-05-22T17:40:53+08:00
lastmod: 2018-05-22T17:40:53+08:00
draft: false
keywords: ['Float', '精度', 'ruby']
description: ""
tags: ["ruby"]
categories: ['Ruby']
author: "renyijiu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: true
mathjax: false
---

最近在写测试时，某一个数据随机生成了一个很大的值，然后跑测试发现一直不过，后面仔细排查发现数据存入数据库前与读取出来的值不一致，最后确定应该是对应数据类型设置为Float导致精度不准的问题，这里回顾下float的精度问题。<!--more-->

ruby中的float精度不准确这个应该是很容易了解的，官方文档中开头位置的就有说明。不过自己脑中一直认为小数为0的浮点数应该是准确的，这次的问题让自己明白了这个想法的错误。

> [Float](https://ruby-doc.org/core-2.3.1/Float.html) objects represent inexact real numbers using the native architecture's double-precision floating point representation.

双精度浮点数(double)使用64位（8字节）来存储一个浮点数。64位又可以分为三个部分：

* 符号位S：第一位是正负数符号位（sign），0代表正数，1代表负数。
* 指数位E：中间的11位数存储指数（exponent），用来表示次方。
* 尾数位M：最后的52位是尾数（mantissa）。

![](https://upload.wikimedia.org/wikipedia/commons/7/76/General_double_precision_float.png)

### 范围

单精度（32位）和双精度（64位）的范围是由指数的位数来决定的。分布如下：

单精度：1bit（符号位）    8bits（指数位）    23bits（尾数位）  

双精度：1bit（符号位）    11bits（指数位）    52bits（尾数位）

科学计数法中的指数也可以为负数，一种方法是指数位部分也设置一个符号位，第二种是IEEE754采取的方式，设置一个偏移，使指数部分永远表现为一个非负数，然后减去某个偏移值才是真实的指数。

64bits浮点数设置的偏移值为1023，即范围为-2^1024 ~ +2^1024，也就是-1.79E+308 ~ +1.79E+308。

单精度的范围为-2^128 ~ +2^128，即-3.40E+38 ~ +3.40E+38。

### 精度

浮点数的精度是由尾数部分控制的。对于二进制的科学计数法，整数部分默认是1，默认不占用位置，因此尾数部分就存储了小数点后面的部分。

单精度：2^23 = 8388608，一共七位，这意味着最多能有7位有效数字，但绝对能保证的为6位，也就是单精度浮点数为6~7位有效数字。

双精度：2^52 = 4503599627370496，一共16位，同理，双精度浮点数的有效数字为15~16位。

当数字超过对应的有效数字位数时就会出现一定的偏差。

### 总结

* 指数和尾数的长度有限制，即对应的浮点数大小及精度有限制。
* 符号位决定正负，指数域决定数量级，尾数域决定精度。

这里写的比较简单，主要是为了解决自己的疑惑（浮点数对应的精度问题），有兴趣的可以看看下面的两篇参考资料，写的比较详细，更加的深入，感觉不错！

### 参考资料

* [该死的IEEE-754浮点数](https://segmentfault.com/a/1190000009084877)
* [JavaScript 浮点数陷阱及解法](https://github.com/camsong/blog/issues/9)



