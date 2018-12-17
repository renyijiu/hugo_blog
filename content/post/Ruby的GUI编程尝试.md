---
title: "Ruby的GUI编程尝试"
date: 2018-12-16T19:54:22+08:00
lastmod: 2018-12-16T19:54:22+08:00
draft: false
keywords: ['Ruby', '图形界面', 'GUI']
description: ""
tags: ['Ruby', 'GUI']
categories: ['Ruby']
author: 'renyijiu'

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

最近天气也是冷的可以，周末正好也没有什么事情，就把以前留的一个坑给补上了，完成了一个前段时间很火的抖音表白软件。然后在写的过程中，也算是对`Ruby`图形界面编程的一个尝试，这里记录一下自己的使用感受。

<!--more-->

### 需求描述

![](https://wx3.sinaimg.cn/large/005NU63ely1fy8tp6sb4tg30iy0a41l0.gif)

从我们的最终效果来看，这个产品的整体逻辑比较简单，主要的功能点就是：

* 当鼠标移动准备去点 '算了吧' 这个按钮时，我们就随机移动按钮，防止用户点击。



### 方案介绍

知道了要做的东西，然后根据这个来介绍不同的GUI编程方案。首先我自己主要是看了下[An overview of Desktop Ruby GUI development in 2018](https://saveriomiroddi.github.io/An-overview-of-ruby-gui-development-in-2018/)、[How are you building Ruby GUI applications in 2018?](https://www.reddit.com/r/ruby/comments/7s3qio/how_are_you_building_ruby_gui_applications_in_2018/)以及[[GUI Frameworks](https://www.ruby-toolbox.com/categories/GUI_Frameworks)](https://www.ruby-toolbox.com/categories/GUI_Frameworks)这几篇资料，其中列举了一些方案介绍，但是其中几个目前已经死亡，有几个项目也已经不太活跃，而部分活跃项目的文档对于新手并不友好，所以整体而言，**我们的可选择性并不大**。我自己尝试了[ruby2d](https://github.com/ruby2d/ruby2d)和[shose4](https://github.com/shoes/shoes4)，下面就大致说下我的看法。

#### ruby2d

这个项目应该算是一个比较新的项目，所以目前还是处于早期的开发阶段。从官方文档的介绍上来看呢，入门上手使用是极其简单的，自己也尝试去编写了demo，发现也确实不错。但是呢按照我们的需求，

1. 这个gem包目前没有`button`这类的基础元素，所以我们只能间接去实现一个类似于按钮的东西，但是考虑到移动之类的效果，整体的复杂度是很高的。
2. 由于没有`button`元素，对应用户的点击事件判断处理也是一个比较复杂的问题
3. 对于鼠标的移动位置，按钮的`hover`事件判断也是一个缺失的功能

出于这主要的三点，我只能放弃这个方案。针对`button`元素，我也已经去[github](https://github.com/ruby2d/ruby2d/issues/132)咨询开发者，已经得到明确的回复，开发者也认为这是必须的元素，后续会考虑增加，自己也会持续关注这个项目。

总的来说，这个项目目前能提供比较简单的2d图形展示，针对用户鼠标点击等交互事件以及基础元素支持不够理想，所以如果只是单纯的需要展示图形信息时，可以考虑此方案。

#### shoes4

引用下官方介绍，`Shoes 4 : the next version of Shoes`。在此之前存在着`shoes`这个项目，这个也是GUI编程的一个不错的方案。`shoes4`则是基于`java`对此进行了重新开发，而`shoes`则是基于`C`的，整体提供的功能是类似的，这点从官方文档也可以看出，两者使用的其实是一份文档🤦‍♂️。

相对于上面的那个gem包，这个就提供了相当全的功能。整体我觉得可以满足我的需求，所以最后这个实现也就是基于这个gem包，具体提供的功能元素可以去看下官方文档自行判断。这里我就说一下在实现过程中自己遇到的几点问题，以供参考：

1. `shoes4`是基于`java`实现的，所以我们需要使用`jruby`
2. 在获取按钮位置时，`absolute_top`以及`absolute_left`值存在问题，并不准确
3. `button`没有`hover`事件，因此我们无法判断鼠标是否移动至按钮上
4. 当鼠标移动至按钮之上时，鼠标的位置会丢失，导致我们无法实时获取坐标
5. 打包文件时，添加的音乐无法正常播放（已提[issue](https://github.com/shoes/shoes4/issues/1573)，不知道是否是bug）

因为3、4两点，导致无法准确的判断鼠标是否移动至按钮上，这里只能采用取巧的方法解决（判断按钮边缘），所以导致整个方案实现效果就没有那么理想。另外打包后测试发现背景音乐无法正常播放，这一点也使这个工具又丧失了一点趣味性。

### 总结

上面就是自己在编写这个玩具时使用两个框架的感受。相对来说，两个方案对于这个需求可能都无法很好的满足，但是个人觉得两个都不失为`ruby`图形编程的优秀方案。因为这次只是为了满足自己的需求，所以未对其他的方案做一些尝试，可能其他的方案更加的强大和易用。

