---
title: "Form表单遇到的小问题"
date: 2017-11-12T17:12:55+08:00
lastmod: 2018-01-14T21:04:52+08:00
draft: false
keywords: []
description: ""
tags: []
categories: ['Rails']
author: ""

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: true
mathjax: false
---


在做后台管理页面时，有一个小需求，提供一个下拉框可以手动修改状态值，为了简单就直接使用form表单去实现，当然也不希望刷新整个页面，大致代码如下：
<!--more-->

```Ruby
<%= form_tag test_path(format: :js), method: :patch, remote: true do %>
  <%= select_tag :state, options_for_select(options, { selected: state }), { onchange: "this.form.submit();" } %>
<% end %>
```

但是测试时发现未达到想要的效果，这里面存在了两个问题：

#### 1. `remote: true`

   在Rails4中，form使用`remote: true`时，form表单不会自动的添加CSRF的隐藏字段，所以当你提交表单时会遇到这个报错`ActionController::InvalidAuthenticityToken`，解决办法也比较简单

   1. 修改配置，`config.action_view.embed_authenticity_token_in_remote_forms = true`（action_view的配置选项）
   2. 手动添加参数保证rails自动添加字段，`:authenticity_token => true`
   3. 引入`jquery-ujs`之类的，会自动的在相关请求头部添加相应字段
   

#### 2. `onchange: "this.form.submit();"`

   这代码是想修改后就提交表单，就不用去写其他的js代码了，但是这个代码中`remote: true`并不会起作用，当选项修改后，`"this.form.submit();"`个人理解是触发浏览器表单提交的行为，会同步提交表单而后页面整体刷新。所以需要修改成ajax提交表单的方式：修改代码为`onchange: "$(this).closest('form').submit();"`，触发`remote: true`机制，这样就可以达到想要的效果了。
