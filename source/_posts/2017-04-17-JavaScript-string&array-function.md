---
layout: post
title: JavaScript常用对数组和字符串的操作方法
date: 2017-04-17
tag: javascript
---

### 对字符串的操作方法

|函数名|作用|用法|
|:---:|:---:|:---:|
|toUpperCase()|把一个字符串全部变为大写并返回一个新的字符串|str.toUpperCase();|
|toLowerCase()|把一个字符串全部变为小写并返回一个新的字符串|str.toLowerCase();|
|indexOf()|搜索并返回指定字符串出现的位置|str.index('Hello');|
|substring()|返回指定索引区间的字符串|str.substring(0, 2);|
|length|返回字符串的长度|str.length;|

### 对数组的操作方法

|函数名|作用|用法|
|:---:|:---:|:---:|
|length|返回数组的长度|arr.length|
|indexOf()|搜索并返回指定元素的位置|arr.indexOf(30)|
|slice()|截取数组的一部分并返回新的数组,参数为空时可以复制一个数组|arr.slice(0, 2);|
|push()|向数组末尾添加元素，返回新数组长度|arr.push(1, 2);|
|pop()|删除数组末尾最后一个元素，返回删除的元素|arr.pop();|
|unshift()|向数组头部添加原素，返回新数组的长度|arr.unshift(1, 2);|
|shift()|删除数组第一个元素，并返回被删除的元素|arr.shift();|
|sort()|对数组进行排序，改变数组本身|arr.sort();|
|reverse()|反转数组元素的顺序，改变数组本身|arr.reverse();|
|splice()|指定位置删除若干元素并添加若干元素(位置，删除元素个数，添加的元素)|arr.splice(2, 0, 'Google');|
|concat()|连接两个数组，并返回链接后的数组|arr1.concat(arr2);|
|join()|用指定字符串连接一个数组中的元素，并返回连接后的数组|arr.join('-');|

