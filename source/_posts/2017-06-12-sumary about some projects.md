---
layout: post
title: 几个小项目总结
date: 2017-06-12
tag:
- javascript
- function
- variable
---
## [Tic Tac Toe游戏](https://codepen.io/ginnko/full/dRPXGv/)
### 1. 变量作用域  
在计算器、番茄闹钟和这个项目中，都遇到了在for循环中用var定义一个循环变量后，即便出了for循环的范围依然可以使用的情况，现在返回头，从廖大角虫的[教程](http://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000/0014344993159773a464f34e1724700a6d5dd9e235ceb7c000)里再明确一下变量作用域。

- 今后在第一行都写上'use strict'，强制要求通过var申明变量。
- 若两个不同的函数各自中声明了相同名字的变量，该变量只在各自的函数体内起作用。
- JavaScript的变量作用域是函数内部，**在for循环等语句块中无法定义具有局部作用域的变量**。比如： 

		function foo() {
			for(var i=0; i<100; i++){
			//
			}
			i += 100; //在for循环外仍然可以使用i！！！
		}

		在语句块中使用let声明局部变量：
		function foo() {
			for(let i=0; i<100; i++){
			//
			}
			i += 100; //在for循环外仍然可以使用i！！！
		}

- 名字空间：

		//唯一的全局变量
		var MYAPP = {};
		//其他变量
		MYAPP.name = 'myapp';
		MYAPP.version = 1.0;
		//其它函数
		MYAPP.foo = function(){
			return "foo";
		};

### 2. 布尔值
false不能用具体某个数字表示：-1 ！== false

### 3. 遇到的具体问题
- Tag div 没有disable这个属性
- 当使用jQuery时，开关click事件可以使用on， off函数，需要一个callback函数。当依次执行关闭、自动开启、手动开启操作时，为避免多次开启，采用下面的方法。[实例](https://codepen.io/ginnko/pen/dRPXGv)：place函数、another函数以及reset函数。

		$(".pit").off("click");//prevent from being disturbed by the same execution in another function
 		$(".pit").on("click", place);

- 数组的push（）函数，如果参数是一个数组，那么结果将是一个二维数组，如果改动这个二维数组中的元素，则原来那个作为参数传入的数组中的相应元素也会被改动，push（）函数执行的是浅复制操作。



## [番茄时钟](https://codepen.io/ginnko/full/EXxXJw/)
### 1. 关于延时函数
这个项目里主要用了setInterval()和clearInterval()两个函数，必须成对出现，可**反复延迟执行**传入setInterval的函数直到遇到清除函数。把setInterval()的返回值赋给某个变量，入下列中的id作为唯一标识，并使用id传入clearInterval()停止循环过程。以下代码出自此项目

	function countSession(){
    if(timeSession < 0){
      st = 1;
      clearInterval(id);//停止循环过程，但此清除函数不是针对下面的SetInterval()函数，详见项目中的代码
      set();
      optimize(timeRelax);
      timeRelax--;
      i++;
      id = setInterval(countRelax, 1000);//每延迟一秒执行一次countRelax函数
      $("circle").attr("fill", "#00FF00");
      $("#progress").text("break");
      flag = true;
 	}else{
    var arr = calculate(timeSession);
    $("#show").text(arr[0] + ":" + arr[1]);
    $("path").attr("stroke-dasharray", "" + i*stepSession + "," + (251.2-i*stepSession));
    timeSession--;
    i++;
    st = 0;
    flag = true;
    }
}

### 2. 关于svg标签
svg标签的使用点击[此处](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Tutorial/Paths)以下代码中，circle标签用来表示大圆，path标签用来表示随着时间变化的外环，text标签用来表示圆内的说明。这个项目参考了[web-tiki](https://codepen.io/web-tiki/full/qEGvMN/)的项目，感谢web-tiki大神。

	<svg role="button" id="svg" viewbox="0 0 100 100">
    	<circle cx="50" cy="50" r="45" fill="#FF4500"/>
    	<path fill="none" stroke-linecap="round" stroke-width="5" stroke="#fff" stroke-dasharray="0 251.2"
          d="M50 10
             a 40 40 0 0 1 0 80
             a 40 40 0 0 1 0 -80"/>
    	<text id="show" x="50" y="50" text-anchor="middle" dy="7" font-size="15">START</text>
    	<text id="progress" x="50" y="75" text-anchor="middle" font-size="10"></text>
    </svg>

### 3. role属性
role属性位于某个标签内，当拥有某个属性值后会使所在标签具有属性值得某些功能。比如上述代码中的svg标签，role="button"是的svg标签拥有了click事件。

## [JavaScrip计算器](https://codepen.io/ginnko/full/EXxXJw/)
### 1. RegExp
这个项目中使用了RegExp的exec()函数，详见这篇[总结]()。
### 2. this & $(this)
this:表示一个html元素， this.title  
$(this):表示一个jQuery对象， $(this).attr("title")





