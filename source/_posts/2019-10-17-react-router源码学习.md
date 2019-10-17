---
layout: post
title: react-router源码学习
date: 2019-10-17
tag: 
- history
- react-router
- react
---

学习react-router嘤...

<!-- more -->

### history

react-router以[history](https://github.com/ReactTraining/history)为基础库，所以先来看一下history的实现。

history主要是对html5提供的history API做了进一步封装，提供了针对浏览器和node.js环境的不同实现，这次主要是看了针对浏览器的实现。

源码不算难，这里记录一下本次的阅读顺序，方便日后参考：可以先看[参考资料2](https://github.com/fi3ework/blog/issues/21)，熟悉使用history实现路由的机制，再参考[参考资料1](https://juejin.im/post/5c049f23e51d455b5a4368bd#heading-7)进一步阅读源码。

看history库除了明白ku自身的实现原理，还能帮助复习下html5提供的history中关键的属性(length,state)、事件(popstate及触发条件)以及方法(go(),pushState(),replaceState())。关于原生history的属性文档，详见[参考资料3](https://developer.mozilla.org/zh-CN/docs/Web/API/History)或[参考资料5的第一部分](https://zhuanlan.zhihu.com/p/55837818)，方法文档详见[参考资料4](https://developer.mozilla.org/zh-CN/docs/Web/API/History_API)。

---

### 参考资料
1. https://juejin.im/post/5c049f23e51d455b5a4368bd#heading-7 (history源码解析，讲的很清楚)
2. https://github.com/fi3ework/blog/issues/21 (第一部分讲了前端路由的两种实现方式，第二部分讲了react-router的实现方式，可以先看下这个熟悉下history的大概实现原理)
3. https://developer.mozilla.org/zh-CN/docs/Web/API/History (mdn 关于history属性的文档)
4. https://developer.mozilla.org/zh-CN/docs/Web/API/History_API (mdn 关于history方法的文档)
5. https://zhuanlan.zhihu.com/p/55837818 (第一部分是关于history属性的总结)