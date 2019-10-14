---
layout: post
title: redux-saga源码学习
date: 2019-10-04
tag: 
- redux-saga
---

上一篇整明白了redux-saga的原理，现在来具体看看代码实现。

讲redux-saga源码的资料真是少的可怜，英文完全没搜到...中文就搜到两篇有价值的，第二篇对整个redux-saga的代码实现做了比较详细的描述。

然而现在的水准做不到完全掌握，能学多少是多少。

<!-- more -->

---
### 辅助理解的变量名

1. cont

  continuation 缩写，一般用于表示 Task / MainTask / ForkQueue 的后继

2. cb

  callback 缩写 或是 currCb 应该是 currentCallback 的缩写。一般用于 effect 的后继/回调函数

3. next

  就是前边的递归函数

### fork model

- parent task: the parent tasks is the aggregation of the main tasks + all its forked tasks
  
- main task: main task is the main flow of the current Genarator

### 主线

1. 关联redux和redux-saga

使用redux中的applyMiddleware将redux中的dispatch注入到redux-saga的sagaMiddleware中

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

---

2. middleware.js

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

---

3. channel.js

这个文件中的部分源码如下：

```js
export function multicastChannel() {
  let closed = false
  let currentTakers = []
  let nextTakers = currentTakers

  function checkForbiddenStates() {
    if (closed && nextTakers.length) {
      throw internalErr(CLOSED_CHANNEL_WITH_TAKERS)
    }
  }

  const ensureCanMutateNextTakers = () => {
    if (nextTakers !== currentTakers) {
      return
    }
    nextTakers = currentTakers.slice()
  }

  const close = () => {
    if (process.env.NODE_ENV !== 'production') {
      checkForbiddenStates()
    }

    closed = true
    const takers = (currentTakers = nextTakers)
    nextTakers = []
    takers.forEach(taker => {
      taker(END)
    })
  }

  // 最重要的两个方法put和take
  // take接收保存回调函数
  // put触发
  return {
    [MULTICAST]: true,
    put(input) {
      if (process.env.NODE_ENV !== 'production') {
        checkForbiddenStates()
        check(input, is.notUndef, UNDEFINED_INPUT_ERROR)
      }

      if (closed) {
        return
      }

      if (isEnd(input)) {
        close()
        return
      }

      const takers = (currentTakers = nextTakers)

      for (let i = 0, len = takers.length; i < len; i++) {
        const taker = takers[i]

        if (taker[MATCH](input)) {
          taker.cancel()
          taker(input)
        }
      }
    },
    // take函数在两个effect中有调用
    // runChannelEffect
    // 以及
    // runTakeEffect
    take(cb, matcher = matchers.wildcard) {
      if (process.env.NODE_ENV !== 'production') {
        checkForbiddenStates()
      }
      if (closed) {
        cb(END)
        return
      }
      cb[MATCH] = matcher
      ensureCanMutateNextTakers()
      nextTakers.push(cb)

      cb.cancel = once(() => {
        ensureCanMutateNextTakers()
        remove(nextTakers, cb)
      })
    },
    close,
  }
}

export function stdChannel() {
  const chan = multicastChannel()
  const { put } = chan
  // 这里重写了put方法
  chan.put = input => {
    if (input[SAGA_ACTION]) {
      put(input)
      return
    }
    asap(() => {
      put(input)
    })
  }
  return chan
}
```
---

4. runSaga.js

`sagaMiddleware.run(helloSaga)`是在启动Generator函数，这里的boundRunSaga是一个偏函数，是绑定了一堆基本参数的runSaga：

```js
  // ...
  boundRunSaga = runSaga.bind(null, {
      ...options,
      context,
      channel,
      dispatch,
      getState,
      sagaMonitor,
    })
  // ...
  sagaMiddleware.run = (...args) => {
    if (process.env.NODE_ENV !== 'production' && !boundRunSaga) {
      throw new Error('Before running a Saga, you must mount the Saga middleware on the Store using applyMiddleware')
    }
    return boundRunSaga(...args)
  }
```

其中的runSaga出自runSaga.js文件。

runSaga函数的最后是下面这几行代码，其中调用proc函数，启动了saga，生成了一个task并返回：

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

5. proc.js

proc函数的主要代码如下：

