---
layout: post
title: redux-saga原理学习
date: 2019-09-28
tag: 
- redux-saga
---
这几天看redux-saga的实现原理，英文资料几乎没搜到有讲redux-saga的原理的，基本都是对saga这个概念做了解释。中文资料倒是很多对redux-saga进行原理解释的，看了好多篇，最后留下两篇，记录在文尾的参考资料中。后面再翻看redux-saga的实现原理，先看[参考资料2](https://www.twblogs.net/a/5d037e74bd9eee487be98be1)，再看[参考资料1](https://zhuanlan.zhihu.com/p/30098155)。

<!-- more -->

---

### effects

从redux-saga文档的学习中得知，effects就是js的纯对象，是发送给middleware的指令，这些指令描述了middleware应该进行的操作信息，middleware实际执行完毕，将结果返回给Generator。

[参考资料2](https://www.twblogs.net/a/5d037e74bd9eee487be98be1)中讲到这样做的好处是：

  - 集中处理异步操作，更好的表达复杂的流程控制
  - 保证action是个纯粹的js对象，风格保持统一
  - 声明式指令，无需在generator中立即执行，只需通知middleware让其执行；借助generator的next方法，向外部暴露每一个步骤

[参考资料2](https://www.twblogs.net/a/5d037e74bd9eee487be98be1)中总结的常用effect指令：

  - put，用来命令middleware向store发起一个action
  - take，用来命令middleware在store上等待指定的action。在发起与pattern匹配的action之前，Generator将暂停
  - call，用来命令middleware以参数args调用函数fn
  - fork，用来命令middleware以非阻塞的形式执行fn
  - race，用来命令middleware在多个effect间运行，相当于Promise.race
  - all，用来命令middleware并行地运行多个effect，并等待它们全部完成，相当于Promise.all

### Saga中的辅助函数

看过那么多篇，感觉[参考资料2](https://www.twblogs.net/a/5d037e74bd9eee487be98be1)是唯一对这个做清晰分类总结的，思路是真的牛逼。辅助函数本质是基于各种effect实现的。

  - takeEvery(take + fork)
  - takeLatest(take + fork + cancel)
  - takeLeading(take + call)
  - throttle(take + fork + delay)
  - debounce(take + fork + delay)
  - retry(call + delay)

### redux-saga原理

下面这张图片出自[参考资料2](https://www.twblogs.net/a/5d037e74bd9eee487be98be1)。

![redux-saga原理](/images/redux_saga/redux-saga-mechanism.png)

redux-saga的核心包括两部分：channel和task。

channel用来触发action，以及注册action的回调函数。

task是Generator函数实际执行的函数，结合递归和Generator的yield以及next完成流程控制。

下面代码实现了这两个核心模块，出自[参考资料1](https://zhuanlan.zhihu.com/p/30098155)：

```js
function channel() {
  let taker;

  function take(cb) {
    taker = cb;
  }

  function put(input) {
    if (taker) {
      const tempTaker = taker;
      taker = null;
      tempTaker(input);
    }
  }

  return {
    put,
    take,
  };
}

const chan = channel();

function take() {
  return {
    type: 'take'
  };
}

function* mainSaga() {
  const action = yield take();
  console.log(action);
}

function runTakeEffect(effect, cb) {
  chan.take(input => {
    cb(input);
  });
}

function task(iterator) {
  const iter = iterator();
  function next(args) {
    const result = iter.next(args);
    if (!result.done) {
      const effect = result.value;
      if (effect.type === 'take') {
        runTakeEffect(result.value, next);
      }
    }
  }
  next();
}

task(mainSaga);

let i = 0;
$btn.addEventListener('click', () => {
  const action =`action data${i++}`;
  chan.put(action);
}, false);

```

---
### 参考资料

1. https://zhuanlan.zhihu.com/p/30098155 (这个写的很好，九浅一深，手动滑稽)
2. https://www.twblogs.net/a/5d037e74bd9eee487be98be1 （这个讲的特别清楚，赞！）