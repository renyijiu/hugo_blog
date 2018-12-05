---
title: "Redis中的hset返回值"
date: 2018-12-05T09:21:07+08:00
draft: false
keywords: ['rails', 'redis', 'hset']
description: ""
tags: ['日常踩坑']
categories: ['Rails']
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

### 问题起源

有一个简单的需求，当用户购买相关的服务后，用户对应的预算热量会采用新的算法进行，服务结束后恢复原来的热量。由于预算热量在各个业务系统中是通用的，内部对用户基础信息做了一层封装，提供了一个gem包去进行写入和读取的操作。

根据上述描述，自己的解决思路如下：

1. 将用户当前的预算热量暂存到其他地方
2. 更新用户当前的预算热量

两次操作在gem包中实际上都是使用了redis的`hset`操作，伪代码如下：

```ruby
if save_the_old_budget_calorie
  update_the_new_budget_calorie
end
```

### 问题所在

当在测试环境中测试时，发现用户的预算热量并没有按照预期设置成功。表现为新用户第一次是可以的，后续就会失败。于是自己就在`console`中测试了一下，发现了问题所在，`save_the_old_budget_calorie`方法第一次返回了`true`值，后续返回了`false`。因为内部是使用了redis的`hset`操作，就去查看了相关的文档，发现了问题所在。

> - `1` if `field` is a new field in the hash and `value` was set.
> - `0` if `field` already exists in the hash and the value was updated.

也就是当对应的`key`已经存在时，就算更新成功了也是返回0值；而对应使用了`redis`这个gem包将值转换成了`boolean`值，也就是返回了`false`。哈，找到了问题，最后修改一下逻辑就解决问题了。





