---
title: "GET 和 POST 的区别"
date: 2015-05-14 22:43:49
lastmod: 2018-01-14T20:25:00+08:00
draft: false
keywords: []
description: ""
tags: []
categories: ['网络']
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

  写这篇文章的原因是因为在面试时被问到了这个问题，结果因为自己只了解了一点，被深入询问时就彻底的崩盘了*（心态真的很重要啊）*，然后就不知道说什么了，所以结束后就自己查询了下资料，记录下来，也算是在这个过程中学到了一些东西。
 <!--more-->

## 简单定义

   在 Http 中定义了与服务器交互的不同方式，最基本的方法有4种，一般是GET，POST，PUT，DELETE 。 URL全称为统一资源定位符，一个URL地址对应着网上的一个资源，而HTTP的GET，POST，PUT，DELETE则分别对应着 **查**，**改**，**增**，**删** 的4个操作，这个概念个人觉得还是比较容易理解和记忆的。

>GET 一般用于 获取/查询 资源信息，POST一般用于 更改 资源信息。


----------


 ★   *一般在浏览器中输入网址访问资源都是通过 **get** 的方式*



----------



## 概念区别

1. 根据HTTP规范，GET用于**信息获取**，而且应该是 *安全* 的和 *幂等* 的
    1. 所谓安全的意味着该操作用于获取信息而非修改信息。换句话说，GET操作请求一般不会产生不良后果。就是说，他仅仅是获取资源信息，就像数据库查询一样，不会修改、增加数据，不会影响资源的状态。
    2. 幂等的意味着对于同一URL的多个请求应该返回同样的结果。

2. 根据HTTP规范，POST表示可能会 **修改** 服务器上资源的请求


## 提交方式的区别

- GET提交：请求的数据会附在URL之后（就是把数据放置在请求行（request line）      中），以?分割URL和传输数据，多个参数用&连接；例如：
login.action? name=suiyi&password=idontknow&verify=%E4%BD%A0 %E5。
URL的编码格式采用的是ASCII码，而不是Unicode。

- POST提交：把提交的数据放置在HTTP包的包体中。

**因此**，GET提交的数据会*在地址栏中显示出来*，而POST提交时，*地址栏不会的改变*。

- GET : 特定浏览器和数据对URL长度有限制，传输数据就会受到URL长度的限制。

- POST：由于不是通过URL传值，理论上数据不受限。


## POST的安全性比GET的安全性要高

1. 通过GET提交数据，用户名和密码将明文出现在URL上

    1. 登录页面可能被浏览器缓存
    2. 其他人查看浏览器的历史纪录，那么别人就可以拿到你的账号和密码了


2. POST是将表单中的数据放在form的数据体中，按照变量和值相对应的方式，传递到action所指向URL。
