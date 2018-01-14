---
title: "Python学习 一 递归"
date: 2014-11-10 23:11:48
lastmod: 2018-01-14T20:55:16+08:00
draft: false
keywords: []
description: ""
tags: []
categories: ['Python']
author: ""

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
---

 递归的方法虽然效率不高，但是有些地方的使用( *对时间的要求不高时* )还是可以轻松的解决一些问题的，也能够用来替代一些循环，在自己的做题中使用还是可以的。这里是使用递归实现的 **二叉查找方法** ，让自己对递归更好的理解。
<!--more-->

## 二分查找的关键
   1. 如果上下限相同，那么就是数字所在的位置，返回
   2. 否则找到两者的中点(上下限的平均值)，查找数字是在左侧还是在右侧，继续查找数字所在的那半部分
   3. 关键的是查找的序列是*有序*且为*升序*

***提示：标准库中的 bisect 模块可以非常有效的实现二分查找 。***

``` python
def search (sequence,number,lower,upper) :
       if lower == uppper :
           assert number == sequence[upper]  #断言，用来判断数字是否为找到的数，否的话引起一个错误
           return upper
       else :
           middle = (lower + upper) //2   #整除
           if number > sequence[middle]
                 return search(sequence,number,middle+1,upper)
           else :
                 return search(sequence,number,lower,middle)
```
