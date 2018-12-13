---
title: "Rails中scope问题踩坑"
date: 2018-12-13T11:56:48+08:00
lastmod: 2018-12-13T11:56:48+08:00
draft: false
keywords: ['Rails', 'scope']
description: ""
tags: ['Rails']
categories: ['日常踩坑']
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

最近在错误系统中突然发现了一个`undefined method 'first_tags' for #<Wisdom::Tag::ActiveRecord_Relation:...>`的错误，其中`first_tags`是`wisdom_tags`这张表中的一个字段，开始以为是其中使用的`scope`的语法问题，但是仔细探究下来发现并不是，是自己以前没有注意的问题，所以记录一下。

<!--more-->

首先在`model`层存在一个`scope`，代码如下：
```ruby
class Wisdom::Tag < ApplicationRecord
  belongs_to :evaluation, :class_name => 'Wisdom::Evaluation', foreign_key: 'evaluation_id'
    
  scope :evaluation_tag, ->(evaluation) { where(week_index: 0, evaluation: evaluation).last }
end
```

在正常的业务情况下，每一条`wisdom_evaluation`都有对应的`wisdom_tags`，因此这个`scope`在业务中也是没有问题的。但是由于特殊情况，存在一条评测数据其对应的标签数据没有成功创建，因此这个`scope`在使用时没有找到对应的数据，**而返回了一个包含全部wisdom_tag数据的`ActiveRecord::Relation`**，这也就导致了后续方法在调用时产生报错。

> Adds a class method for retrieving and querying objects. The method is intended to return an ActiveRecord::Relation object, which is composable with other scopes. If it returns `nil` or `false`, an all scope is returned instead.

官方对应的文档上其实也有说明，另外这里[stackoverflow](https://stackoverflow.com/questions/20942672/rails-scope-returns-all-instead-of-nil)有个不错的回答。 **当`scope`返回`nil`或者`false`时，所有的记录会被返回，这个设计是有意为之，是为了`scopes`的链式调用，这一点也是和`class_method`的关键区别**，具体的例子可以去提供的回答中看一下，写的挺详细的，这一点也是自己以前没有注意到的。

所以针对这种情况，当你需要`first`、`last`或者`find`这类的返回具体的数据的时候，优先考虑使用`class_method`来替代使用`scope`，避免出现类似的bug哈。

