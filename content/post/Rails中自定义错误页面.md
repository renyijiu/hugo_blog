---
title: "Rails中自定义错误页面"
date: 2016-10-30 14:24:10
lastmod: 2018-01-14T20:57:32+08:00
draft: false
keywords: []
description: ""
tags: []
categories: []
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


Rails中当出现500和404错误时，默认使用的是Public下的错误页面，如500.html。在开发过程中或者是一些内部应用中，我们希望可以将错误信息暴露出来，以方便我们快速定位及解决问题。那么我们可以对错误页面的信息进行自定义。<!--more-->

## 修改配置

首先需要在config中开启这个设定，使用自定义的错误方式。

```ruby
# config/application.rb   
config.exceptions_app = self.routes
```

设定后应该就是使用自己的错误定义了，也可以将public下的错误页面删了，不删也没什么关系。

## 定义错误动作

使用自己的错误定义后，那就和其他的操作一样，首先定义路由跳转

```ruby
#通过数组定义错误代码，自己定制 
%w(404 500).each do |code|
     match code, to: "error#show", code: code, :via => [:get, :post, :put, :patch, :delete]
 end

#正则表达式匹配路由，这个是项目使用的一种，将服务器错误（5XX）进行跳转
match '/:code', to: "error#show", code: :code, via: :all, :constraints => { :code => /5(10|0\d){1}/ }
```

**这里需要说一下**，当请求出错，使用路由跳转时，原来的请求方式和请求参数会被携带着转向错误处理。例如，当使用get时，参数为params={"test":"测试"}，这个请求服务器出错，路由跳转错误处理时，使用的依旧是get方式，并带有原来的参数，所以需要确定自己需要对那些请求方式进行处理，简单的使用via: :all对所有的http请求进行匹配。

在写好route后，需要相对应的cotroller文件

```ruby
class ErrorController < ApplicationController
  def show
    #自定义错误处理代码
    ......
  end
end
```

如果需要将错误的log信息显示，可以通过环境变量去进行提取

```ruby
#获取错误log信息，进行自定义显示
def env_exception
  @ex ||= env['action_dispatch.exception']
end

@err_mess = env_exception ? env_exception.message : ''
@err_info = env_exception ? env_exception.backtrace.join("\n") : ''

```

## Rails内自带的500错误提示

```ruby
"500 Internal Server Error\n" 
"If you are the administrator of this website, then please read this web application's log file and/or the web server's log file to find out what went wrong."
```

这个是Rails自带的错误提示，当你自定义的错误页面代码出错时，无法展示了，就会转向这个告诉你出问题了，这是你就需要检查自己的错误信息页面哪里出问题了。

## 其它的思路

自定义错误信息也不一定去展示，在```error#show```中可以进行一些有趣的操作，常见的是加入邮件发送，通知管理员出问题了。另外也可以去调用一些消息通知，例如，添加一个钉钉的消息通知接口，出现错误后调用接口，将错误日志发给管理员，更加方便快捷。

总的来说，这个自定义错误还是比较实用的，可以提高错误的通知效率，使得错误快速定位并解决，而不是说用户无法使用抱怨了，这才发现线上环境出现了错误。

