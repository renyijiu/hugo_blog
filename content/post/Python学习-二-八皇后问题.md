---
title: "Python学习 二 八皇后问题"
date: 2014-11-19 21:39:44
lastmod: 2018-01-14T20:56:37+08:00
draft: false
keywords: []
description: ""
tags: []
categories: ['Python']
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


## 八皇后问题
  在8*8的国际象棋上摆放八个皇后，使其不能互相攻击，即任意两个皇后都不能处于同一行、同一列或同一斜线上，问有多少种摆法？
<!--more-->

## 分析
  首先尝试放置第一个皇后(在第一行)，然后放置第二个皇后，以此类推。如果发现不能放下最后一个皇后，就回溯到上一步，试着将皇后放到其他的位置。

## 方案
使用生成器。生成器是逐渐产生结果的复杂递归算法的理想实现工具。本解决方法是 <**pyhton基础教程** >中的代码。


```python
#定义冲突函数,用于判断下一个皇后的位置会不会有冲突。
def conflict (state , nextX) :       #nextX代表下一个皇后的水平位置(列)，nextY代表垂直位置
	nextY = len (state)         #state[0] = 0,表示皇后处于第一行第一列(从0开始计数)
	for i in range (nextY) :    #range  包括起始，不包括结尾，后面的数为下个序列第一个数的编号
		if abs (state[i]-nextX) in (0,nextY-i) :
		              return True
	return False

#如果只剩一个皇后没有放置，那么遍历它所有可能的位置，并且返回没有冲突的位置
 def queens (num,state) :    #num参数是皇后的总数,state参数是存放前面皇后的位置信息的元祖
 	if len(state) == num-1:
 		for pos in range (num) :
 			if not conflict(state, pos) :
 				yield pos

#程序从前面的皇后得到了包含信息的元组,并且要为后面的皇后提供当前皇后的每种合法的位置信息
.......
      else :
      	for pos in range(num) :
      		if not conflict (state, pos) :
      			for result in queens(num,state + (pos,)) :    #生成器，解决问题的关键所在
      				yield (pos,) + result

#定义一个输出函数
def prettyprint(solution) :
	def line(pos,length = len(solution)) :
		return '. ' * (pos) + 'X ' + '. ' * (length - pos -1)
	for pos in solution :
		print line(pos)

#中间优化的代码
def queens(num = 8,state = ()) :
	for pos in range (num) :
		if not conflict(state, pos) :
			if len(state) == num -1 :
				yield (pos,)
			else :
				for return in queens(num,state + (pos,)) :
					yield (pos,) + result

```

