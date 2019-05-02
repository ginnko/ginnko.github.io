---
layout: post
title: ES6之后添加的日常开发用的到的新特性
date: 2019-04-08
tag: javascript
---

# 数组的新函数

1. `Array.prototype.includes()`

  判断一个数组是否包含一个指定的值，包含返回`true`，否则返回`false`。

2. `Array.prototype.flat()`

  这个方法按照一个可指定的深度递归遍历数组，将所有元素与遍历到的子数组中的元素合并为一个新数组返回（另外可以利用这个函数进行数组空项去除）。

<!-- more -->

3. `Array.prototype.flatMap()`

  这个方法首先使用映射函数映射每个元素，然后将结果压缩成一个新数组。
  
# 对象的新函数

1. `Object.values()`

  这个方法不是定义在原型上的，返回指定对象自身属性的所有值，不包含继承的值。

2. `Object.entries()`

  这个方法也没有定义在原型上，返回指定对象自身可枚举属性的键值对的数组

  ```js
    for (let [key, value] of Object.entries(obj1)) {
      console.log(`key: ${key}, value: ${value}`);
    }
  ```
3. `Object.fromEntries()`

  上一个函数的反转。

# 函数的新方法

1. `Function.prototype.toString()`

  改进版的返回精确字符，包含空格和注释

# 异步迭代器

```js
// 现在后台面板
// 拿到单品id
// 进行单品详细信息轮寻获取的时候好像用到了类似的方法
async function process(array) {
  for await (let i of array) {
    doSomething(i);
  }
}

// 下面这两种方式都无法正确执行异步过程

async function process(array) {
  for (let i of array) {
    await doSomething();
  }
}

// 或者

async function process(array) {
  array.forEach(async arr => {
    await doSomething();
  });
}
```

# Promise.finally()

不论是`resolve`还是`reject`最后都可以执行`finally`中的代码。

# 新的基本累心

- String
- Number
- Boolean
- Nill
- Undefined
- Symbol
- **BigInt**：这个好像还在提案中


参考:

1. https://juejin.im/post/5ca2e1935188254416288eb2