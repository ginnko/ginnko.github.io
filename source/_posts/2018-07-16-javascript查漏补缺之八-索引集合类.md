---
layout: post
title: javascript查漏补缺之八——《索引集合类》
date: 2018-07-16
tag: javascript
---

[链接地址](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Indexed_collections)

1. 用来处理DOM集合的一个更高效的土法子

  从来没有这样用过诶。`div = divs[i]`这个赋值表达式，当i = length时，返回值为false就会停止这个循环。

  ```
  var divs = document.getElementsByTagName('div');
  for (var i = 0, div; div = divs[i]; i++) {
    /* Process div in some way */
  }
  ```
<!-- more -->

2. 以下是几个自己没有怎么常用的数组方法

  1. `join()`

    ```
    var myArray = new Array("Wind", "Rain", "Fire");
    var list = myArray.join(" - "); // list is "Wind - Rain - Fire"
    ```

  2. `Array.prototype.reverse()`

3. `map(callback[, thisObject])`之类的函数有第二个参数，thisObject 变成回调函数内部的 this 关键字的值。如果没有提供，例如函数在一个显示的对象上下文外被调用时，this 将引用全局对象(window)。 

4. 字符串使用数组函数

  ```
  Array.prototype.forEach.call('a string', function(chr) {
    console.log(chr);
  });
  ```