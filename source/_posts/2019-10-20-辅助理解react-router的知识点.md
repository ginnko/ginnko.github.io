---
layout: post
title: 辅助理解react-router的知识点
date: 2019-10-20
tag: 
- history
- react-router
- react
---

本文主要是记录html5原生history的属性和方法以及React的Context、Provider、Consumer、Children、HOC相关概念。

<!-- more -->

## 使用html5的history

### popstate事件

下面摘自[参考资料3](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/onpopstate)。

每当处于激活状态的历史记录条目发生变化时，popstate事件就会在对应的window对象上触发。如果当前处于激活状态的历史记录条目是由history.pushState()方法创建，或者由histoary.replaceState()方法修改过，则popstate事件对象的state属性包含了这个历史条目的state对象的一个拷贝。

**调用history.pushState()或者history.replaceState()不会触发popstate事件。popstate事件只会在浏览器某些行为下触发，比如点击后退、前进按钮，或者调用history.back()、history.forward()、history.go()方法。**

当网页加载时，各浏览器对popstate事件是否触发会有不同的表现，Chrome和Safari会触发popstate事件，而Firefox不会。

即便进入了那些非pushState和replaceState方法作用过的没有state对象关联的那些网页，popstate事件也仍然会被触发。

---

### pushState()

[参考资料2](https://developer.mozilla.org/en-US/docs/Web/API/History_API/Working_with_the_History_API)中给出的例子： 

```js
  let stateObj = {
      foo: "bar",
  };

  history.pushState(stateObj, "page 2", "bar.html");
```

调用history.pushState()将导致地址栏的地址变为`http://mozilla.org/bar.html`，但是浏览器不会加载`bar.html`，甚至不会检查`bar.html`是否存在。

假设现在用户又访问了 `http://google.com`，然后点击了返回按钮。此时，地址栏将显示 `http://mozilla.org/bar.html`，history.state 中包含了 stateObj 的一份拷贝。页面此时展现为bar.html。且因为页面被重新加载了，所以popstate事件将不会被触发。（这里有个问题：根据上面的描述，这里说popstate不会触发是FF不会触发？Chrome还是会触发吧？）

如果我们再次点击返回按钮，页面URL会变为`http://mozilla.org/foo.html`，文档对象document会触发另外一个 popstate 事件，这一次的事件对象state object为null。 这里也一样，返回并不改变文档的内容，尽管文档在接收 popstate 事件时可能会改变自己的内容，其内容仍与之前的展现一致。

pushState()需要三个参数：状态、标题（忽略）、url。

1. 状态对象：是一个js的普通对象，通过pushState()创建新的历史条目。无论什么时候用户导航到新的状态，popstate时间会被触发，且该时间的state属性包含该历史记录条目状态对象的副本。

### replaceState()

和pushState()方法非常相似，区别在于replaceState()是修修改了当前的历史记录项而不是新建一个，但是这并不会阻止其在全局浏览器历史中创建一个新的历史记录项。

这个函数的使用场景在于为了响应用户操作，想要更新状态对象或者当前历史记录的URL。

---

## React部分

关于Context、Provider、Consumer可以看[参考资料4](https://juejin.im/post/5a90e0545188257a63112977)和[参考资料5](https://reactjs.org/docs/context.html)，目前先知道怎么用，用来帮助理解react-router，后续看react原理和源码的时候再来深入。



### 参考资料

1. https://developer.mozilla.org/zh-CN/docs/Web/API/History_API
2. https://developer.mozilla.org/en-US/docs/Web/API/History_API/Working_with_the_History_API
3. https://developer.mozilla.org/zh-CN/docs/Web/API/Window/onpopstate
4. https://juejin.im/post/5a90e0545188257a63112977
5. https://reactjs.org/docs/context.html