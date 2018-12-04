---
title: "Hash使用小技巧"
date: 2018-08-01T16:40:58+08:00
lastmod: 2018-08-01T16:40:58+08:00
draft: false
keywords: ["Ruby", "Hash"]
description: "Ruby中的Hash使用技巧"
tags: ["Ruby", "Hash"]
categories: ["ruby"]
author: "renyijiu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
autoCollapseToc: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: true
mathjax: false
---

Ruby中的Hash是经常使用的，每天写代码几乎都要和它打交道。自己在使用中以及日常查资料中看到一些Hash的使用技巧，这里记录一下，希望对大家有用。

<!--more-->

### 1. 初始化

常见的方式就不在赘述了，一次查资料时看到一个比较少用的方式：

```ruby
Hash[:a, :b]
=> {:a=>:b}

Hash[[:a, :b], [:c, :d]]
=> {[:a, :b]=>[:c, :d]}
```

### 2. 设置默认值

我们可以在创建Hash的时候通过设置代码块来实现自定义默认值

```ruby
irb> hash = Hash.new { |hash, key| hash[key] = key }
=> {}
irb> hash[:a]
=> :a

# 或者使用default_proc设置
a.default_proc = Proc.new do |hash, key|
  hash[key] = key
end
=> #<Proc:0x00007feaf00ba738@(irb):19>
irb> a
=> {}
irb> a[:a]
=> :a
```

这里的`default_proc`很有意思，我们可以深入的利用一下去生成一个树形结构的hash

```ruby
irb> tree = Hash.new{|hash, key| hash[key] = Hash.new(&hash.default_proc)}
=> {}
irb> tree[:a][:b][:c]
{}
irb> tree[:a][:b][:d] = ['hello', 'world']
[
    [0] "hello",
    [1] "world"
]
irb> tree[:a][:e] = 'hello world'
"hello world"
irb> tree
{
    :a => {
        :b => {
            :c => {},
            :d => [
                [0] "hello",
                [1] "world"
            ]
        },
        :e => "hello world"
    }
}
```

### 3. 替换

String中的`%`大家比较熟悉，`str % arg → new_str`，但其实我们也可以结合hash去做一些有意思的操作。

```ruby
irb> list = {language: 'Ruby'}
{
    :language => "Ruby"
}
irb> string = "The best language is %{language}"
"The best language is %{language}"
irb> string % list
"The best language is Ruby"	
```

正则替换`gsub`也是同理哈

```ruby
irb> string = "The best language is PHP"
"The best language is PHP"
irb> language = {"PHP" => "Ruby"}
{
    "PHP" => "Ruby"
}
# 使用default_proc确保只替换hash存在的单词
irb> language.default_proc = Proc.new{|hash, key| key}
#<Proc:0x00007fe5729d73f8@(irb):3>
irb> string.gsub(/\w+/, language)
"The best language is Ruby"
```

### 4. 合并

合并两个hash，当存在对应新的value时，覆盖原来的值；若无，则保留原来的值

```ruby
def custom_merge(old_hash, new_hash)
  old_hash.merge(new_hash){|_, old_val, new_val| new_val.nil? ? old_val : new_val}
end

# 示例
irb> a = {a: 'hello', b: 'world'}
{
    :a => "hello",
    :b => "world"
}
irb> b = {a: 'hello world', c: 'Ruby'}
{
    :a => "hello world",
    :c => "Ruby"
}
irb> custom_merge(a, b)
{
    :a => "hello world",
    :b => "world",
    :c => "Ruby"
}
```

### 5. 格式化

将hash中的key转成symbol格式，采用递归确保所有的key转换

```ruby
def symbolize(obj)
  if obj.is_a? ::Hash
    return obj.inject({}) do |memo, (k, v)|
      memo.tap { |m| m[k.to_sym] = symbolize(v) }
    end
  elsif obj.is_a? ::Array
    return obj.map { |memo| symbolize(memo) }
  end

  obj
end

#示例
irb> a = {'1' => 'a', '2' => 'b', '3' => 'c', d: {'hello' => "world", 'test' => [1, 2, 3]}}
{
    "1" => "a",
    "2" => "b",
    "3" => "c",
     :d => {
        "hello" => "world",
         "test" => [
            [0] 1,
            [1] 2,
            [2] 3
        ]
    }
}
irb(main):030:0> symbolize(a)
{
    :"1" => "a",
    :"2" => "b",
    :"3" => "c",
      :d => {
        :hello => "world",
         :test => [
            [0] 1,
            [1] 2,
            [2] 3
        ]
    }
}
```