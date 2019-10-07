---
layout: post
title: redux-saga源码学习
date: 2019-10-04
tag: 
- redux-saga
---

上一篇整明白了redux-saga的原理，现在来具体看看代码实现。

讲redux-saga源码的资料真是少的可怜，英文完全没搜到...中文就搜到两篇有价值的，第二篇对整个redux-saga的代码实现做了比较详细的描述。

<!-- more -->

---
### 辅助理解的变量名

1. cont

  continuation 缩写，一般用于表示 Task / MainTask / ForkQueue 的后继

2. cb

  callback 缩写 或是 currCb 应该是 currentCallback 的缩写。一般用于 effect 的后继/回调函数

3. next

  就是前边的递归函数

### 主线

1. 关联redux和redux-saga，使用redux中的applyMiddleware将redux中的dispatch注入到redux-saga的sagaMiddleware中

redux的applyMiddleware文件中的这几行代码完成了主要工作：

```js
const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    const chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch)
```

下面是使用redux和redux-saga关联的实际使用示例代码：

```js
  import { createStore, applyMiddleware } from 'redux'
  import createSagaMiddleware from 'redux-saga'

  // ...
  import { helloSaga } from './sagas'

  const sagaMiddleware = createSagaMiddleware()
  const store = createStore(
    reducer,
    applyMiddleware(sagaMiddleware)
  )
  sagaMiddleware.run(helloSaga)

  const action = type => store.dispatch({type})
```

关联后，控制权移入sagaMiddleware函数中，接入redux-saga最终要的一行代码是：`channel.put(action)`，这行代码背后的含义是当出现action的操作时，会执行提前绑定好的回调函数。

```js
  function sagaMiddleware({ getState, dispatch }) {
    boundRunSaga = runSaga.bind(null, {
      ...options,
      context,
      channel,
      dispatch,
      getState,
      sagaMonitor,
    })
    // 如果仅redux-saga和redux一起使用
    // 那么此处的next等于dispatch
    return next => action => {
      if (sagaMonitor && sagaMonitor.actionDispatched) {
        sagaMonitor.actionDispatched(action)
      }
      // 此处执行reducer
      // 如果action在reducer中有对应的处理类型
      // 则返回对应的处理结果
      // 没有的话直接返回state
      const result = next(action) // hit reducers
      // 异步的则走下面这行代码
      // 同步action不存在对应的处理过程
      // 执行
      // const sagaMiddleware = createSagaMiddleware()
      // const store = createStore(
      //   reducer,
      //   applyMiddleware(sagaMiddleware)
      // )
      // 的时候还没有action传进来
      // 所以最里面的这个函数并没有执行
      channel.put(action)
      return result
    }
  }
```

`sagaMiddleware.run(helloSaga)`是在启动Generator函数，这里的boundRunSaga是一个偏函数，是绑定了一堆基本参数的runSaga：

```js
  sagaMiddleware.run = (...args) => {
    if (process.env.NODE_ENV !== 'production' && !boundRunSaga) {
      throw new Error('Before running a Saga, you must mount the Saga middleware on the Store using applyMiddleware')
    }
    return boundRunSaga(...args)
  }
```
2. 启动saga

runSaga函数的最后是下面这几行代码，其中调用proc函数，生成了一个task并返回：

```js
return immediately(() => {
    const task = proc(env, iterator, context, effectId, getMetaInfo(saga), /* isRoot */ true, undefined)

    if (sagaMonitor) {
      sagaMonitor.effectResolved(effectId, task)
    }

    return task
  })
```

首先看一下immediately函数，这个函数位于scheduler.js文件中：

```js
  export function immediately(task) {
    try {
      suspend()
      return task()
    } finally {
      // 函数体内的代码执行顺序是
      // 1. suspend()
      // 2. flush()
      // 3. return task()
      // 首次执行的时刻
      // 是在执行sagaMiddleware.run触发的
      // 此处queue中没有任何task
      // 所以此处的flush相当于白执行
      // 重点在上面的task
      // 当执行sagaMiddleware.run时，task为
      // () => {
      //   const task = proc(env, iterator, context, effectId, getMetaInfo(saga), /* isRoot */ true, undefined)
    
      //   if (sagaMonitor) {
      //     sagaMonitor.effectResolved(effectId, task)
      //   }
    
      //   return task
      // }
      // 也就是触发了proc函数
      flush()
    }
  }
```

然后来到proc函数，

### 卤煮

1. `is`文件中有20种对不同类型的判断，感觉多数都用了鸭子类型，鸭子类型牛逼。
2. `once`，这个函数出自`utils`文件，用来确保一个函数只执行一次，在underscore中也有这个工具函数，但好像写过去就过去了，实际开发中没有用到过，现在拿出来再熟悉一下，用闭包实现的，很好玩：

  ```js
  function once(fn) {
    let called = false
    return () => {
      if (called) {
        return
      }
      called = true
      fn()
    }
  }
  ```
---

### 参考资料

1. https://zhuanlan.zhihu.com/p/30098155 (这篇文章的前半部分介绍的原理，后半部分举例了fork这个effect的实现和takeEvery这个帮助函数的实现)
2. https://zhuanlan.zhihu.com/p/37356948 (这篇文章讲的很好，只是结构有问题，感觉分块儿来讲第一次看起来一头雾水，但瑕不掩瑜)