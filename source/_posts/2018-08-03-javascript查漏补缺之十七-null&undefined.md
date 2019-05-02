---
layout: post
title: javascript查漏补缺之十七 ——《null&undefined》
date: 2018-08-03
tag: javascript
---

[链接地址(评论区RedNax的评论)](http://www.ruanyifeng.com/blog/2014/03/undefined-vs-null.html)

感觉这段评论将`undefined`和`null`的不同说的很清楚，而且也说明白了应该在什么地方使用。

react的源码`ReactElement.js`中的`createElement`函数中就有如下定义：

如此看来这样的定义才是合理的。

```javascript
  let key = null;
  let ref = null;
  let self = null;
  let source = null;
```
<!-- more -->

1. null 和 undefined在现代JS语义里面是有明确区别的：

    null 表示一个值被定义了，定义为“空值”；

    undefined 表示根本不存在定义。

    所以设置一个值为 null 是合理的，如objA.valueA = null;但设置一个值为 undefined 是不合理的，如objA.valueA = undefined; // 应该直接使用 delete objA.valueA; 任何一个存在引用的变量值为undefined都是一件错误的事情。

    这样判断一个值是否存在，就可以用objA.valueA === undefined // 不应使用 null 因为 undefined == null，而 null 表示该值定义为空值。

    这个语义在JSON规范中被强化，这个标准中不存在 undefined 这个类型，但存在表示空值的 null 。在一些使用广泛的库（比如jQuery）中的深度拷贝函数会忽略 undefined 而不会忽略 null ，也是针对这个语义的理解。


2. JS 中同时存在 undefined 和 null 是合理的。

    首先在 Java 中不存在 undefined 是很合理的：Java 是一个静态类型语言，对于 Java 来说不可能存在一个“不存在”的成员（不存在的话直接就编译失败了），所以只用 null 来表示语义上的空值。而 JavaScript 是一门动态类型语言，成员除了表示存在的空值外，还有可能根本就不存在（因为存不存在只在运行期才知道），所以这就要一个值来表示对某成员的 getter 是取不到值的。


3. typeof null 结果是 ”object“ 更像是一个设计失误

    因为 typeof null === "object" 而认为 null 语义是表示空对象是个不谨慎的猜测，感觉像是先射箭后画靶一般。简单的反例：在强类型数据交换协议 odata（http://www.odata.org/）的 JSON 格式中，即使一个成员定义为特定类型（比如string），也可以设置其值为 null 来表示这个值是空值，这可不是表示这个成员是空对象，只是说值为空而已（和空字符串、0、false有所区别）。