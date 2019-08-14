---
title: "[Spring Boot]insert not found问题"
date: 2019-08-14T18:48:09+08:00
lastmod: 2019-08-14T18:48:09+08:00
draft: false
keywords: ["spring", "spring-boot", "mybatis", "java"]
description: "mybatis insert not found"
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

个人项目中编写了一个简单的注册接口，但是在过程中遇到了几个小问题，这里记录一下。真所谓万事开头难，一个简单的接口，搭建项目碰到了各种小问题😂。

<!--more-->

## 问题1

在启动项目时，提示了

```java
Field userMapper in com.renyijiu.strangerbackend.service.Impl.AccountServiceImpl required a bean of type 'com.renyijiu.strangerbackend.dao.mapper.UserMapper' that could not be found.

The injection point has the following annotations:
	- @org.springframework.beans.factory.annotation.Autowired(required=true)


Action:

Consider defining a bean of type 'com.renyijiu.strangerbackend.dao.mapper.UserMapper' in your configuration.
```

根据错误信息，提示`UserMapper`缺少`bean`注解导致的无法自动`@Autowired`导致的错误。

### 解决

1. 我们可以直接在`UserMapper`接口类上增加`@Mapper`注解，但是这个方案不好的一点是每一个新增的`Mapper`接口类都需要增加注解，而这文件一般都是使用插件自动生成的，很容易忽略这步操作。
2. 使用`@MapperScan`注解（对于spring版本有一定要求，网上看到是3.x+），如下：

```java
@SpringBootApplication
@MapperScan("com.renyijiu.strangerbackend.dao.mapper") // 新增
public class StrangerBackendApplication {

	public static void main(String[] args) {
		SpringApplication.run(StrangerBackendApplication.class, args);
	}

}
```

当然也是可以通过xml文件进行配置，但是自己不喜欢这种方式，所以就忽略了。

## 问题2

在测试注册接口时，提示了

```java
org.apache.ibatis.binding.BindingException: Invalid bound statement (not found): com.renyijiu.strangerbackend.dao.mapper.UserMapper.insert
```

这个错误自己看着很懵，搜索了一下，看到的方案大致都是：

>1. mapper.xml的namespace要写所映射接口的全称类名。
>2. mapper.xml中的每个statement的id要和接口方法的方法名相同
>3. mapper.xml中定义的每个sql的parameterType要和接口方法的形参类型相同
>4. mapper.xml中定义的每个sql的resultType要和接口方法的返回值的类型相同
>5. mapper.xml要和对应的mapper接口在同一个包下
>6. mapper.xml的命名规范遵守: 接口名+Mapper.xml

但由于文件是`mybatis-generator`插件自动生成的，完全没有上述的情况，所以还需要寻找解决方案。

### 解决

当你的Mapper.xml与Mapper.class在同一个文件夹下且同名时，spring扫描Mapper.class的同时会自动扫描同名的Mapper.xml并装配到Mapper.class。

但是我在自动生成时，将xml文件放置在`resources`下自定义文件夹`postgres`中了，导致了这个问题的出现。

所以我们需要在`application.properties`中配置一下，手动设置文件路径解决这个问题。

```java
mybatis.mapperLocations=classpath:postgres/*.xml
```



## 总结

上述问题都比较简单，通过搜索查找资料慢慢都可以解决，不过对于自己而言，如果不从头开始搭建项目，可能也不会知道这些小细节的存在。