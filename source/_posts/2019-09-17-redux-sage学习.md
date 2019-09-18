---
layout: post
title: redux-saga学习
date: 2019-09-17
tag: 
- redux-saga
---

redux-saga学习中。

<!-- more -->

### redux-saga中的重要概念

1. saga

官网中关于saga有这样的描述:

> a saga is like a separate thread in your application that's solely responsible for side effects. 

[资料2](https://blog.couchbase.com/saga-pattern-implement-business-transactions-using-microservices-part/)和[资料3](https://blog.couchbase.com/saga-pattern-implement-business-transactions-using-microservices-part-2/)中描述了在微服务中的saga模型,我的理解是`saga`像是一个单位，用于完成一个完整功能,这个完整功能依赖多个有先后依存顺序的事务的执行。文中描述了两种`saga`的实现，一种是每一个事务顺序执行，当前事务执行完毕向后一个事务发出事件；另一种实现是建立一个saga的协调中心，协调中心负责调度各个事务。

感觉上面的描述没有指出saga的重点，[资料4](https://zhuanlan.zhihu.com/p/41661402)描述saga的一个特点：

>saga需要一个“从中间开始执行”的模式。

>本质上其实是事务的异常中止和恢复。解决起来也并不复杂：随时保存saga的当前状态，并在异常结束之后恢复状态。



---

### 参考资料

1. https://redux-saga.js.org/
2. https://blog.couchbase.com/saga-pattern-implement-business-transactions-using-microservices-part/
3. https://blog.couchbase.com/saga-pattern-implement-business-transactions-using-microservices-part-2/
4. https://zhuanlan.zhihu.com/p/41661402

