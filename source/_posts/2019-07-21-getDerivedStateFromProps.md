---
layout: post
title: 生命周期函数getDerivedStateFromProps的使用
date: 2019-07-021
tag: 
- React
- 生命周期函数
---

以前写的代码，遇到`组件内的状态即来自组件外部props，也来自组件内部的state更新`这种情况，都是在生命周期函数`componentDidUpdate`中解决的，在其他地方看到`getDerivedStateFromProps`函数的使用，突然发现这个函数才是真正为这个场景而生的。

<!-- more -->

## 这个函数使用的场景是

**组件内的状态即来自组件外部props，也来自组件内部的state更新。**

这个函数是静态函数，在`render()`之前执行，`setState()`和`forceUpstate()`会触发`getDerivedStateFromProps`函数。

## 注意使用条件

1. 在使用此生命周期时，要注意把传入的 prop 值和之前传入的 prop 进行比较。
2. 因为这个生命周期是静态方法，同时要保持它是纯函数，不要产生副作用。


## 参考

1. https://juejin.im/post/5c3ad49be51d45521053fde0
2. https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#anti-pattern-unconditionally-copying-props-to-state