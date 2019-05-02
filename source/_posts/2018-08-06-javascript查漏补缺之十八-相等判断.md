---
layout: post
title: javascript查漏补缺之十八 ——《相等判断》
date: 2018-08-06
tag: javascript
---

[链接地址](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness)

### loose equality

![==转换](/images/js/7.png)

1. `Number`，`String`，`Boolean`类型在和`Number`，`String`，`Boolean`类型比较的时候，如果两个不是相同类型，会将两个比较的值都转成数字后再进行比较。转换时，相当于使用`+`。

2. `Number`，`String`，`Boolean`类型在和`Object`类型比较的时候，会把`Object`类型转换成基础类型。转换过程会调用`A.toString`和`A.valueOf`这两个方法，不同类型进行转换时，这两个方法调用的顺序不同。转换顺序详见[规范](http://ecma-international.org/ecma-262/5.1/#sec-8.12.8)。

    *此处要加上几个关于对象转换成基本值的示例，一步步按转换步骤写。暂时没有找到...明天看下犀牛书的说明好了。*

    犀牛书原文：

    >"+"和“=”应用的对象到原始值的转换包含日期对象的一种特殊情形。日期类是js语言核心中唯一的预先定义类型，它定义了有意义的向字符串和数字类型的转换。对于所有非日期的对象来说，**对象到原始值的转换基本上是对象到数字的转换**`（首先调用valueof()）`，日期对象则使用对象到字符串的转换模式，然而，这里的转换和上文讲述的并不完全一致：通过valueOf或toString返回的原始值将被直接使用，而不会被强制转换为数字或字符串。

    > 对象转换为数字的细节解释了为什么空数组会被转换为数字0以及为什么具有单个元素的数组同样会被转换成一个数字。数组集成了默认的valueOf方法，这个方法返回一个对象而不是一个原始值，因此，数组到数字的转换则调用toString()方法。空数组转换成为空字符串，空字符串转为数字0。含有一个元素的数组转换为字符串的结果和这个元素转换字符串的结果一样。如果数组只包含一个数字元素，这个数字转换为字符串，再转换为数字。

### 同值相等判断

`Object.is()`除下面的特殊情况外，基本类似`===`的比较结果。

**通常要避免使用这个函数，比较NaN时，可以使用isNaN函数。**

```js
Object.is(NaN, NaN) // true

Object.is(+0, -0) // false

Object.is(+0, 0) // true

Object.is(-0, 0) // false
```

### [严格相等比较算法](http://ecma-international.org/ecma-262/5.1/#sec-11.9.6)

The comparison x === y, where x and y are values, produces true or false. Such a comparison is performed as follows:

  1. If Type(x) is different from Type(y), return false.
  2. If Type(x) is Undefined, return true.
  3. If Type(x) is Null, return true.
  4. If Type(x) is Number, then
      1. If x is NaN, return false.
      2. If y is NaN, return false.
      3. If x is the same Number value as y, return true.
      4. If x is +0 and y is −0, return true.
      5. If x is −0 and y is +0, return true.
      6. Return false.
  5. If Type(x) is String, then return true if x and y are exactly the same sequence of characters (same length and same characters in corresponding positions); otherwise, return false.
  6. If Type(x) is Boolean, return true if x and y are both true or both false; otherwise, return false.
  7. Return true if x and y refer to the same object. Otherwise, return false.

### [绝对相等比较算法](http://ecma-international.org/ecma-262/5.1/#sec-11.9.3)

共10个steps，详见链接。下面是一些注意：

1. Given the above definition of equality:

    - String comparison can be forced by: "" + a == "" + b.
    - Numeric comparison can be forced by: +a == +b.
    - Boolean comparison can be forced by: !a == !b.

2. The equality operator is not always transitive. For example, there might be two distinct String objects, each representing the same String value; each String object would be considered equal to the String value by the == operator, but the two String objects would not be equal to each other. For Example:

    - new String("a") == "a" and "a" == new String("a")are both true.
    - new String("a") == new String("a") is false.
