---
title: "Rails过滤器及回调顺序踩坑"
date: 2018-05-19T13:38:36+08:00
lastmod: 2018-05-19T13:38:36+08:00
draft: false
keywords: ['Rails', '回调', '过滤器']
description: ""
tags: ["Rails"]
categories: ['Rails']
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

最近在做项目时碰到了一个问题，自己在model中定义了几个`after_commit`回调（有一个依赖于上一个回调的数据结果），但是运行结果并没有达到预期的效果。开始以为是没有触发导致的问题，但实际后来查看日志发现已经触发了，只是呢顺序没有自己预期的那样，看来自己对于这个回调顺序没有完全了解，所以写一篇文章记录下这个问题。

<!--more-->

本文测试代码基于Rails 5.2.0， Ruby 2.5.1版本，其他版本结果不确保一定一致。

### 过滤器

过滤器（filter）是一种方法，在控制器动作运行之前、之后，或者前后运行。每类过滤器有着自己常用的使用场景，这里就不具体介绍了，下面看看过滤器的执行顺序。

```ruby
class HomeController < ApplicationController
  before_action :before1
  around_action :around1
  after_action :after1

  around_action :around2
  before_action :before2
  after_action :after2
  
  before_action :before3
  after_action :after3
  around_action :around3

  def index
    render plain: "Hello World"
  end

  def before1
    puts "before1"
  end

  def before2
    puts "before2"
  end
  
  def before3
    puts "before3"
  end

  def after1
    puts "after1"
  end

  def after2
    puts "after2"
  end
  
  def after3
    puts "after3"
  end

  def around1
    puts "around1 (before)"
    yield
    puts "around1 (after)"
  end

  def around2
    puts "around2 (before)"
    yield
    puts "around2 (after)"
  end
  
  def around3
    puts "around3 (before)"
    yield
    puts "around3 (after)"
  end
end
```

运行一下测试代码输出日志如下：

```
before1
around1 (before)
around2 (before)
before2
before3
around3 (before)
around3 (after)
after3
after2
around2 (after)
after1
around1 (after)
```

从输出日志可以明确地看出 **过滤器是按照定义的顺序执行的**，对应日志理解这个逻辑应该不难

### 回调

Rails提供了十几种回调操作，这里具体就不介绍每个回调的作用了，在官方文档中都有着具体的说明，这里说一下自己对于回调执行顺序的了解。

测试代码如下：

```ruby
# home_controller.rb

class HomeController < ApplicationController
  def index
    User.create(user_key: 'hello_world', name: 'test')

    render plain: "Hello World"
  end
end

# user.rb

class User < ApplicationRecord
  after_commit :commit_log_1
  after_commit :commit_log_2

  after_save :save_log_1
  after_save :save_log_2

  private

  def save_log_1
    p "save log 1"
  end

  def save_log_2
    p "save log 2"
  end

  def commit_log_1
    p "commit log 1"
  end

  def commit_log_2
    p "commit log 2"
  end
end
```

代码运行输出日志如下：

```
"save log 1"
"save log 2"
"commit log 2"
"commit log 1"
```

这里就可以明显的看出来，`after_save`回调是按照**定义顺序**执行的，而`after_commit`回调是按照**定义顺序逆序**执行的，其实官方文档中有所解释，只是自己一直没有注意到这方面，所以踩坑了。

> 非事务性回调按照定义顺序执行，事务性回调按照定义顺序**逆序**执行(后定义的先执行)

而按照文档中给的事务性回调目前只有两个：`after_commit`以及`after_rollback`，所以这里问题原因就已经明白，`after_commit`按照定义顺序相反的执行，修改一下顺序即可达到预期效果。