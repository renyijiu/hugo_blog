---
title: "Rails测试中@app变量问题"
date: 2018-12-09T17:42:28+08:00
lastmod: 2018-12-09T17:42:28+08:00
draft: false
keywords: ['rails', '@app', '测试']
description: ""
tags: []
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

前段时间在写项目的测试时，碰到了一个比较奇怪的问题，一个比较普通的测试代码一直无法通过，当时也是花费了较多的时间在进行排查。现在来记录一下这个问题，防止下次再次踩坑。

<!--more-->

下面示例代码，基于`ruby 2.5.3p105`和`Rails 5.2.2`，其他版本无法确保问题可以直接复现。

在我们的业务中存在着一张`apps`的表，然后存在一个简单的`controller`对其进行一些数据操作，代码如下：
```ruby
# model app.rb
# 对应的controller

class AppsController < ApplicationController
  before_action :set_app
  
  def update
    @app.update(name: 'hello')
    
    redirect_to apps_path
  end

  private
  
  def set_app
    @app = App.find(params[:id])  
  end
end
```

我们对这个`controller`编写对应的测试，代码如下：
```ruby
require 'test_helper'

class AppsControllerTest < ActionDispatch::IntegrationTest
  def setup
    @app = App.create(name: 'renyijiu')
  end

  def test_should_update_the_name
    put app_path(id: @app.id)

    assert_redirected_to apps_path
  end
end

```

代码比较简单，看起来应该是没有什么问题的，但是呢运行测试发现却会提示错误，
`Minitest::UnexpectedError: NoMethodError: undefined method 'app_path'`，看到这个错误自己也是一头的雾水，查看路由可以发现这个路径是存在的，那么问题出在哪里呢？

主要就是下面这行代码，
```ruby
  def setup
    @app = App.create(name: 'renyijiu')
  end
```

让我们换个变量名字试试，
```ruby
require 'test_helper'

class AppsControllerTest < ActionDispatch::IntegrationTest
  def setup
    @my_app = App.create(name: 'renyijiu')
  end

  def test_should_update_the_name
    put app_path(id: @my_app.id)

    assert_redirected_to apps_path
  end
end
```
这时发现我们的测试竟然顺利的通过了，问题也就顺利的定位到了，这个`@app`的变量命名导致了问题。

查看[github issue](https://github.com/rails/rails/issues/26835)可以发现，这个问题其实已经有人遇到过了，但是呢一直没有解决。主要就是我们继承的`ActionDispatch`内部使用了`@app`这个变量，我们在测试中使用这个变量时，直接产生了覆盖的问题，导致测试无法[include url_helpers](https://github.com/rails/rails/blob/bad1041b82df941d588ae2565f62424d88104933/actionpack/lib/action_dispatch/testing/integration.rb#L336)，所以产生了上述的错误。当然在issues中对具体的代码也有所讨论，也有人想尝试修复这个问题，但是呢因为有着大量的测试代码依赖于此，需要因此进行更新，这个工作相对来说回报不大，另外也因为这个问题是一个低优先级的问题。所以这个问题在写这篇文章时问题依旧还存在，自己也会对此持续关注，尝试源码解读，万一有机会提PR呢，哈哈哈！

所以基于前面所说的一切，目前解决这个问题就最简单的方法：**避开@app的命名，使用一个新的变量名替代，例如@my_app之类的**。

