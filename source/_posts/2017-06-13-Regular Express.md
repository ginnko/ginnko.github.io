---
layout: post
title: RegExp
date: 2017-06-13
tag: RegExp
---
### 1. 参考资料
1. [Mozilla MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp)：详细的RegExp说明和使用
2. [在线神器](http://regexr.com/)：方便的在线测试和快速构造
3. [廖大角虫教程](http://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000/001434499503920bb7b42ff6627420da2ceae4babf6c4f2000)：应用和主要函数一览无余  

<!-- more -->

### 2. 函数
1. [exec()函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/exec)  
**不要把正则表达式字面量（或者正则表达式构造器）放在 while 条件表达式里。由于每次迭代时 lastIndex 的属性都被重置，如果匹配，将会造成一个死循环。**

		var myRe = /ab*/g;
		var str = 'abbcdefabh';
		var myArray;
		while ((myArray = myRe.exec(str)) !== null) {
		  var msg = 'Found ' + myArray[0] + '. ';
		  msg += 'Next match starts at ' + myRe.lastIndex;
		  console.log(msg);
		}
        //Found abb. Next match starts at 3
        //Found ab. Next match starts at 9







