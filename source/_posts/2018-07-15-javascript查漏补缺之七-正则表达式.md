---
layout: post
title: javascript查漏补缺之七——《正则表达式》
date: 2018-07-15
tag: javascript
---



#### 正则表达式中的特殊字符

1. **?**

    1. 单独使用：匹配前面一个表达式0次或者1次。等价于 {0,1}。

    2. 如果紧跟在任何量词 `*、 +、? 或 {}` 的后面，将会使量词变为非贪婪的（匹配尽量少的字符），和缺省使用的贪婪模式（匹配尽可能多的字符）正好相反。
2. `x(?=y)`: 正向肯定查找。

    > /Jack(?=Sprat)/会匹配到'Jack'仅仅当它后面跟着'Sprat'。/Jack(?=Sprat|Frost)/匹配‘Jack’仅仅当它后面跟着'Sprat'或者是‘Frost’。但是‘Sprat’和‘Frost’都不是匹配结果的一部分。

<!-- more -->

3. `x(?!y)`: 正向否定查找。

    > /\d+(?!\.)/匹配一个数字仅仅当这个数字后面没有跟小数点的时候。正则表达式/\d+(?!\.)/.exec("3.141")匹配‘141’但是不是‘3.141’

4. `[^xyz]`: 方向字符集和。匹配**任何没有包含**在方括号中的字符。

5. `.`: 匹配除换行符之外的任何单个字符。

6. `\w`： 匹配一个单字字符（字母、数字或者下划线）。等价于[A-Za-z0-9_]。

#### 正则表达式的方法

1. `RegExp.prototype.exec()`：这个函数类似`String.prototype.match()`,但是返回数组的同时还会同时更新正则表达式的属性。

    - 返回的数组和更新的正则表达式的附带属性见下图

        ![exec()返回值](/images/js/5.png)

    - 使用`exec（）`函数的一个方便用法的示例

        ```
        var myRe = /ab*/g;
        var str = 'abbcdefabh';
        var myArray;
        while ((myArray = myRe.exec(str)) !== null) {
        var msg = 'Found ' + myArray[0] + '. ';
        msg += 'Next match starts at ' + myRe.lastIndex;
        console.log(msg);
        }
        ```

        *注意：*

        1. 不要把正则表达式字面量（或者RegExp构造器）放在 while 条件表达式里（否则每次都会重新设置，会造成while的死循环）;

        2. 要加g标志（否则不会查找下一个，会造成while的死循环）。

2. `RegExp.prototype.test()`： 类似`String.prototype.search()`,只是返回值为布尔值。

3. `String.prototype.split()`: 这个函数是String原型对象上的方法。

    - 不使用捕获组括号
        ```
        var myString = "Hello 1 word. Sentence number 2.";
        var splits = myString.split(/\d/);
        console.log(splits);
        //[ "Hello ", " word. Sentence number ", "." ]
        ```

    - 使用捕获组括号
        ```
        var myString = "Hello 1 word. Sentence number 2.";
        var splits = myString.split(/(\d)/);
        console.log(splits);
        //[ "Hello ", "1", " word. Sentence number ", "2", "." ]
        ```
