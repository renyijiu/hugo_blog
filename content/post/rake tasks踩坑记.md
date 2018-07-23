---
title: "Rake tasks踩坑记"
date: 2018-07-21T17:47:53+08:00
lastmod: 2018-07-21T17:47:53+08:00
draft: false
keywords: ['Rake', 'Rails']
description: ""
tags: ['Rake', 'Rails']
categories: ['Rails']
author: "renyijiu"
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

最近在工作中编写Rake  tasks时碰到了一个以前没有注意的问题，不同命名空间下定义的同名methods出现了覆盖的情况导致bug的产生，自己也是排查很久后才发现重名的问题，所以就把这个记录下来，避免以后再次踩坑。<!--more-->

### 示例一

```ruby
# lib/tasks/hello_one.rake

desc "puts hello world one"
task :hello_one do
  hello_world
end

def hello_world
  p "hello tasks one!"
end
```

运行上面的task，结果也是和我们期望的一样

    $ rake 'hello_one'
    
    "hello world one"    

然后写下另外一个task进行测试

```ruby
# lib/tasks/hello_two.rake

desc "puts hello world two"
task :hello_two do
  hello_world
end

def hello_world
  p "hello world two"
end
```

运行也得到了期望的结果

    $ rake 'hello_two'
    
    "hello world two"

但是现在重新运行`rake 'hello_one'`时却发现输出也变成了`hello world two`，问题就出来了。当你运行rake任务时，过程是这样的：

1. `rake 'hello_one'`
2. Rails按照文件名称顺序自动加载所有的rake文件
3. `lib/tasks/hello_one.rake`文件被加载，定义`hello_world`方法
4. `lib/tasks/hello_two.rake`文件被加载，`hello_world`方法被重新定义
5. 运行task代码，最后定义的`hello world`方法被调用

上述示例可以简单的说明问题，但是一般我们编写时会带有`namespace`，那现在看看是否会出现上面的情况呢？

### 示例二

```ruby
# lib/tasks/hello_one.rake
namespace :one do
  desc "puts hello world one"
  task :hello_one do
    hello_world
  end
  
  def hello_world
    p "hello tasks one!"
  end
end

# lib/tasks/hello_two.rake
namespace :two do
  desc "puts hello world two"
  task :hello_two do
    hello_world
  end

  def hello_world
    p "hello world two"
  end
end
```

运行上述代码，结果如下：

```
$ rake 'one:hello_one'
"hello world two"

$ rake 'two:hello_two'
"hello world two"
```

从运行结果可以看出来，`namespace`对于方法定义没有影响，方法依旧是全局性质的，问题依旧存在。

对于这个一般有几个常用的解决办法：

1. 确保每一个定义的方法名称唯一（最直接也最简单，但是后续容易重名踩坑）
2. 把方法的逻辑写回到task中去，不单独定义相关的method（当然这是相反的操作，如果方法简单那是可以这么操作的）
3. 将方法逻辑抽离，单独封装成Class或者是Module，然后在task中调用即可（当然若与model相关，可以写成对应的实例方法或者类方法，这都是可以的）

第三个相对是一个较好的解决方案，但是呢具体的使用还是需要根据当时的业务场景及代码复杂度等因素去选择判断，解决问题的方案才是好方案。


