---
title: "Rails Controller中的concerns测试编写"
date: 2018-08-24T09:58:12+08:00
lastmod: 2018-08-24T09:58:12+08:00
draft: false
keywords: ['测试', 'Rails']
description: "Rails Controller下concerns测试"
tags: ['Rails', 'Test']
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

最近在完善项目中的测试代码，常见的`Controller`以及`Model`层的测试代码写的比较熟悉了。在查看测试覆盖率报告时发现`Controller`中`concerns`下文件基本没有测试，然后自己对如何测试`concerns`也不是很了解，就搜索了资料记录下来，方便后续回顾。

<!--more-->

`concerns`目录下一般来说是一些独立的逻辑模块或者是重复使用的功能模块，这样可以提升代码的可读性以及维护性。下面是一个例子：

```ruby
# /app/controllers/concerns/error_trackable.rb

module ErrorTrackable
  extend ActiveSupport::Concern
    
  def error
    p "Hello, Ruby!"
    
    true
  end
end
```

我们在使用的时候可以在`controller`中引入即可

```ruby
class AdminController < ApplicationController
  include ErrorTrackable    

end		
```

然后通过编写`AdminController`的测试代码也可以对`concerns`进行测试，但是现在这样就会导致代码**过于耦合**，一旦`AdminController`中不再引入相关文件，就会导致测试代码无法通过。另外一点我们可能需要在所有引入相关文件的地方都对此编写测试代码，保证逻辑正确性，但是这不符合DRY的原则啊。

所以我们通过构建一个假的`controller`，引入相关的文件功能，然后对这个构建的''假的"`controller`编写测试代码进行相关的测试，测试代码如下：

```ruby
require 'test_helper'

class ErrorTrackableTestController < ApplicationController
  include ErrorTrackable
end

class ErrorTrackableTestControllerTest < ActionDispatch::IntegrationTest

  test "should get error" do
    res = ErrorTrackableTestController.new.error

    assert_equal true, res
  end
  
end
```

这样子我们就可以对`concerns`中的代码进行相关的测试了，代码也不会依赖于我们原来的逻辑，另外保证了代码的测试覆盖率，终于可以愉快的玩耍了。