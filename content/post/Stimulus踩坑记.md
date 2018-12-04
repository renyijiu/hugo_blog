---
title: "Stimulus踩坑记"
date: 2018-03-31T11:14:00+08:00
lastmod: 2018-03-31T11:14:00+08:00
draft: false
keywords: []
description: "Stimulus踩坑"
tags: ["stimulus"]
categories: ["前端"]
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

上一篇在介绍Stimulus时，有说到部分插件会使用`data-target`类的属性值导致与Stimulus产生冲突，导致功能失效或者是报错，下面就介绍下两种自己的处理方法。

<!--more-->

例如，在使用Bootstrap的tabs时，tabs可以通过自定义`data-target`属性去关联对应的panel，这就会与框架产生冲突导致tab切换功能的失效（前提是你在tab元素上设置了`data-target`属性）。自己想到的有两种方式（当然你要是开口就是修改bootstrap框架源码之类的，那就没必要往下看了，手动斜眼）去解决问题。

### 一、修改属性名称

在框架初始化时手动传入Schema作为第二个参数去自定义名称。当然不推荐这么做，因为你自己了解缘由，但是当其他人接手代码后很容易入坑，尤其是没有相关文档说明的情况下。

```javascript
const application = new Application(document.documentElement, {
  controllerAttribute: "data-controller",
  actionAttribute: "data-action"
  targetAttribute: "data-target"
})
// ...
application.start()
```

不过如果像你使用`datetimepicker`这类的插件，它们在实现上使用了`data-action`属性，而你自己又不能去修改插件代码，所以在一开始引入Stimulus框架时就去修改对应的参数名称，直接避开这类的兼容问题，不失为一种最佳的方案（但是请说明清楚防止别人掉坑）。

### 二、重新实现

这一种的话都是基于我们的功能相对较为简单或者是为了以后的复用，我们可以基于Stimulus自己去实现相对应的功能。下面举个例子，简单实现对应的tab切换功能。

```html	
<!-- HTML页面代码 -->

<div data-controller="tabs">
  <div class="tabs">
    <ul>
      <li data-target="tabs.tab" data-action="click->tabs#change">
        <a>Tab 1</a>
      </li>
      <li data-target="tabs.tab" data-action="click->tabs#change">
        <a>Tab 2</a>
      </li>
    </ul>
  </div>
  
  <div class="hidden" data-target="tabs.panel">
    Tab panel 1
  </div>
  
  <div class="hidden" data-target="tabs.panel">
    Tab panel 2
  </div>
</div>

```

对应的JavaScript代码：

```javascript
const stimulus = Stimulus.Application.start();
stimulus.register("tabs", class extends Stimulus.Controller {

  static get targets() {
    return ["tab", "panel"];
  }

  initialize() {
    this.showTab();
  }

  change(event) {
    this.index = this.tabTargets.indexOf(event.currentTarget);
  }

  showTab() {
    this.tabTargets.forEach((tab, index) => {
      const panel = this.panelTargets[index];
      tab.classList.toggle("active", index === this.index);
      panel.classList.toggle("hidden", index !== this.index);
    })
  }

  get index() {
    return parseInt(this.data.get("index") || 0);
  }

  set index(value) {
    this.data.set("index", value);
    this.showTab();
  }
})

```

这样子就是一个简单的tab切换功能，好处就是慢慢积累成自己的“组件库”，复用也是极为方便的。

所以总结来说，一种是从开始去避开问题的出现，另一种算是曲线救国的方案，主要还是看具体项目去决定怎么处理类似出现的问题的吧。

