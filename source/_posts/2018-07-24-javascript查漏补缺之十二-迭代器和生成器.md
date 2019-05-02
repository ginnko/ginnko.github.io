---
layout: post
title: javascript查漏补缺之十二 ——《迭代器和生成器》
date: 2018-07-24
tag: javascript
---

[链接地址](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Iterators_and_Generators)

1. 迭代器

    迭代器是一个**对象**，知道如何每次访问集合中的一项，并跟踪该序列中的当前位置。它提供一个`next()`方法，用来返回序列中的下一项。这个方法返回包含两个属性：`done`和`value`。

    ```javascript
    //创建一个迭代器对象

    function makeIterator(array) {
      var nextIndex = 0;
      return {
        next: function() {
          return nextIndex < array.length ?
              {value: array[nextIndex++], done: false} :
              {done: true};
        }
      };
    }

    //使用迭代器

    var it = makeIterator(['yo', 'ya']);
    console.log(it.next().value); // 'yo'
    console.log(it.next().value); // 'ya'
    console.log(it.next().done); // true
    ```
<!-- more -->

2. 生成器

    迭代器需要显式地维护其内部状态，**生成器**允许自定义一个包含自有迭代算法的函数，同时它可以自动维护自己的状态。

    ```javascript
    function* idMaker() {
      var index = 0;
      while(true) {
        yield index++;
      }
    }

    var gen = idMaker();

    console.log(gen.next().value); // 0
    console.log(gen.next().value); // 1
    console.log(gen.next().value); // 2
    ```

3. 可迭代对象

    为了实现可迭代，一个对象必须实现`@@iterator`方法，就是说这个对象或其原型链中的一个对象必须具有带`Symbol.iterator`键的属性。

    ```javascript
    var myIterable = {};
    myIterable[Symbol.iterator] = function* () {
      yield 1;
      yield 2;
      yield 3;
    };

    for (let value of myIterable) {
      console.log(value);
    }

    // 1
    // 2
    // 3

    [...myIterable] // [1, 2, 3]

4. 高级生成器

    The next() 方法也接受可用于修改生成器内部状态的值。传递给next()的值将被视为暂停生成器的最后一个yield表达式的结果。

    ```javascript
    使用 next(x) 重新启动 fibonacci 序列生成器：

    function* fibonacci() {
      var fn1 = 0;
      var fn2 = 1;
      while(true) {
        var current = fn1;
        fn1 = fn2;
        fn2 = current + fn1;
        var reset = yield current;
        if (reset) {
          fn1 = 0;
          fn2 = 1;
        }
      }
    }

    var sequence = fibonacci();
    console.log(sequence.next().value);     // 0
    console.log(sequence.next().value);     // 1
    console.log(sequence.next().value);     // 1
    console.log(sequence.next().value);     // 2
    console.log(sequence.next().value);     // 3
    console.log(sequence.next().value);     // 5
    console.log(sequence.next().value);     // 8
    console.log(sequence.next(true).value); // 0
    console.log(sequence.next().value);     // 1
    console.log(sequence.next().value);     // 1
    console.log(sequence.next().value);     // 2
    ```