---
title: "Stringy Strings"
date: 2017-08-21T22:08:36+08:00
lastmod: 2018-01-14T21:01:23+08:00
draft: false
keywords: ['codewars']
description: ""
tags: ['codewars']
categories: ['codewars']
author: ""

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
autoCollapseToc: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
---


```
write me a function stringy that takes a size and returns a string of alternating '1s' and '0s'.

the string should start with a 1.

a string with size 6 should return :'101010'.

with size 4 should return : '1010'.

with size 12 should return : '101010101010'.

The size will always be positive and will only use whole numbers.
```

<!--more-->

#### 题目

题目意思很简单，函数输入一个正整数，返回对应长度的"1010..."字符串，字符串总是以'1'开头。

#### 解法

自己的开始想到的第一解法就是遍历长度，每一位判断为1或者0，最后组装成字符串

```ruby
def stringy(size)
  size.times.map{|n| n.even? ? '1' : '0' }.join
end
```



另一种解法，因为是"10"的循环，所以

```ruby
def stringy(size)
  "10" * (size / 2) + "1" * (size % 2)
end
```



但是看别人的解法时发现ruby有提供一个`rjust`函数，因此更简单的解法：

```ruby
def stringy(s)
  "".rjust(s, "10")
end
```

