---
layout: post
title: javascript查漏补缺之十九 ——《闭包》
date: 2018-08-14
tag: javascript
---

[链接地址](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)

### 关于`lexical`的解释

> The word "lexical" refers to the fact that lexical scoping uses the location where a variable is declared within the source code to determine where that variable is available. Nested functions have access to variables declared in their outer scope.

### 关于闭包

> A closure is the combination of a function andn the lexical environment within which that function was declared.

**闭包具有封装和隐藏数据的优点。**

比如下面这个例子

```js
function makeAdder(x) {
  return function(y) {
    return x + y;
  };
}

var add5 = makeAdder(5);
var add10 = makeAdder(10);

console.log(add5(2));  // 7
console.log(add10(2)); // 12
```

`add5`和`add10`都是闭包，它们共享相同的函数体定义，但是存储了不同的词法环境。在`add5`的词法环境中，`x`是5；在`add10`的词法环境中，`x`是10。

再比如下面这个例子：

```js
var counter = (function() {
  var privateCounter = 0;
  function changeBy(val) {
    privateCounter += val;
  }
  return {
    increment: function() {
      changeBy(1);
    },
    decrement: function() {
      changeBy(-1);
    },
    value: function() {
      return privateCounter;
    }
  };   
})();

console.log(counter.value()); // logs 0
counter.increment();
counter.increment();
console.log(counter.value()); // logs 2
counter.decrement();
console.log(counter.value()); // logs 1
```

在前面的例子中，每一个闭包都有它自己的词法环境。然而，这里我们创建了一个单一的词法环境，这个环境被三个函数共享。这个共享的词法环境是在一个匿名函数的函数体中创建的，这个匿名函数创建完就执行了。这个词法环境包含两个私有条目：一个叫做`privateCounter`的变量以及一个叫做`changeBy`的函数。这两个私有条目都不可以从匿名函数外访问到。相反，它们只能被匿名函数返回的三个公共函数访问。

### 一个常见的闭包错误

```js
<p id="help">Helpful notes will appear here</p>
<p>E-mail: <input type="text" id="email" name="email"></p>
<p>Name: <input type="text" id="name" name="name"></p>
<p>Age: <input type="text" id="age" name="age"></p>

function showHelp(help) {
  document.getElementById('help').innerHTML = help;
}

function setupHelp() {
  var helpText = [
      {'id': 'email', 'help': 'Your e-mail address'},
      {'id': 'name', 'help': 'Your full name'},
      {'id': 'age', 'help': 'Your age (you must be over 16)'}
    ];

  for (var i = 0; i < helpText.length; i++) {
    var item = helpText[i];
    document.getElementById(item.id).onfocus = function() {
      showHelp(item.help);
    }
  }
}

setupHelp();
```

原因：传给`onfocus`的函数是闭包。循环创建了这三个闭包，但这三个共享了相同的唯一的词法环境,这个此法环境有一个改变中的变量`item.help`。**当`onfocus`的回调函数执行的时候，`item.help`的值才会确定。(只有函数执行的时候才会上溯搜寻参数)**而此时，循环已经结束，被三个闭包共享的变量对象`item`指向`helpText`数组的最后一项。

解决的关键是让这**三个闭包不共享相同的词法环境**。

1. 方法1

在循环词法环境和比包之间创建一个过渡的嵌套环境，立即执行保存了正确的参数，同时也给闭包创建了不同于其他两个比包的词法环境。

```js
function showHelp(help) {
  document.getElementById('help').innerHTML = help;
}

function makeHelpCallback(help) {
  return function() {
    showHelp(help);
  };
}

function setupHelp() {
  var helpText = [
      {'id': 'email', 'help': 'Your e-mail address'},
      {'id': 'name', 'help': 'Your full name'},
      {'id': 'age', 'help': 'Your age (you must be over 16)'}
    ];

  for (var i = 0; i < helpText.length; i++) {
    var item = helpText[i];
    document.getElementById(item.id).onfocus = makeHelpCallback(item.help);
  }
}

setupHelp();
```

2. 方法2

使用匿名闭包，和上面的方法本质相同，这种方法是把循环的整个过程通过匿名立即执行函数保存成一个区别于其它两个的词法环境。

```js
function showHelp(help) {
  document.getElementById('help').innerHTML = help;
}

function setupHelp() {
  var helpText = [
      {'id': 'email', 'help': 'Your e-mail address'},
      {'id': 'name', 'help': 'Your full name'},
      {'id': 'age', 'help': 'Your age (you must be over 16)'}
    ];

  for (var i = 0; i < helpText.length; i++) {
    (function() {
       var item = helpText[i];
       document.getElementById(item.id).onfocus = function() {
         showHelp(item.help);
       }
    })(); // Immediate event listener attachment with the current value of item (preserved until iteration).
  }
}

setupHelp();
```

3. 方法3

使用`let`，每个闭包绑定了块作用域的变量，这就不再需要闭包创建独立的词法环境了。

```js
function showHelp(help) {
  document.getElementById('help').innerHTML = help;
}

function setupHelp() {
  var helpText = [
      {'id': 'email', 'help': 'Your e-mail address'},
      {'id': 'name', 'help': 'Your full name'},
      {'id': 'age', 'help': 'Your age (you must be over 16)'}
    ];

  for (var i = 0; i < helpText.length; i++) {
    let item = helpText[i];
    document.getElementById(item.id).onfocus = function() {
      showHelp(item.help);
    }
  }
}

setupHelp();
```

4. 方法4

使用`forEach`替代`for`循环。

```js
function showHelp(help) {
  document.getElementById('help').innerHTML = help;
}

function setupHelp() {
  var helpText = [
      {'id': 'email', 'help': 'Your e-mail address'},
      {'id': 'name', 'help': 'Your full name'},
      {'id': 'age', 'help': 'Your age (you must be over 16)'}
    ];
  
  helpText.forEach(function(text) {
    document.getElementById(text.id).onfocus = function() {
      showHelp(text.help);
    }
  });
}

setupHelp();
```