---
layout: post
title: react-redux源码学习
date: 2019-10-22
tag: 
- react-redux
- react
- redux
---

暂停

新的react-router大量使用了react的hook，本文主要参考[参考资料1](http://xzfyu.com/2018/07/08/react/react%E7%9B%B8%E5%85%B3/react-redux%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/)和[参考资料4](https://juejin.im/post/5c0e2a616fb9a049a9798997)，这两篇还比较新。

<!-- more -->

### context

[参考资料2](https://juejin.im/post/5b32f145f265da596a3682b4)在讲context的时候说道有使用`getChildContext`函数，但是在react文档[参考资料3](https://reactjs.org/docs/legacy-context.html#updating-context)中有下面的描述：

>Don’t do it.

>React has an API to update context, but it is fundamentally broken and you should not use it.

有点尴尬。[参考资料2](https://juejin.im/post/5b32f145f265da596a3682b4)中描述了不建议使用的原因，就是：context里面的数据能被随意地接触修改，导致程序运行的不可预料。而redux却能做到修改数据的行为可预测可追踪。


### 参考资料

1. http://xzfyu.com/2018/07/08/react/react%E7%9B%B8%E5%85%B3/react-redux%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/
2. https://juejin.im/post/5b32f145f265da596a3682b4
3. https://reactjs.org/docs/legacy-context.html#updating-context
4. https://juejin.im/post/5c0e2a616fb9a049a9798997
5. https://codar.club/blogs/analysis-of-connect-method-of-react-redux.html