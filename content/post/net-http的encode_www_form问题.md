---
title: "Net Http的encode_www_form问题"
date: 2018-12-06T18:29:07+08:00
lastmod: 2018-12-06T18:29:07+08:00
draft: false
keywords: ['ruby', 'url', 'encode']
description: ""
tags: []
categories: ['Ruby']
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

最近业务又涉及到了支付宝小程序，所以需要更新[alipay_mini](https://github.com/renyijiu/alipay_mini)这个gem包，增加一个创建订单的接口，去实现支付宝支付的功能模块。因为包里使用的是ruby自带的`net/http`进行网络请求，当测试的时候发现阿里那边接口返回的一直是`签名验证不一致`，但是测试其他接口却是正常的，能够获取到相关的信息。于是开始了排查之路，毕竟刚写的接口，肯定是哪里出了问题，于是与阿里返回的验证字符串进行了比较，慢慢的发现了问题所在。

<!--more-->
### 问题描述

当一个单层的hash值使用`net/http`去构建成url的query字符串时，我们发现结果是正常的，如下面代码所示：

```ruby
> hash = {hello: 'world', language: 'ruby'}
  {
    :hello => "world",
    :language => "ruby"
  }

# 输出结果
> URI.encode_www_form(hash)
  "hello=world&language=ruby"
```

但是呢当一个多层的hash使用相同的方式进行操作时问题就出来了，代码如下：
```ruby
hash = {hello: {language: :ruby, hello: {name: :renyijiu}}}
  {
    :hello => {
      :language => :ruby,
      :hello => {
        :name => :renyijiu
      }
    }
  }

# 输出结果
> URI.encode_www_form(hash)
  "hello=%7B%3Alanguage%3D%3E%3Aruby%2C+%3Ahello%3D%3E%7B%3Aname%3D%3E%3Arenyijiu%7D%7D"

```

这样子可能无法直观的看出问题，我们将输出的字符串进行解码：

```ruby
"hello=%7B%3Alanguage%3D%3E%3Aruby%2C+%3Ahello%3D%3E%7B%3Aname%3D%3E%3Arenyijiu%7D%7D" 

# 对应的是：
"hello={:language=>:ruby, :hello=>{:name=>:renyijiu}}"
```
现在可以发现的是，**多层级的hash值其较深的层次在`urlencode`时并没有达到转换的效果，只有最外层的hash是成功处理的**。

### 问题解决

确定了问题的原因，那就开始解决问题了。当然如果我们使用`rails`时，其提供了几个对应的方法可以去达到效果，例如
```ruby
hash.to_query

hash.to_param
```

但是由于这次是在写一个gem包，我们不能依赖于`rails`的语法糖，只能靠自己去处理了，所以需要编写一个函数，去把多层的`hash`转换成单层的，然后再去使用`encode_www_form`就可以解决这个问题了。
```ruby

def to_query(value, key = nil, out_hash = {})
  case value
  when Hash
    value.each { |k, v| to_query(v, append_key(key, k), out_hash) }  
  when Array
    value.each { |v| to_query(v, "#{key}[]", out_hash) }    
  when nil
    ''
  else
    out_hash[key] = value
  end

  out_hash
end

def append_key(root_key, key)
  root_key.nil? ? :"#{key}" : :"#{root_key}[#{key.to_s}]"
end

```
测试下实际的效果，看看能否达到的效果：
```ruby
hash = {hello: {language: :ruby, hello: {name: :renyijiu}}}
  {
    :hello => {
      :language => :ruby,
      :hello => {
        :name => :renyijiu
      }
    }
  }

# 使用函数转换
> to_query(hash)
  {
    :"hello[language]" => :ruby,
    :"hello[hello][name]" => :renyijiu
  }
```

可以看出来，我们将多层嵌套的`hash`转换成了单层级，然后使用`encode_www_form`去转化成`url`格式：
```ruby
> hash = {
    :"hello[language]" => :ruby,
    :"hello[hello][name]" => :renyijiu
  }

> URI.encode_www_form(hash)
  "hello%5Blanguage%5D=ruby&hello%5Bhello%5D%5Bname%5D=renyijiu"
```

现在这样子问题就顺利解决了，参数验证也通过了。

### 结语
这个问题自己也是第一次遇到，以前都是使用`rails`或者是其他的方案，可能在深层次已经帮你解决了这个问题，并对你隐藏了这个问题的存在，这次的实践让自己发现了这个问题，所以记录下提醒自己哈。