```js
function proc(env, iterator, parentContext, parentEffectId, meta, isRoot, cont) {
  // 一般情况下env.finalizeRunEffect是identity
  const finalRunEffect = env.finalizeRunEffect(runEffect)
  next.cancel = noop
  // mainTask如上面核心概念中描述的
  // 表示当前Generator的执行状态
  const mainTask = { meta, cancel: cancelMain, status: RUNNING }
  // newTask详见newTask.js文件
  const task = newTask(env, mainTask, parentContext, parentEffectId, meta, isRoot, cont)
  // executingContext用在effect runner函数中
  const executingContext = {
    task,
    digestEffect,
  }
  function cancelMain() {
    if (mainTask.status === RUNNING) {
      mainTask.status = CANCELLED
      next(TASK_CANCEL)
    }
  }

  next()

  return task
  // next函数是完成Generator运行的地方
  function next(arg, isErr) {
    try {
      let result
      if (isErr) {
        result = iterator.throw(arg)
        // user handled the error, we can clear bookkept values
        sagaError.clear()
      } else if (shouldCancel(arg)) {
        // 取消会对表示状态的status设置值CANCELLED
        mainTask.status = CANCELLED
        // next.cancel是在下面的digestEffect函数中具体设置的
        // 执行取消命令
        // 不同的effect的cancel不同
        // take类型的表示从回调池子中删除对应的回调
        next.cancel()

        // 设置done: true
        result = is.func(iterator.return) ? iterator.return(TASK_CANCEL) : { done: true, value: TASK_CANCEL }
      } else if (shouldTerminate(arg)) {
        // 设置done: true
        result = is.func(iterator.return) ? iterator.return() : { done: true }
      } else {
        // 经过上面各层的剥离
        // 迭代器对象在此处给出下一次Generator运行时的值
        // 赋给result
        result = iterator.next(arg)
      }
      // 如果没有结束
      // 将值交付予digestEffect函数
      if (!result.done) {
        digestEffect(result.value, parentEffectId, next)
      } else {
        if (mainTask.status !== CANCELLED) {
          mainTask.status = DONE
        }
        // 此处执行mainTask的回调
        mainTask.cont(result.value)
      }
    } catch (error) {
      if (mainTask.status === CANCELLED) {
        throw error
      }
      mainTask.status = ABORTED

      mainTask.cont(error, true)
    }
  }

  // 这里的effect就是上面传入的result.value
  // 经由内部的类型判断
  // 进入不同的分之
  function runEffect(effect, effectId, currCb) {
    if (is.promise(effect)) {
      resolvePromise(effect, currCb)
    } else if (is.iterator(effect)) {
      // resolve iterator
      proc(env, effect, task.context, effectId, meta, /* isRoot */ false, currCb)
    } else if (effect && effect[IO]) {
      const effectRunner = effectRunnerMap[effect.type]
      effectRunner(env, effect.payload, currCb, executingContext)
    } else {
      // anything else returned as is
      currCb(effect)
    }
  }

  function digestEffect(effect, parentEffectId, cb, label = '') {
    const effectId = nextEffectId()
    env.sagaMonitor && env.sagaMonitor.effectTriggered({ effectId, parentEffectId, label, effect })

    // 下面这个变量用来标识当前effect的执行状态，完成和取消两种状态不能并存  
    let effectSettled

    function currCb(res, isErr) {
      if (effectSettled) {
        return
      }

      effectSettled = true
      // 此处的cb是上面定义的next函数
      cb.cancel = noop
      if (env.sagaMonitor) {
        if (isErr) {
          env.sagaMonitor.effectRejected(effectId, res)
        } else {
          env.sagaMonitor.effectResolved(effectId, res)
        }
      }

      if (isErr) {
        sagaError.setCrashedEffect(effect)
      }
      // 这里递归执行了next
      cb(res, isErr)
    }
    currCb.cancel = noop

    // s此处给next函数的cancel属性赋予了明确的执行函数
    cb.cancel = () => {
      // prevents cancelling an already completed effect
      if (effectSettled) {
        return
      }

      effectSettled = true
      // currCb.cancel的定义由它的下级更具体的执行函数定义
      // 比如take类型的effect，cancel()定义在channel中
      // fork类型的effect，是不可取消的类型，所以没有cancel()函数
      currCb.cancel()
      currCb.cancel = noop // defensive measure

      env.sagaMonitor && env.sagaMonitor.effectCancelled(effectId)
    }

    finalRunEffect(effect, effectId, currCb)
  }
}
```
---

6. newTask.js

