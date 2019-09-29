---
layout: post
title: redux-saga文档学习
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

2. Effect

> Effects are plain JavaScript objects which contain instructions to be fulfilled by the middleware.

Effects的类型是js的空白对象，包含能被middleware执行的指令(原文没写，但感觉就是一个action)。在redux-saga中使用`put`来发动一个Effect。

---
### redux-saga的使用

现在公司的框架使用了antd pro，其中的dva对redux-saga进行了封装，不太清楚原生redux-saga如何使用，现在来学一波，参考资料为[redux-saga官网教程](https://redux-saga.js.org/)，主要目的是：

  - 熟悉redux-saga中的概念
  - 熟悉用法，为后面学习设计思想和源码实现打基础

1. 使用saga的步骤

  1. 在单独的文件中创建saga
  2. 向主文件中导入saga，如`helloSaga`
  3. 使用从`redux-saga`中导入的工厂函数`createSagaMiddleware`创建中间件`sagaMiddleware`
  4. 使用从`redux`中导入的`applyMiddleware`连接第三步创建的中间件`sagaMiddleware`
  5. 使用`sagaMiddleware.run(helloSaga)`启动saga

2. 创建saga

[参考资料1](https://redux-saga.js.org/docs/introduction/BeginnerTutorial.html)中有下面的描述：

>Sagas are implemented as Generator functions that yield objects to the redux-saga middleware. The yielded objects are a kind of instruction to be interpreted by the middleware. When a Promise is yielded to the middleware, the middleware will suspend the Saga until the Promise completes.

saga这种东西是由生成器函数实现的，生成器函数会给redux-saga的中间件yield（这里没有翻译，是为了表示生成器的运行过程，防止日后再回来看有误解）对象。中间件会将yield的对象翻译成指令。当saga yield一个Promise给中间件的时候，中间件将暂停这个saga的直到Promise有了结果。

> Once the Promise is resolved, the middleware will resume the saga, executing code until the next yield.

一旦一个Promise被成功解析，中间件将重启这个saga，执行下一个yield。

> put is one example of what we call an Effect. Effects are plain Javascript objects which contain instructions to be fulfilled by the middleware. When a middleware retrives an Effect yielded by a saga, the saga is paused until the Effect is fulfilled.

上面的重点在当一个中间件获得一个saga yield的Effect，saga也会暂停执行直到这个yielded的Effect执行完成(Effect的概念见上)。

一个列子：

```js
export function* incrementAsync() {
  yield delay(1000);
  // 下面这行代码指示中间件分发一个`INCREMENT`action
  yield put({
    type: 'INCREAMENT'
  });
}
```

上面的代码从saga的视角描述是：incrementAsync这个saga通过调用delay(1000)睡了1s，然后分发了INCREMENT action。

3. api

  1. put：发出一个Effect
  2. takeEvery：redux-saga提供的辅助函数，作用感觉是监控saga的启动，允许多个fetchData实例同时存在。

  ```js
  import { takeEvery } from 'redux-saga/effects'

  function* watchFetchData() {
    yield takeEvery('FETCH_REQUESTED', fetchData)
  }
  ```
  3. takeLatest：功能和takeEvery类似，但同时只允许一个saga在运行。
  4. all：返回由运行多个saga产生的结果组成的数组，多个saga会并行执行。
  5. call：类似put，返回一个Effect指示中间件使用给定的参数调用给定的函数。事实上，put或call本身不执行dispatch或是异步调用，它们只是返回js中的空对象(plain object)。具体的执行是由中间件来决定，如果Effect的类型是`put`，中间件就向store分发一个action，如果是一个`call`类型，就调用这个给定函数。
  6. take：先看两个例子：

  ```js
    <!-- take -->
    import { take, put } from 'redux-saga/effects'

    function* watchFirstThreeTodosCreation() {
      for (let i = 0; i < 3; i++) {
        const action = yield take('TODO_CREATED')
      }
      yield put({type: 'SHOW_CONGRATULATION'})
    }

    <!-- takeEvery -->
    import { select, takeEvery } from 'redux-saga/effects'

    function* watchAndLog() {
      yield takeEvery('*', function* logger(action) {
        const state = yield select()

        console.log('action', action)
        console.log('state after', state)
      })
    }
  ```
   take的工作方式是将暂停这个迭代器，直到一个匹配action被分发。

  官网关于takeEvery的工作原理的说明：

   > In the case of takeEvery, the invoked tasks have no control on when they'll be called. They will be invoked again and again on each matching action. They also have no control on when to stop the observation.

  翻译：被调用的任务(logger)控制不了它们何时被调用。只要满足匹配项，它们就会一次一次被调用。它们也控制不了何时终止观察过程。

  官网关于take的工作原理的说明：

  > In the case of take, the control is inverted. Instead of the actions being pushed to the handler tasks, the saga is pulling the action by itself. It looks as if the saga is performing a normal function call action=getNextAction(), which will resolve when the action is dispatched.

  翻译：在take的使用中，action是由take主动发起的（这里类似put或者call），和takeEvery将action推向处理任务不同，这个saga(watchFirstThreeTodosCreation)是自己拉取(take)action，就好像调用了一个普通的函数：action=getNextAction()，当action被分发后完成解析。
  
  看上面的第一个例子，watchFirstThreeTodosCreation运行后，会进入一个次数为三次的循环，每次循环的开始都会暂停在：const action = yield take('TODO_CREATED')处，当这个action(TODO_CREATED)完成分发后，这一行代码执行完，进入下一个循环，重复同样的过程，直到三次结束，最后调用：yield put({type: 'SHOW_CONGRATULATION'})发出一个Effect，然后就退出了监控，这个迭代器也可以被垃圾回收。

  另外，可以并发的观察多个action：

  ```js
    yield take(['LOGOUT', 'LOGIN_ERROR'])
  ```

  7. fork：当fork一个任务时，这项任务会在后台启动，后续的任务无需等待fork的这项任务完成，实现无阻塞工作流程。
  8. cancel：取消一个fork的任务
   
   ```js
    function* loginFlow() {
      while (true) {
        const {user, password} = yield take('LOGIN_REQUEST')
        // fork return a Task object
        const task = yield fork(authorize, user, password)
        const action = yield take(['LOGOUT', 'LOGIN_ERROR'])
        if (action.type === 'LOGOUT')
          yield cancel(task)
        yield call(Api.clearItem, 'token')
      }
    }
   ```

   cancel一个task，会向下传递，将任务中的任务也取消，比如下例，取消了subtask，同时会取消subtask2：

   ```js
    function* main() {
      const task = yield fork(subtask)
      ...
      // later
      yield cancel(task)
    }

    function* subtask() {
      ...
      yield call(subtask2) // currently blocked on this call
      ...
    }

    function* subtask2() {
      ...
      yield call(someApi) // currently blocked on this call
      ...
    }
   ```
  9. all：并发执行任务，这个在项目中已经接触过了， 如果有一个任务失败了，其他的都会被取消
  10. race：第一个执行完毕的任务作为最后的结果，其他任务将被自动取消

4. redux-saga中原理模型

  1. fork model：参见这里：https://redux-saga.js.org/docs/advanced/ForkModel.html，需要的时候再来看。
---

### 参考资料

1. https://redux-saga.js.org/
2. https://blog.couchbase.com/saga-pattern-implement-business-transactions-using-microservices-part/
3. https://blog.couchbase.com/saga-pattern-implement-business-transactions-using-microservices-part-2/
4. https://zhuanlan.zhihu.com/p/41661402

