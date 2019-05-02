---
layout: post
title: javascript查漏补缺之五——《数字和日期》
date: 2018-07-07
tag: javascript
---

[译文地址](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Numbers_and_dates)

1. js中数字的范围和类型

  范围： -(2^53 - 1) ~ (2^53 - 1)

  类型： 普通数字， +Infinity， -Infinity， NaN

<!-- more -->

2. 数字对象属性和方法的引用

  属性包括最大值、最小值、非数、正无穷、负无穷、比较值、最大安全数、最小安全数。

  ![数字对象属性](/images/js/1.png)

  方法调用的时候要使用`Number.`的形式，忘记这个在哪儿看到的了，总之就是有好处就对了。

  ![数字对象的方法](/images/js/2.png)

3. 实现精确四舍五入保留小数位数的方法

    1. `Number.prototype.toFixed()`：这个写在数字对象原型上的方法就可以做到，只不过会返回字符串形式的结果。（负数因为操作符的优先级问题，在不加括号的情况下不会返回字符串形式）

    2. 使用这种方法保留小数后几（3）位：

        ```
        const rounded = Math.round(output * 1000) / 1000;
        ```

4. Math对象的方法

    Math对象不可以自定义。

    1. `floor（）`往下舍;

    2. `ceil（）`往上进;

    3. `round（）`四舍五入，只可保证整数部分是精确的;

    4. `trunc（）`截掉数字的小数部分。

    5. `random（）`返回一个浮点,  伪随机数在范围[0，1)， **不能使用它们来处理有关安全的事情。使用Web Crypto API 来代替, 和更精确的window.crypto.getRandomValues() 方法。**

        1. 得到一个大于等于0，小于1之间的随机数

        ```
        function getRandom() {
          return Math.random();
        }
        ```

        2. 得到一个两数之间的随机数

        ```
        function getRandomArbitrary(min, max) {
          return Math.random() * (max - min) + min;
        }
        ```

        3. 得到一个两数之间的随机整数

        ```
        function getRandomInt(min, max) {
          min = Math.ceil(min);
          max = Math.floor(max);
          return Math.floor(Math.random() * (max - min)) + min; //The maximum is exclusive and the minimum is inclusive
        }
        ```

        4. 得到一个两数之间的随机整数，包括两个数在内

        ```
        function getRandomIntInclusive(min, max) {
          min = Math.ceil(min);
          max = Math.floor(max);
          return Math.floor(Math.random() * (max - min + 1)) + min; //The maximum is inclusive and the minimum is inclusive 
        }
        ```

5. 日期对象

    1. 如果不使用*new*调用`Date()`,则返回的是当前日期时间的字符串。

    2. 使用*new*调用`new Date()`。使用这种方式创建的时间对象都可以使用Date对象的相关方法进行操作。

        1. 如果不传入参数，则返回当前日期时间的时间对象;

        2. 传入`1995, 11, 25, 9, 30, 0`表示传入的是1995年11月25日9点30分0秒。
    
    3. Date对象的常用方法

        1. `Date.prototype.getDate()`: 根据本地时间返回指定日期对象的月份中的第几天（1-31）。

        2. `Date.prototype.getDay()`: 根据本地时间返回指定日期对象的星期中的第几天（0-6）。

        3. `Date.prototype.getFullYear()`: 根据本地时间返回指定日期对象的年份。

        4. `Date.prototype.getHours()`: 根据本地时间返回指定日期对象的小时（0-23）。

        5. `Date.prototype.getSeconds()`: 根据本地时间返回指定日期对象的秒数（0-59）。

        6. `Date.prototype.getMinutes()`: 根据本地时间返回指定日期对象的分钟（0-59）。

        7. `Date.prototype.getMonth()`: 根据本地时间返回指定日期对象的月份（0-11）。

        8. `Date.prototype.toDateString()`: 返回一个人类可读的日期字符串，比如 *"Mon Jul 09 2018"* 。

        9. `Date.prototype.toLocalDateString()`： 返回一个 *"2018/7/9"* 这样形式的日期字符串。术语叫**该字符串格式与系统设置的地区关联**。

        10. `Date.prototype.toLocaleString()`:
        返回一个 *"2018/7/9 上午10:48:34"* 这样形式的日期字符串。术语叫**该字符串与系统设置的地区关联**。

        11. `Date.prototype.toLocaleTimeString()`:返回一个 *"上午10:48:34"* 这样形式的时间字符串。

        12. `Date.prototype.toString()`: 返回 *"Mon Jul 09 2018 10:48:34 GMT+0800 (CST)"* 这种形式的字符串。感觉上面那种 *toLocale* 形式的更有用些。



