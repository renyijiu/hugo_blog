---
title: "Node Js开发指南 学习笔记"
date: 2017-04-05 11:15:35
lastmod: 2018-01-14T20:54:12+08:00
draft: false
keywords: []
description: ""
tags: []
categories: ['JavaScript']
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


前面在看`node.js开发指南`学习的时候，发现书中的版本比较旧了，一些代码在新的版本上存在一些不一致，会出现报错情况，下面是自己在跟随书本编写时遇到的一些问题及解决办法。<!--more-->

#### 运行

书中写的`npm app.js`，这个按照`express 4`的文档应该是`npm start`，另外在`package.json`中也有定义

```javascript
  "scripts": {
    "start": "node ./bin/www"
  }
```

所以也可以使用`node ./bin/www`

#### 模板

在`express 4`中`<%- body %>`这种模板直接使用会报错误，可以使用`<%- include .... %>`的方式，

如果需要使用原来的方法，可以使用`express-ejs-layouts`这个包.

#### 报错信息

###### 1. TypeError

```javascript
TypeError: Cannot read property 'Store' of undefined
```

如果遇到这个报错信息，可以使用下面的代码方式解决

```javascript
var expressSession = require('express-session');
var MongoStore = require('connect-mongo')(expressSession);
```

###### 2. Mongo Error

```javascript
Error: Connection strategy not found
```

书中`connect-mongo`使用的是旧的写法建立新的连接，改成下列方式

```javascript
app.use(expressSession({
    secret: setting.cookieSecret,
    store: new MongoStore({
      url: 'mongodb://localhost/microblog'
    })
}));
```

###### 3. Form 

POST的表单数据为空，纠结了自己好久才发现，表单`input`的`name`属性没有填写，所以导致了`POST`过来的数据一直为空

###### 4. Helpers   

新版本的Express已经不支持`app.dynamicHelpers`方法，需要使用`locals`来代替

```javascript
app.use(function(req, res, next) {
  res.locals.user = req.session.user;
  var err = req.flash('error');
  var success = req.flash("success");

  res.locals.error = err.length ? err : null;
  res.locals.success = success.length ? success : null;
  next();
});
```

另外，需要将上述代码放在路由定义之前，不然在页面上中是无法获取到`user, error, success`参数的。

###### 5. Mongo Port

` Cannot read property 'DEFAULT_PORT' of undefined。`

需要将这个直接替换成mongodb的端口，一般默认是`27017`；但推荐是写在配置文件中引入。

#### 结语

上面是在这本书学习过程中遇到的几个主要的问题，写出来也算为后来的学习者提供点资料吧，因为在自己的查询过程中，发现这个错误信息的资料，尤其是中文资料真的不多，对新手不是很友好。

