---
title: "Ubuntu下搭建LAMP服务器"
date: 2014-11-28 15:43:00
lastmod: 2018-01-14T20:58:38+08:00
draft: false
keywords: []
description: ""
tags: []
categories: ['Ubuntu']
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


本文主要记录下在Ubuntu下搭建LAMP服务器的大致流程

		LAMP = Linux + Apache + Mysql + PHP
<!--more-->

## 安装Apache2
第一步是安装Apache2，在Ubuntu下只需要使用  apt-get install  命令即可

		$ sudo apt-get install apache2

## 安装PHP
安装PHP需要安装两个包，一个是 `PHP5` 本身，二是Apache2的 `PHP模块` ，安装命令如下：

		$ sudo apt-get install php5 libapache2-mod-php5

## 配置Mysql的root密码
如果你在用上述的命令安装完Mysql后使用命令：

		$ mysql -uroot -p

**注意**：这里不用输入密码，直接敲回车键就能进入Mysql中

进入后我们可以使用命令来重新设置root密码，也可以同样地新建用户

```
>USE mysql;
>UPDATE user SET password = PASSWORD('你要设置的密码') WHERE user= 'root';
>flush privilege;
>exit;

```
登录Mysql的操作就是上面的命令：

		$ mysql -u('你的用户名') -p

回车后输入你的密码即可

## 启动Apache2

现在我们用命令启动`apache2`服务：

		$ sudo service apache2 start

然后我们通过浏览器访问<http://localhost/>,应该能看到一个关于apache2的网页，有一行是：

		It Works!

到这，一台基于Ubuntu能够正常使用的服务器已经搭好了。

## 安装PHPMyAdmin

PHPMyAdmin是一款基于PHP的数据库管理网站，在Ubuntu下安装也比较简单：

		$ sudo apt-get install phpmyadmin

然后我们将PHPMyAdmin的配置复制到apache2的配置文件夹下：

		$ sudo cp /etc/phpmyadmin/apache.conf /etc/apache2/sites-available/phpmyadmin.conf

		$ sudo ln -s /etc/apache2/sites-available/phpmyadmin.conf /etc/apache2/sites-enabled/phpmyadmin.conf


最后，我们重启apapche2:

		$ sudo service apache2 restart

然后我们访问<http://localhost/phpmyadmin/>,用你的数据库帐号密码登录

**提醒**:这里存在着一个问题，用你新建的账户无法登录，我自己测试只能用root账户登录，不知道原因，待找到解决办法再更新本文！

