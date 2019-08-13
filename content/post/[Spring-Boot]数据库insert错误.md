---
title: "[Spring Boot]数据库insert错误"
date: 2019-08-13T12:18:07+08:00
lastmod: 2019-08-13T12:18:07+08:00
draft: false
keywords: ["Spring", "spring-boot", "database", "java"]
description: "[Spring Boot]数据库insert错误"
tags: ["spring-boot"]
categories: ["Java"]
author: "renyijiu"

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

开始写Java也有两个多月了，虽然深度参与了一个项目（基于spring-boot对一个Ruby项目进行重构，使用Java重新实现），但是因为基础框架是同事搭建的，所以可能对框架其中的一些东西不甚了解，更加偏向于业务的深度。所以就决定自己从零写一个小项目，尝试去体验spring-boot的整体开发流程。

当然一开始就碰到了各种小问题，也都通过查资料等解决了，所以决定记录一下，大部分问题可能都很简单，但可能其他人在开始学习时也会遇到一样的问题呢，留个思路在网上，万一帮助了其他新人也是极好的。

<!--more-->

## 问题

开始写了一个注册的接口，但是在运行插入数据的时候发现提示：

`Error updating database. Cause: org.postgresql.util.PSQLException: ERROR: null value in column "id" violates not-null constraint`

从错误信息看，字段违反了非空原则，这一列不能为空。`ID`这一列是PostgreSQL的主键。`PostgreSQL`建表的时候，`ID`字段设置成了`serial`字段，会自动生成一个关联的序列，实现自增的功能。如下图所示：

![数据库信息](/img/C5981B38-41AC-4A75-8288-6BE16CA74D12.png)

由此想到了查看我们对应的生成sql寻找问题，查看对应的`userMapper.insert(record)`方法，在对应的`UserMapper.xml`中定义如下：

```sql
<insert id="insert" parameterType="com.renyijiu.strangerbackend.dao.entry.User">
  insert into public.user (id, uuid, nickname,password, 
                           email, created_at,updated_at,
                           avatar_url)
  values (#{id,jdbcType=BIGINT}, 
          #{uuid,jdbcType=BIGINT}, 
          #{nickname,jdbcType=VARCHAR},
          #{password,jdbcType=VARCHAR}, 
          #{email,jdbcType=VARCHAR},
          #{createdAt,jdbcType=TIMESTAMP},
          #{updatedAt,jdbcType=TIMESTAMP},
          #{avatarUrl,jdbcType=VARCHAR})
</insert>
```

这个方法直接生成了对应的sql，其中没有对空值进行过滤处理，导致ID插入时强行设置成了`null`，而导致相关报错。

## 解决

为了解决这个问题，我们可以使用`userMapper`的`insertSelective`来代替`insert`方法，因为这个方法在对应的sql生成时，增加空值的判断，具体可以看下对应的xml文件中此方法的sql。

另外，这里注意的一点是在使用`mybatis-generator`自动生成相关文件时，配置文件中的

![xml配置](/img/386CA5B0-BBB6-4740-B1DC-785FC1E32281.png)

使用`MyBatis3Simple`时生成只有比较简单的sql，不存在Example相关的代码和方法，开始自己没有意识到自己的配置问题，还打算手动修改对应的sql解决问题🤦‍。另外，值得注意的是，重复执行`mybatis-generator`插件，**对应的xml文件不是重新覆盖的，而是增量的**，自己因为这个也碰到了另外的报错问题，当然这是后话了。