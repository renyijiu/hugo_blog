---
title: "JavaScript 学习笔记"
date: 2017-04-15 17:58:24
lastmod: 2018-01-14T20:50:47+08:00
draft: false
keywords: ['javascript', '笔记']
description: ""
tags: []
categories: ['JavaScript']
author: "renyijiu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
autoCollapseToc: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
---


这个是自己在跟着[廖雪峰的JavaScript教程](http://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000)学习时记录的一些笔记，这里整理一下，也算方便自己后面进行知识回顾。个人感觉这个教程作为入门了解一些专业名词和基础知识还是可以的，但是由于教程的内容包含的内容较为广，跨度较大，一些知识只是简单的叙述了一下，深入的理解和学习还是需要去查阅其他更详细的资料。

<!--more-->

#### 数据类型和变量

1. "use strict"，在strict模式下运行的JavaScript代码，强制使用`var`申明变量，未使用`var`申明变量就使用的，将会导致运行错误；未声明可能导致成为全局变量，污染全局变量环境。

   ```javascript
   ReferenceError: xxx is not defined
   ```

#### 字符串

1. ES6新增了一种模板字符串（若不支持，使用`+`连接字符串）

   ```javascript
   var name = 'hello world';
   var time = '2017-01-01';
   var message = `${name}, 今天是${time}!`;
   alert(message); // hello world, 今天是2017-01-01!
   ```

#### 数组

1. 直接给`Array`的`length`赋值，可能会导致`Array`大小的改变
2. 通过索引赋值时，当索引超出数组大小时，同样会引起`Array`大小的改变

#### 对象

1. 判断对象是否拥有某一属性，使用`in`操作符
2. 使用`in`判断某一属性是否存在时，属性不一定是对象本身的，可能是继承得到的
3. 判断一个属性是否是对象自身拥有的，而不是继承得到的，可以使用`hasOwnProperty()`方法

#### 条件判断

1. Javascript把`null`、`undefined`、`0`、`NaN`和空字符串`''`视为`false`，其他值均为`true`

#### 循环

1. `for...in`对`Array`的循环得到的是`String`而不是`Number`

#### 函数

1. JavaScript函数允许传入任意个数的参数而不影响调用
2. 关键字`arguments`只在函数内部起作用，可以获取到当前函数调用者传入的所有参数

#### 方法

1. 在一个方法内部，`this`是一个特殊变量，始终指向当前对象

2. 单独调用一个函数，此时，高函数的`this`指向全局变量，也就是`window`

3. 使用apply()和call()对`this`的指向进行控制，防止产生错误

4. 利用apply()动态的改变函数的行为

   假定我们想统计一下代码一共调用了多少次`parseInt()`，可以把所有的调用都找出来，然后手动加上`count += 1`，不过这样做太傻了。最佳方案是用我们自己的函数替换掉默认的`parseInt()`：

   ```javascript
   var count = 0;
   var oldParseInt = parseInt; // 保存原函数

   window.parseInt = function () {
       count += 1;
       return oldParseInt.apply(null, arguments); // 调用原函数
   };

   // 测试:
   parseInt('10');
   parseInt('20');
   parseInt('30');
   count; // 3
   ```

#### map

```javascript
'use strict';
var arr = ['1', '2', '3'];
var r;
r = arr.map(parseInt);

// 结果返回为[1, NaN, NaN]
```

原因是在调用`arr.map(parseInt)`的时候，传给`map`的参数为三个：

- 当前的值(currentValue)  `// 依次是'1','2','3'`
- 当前值的索引(currentIndex)  `// 依次是0，1，2`
- 当前的数组(currentArray) `// 都是['1','2','3']`

而每次调用parseInt函数时，函数接受两个参数，因此只传入了两个值(`currentValue, currentIndex`)

```javascript
parseInt('1', 0) // 1，当第二个参数转换后为0或者NaN时，会被忽略
parseInt('2', 1) // NaN
parseInt('3', 2) // NaN
```

所以有效的写法是`arr.map(function(x) { return parseInt(x)});`

#### sort

1. `Array`的`sort()`方法默认把所有元素先转换成`String`后再进行排序

#### 包装对象

1. 包装对象看上去和原来的值一样，但是他们的类型已经发生了改变，变成了`object`，并且使用`===`与原始值比较会返回`false`

   ```javascript
   typeof new Number(123); // 'object'
   new Number(123) === 123; // false

   typeof new Boolean(true); // 'object'
   new Boolean(true) === true; // false

   typeof new String('str'); // 'object'
   new String('str') === 'str'; // false
   ```

#### 箭头函数

易犯的错误：**箭头函数没有自己的`this`**，总是指向定义时所在的作用域的`this`值

```javascript
let a = {
  foo: 1,
  bar: () => console.log(this.foo)
}

a.bar()  //undefined
```

上述代码中，箭头函数中的`this`并不是指向`a`这个对象的，**使用的是词法作用域内的this**；对象`a`并不能构建自身的作用域，不能够新建自己的`this`值，所以向上查找到达全局作用域，`this`就是指向全局作用域，而全局作用域不存在相应的属性值，因此为`undefined`

下述代码中，如果使用普通函数时，输出结果就是符合预期的，这是因为

> 在函数运行时，this关键字并不会指向正在运行的函数本身，而是指向调用该函数的对象

也就是`a.bar()`函数执行时作用域绑到了`a`对象，`this`就是指向了`a`对象。

```javascript
let a = {
  foo: 1,
  bar: function() { console.log(this.foo) }
}

a.bar()  // 1

var c = a.bar;
c(); // undefined
```

此时调用函数`bar`的是`window`对象，所以函数`bar`的`this`被指向了`window`，而`window`不存在`foo`属性，所以结果是`undefined`

#### 面向对象编程

1. JavaScript的原型链和Java的Class的区别是，JavaScript没有`Class`的概念，所有的对象都是实例，所谓的**继承关系不过是把一个对象的原型指向另外一个对象而已**。
2. `prototype`是**函数**的一个属性(每个函数都有一个`prototype`属性)，这个属性是一个指针，指向一个对象。它是显示修改对象的原型的属性。
3. `__proto__`是一个对象拥有的内置属性，(**请注意：`prototype`是函数的内置属性，`__proto__`是对象的内置属性**)，是JS内部使用寻找原型链的属性。

#### 创建对象

1. 如果原型链很长，那么访问一个对象的属性就会因为花更多的时间查找而变慢，所以不要把原型链搞得太长

2. 使用`new`，变成了一个构造函数，他绑定的`this`指向新创建的对象，并默认返回`this`

   ```javascript
   function Student(name) {
       this.name = name;
       this.hello = function () {
           alert('Hello, ' + this.name + '!');
       }
   }

   var xiaoming = new Student('小明')；
   xiaoming.name; // "小明"
   xiaoming.hello(); // "Hello,小明!"
   ```

   ```javascript
   var xiaohong = new Student("小红")；
   xiaohong.name; // "小红"

   // 注意这里
   xiaohong.hello; // function Student/this.hello()
   xiaoming.hello; // function Student/this.hello()
   xiaohong.hello === xiaoming.hello // false
   ```

   从上述代码可以看出，`xiaoming.hello`和`xiaohong.hello`都是函数，但是它们是不同的函数，虽然函数名称和代码是相同的！

   所以，当我们通过`new Student()`创建了较多对象时，每个对象都有一个自己的`hello`函数，这样子来说是比较耗费内存的，实际上只需要共享同一个函数就可以了，这样子可以节省资源。

   ```javascript
   function Student(name) {
     this.name = name;
   }

   Student.prototype.hello = function() {
     alert("Hello," + this.name + "!");
   };
   ```

   *为了区分普通函数和构造函数，按照约定，构造函数首字母应该大写，而普通函数首字母应该小写*

#### 原型继承

1. 定义新的构造函数，并在内部用`call()`调用希望"继承"的构造函数，动态修改`this`
2. 借助中间函数`F`实现原型链继承（可以通过封装的`inherits`函数完成）
3. 继续在新的构造函数上的原型上定义新的方法

#### 浏览器对象

##### location

1. `location`对象表示当前页面的URL信息
2. 加载一个新页面，可以调用`location.assign()`
3. 重新加载当前页面，调用`location.reload()`

#### 扩展

##### 编写jQuery插件

1. 给`$.fn`绑定函数，实现插件的代码逻辑
2. 插件函数最后要`return this`，以实现链式调用
3. 插件函数要有默认值，绑定在`$.fn.<pluginNane>.defaults`上
4. 用户在调用时可以传入设定值以覆盖默认值

#### 模块

1. 在Node环境中，一个.js文件就称为一个模块(module)
2. 提高了代码的可维护性
3. 模块的名字就是文件名（去掉`.js`的后缀）
4. 模块中对外输出变量，用`module.exports = variable;`
5. 引入其他模块输出的对象，用`var foo = require('other_module');`

#### Stream

1. 流的特点是数据是有序的，而且必须是依次读取，或者依次写入
2. `pipe()`把一个文件流和另一个文件流串起来，这样子源文件的所有数据就自动写入到目标文件了

#### HTTP

1. `request`对象封装了HTTP请求，调用`request`对象的属性和方法就可以拿到所有的HTTP请求的信息
2. `response`对象封装了HTTP响应，操作`response`对象的方法，就可以把HTTP响应返回给浏览器

#### MVVM

1. 关注Model的变化，让MVVM框架取去自动更新DOM的状态
2. 单向绑定：把Model绑定到View，当我们使用JavaScript代码更新Model时，View会自动更新
3. 双向绑定： 当用户更新了View，Model的数据也会自动被更新
4. 让Model和DOM结构保持同步
5. 优势是前端逻辑复杂的页面，尤其是需要大量DOM操作的逻辑，利用MVVM可以极大地简化前端页面逻辑

