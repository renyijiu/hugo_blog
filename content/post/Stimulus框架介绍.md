---
title: "Stimulus框架简介"
date: 2018-03-24T14:18:58+08:00
lastmod: 2018-03-24T14:18:58+08:00
draft: false
keywords: ["Stimulus"]
description: "stimulus 前端框架"
tags: []
categories: ["前端"]
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

Stimulus是2018年年初Basecamp开源的一个新的前端框架，是一个轻量级的框架（所做的事情很简洁）。

从官方介绍可以看出，它的出现是Basecamp目前对于Rails服务端渲染中如何优雅的组织Javascript代码的一种实践，这也表明了它与目前的前端三大框架（Vue、React、Angular）完全不是一个路子，毕竟诞生原因和目标都不一样。这一点在[origin](https://stimulusjs.org/handbook/origin)详细地说明了，有兴趣的可以看一看。

<!--more-->

它的用法其实使用官方的一张图就可以很直白的了解：

![stimulus.png](https://i.loli.net/2018/03/24/5ab5f63f87527.png)

框架通过`data-contoller` `data-target` `data-action`等属性将HTML元素与JavaScript行为绑定。

- `data-controller` 将HTML与对应的JavaScript代码进行关联
- `data-action` 进行相应的事件关联处理
- `data-target` 提供了三个自定义方法实现对于指定HTML元素的关联引用

上图可以看出整个框架所做的事情非常的简洁（为已经渲染的HTML添加行为），整个框架的使用也非常的简单，具体使用就不说了，可以看看官方的[handbook](https://stimulusjs.org/handbook/introduction)，毕竟只有6页。

上面就是框架的大概介绍了。然后最近在NICE项目中我们引入了Stimulus框架，用于对管理后台部分的JavaScript代码进行重构，大致说一下框架的使用心得。

在我们的项目中，对于用户的产品大多是前后分离，前端使用主流框架开发，这个不需要我们担心。而在对应的管理后台中，采用的是Rails服务端渲染方式，另外目前还是Rails 4的版本，没有Rails 5中的webpacker之类的前端工具链，也不会主动去引入这一套使用在后台的开发中。其次js代码主要还是JQuery一把梭，并且管理后台的交互性等都相对简单，这是项目背景。因此，是可以采用这个框架的，毕竟没有复杂的前端交互等需求。

从整体的重构来说，JavaScript的代码量整体来说并没有减少，可能比原来的还多了（因为框架本身处理的相对简单明确，提供了事件绑定，剩下的你所需要的交互效果、数据请求等等依旧需要自己去处理实现，所以并不会减少太多的代码）。但是呢个人觉得有了这个框架，会让代码相对的规范化，去遵守框架提供的一种实践方式。而不会像以前常见的代码，多人修改后，页面元素充满了难以理解的class、id等。HTML应该是具有可读性的，阅读HTML也可以了解到js代码想要做什么。

另外，作为亲儿子出身，它与rails推的Turbolinks是完美结合的。这一点可能不需要像当时一样，开启Turbolinks之后引发了原有js代码一堆的问题，导致很多人第一时间放弃了这个功能。而且，代码是可以复用的，例如可以将tab的切换写成一个tab_controller.js文件，后续就可以直接重复使用了，慢慢的积累成"组件库"。

当然引入新框架也是会产生问题的：

- 与部分插件会产生冲突，例如bootstrap-datetimepicker插件与其均使用`data-action`属性，bootstrap的tabs切换使用了`data-target`属性等等，导致插件的失效或者报错之类。
- 框架本身也存在一些使用上的不便，一些例如controller之间如何通信调用之类的问题还没有好的方法等。

由于后台系统无复杂js交互逻辑，就目前所遇到的问题来看，大部分其实都已经有人遇到过，也提供了解决办法。（例如，插件使用了相同的属性名，你可以在框架初始化时自定义名称，如改成`data-saction`之类的，在使用一开始就避开这个问题，当然也有其他的方式，详细后续再介绍）。

因此总的来说，Stimulus提供了一种目前Rails中组织编写JavaScript代码的实践规范，弥补了一直以来的缺失。在前端需求没有那么强烈的后台管理系统之中，可以考虑引入框架去规范代码实现，毕竟这一整套方案（Stimulus + Turbolinks + SJR）是不错的选择。



