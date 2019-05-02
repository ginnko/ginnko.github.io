---
layout: post
title: javascript查缺补漏之二 ——《语法与类型》
date: 2018-07-04
tag: javascript
---

[原文地址](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Grammar_and_Types)

1. 在 JavaScript 中大小写敏感，变量 früh 和 Früh 则是两个不同的变量。

2. 当你对一个 null 变量求值时，空值 null 在数值类型环境中会被当作0来对待，而布尔类型环境中会被当作 false。

<!-- more -->

3. 局部变量：在函数内部声明的变量。

4. 函数提升：只有函数声明会被提升到顶部，而不包括函数表达式。

5. 对象属性和数组元素不受保护，可以使用`const`来声明对象名或数组名

    ```
    const MY_OBJECT = {"key": "value"};
    MY_OBJECT.key = "otherValue";

    const MY_ARRAY = ['HTML','CSS'];
    MY_ARRAY.push('JAVASCRIPT');
    console.log(MY_ARRAY); //logs ['HTML','CSS','JAVASCRIPT'];
    ```
6. Object.prototype.toString()

    每个对象都包含一个`toString()`属性，默认情况会返回 "[object type]"（注意！！！此处返回值确实会带着引号），其中type是对象的类型。

    可以在对象中重写`toString()`来覆盖默认返回值。

7. 给一个对象定义属性和值的一个不常用的写法：

    ```
    let a = {};
    Object.defineProperty(a, mySymbol, { value: 'Hello!' });
    ```

8. 魔术字符串

    魔术字符串指的是，在代码之中多次出现、与代码形成强耦合的某一个具体的字符串或者数值。

9. `for in` vs `Object.keys` vs `Object.getOwnPropertyNames`

    - `for in` ：用来获取某个对象（**该对象**）及其**原型链**上的可枚举属性。如果仅想获取该对象自身的属性，要借助`hasOwnProperty`方法。

        ```
        for (var key in child) {
            if (child.hasOwnProperty(key)) {
                console.log(key);
            }
        }
        ```
    
    - `Object.keys` ：用来获取某个对象自身的可枚举属性。其效果等于`for in` + `hasOwnProperty`。

    - `Object.getOwnPropertyNames` ： 用来获取某个对象自身可枚举以及不可枚举的属性（不可获取以symbol命名的属性名）。

10. 不要混淆原始的布尔值true和false与 Boolean对象的真和假

    ```
    var b = new Boolean(false);
    if (b) // this condition evaluates to true
    if (b == true) // this condition evaluates to false
    ```