在saga的启动函数proc函数中，会创建一个task对象，

下面是创建task对象的主要代码：

```js
// newTask创建的对象用途和上面的mainTask类似
// 都是用来控制一个流程的
// 但newTask创建的对象更为强大
// 它同时创建了一个数组
// 用于保存后续fork的task
// 控制main task 和后续fork task的关系
// 感觉这个newTask返回的对象就是上面核心概念中说的parent task
export default function newTask(env, mainTask, parentContext, parentEffectId, meta, isRoot, cont = noop) {
  let status = RUNNING
  let taskResult
  let taskError
  let deferredEnd = null

  const cancelledDueToErrorTasks = []

  const context = Object.create(parentContext)
  // 该函数是fork model的具体实现
  // 由此创建了fork的多分支
  // 具体见forkQueue函数
  const queue = forkQueue(
    mainTask,
    function onAbort() {
      cancelledDueToErrorTasks.push(...queue.getTasks().map(t => t.meta.name))
    },
    end,
  )

  /**
   This may be called by a parent generator to trigger/propagate cancellation
   cancel all pending tasks (including the main task), then end the current task.

   Cancellation propagates down to the whole execution tree held by this Parent task
   It's also propagated to all joiners of this task and their execution tree/joiners

   Cancellation is noop for terminated/Cancelled tasks tasks
   **/
  function cancel() {
    if (status === RUNNING) {
      // Setting status to CANCELLED does not necessarily mean that the task/iterators are stopped
      // effects in the iterator's finally block will still be executed
      status = CANCELLED
      queue.cancelAll()
      // Ending with a TASK_CANCEL will propagate the Cancellation to all joiners
      end(TASK_CANCEL, false)
    }
  }

  function end(result, isErr) {
    if (!isErr) {
      // The status here may be RUNNING or CANCELLED
      // If the status is CANCELLED, then we do not need to change it here
      if (result === TASK_CANCEL) {
        status = CANCELLED
      } else if (status !== CANCELLED) {
        status = DONE
      }
      taskResult = result
      deferredEnd && deferredEnd.resolve(result)
    } else {
      status = ABORTED
      sagaError.addSagaFrame({ meta, cancelledTasks: cancelledDueToErrorTasks })

      if (task.isRoot) {
        const sagaStack = sagaError.toString()
        // we've dumped the saga stack to string and are passing it to user's code
        // we know that it won't be needed anymore and we need to clear it
        sagaError.clear()
        env.onError(result, { sagaStack })
      }
      taskError = result
      deferredEnd && deferredEnd.reject(result)
    }
    task.cont(result, isErr)
    task.joiners.forEach(joiner => {
      joiner.cb(result, isErr)
    })
    task.joiners = null
  }

  function setContext(props) {
    if (process.env.NODE_ENV !== 'production') {
      check(props, is.object, createSetContextWarning('task', props))
    }

    assignWithSymbols(context, props)
  }

  function toPromise() {
    if (deferredEnd) {
      return deferredEnd.promise
    }

    deferredEnd = deferred()

    if (status === ABORTED) {
      deferredEnd.reject(taskError)
    } else if (status !== RUNNING) {
      deferredEnd.resolve(taskResult)
    }

    return deferredEnd.promise
  }

  const task = {
    // fields
    [TASK]: true,
    id: parentEffectId,
    meta,
    isRoot,
    context,
    joiners: [],
    queue,

    // methods
    cancel,
    cont,
    end,
    setContext,
    toPromise,
    isRunning: () => status === RUNNING,
    /*
      This method is used both for answering the cancellation status of the task and answering for CANCELLED effects.
      In most cases, the cancellation of a task propagates to all its unfinished children (including
      all forked tasks and the mainTask), so a naive implementation of this method would be:
        `() => status === CANCELLED || mainTask.status === CANCELLED`

      But there are cases that the task is aborted by an error and the abortion caused the mainTask to be cancelled.
      In such cases, the task is supposed to be aborted rather than cancelled, however the above naive implementation
      would return true for `task.isCancelled()`. So we need make sure that the task is running before accessing
      mainTask.status.

      There are cases that the task is cancelled when the mainTask is done (the task is waiting for forked children
      when cancellation occurs). In such cases, you may wonder `yield io.cancelled()` would return true because
      `status === CANCELLED` holds, and which is wrong. However, after the mainTask is done, the iterator cannot yield
      any further effects, so we can ignore such cases.

      See discussions in #1704
     */
    isCancelled: () => status === CANCELLED || (status === RUNNING && mainTask.status === CANCELLED),
    isAborted: () => status === ABORTED,
    result: () => taskResult,
    error: () => taskError,
  }

  return task
}
```
---

