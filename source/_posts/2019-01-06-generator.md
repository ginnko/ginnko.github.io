---
layout: post
title: generator函数初步学习
date: 2019-01-06
tag: javascript
---

看了许多材料后，感觉下面这三篇文章对于理解generator函数有非常大的帮助，囊括了常见用法、控制流程、循环操作等，理解的程度已经基本能满足现在的需求。

1. https://davidwalsh.name/es6-generators
2. https://davidwalsh.name/es6-generators-dive
3. https://davidwalsh.name/async-generators


下面的内容为阅读材料时随手做的笔记，未经过整理。

<!-- more -->

## 关于generator函数的基本概念

[这篇](https://davidwalsh.name/es6-generators)文章描述generator使用了`...so powerful for the future of JS.`。

万万没想到！之前对generator的认知仅停留在是个新的函数种类，没想到这里厉害。

到目前已经遇到两种衍生的技术了：

1. async函数

2. saga结合成的redux-saga。

原文：

>If you 've ever read anything about concurrency or threaded programming, you may have seen the term 'cooperative', which basically indicates that a process(in our case, a function) itself chooses when it will allow an interruption, so that it can cooperate with other code. This concept is contrasted with 'preemptive', which suggests that a process/function could be interrupted against its will.

就是说，并行编程有两种，在一段程序运行的时候可以被打断，执行另一段程序，主动打断的称为 **cooperative**，被动打断的称为 **preemptive**。Generator函数就是前者，它通过`yield`关键字，从generator函数内部打断执行过程，需要从外部恢复执行。

Generator函数并不仅仅是简单的打断、启动这种流程控制，它还允许向generator函数的内部和外部进行双向的数据传递。

### 创建iterator

调用一个generator函数便会创建一个iterator，并且创建的过程并不会执行generator函数中的内容。


## 关于generator的工作流程

以下内容出自[此处](https://github.com/gajus/gajus.com-blog/blob/master/posts/the-definitive-guide-to-the-javascript-generators/index.md)。

1. Advancing the Generator

使用next()函数

2. Pass a Value To the Iterator

在function*函数中使用yield关键字

3. Receive a Value From the Iterator

yield keyword can receive a value back from the iterator

```js
  const generatorFunction = function* () {
      console.log(yield);
  };
  
  const iterator = generatorFunction();
  
  iterator.next('foo');
  iterator.next('bar');
  
  // bar
```

好奇怪，为啥执行第一个next不输出`foo`，而是将这个值丢弃？

4. 内部执行顺序

执行顺序

```js
var foo, f;
foo = function* () {
  console.debug('generator 1');
  console.debug('yield 1', yield 'A');
  console.debug('generator 2');
  console.debug('yield 2', yield 'B');
  console.debug('generator 3');
}

f = foo();

console.log('tick 1');
console.log(f.next('a'));
console.log('tick 2');
console.log(f.next('b'));
console.log('tick 3');
console.log(f.next('c'));
console.log('tick 4');
console.log(f.next('d'));
```

执行结果

```js
tick 1
generator 1
{ value: 'A', done: false }
tick 2
yield 1 b
generator 2
{ value: 'B', done: false }
tick 3
yield 2 c
generator 3
{ value: undefined, done: true }
tick 4
{ value: undefined, done: true }
```

5. 使用`for...of...`循环

准则：
  1. The iteration will continue as long as done property is false.
  2. The for..of loop cannot be used in cases where you need to pass in values to the generator steps.
  3. The for..of loop will throw away the return value.

6. 委托yield

```js
let index;

const foo = function* () {
  yield 'foo';
  yield * bar();
};

const bar = function* () {
  yield 'bar';
  yield * baz();
};

const baz = function* () {
  yield 'baz';
}

for (index of foo()) {
  console.log(index);
}

// foo
// bar
// baz
```

委托一个generator到另一个generator相当于将目标generator的函数体导入到目的generator中，
相当于下面这种代码形式：

```js
let index;
 
const foo = function* () {
    yield 'foo';
    yield 'bar';
    yield 'baz';
};
 
for (index of foo()) {
    console.log(index);
}
 
// foo
// bar
// baz
```

7. 用yield控制异步流程

```js
const foo = (name, callback) => {
  setTimeout(() => {
    callback(name);
  }, 100);
};

const curry = (method, ...args) => {
  return (callback) => {
    args.push(callback);
    return method.apply({}, args);
  };
};

const controller = (generator) => {
  const iterator = generator();
  const advancer = (response) => {
    let state;
    state = iterator.next(response);
    if (!state.done) {
      state.value(advancer);
    }
  };
  advancer();
};

controller(function* () {
  const a = yield curry(foo, 'a');
  const b = yield curry(foo, 'b');
  const c = yield curry(foo, 'c');

  console.log(a, b, c);
});
```

8. 带错误处理的yield异步流程控制

```js
const foo = (parameters, callback) => {
  setTimeout(() => {
    callback(parameters);
  }, 100);
};

const curry = (method, ...args) => {
  return (callback) => {
    args.push(callback);
    return method.apply({}, args);
  };
};

const controller = (generator) => {
  const iterator = generator();
  const advancer = (response) => {
    if (response && response.error) {
      return iterator.throw(response.error);
    }

    const state = iterator.next(response);

    if (!state.done) {
      state.value(advancer);
    }
  }
  advancer();
};

controller(function* () {
  let a, b, c;
  try {
    a = yield curry(foo, 'a');
    b = yield curry(foo, {error: 'Something went wrong.'});
    c = yield curry(foo, 'c');
  } catch (e) {
    console.log(e);
  }

  console.log(a, b, c);
});
```

## generator的一个理解角度

以下部分出自[此处](https://goshakkk.name/javascript-generators-understanding-sample-use-cases/)

翻译其中的understanding部分：

当一个函数被调用，其中的命令会一个接一个的按顺序执行，函数能够通过`return`将一些值传给它的调用者。

```js
function regular() {
  doA();
  doB();
}
```

**`generators`的出现能够像对待一个`program`一样对待一个函数。**这个函数能够根据你定义的规则执行。所以我们可以将这个`generator`函数成为一个`program`。

为了执行一个`program`，我们需要一个`interpreter`，这个`interpreter`将“take the program in and run it”。

```js
interpreter(function* program() {

});
```

`yield`命令告诉一个`program`跳出generator调到`interpreter`中。`program`和`interpreter`的交流是双向的：

1. `program`可以向`interpreter`发送一些东西
2. `interpreter`也可以向`program`返回一些东西

基于上面的描述，可以从下面的角度描述下面这行代码：

`const a = yield b;`这行代码表示了我们如何发送一个命令b到`interpreter`然后把它的结果给a。`generator`函数将会暂停，直到`interpreter`告诉它才会继续执行后面的代码。

从redux-saga的角度阐述这个概念。

redux-saga没有将`side-effect`分散进多个action creator 和 reducer中，而是按逻辑组合多个片段行为，这个组合体被成为一个**saga**。

redux-saga中的`helper`函数，相当于在翻译`command`。

```js
function* welcomeSaga() {
  yield take('REGISTRATION_FINISHED');
  yield put(showWelcomePopup());
}

sagaMiddleware.run(welcomeSaga);
```

sagaMiddleware感觉就是一个interpreter。

