---
layout: post
title: react-router常用组件原理学习
date: 2019-10-17
tag: 
- history
- react-router
- react
---

react-router的功能实现感觉可以分为两部分：

1. html5的history，这个api提供了浏览器地址栏的地址变化的一系列控制。
2. 路由和组件的匹配，react-router主要是实现了这一部分，涉及context、provider、高阶组件等。

react-redux中应该也会涉及到2中说的这些概念和用法，但猜测应该更侧重于组件更新的优化上。这次先试探一波，除了路由的实现外，熟悉下这些概念。

p.s. 这段写于看完下面说的几部分之后

看完之后，感觉react-router的实现原理并不难，需要着重掌握的反而是history和React中的几个概念：Context、Provider、Consumer、Children、HOC。这几个概念还真不熟悉，所以决定再起一篇，用来着重记录。

<!-- more -->

### history

react-router以[history](https://github.com/ReactTraining/history)为基础库，所以先来看一下history的实现。

history主要是对html5提供的history API做了进一步封装，提供了针对浏览器和node.js环境的不同实现，这次主要是看了针对浏览器的实现。

源码不算难，这里记录一下本次的阅读顺序，方便日后参考：可以先看[参考资料2](https://github.com/fi3ework/blog/issues/21)，熟悉使用history实现路由的机制，再参考[参考资料1](https://juejin.im/post/5c049f23e51d455b5a4368bd#heading-7)进一步阅读源码。

看history库除了明白库自身的实现原理，还能帮助复习下html5提供的history中关键的属性(length,state)、事件(popstate及触发条件)以及方法(go(),pushState(),replaceState())。关于原生history的属性文档，详见[参考资料3](https://developer.mozilla.org/zh-CN/docs/Web/API/History)或[参考资料5的第一部分](https://zhuanlan.zhihu.com/p/55837818)，方法文档详见[参考资料4](https://developer.mozilla.org/zh-CN/docs/Web/API/History_API)。

关于这一部分，在下一篇中有详细记录。

---

### React-Router

这次主要看react-router和react-router-dom两个文件夹的文件代码。[参考资料2](https://github.com/fi3ework/blog/issues/21)中有讲：

>react-router 负责通用的路由逻辑，react-router-dom 负责浏览器的路由管理

但目前着实没感觉出来，手动扶额...感觉[参考资料6](https://reacttraining.com/react-router/web/api/Route)说的更靠谱一些，react-router提供了底层的实现，react-router-dom在其基础上封装了更常用的方法。

---

### Router.js

这个文件位于react-router中，源码能看明白，就不写了，这里想写一下相关的内容：

1. Router这个组件的作用

  看过源码后，感觉Router是为创建Context，提供Provider存在的。Router是一个很基础的组件，后面要看的BrowserRouter就是在Router的基础上构造的。具体的用途就是包裹其他组件，方便被包裹的组件获取参数，以及这些参数有更新时，控制被包括组件更新。

2. context

  首先，context这个东西的存在就是为了避免嵌套组件层层传参数的繁琐。相当于一个全局的池子，其中的属性，满足`条件`的组件都可以访问，而不需要层层传递。

  React有这样一个[api](https://reactjs.org/docs/context.html#reactcreatecontext)：`React.createContext`，用来创建一个context，react-router没有直接使用这个api，而是用了一个pollifill，功能应该是一样的。

3. Provider

  Provider是一个组件，通过上面的`React.createContext`创建的context会返回一个Provider。这个组件就是上面2中所说的条件，被这个Provider组件包裹后，才能访问context中的属性。同时，Provider还负责当有属性更新时，驱动子组件更新。[文档](https://reactjs.org/docs/context.html#contextprovider)中还说道，使用这种方式的驱动更新不受`shouldComponentUpdate`的干预。

---

### BrowserRouter.js

这个组件就是把上面说的history库的一个实例对象传入Router组件中，创建所需的全局池子，更新方式以及注入路由相关的属性。

源码很简单，不写。

---

### Route.js

这个组件用来实现路由和业务组件的匹配。

使用Route组件的方式是这样的：

```js
<Router>
  <div>
    <Route exact path="/">
      <Home />
    </Route>
    <Route path="/news">
      <NewsFeed />
    </Route>
  </div>
</Router>
```
传入一个path属性，如果当前url和path匹配，就渲染其包裹的组件。

Route源码就是是做了上述的两件事，第一件是判断当前url和path是否匹配，第二件是处理渲染的组件。但实现感觉有点乱...[参考资料6](https://reacttraining.com/react-router/web/api/Route)中关于这一部分的代码有详细的说明。

---

### Switch.js

这个组件的作用是当有多个path属性相近(比如都包含'/')的时候，多个Route组件都会渲染，而Switch做的事情是使用循环迭代其包裹的子组件，将匹配成功的第一个Route赋值给中间变量，最后再渲染这个。有一个疑问是，为何源码没有在找到第一个满足条件的Route后就停止迭代？

---

### Link.js

Link的源码同样不难，本质是调用了history.push()。

如果没有传component，Link默认使用LinkAnchor。LinkAnchor底层就是a标签，一开始没细看，有一点让我很困惑，就是a标签点击后同样会触发url的改变，这样就会脱离history的管理。然而实际上，LinkAnchor中有这样一行代码：

```js
  event.preventDefault();
  navigate();
```

第一行代码阻止了a标签被点击时的默认行为，路由的管理由history接手。

---

### 参考资料
1. https://juejin.im/post/5c049f23e51d455b5a4368bd#heading-7 (history源码解析，讲的很清楚)
2. https://github.com/fi3ework/blog/issues/21 (第一部分讲了前端路由的两种实现方式，第二部分讲了react-router的实现方式，可以先看下这个熟悉下history的大概实现原理)
3. https://developer.mozilla.org/zh-CN/docs/Web/API/History (mdn 关于history属性的文档)
4. https://developer.mozilla.org/zh-CN/docs/Web/API/History_API (mdn 关于history方法的文档)
5. https://zhuanlan.zhihu.com/p/55837818 (第一部分是关于history属性的总结)
6. https://reacttraining.com/react-router/web/api/Route (分析了代码的作用，但感觉么有灵魂)