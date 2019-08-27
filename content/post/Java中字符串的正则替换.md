---
title: "Java中字符串的正则替换"
date: 2019-08-26T16:40:13+08:00
lastmod: 2019-08-26T16:40:13+08:00
draft: false
keywords: ["Java", "正则表达式", "字符替换"]
description: "Java中字符串的正则替换"
tags: ["Java", "正则表达式"]
categories: ["Java"]
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

最近有个需求中间过程比较复杂，再三考虑决定使用正则表达式替换逻辑去简化复杂度，但是在处理的过程中发现正则表达式全局替换特定匹配组并没有很好的现成方法，所以记录下自己的解决方案。

<!--more-->

## 问题描述

```json
[
  {"category": "java", "id": 1,"relation": "不等于","value": 0},
  {"category": "ruby", "pattern_id": 1, "relation": "不缺失", "value": 0},
  {"category": "python","card_id": 66,"relation": "缺失","value": 0,"card_version": 1}
]
```

存在一个格式类似于上面示例的json字符串模块数据，由于实际json的格式存在层级嵌套，所需的数据存在于最深的层级，另一个嵌套中还存在多种类型的数据，每类数据格式之间的细微差别导致难以进行统一读取处理，综合考虑使用正则表达式进行处理。通过编写正则表达式，全局提取出符合要求格式中的ID值数据，使用对应的数据进行替换。

## 解决方案

考虑为每一个类型编写一个正则表达式，分离正则表达式的复杂度，其中一个类型示例：

```java
String REG = "\\{.*?[\"']category[\"']:\\s*[\"']%s[\"'][^\\}\\]]*?[\"']id[\"']:\\s*(%s).*?\\}"
```

这是个正则模版，`category`和`id`数据是通过`String.format`进行传递的，正则的内容就是匹配符合要求类型以及ID的字符串，提取出对应的ID数据，方便进行后续的替换操作。

另外由于需求上需要记录替换字符串的位置方便后续的逻辑操作，这个要求也就排除了自带提供的`replaceAll(String regex, String replacement)`方法，因为我们无法通过这个方法获取到替换的位置。然后网络上搜索了一下，也没有现成方法去全局的搜索替换特定的匹配组数据，所以就需要自己写一个方法。

```java
/**
* @param: regex regex string
* @param: source content string
* @param: oldId -> newId map
* @return: replaced string
*/
private static String replaceMatchId(String regex, String source, Map<Integer, Integer> compareMap) {
  StringBuilder stringBuilder = new StringBuilder(source);
  Matcher m = Pattern.compile(regex).matcher(source);
  while (m.find()) {
    log.info("string group: {}, start: {}, end: {}", m.group(1), m.start(1), m.end(1));

    if (StringUtils.isBlank(m.group(1)) || "null".equals(m.group(1))) {
      continue;
    }
    Integer id = Integer.parseInt(m.group(1));
    Integer newId = (compareMap.get(id) == null) ? id : compareMap.get(id);
    stringBuilder = stringBuilder.replace(m.start(1), m.end(1), newId.toString());
  }
  return stringBuilder.toString();
}
```

方法的大致逻辑就是，正则匹配获取各个匹配项，循环去替换对应匹配位置的id数据（忽略了记录位置的逻辑）。这个方法一定情况下是可以正确运行的，但是当一次测试时，原有ID与替换ID的位数不一致，例如`{100 -> 10001}`时，问题就出来了。在第一次替换完成后，由于位数的不一致，导致字符串的位置发生了偏移，导致了后续的数据替换产生了错误。因为在原来位置的数据偏移后并不是我们原来提取的id数据，直接替换后会产生各种问题，最直接的就是无法正确的反序列化成json格式数据，产生报错。

所以这个方法是无法使用的，修改逻辑，代码如下：

```java
private String replaceAll(String regex, String source, int groupToReplace, String replacement) {
  StringBuffer sb = new StringBuffer();
  Matcher m = Pattern.compile(regex).matcher(source);
  while (m.find()) {
    StringBuilder buf = new StringBuilder(m.group());
    buf.replace(m.start(groupToReplace) - m.start(), m.end(groupToReplace) - m.start(), replacement);
    m.appendReplacement(sb, buf.toString());
  }
  m.appendTail(sb);
  return sb.toString();
}
```

通过字符串的拼接操作替代原有的直接替换对应位置数据的逻辑，这样子即可。

## 总结

考虑问题还是需要多思考边界的正确性，这样才能保证程序的健壮性。