7. forkQueue.js

forkQueue是fork模型的实现，感觉代码实现很直观。

代码如下：

```js
function forkQueue(mainTask, onAbort, cont) {
  let tasks = []
  let result
  let completed = false

  addTask(mainTask)
  const getTasks = () => tasks

  function abort(err) {
    onAbort()
    cancelAll()
    cont(err, true)
  }

  function addTask(task) {
    tasks.push(task)
    task.cont = (res, isErr) => {
      if (completed) {
        return
      }

      remove(tasks, task)
      task.cont = noop
      if (isErr) {
        abort(res)
      } else {
        if (task === mainTask) {
          result = res
        }
        if (!tasks.length) {
          completed = true
          cont(result)
        }
      }
    }
  }

  function cancelAll() {
    if (completed) {
      return
    }
    completed = true
    tasks.forEach(t => {
      t.cont = noop
      t.cancel()
    })
    tasks = []
  }

  return {
    addTask,
    cancelAll,
    abort,
    getTasks,
  }
}
```
---
8. resolvePromise.js

proc函数中的runEffect函数中的第一种情况是用来处理promise，比如发ajax请求数据，处理promise使用了resolvePromise，代码如下，也很直观：

```js
function resolvePromise(promise, cb) {
  const cancelPromise = promise[CANCEL]

  if (is.func(cancelPromise)) {
    cb.cancel = cancelPromise
  }

  promise.then(cb, error => {
    cb(error, true)
  })
}
```
---
9. io.js

runEffect函数中的第三种请求是如果判断传入的参数是effect，则执行effectRunner，先看下effect的工厂函数。

```js
const makeEffect = (type, payload) => ({
  [IO]: true,
  // this property makes all/race distinguishable in generic manner from other effects
  // currently it's not used at runtime at all but it's here to satisfy type systems
  combinator: false,
  type,
  payload,
})
```
makeEffect返回一个纯对象，对象的属性包含了相关信息用于在effectRunner中执行。

call、put、take、fork、all等effect都是利用在makeEffect构造的。

---
10. effectRunnerMap.js

这个函数中包含了各种effect的实际执行环境函数，以runForkEffect函数为例：

```js
function runForkEffect(env, { context, fn, args, detached }, cb, { task: parent }) {
  const taskIterator = createTaskIterator({ context, fn, args })
  const meta = getIteratorMetaInfo(taskIterator, fn)

  immediately(() => {
    // 在执行下面这行代码的同时，会启动child的saga
    const child = proc(env, taskIterator, parent.context, currentEffectId, meta, detached, undefined)

    if (detached) {
      cb(child)
    } else {
      if (child.isRunning()) {
        // 此处将fork task装入parent task的queue中
        // 形成多分支
        parent.queue.addTask(child)
        cb(child)
      } else if (child.isAborted()) {
        parent.queue.abort(child.error())
      } else {
        cb(child)
      }
    }
  })
  // Fork effects are non cancellables
}
```


相比下面的runTakeEffect函数，runForkEffect会起一个新的saga，新的saga中有自己的next迭代函数，每一个fork分支对应一个，所以不存在阻塞的问题（这里的阻塞感觉是相对Generator的主执行环境而言，这里没有看到有起其他的工作线程，相对于主线程而言还是要按顺序线性执行）。但其他的effect，比如take，是运行在mainTask所在的那个next中，就出现了等待，常用的call也是这样。另外使用take会使middleware等待出现一个特定的action（其实看源码是通过类型判断的，并没有具体到某一个）这是因为将next传入了channel的队列中，成为了channel的回调，看起来像是middleware的等待。

```js

function runTakeEffect(env, { channel = env.channel, pattern, maybe }, cb) {
  const takeCb = input => {
    if (input instanceof Error) {
      cb(input, true)
      return
    }
    if (isEnd(input) && !maybe) {
      cb(TERMINATE)
      return
    }
    // 将next传入channel的队列中
    cb(input)
  }
  try {
    channel.take(takeCb, is.notUndef(pattern) ? matcher(pattern) : null)
  } catch (err) {
    cb(err, true)
    return
  }
  cb.cancel = takeCb.cancel
}
```
---
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