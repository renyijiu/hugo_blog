---
title: "Hexo的基本命令"
date: 2014-10-28 21:39:13
lastmod: 2018-01-14T20:45:36+08:00
draft: false
keywords: []
description: ""
tags: []
categories: []
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

   这个博客使用的就是hexo，一些命令可能随时间会有部分遗忘，所以写一篇文章记录下来方便自己下次查询。
   <!--more-->

## 新建博文
		hexo new [layout] "postName"

  其中layout是可选参数，默认值为post。

## 新建页面
		hexo new page "about"

  在hexo\source\下会生成about目录，里面有个index.md，直接编辑就可以了，然后在主题的_config.yml中在menu中添加下相对路径。

## read more功能
   编辑博文时可以添加这个以显示在页面你想显示的摘要类内容，可以自己控制长度，当然长度须在主题允许的长度内，也可更改主题内限制长度达到相同的效果。

        <!--more-->

## 清除
   有时直接生成会出现一些未知的错误，例如自己写的东西推送不上去，或者生成的内容不对，这时可能就需要清空下。

        hexo clean

## 生成静态页面

		hexo generate

## 本地启动
		hexo server
   执行这个命令，启动本地服务，进行文章预览，根据显示效果在根据自己的需求进行修改。浏览器输入  http://localhost:4000  可以查看

## 上传更新
		hexo deploy

## 常用命令
```
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
hexo generate #生成静态页面至public目录
hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
hexo deploy #将.deploy目录部署到GitHub
```

## 常用复合命令
```
hexo deploy -g
hexo server -g
```

## 命令简写
```
hexo n == hexo new
hexo g == hexo generate
hexo s == hexo server
hexo d == hexo deploy
```
