---
title: "dup和clone的区别"
date: 2019-01-08T16:21:39+08:00
lastmod: 2019-01-08T16:21:39+08:00
draft: false
keywords: ['ruby', 'dup', 'clone']
description: "ruby中dup和clone的区别"
tags: ['Rails']
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

最近在项目中出现了一个比较奇怪的问题，部分记录出现了重复操作，开始自己以为是数据操作的代码上出现了问题，但是经过定位发现是一个自己之前没有注意到的语法使用导致的问题。

<!--more-->

项目代码基于`Ruby 2.4.4`以及`Rails 5.1.6`，其他版本无法保证完全一致。

在项目中有如下代码：

```ruby
# snack_foods为ActiveRecord的查询结果
tmp_snack_foods = snack_foods.to_a.dup

snack_foods.each do |snack_food|
  if MEAL_TIME_TYPE.include?(snack_food.time_type_value)
    tmp_snack_foods.delete(snack_food) if delete_snack_food(snack_food)
  end
end

check_snack_calory_rule(tmp_snack_foods)
```

整体代码逻辑就是拷贝`snack_foods`至一个临时变量`tmp_snack_foods`，然后遍历`snack_foods`，符合条件时从`tmp_snack_foods`中删除对应的项。但是在实际运行时发现数据没有正确的删除，导致后续又进行了一遍处理。

首先来看下`Ruby`语法中在`Object`上`dup`和`clone`的区别：

* `clone`会拷贝单例方法，而`dup`不会

```ruby
o = Object.new
def o.hello
  'hello'
end

o.dup.hello    # raises NoMethodError
o.clone.hello  # return 'hello'
```

* `clone`会保留`frozen`的状态，而`dup`不会

```ruby
class Hello
  attr_accessor :hello
end

o = Hello.new
o.freeze

o.dup.hello = 'world' # success
o.clone.hello = 'world' # untimeError: can't modify frozen Hello
```

上面是两者的区别，而当处理`ActiveRecord`数据时，两者最大的区别是：

* `dup`创建了一个**无ID**的新对象，我们可以使用`save`去保存数据

```ruby
irb(main)> food = Wisdom::Food.last
#<Wisdom::Food:0x00007ff5c7036ff0> {
               :id => 395,
             :name => "胡萝卜黑木耳包子",
             :code => "huluoboheimuerbaozi",
        :time_type => "lunch"
}

irb(main)> tmp_food = food.dup
#<Wisdom::Food:0x00007ff5c59f40a8> {
               :id => nil,
             :name => "胡萝卜黑木耳包子",
             :code => "huluoboheimuerbaozi",
        :time_type => "lunch"
}

irb(main)> tmp_food.save
   (0.4ms)  BEGIN
  SQL (3.8ms)  INSERT INTO `wisdom_foods` ...
   (1.4ms)  COMMIT
true
```

* 而`clone`创建了一个拥有**相同的ID**的对象，因此所有对新对象的修改都会覆盖原来的对象数据。

```ruby
irb(main):001:0> food = Wisdom::Food.last
#<Wisdom::Food:0x00007ff5c5939528> {
               :id => 396,
             :name => "胡萝卜黑木耳包子",
             :code => "huluoboheimuerbaozi",
        :time_type => "lunch"
}

irb(main):002:0> tmp_food = food.clone
#<Wisdom::Food:0x00007ff5c591e868> {
               :id => 396,
             :name => "胡萝卜黑木耳包子",
             :code => "huluoboheimuerbaozi",
        :time_type => "lunch"
}

irb(main):003:0> tmp_food.update(name: '胡萝卜木耳包子')
   (0.3ms)  BEGIN
  SQL (2.8ms)  UPDATE `wisdom_foods` SET `name` = '胡萝卜木耳包子' WHERE `wisdom_foods`.`id` = 396
   (1.5ms)  COMMIT

irb(main):004:0> food
#<Wisdom::Food:0x00007ff5c5939528> {
               :id => 396,
             :name => "胡萝卜木耳包子",
             :code => "huluoboheimuerbaozi",
        :time_type => "lunch"
}
```

明白了上述两个方法的区别后就会发现问题所在:

```ruby
# 这行代码导致的是tmp_snack_foods里面的值都是不包含ID的新对象
tmp_snack_foods = snack_foods.to_a.dup

# 因此导致这里的删除无法生效，数组保留传到了后续的操作上导致报错
tmp_snack_foods.delete(snack_food)
```

针对上述问题，修改成`clone`应该就可以达到效果了。这一个问题自己以前确实不太了解，这一次正好是碰到了知识盲区，学到了两者的差别，下次使用就会注意这个问题了。

