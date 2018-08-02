---
title: "Form表单上传文件的问题"
date: 2017-11-18T22:24:05+08:00
lastmod: 2018-01-14T21:04:03+08:00
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


后台优化时，需要增加一个给用户发送图片消息的功能，为了兼容原有代码，使用了form表单上传图片文件，大致代码如下：<!--more-->

```ruby
<%= form_tag test_path, multipart: true, remote: true do %>
	<input id="message_photo" class="form-control" type="file" name="message[qiniu_key]">
	<span class="glyphicon glyphicon-picture" id="img-upload-icon"></span>
<% end %>
```

原本预想的是通过添加`remote: true`，配合js监听事件来达到无刷新上传的效果，但实际测试发现每次文件上传后页面都会整体刷新，下面就分析下这个问题。

### jquery-ujs

在Rails5.1之前默认是通过依赖jQuery来提供内置ajax的支持（Rails5.1默认已经去除了对于jQuery的依赖，使用原生js进行相关开发实现），所以默认都包含了`jquery-rails`这个gem包。而这个gem包包含了`jquery-ujs`从而提供了`data-remote`等类似的功能方法。

所以查看`jquery-ujs`中的[rails.js](https://github.com/rails/jquery-ujs/blob/4b6e30f68ff1244fc0c790641d3408c2695a29bd/src/rails.js#L500)源代码可以发现，

```javascript
if (remote) {
  nonBlankFileInputs = rails.nonBlankInputs(form, rails.fileInputSelector);
  if (nonBlankFileInputs) {
    // Slight timeout so that the submit button gets properly serialized
    // (make it easy for event handler to serialize form without disabled values)
    setTimeout(function(){ rails.disableFormElements(form); }, 13);
    var aborted = rails.fire(form, 'ajax:aborted:file', [nonBlankFileInputs]);

    // Re-enable form elements if event bindings return false (canceling normal form submission)
    if (!aborted) { setTimeout(function(){ rails.enableFormElements(form); }, 13); }

    return aborted;
  }
  
  rails.handleRemote(form);
  return false;
  ...
}
```

当使用`remote: true`时，如果存在非空的`type=file`的输入框时，代码会进入if判断，然后返回，跳过了`rails.handleRemote(form);`这个函数的执行过程，这样就回退到浏览器的表单提交，也就是同步提交后刷新页面。

上面可以看出，源代码设计就是这样子，无法通过`remote: true`来异步上传文件。这个是个历史遗留问题，旧浏览器在当时不支持这种通过ajax的方式上传文件，所以没有办法。但是在HTML5中其实是已经支持异步上传文件的，可以通过`FormData`去实现这个功能（如果不考虑兼容旧浏览器，这是个不错的方法）。

后面发现项目的wiki中也算提供了一种折中的解决方式，

```javascript
$('form').bind('ajax:aborted:file', function(event, elements){
  // Implement own remote file-transfer handler here for non-blank file inputs passed in `elements`.
  // Returning false in this handler tells rails.js to disallow standard form submission
  return false;
});
```

提供了一个Hook方式，通过监听这个事件，自己编写添加上传文件的代码。另外[Remotipart](https://os.alfajango.com/remotipart/)这个gem包就是通过这个Hook去封装了文件的上传，为开发者提供了便利。
