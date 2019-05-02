---
layout: post
title: javascript查漏补缺之二十 ——《继承和原型链》
date: 2018-08-15
tag: javascript
---

[链接地址](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)

### 获取和设置原型对象的标准方法

- 获取：`Object.getPrototypeOf()`

以后可以不用`__proto__`这个属性了。

### 设置一个对象的属性

通常，设置一个对象的属性会创建这个对象自身的属性。但如果这个对象继承了`getter`或`setter`属性，就会出现例外。

比如下面这段代码：

```js
var o = {
  a: 7,
  get b() { 
    return this.a + 1;
  },
  set c(x) {
    this.a = x / 2;
  }
};

var ins = Object.create(o);

ins.c = 50;//这行代码并不会给ins设置自己的c属性，而是会沿着原型链找到作为`setter`的c属性，结果是ins中多了一个属性a，属性值为25。
```

### 不同方式创建对象以及导致的原型链

1. 使用字面两创建

    - `var o = {a: 1};`, 原型链：`o ---> Object.prototype ---> null`。
    - `var b = ['yo', 'whadup', '?'];`, 原型链： `b ---> Array.prototype ---> Object.prototype ---> null`。
    - `function f() {return 2}`, 原型链： `f ---> Function.prototype ---> Object.prototype ---> null`。

2. 使用构造函数创建

    ```js
    function Graph() {
      this.vertices = [];
      this.edges = [];
    }

    Graph.prototype = {
      addVertex: function(v) {
        this.vertices.push(v);
      }
    };

    var g = new Graph();
    ```

3. 使用`Object.create`创建

    ```js
    var a = {a: 1}; 
    // a ---> Object.prototype ---> null

    var b = Object.create(a);
    // b ---> a ---> Object.prototype ---> null
    console.log(b.a); // 1 (inherited)

    var c = Object.create(b);
    // c ---> b ---> a ---> Object.prototype ---> null

    var d = Object.create(null);
    // d ---> null
    console.log(d.hasOwnProperty); 
    // undefined, because d doesn't inherit from Object.prototype
    ```

4. 使用`class`创建

    ```js
    class Polygon {
    constructor(height, width) {
      this.height = height;
      this.width = width;
      }
    }

    class Square extends Polygon {
      constructor(sideLength) {
        super(sideLength, sideLength);
      }
      get area() {
        return this.height * this.width;
      }
      set sideLength(newLength) {
        this.height = newLength;
        this.width = newLength;
      }
    }

    var square = new Square(2);
    ```

### `hasOwnProperty`

这个属性定义在`Object.prototype`上，是js中唯一一个处理对象属性且不会贯穿原型链的处理方法。

### [扩展原型链的方法比较](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain#Summary_of_methods_for_extending_the_protoype_chain)

MDN上推荐使用下面两种方法进行原型链的扩展。不推荐使用`Object.setPrototypeOf`以及`__proto__`。

另外感觉`class`也是一种实现扩展原型链的方法，感觉这里落下了。

1. 传统的方法

```js
function foo() {}
foo.prototype = {
  foo_prop: 'foo val'
};
function bar() {}
var proto = new foo;
proto.bar_prop = 'bar val';
bar.prototype = proto;
var inst = new bar;
```

2. 使用`Object.create`

下面的例子显示，`Object.create`可以传入第二个参数，表示添加到新创建对象的可枚举属性（即其自身定义的属性，而不是其原型链上的枚举属性）。

```js
function foo() {}
foo.prototype = {
  foo_prop: 'foo val'
};
function bar() {}
var proto = Object.create(foo.prototype);
proto.bar_prop = 'bar val';
bar.prototype = proto;
var inst = new bar;
```

或者

```js
function foo() {}
foo.prototype = {
  foo_prop: 'foo val'
};
function bar() {}
var proto = Object.create(
  foo.prototype,
  {
    bar_prop: {
      value: 'bar val'
    }
  }
);
bar.prototype = proto;
var inst = new bar;
```

### `prototype`和`Object.getPrototypeOf`

当你调用
```js
var o = new Foo();
```
时，javascript实际执行的是
```js
var o = new Object();
o.[[Prototype]] = Foo.prototype;
Foo.call(o);
```