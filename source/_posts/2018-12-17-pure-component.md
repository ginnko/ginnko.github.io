---
layout: post
title: React纯组件
date: 2018-12-17
tag: React
---

## 参考资料

1. https://reactjs.org/docs/react-api.html#reactpurecomponent
2. https://segmentfault.com/a/1190000014979065
3. https://www.zcfy.cc/article/why-and-how-to-use-purecomponent-in-react-js-60devs
4. https://juejin.im/post/5b1caceb5188257d63226743

## 主要内容

React为了提升性能，产生了纯组件。纯组件的子组件也是纯组件。

关于纯组件自己的理解：类似于纯函数，只要是一样的state和props，就必定是同样的结果，可以不用再render，直接使用上次的结果。

`React.PureComponent`通过减少应用中的渲染次数提升性能可以避免手写检查代码。

纯组件忽略重新渲染时，会影响它和它的所有子元素，所以出组件的最佳使用场景就是展示组件，既没有子组件，也没有依赖应用的全局状态。

解决纯组件的初始化问题：

1. default Props
2. 向子组件中传入的函数传引用，而不是函数本身，否则会重新创建函数对象

不要在render中创建一个新函数或对象或方法。任何包含子元素的组件，shallowEqual检查总会返回false。

## 使用细节

这个要补充一下