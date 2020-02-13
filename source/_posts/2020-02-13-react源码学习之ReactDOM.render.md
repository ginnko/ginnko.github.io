---
layout: post
title: react源码学习之ReactDOM.render
date: 2020-02-13
tag: 
- react
- ReactDOM.render
---

基于16.12.0。

ReactDOM.render是将react元素和dom节点对象绑定的api。整个流程主要负责三件事：

1. 创建ReactRoot
2. 创建FiberRoot和Fiber对象
3. 创建更新

目前还不明白上面三个对象的逻辑含义。

再之后将进入调度过程。进入调度之后，不论是通过setState还是ReactDOM.render，在创建完更新后，它们的后续操作都是由调度器管理的，和创建更新调用的哪个api没有任何关系。

<!-- more -->

### 流程

![ReactDOM.render流程](/images/react/ReactDOM.render流程图.png)

### 主要函数

1. ReactDOM.render

位置：packages/react-dom/src/client/ReactDOMLegacy.js

常用参数：
  - element：React元素，
  - container：DOM节点对象

```js
// 去除dev的条件判断和不相关的判断代码

function render(
  element: React$Element<any>,
  container: DOMContainer,
  callback: ?Function,
) {
  return legacyRenderSubtreeIntoContainer(
    null,
    element,
    container,
    false,
    callback,
  );
}
```

2. legacyRenderSubtreeIntoContainer

这里的两个重要函数：

- legacyCreateRootFromDOMContainer：创建了root对象，并将其_internalRoot属性赋值给fiberRoot
- updateContainer：在这个函数中将创建"更新"，更新用来标记React应用中用来更新的地点

在这个函数中调用上面两个函数完成开头说的三个对象的创建工作。

```js
// 去除dev的条件判断和不相关的判断代码
// 仅显示首次渲染的代码

function legacyRenderSubtreeIntoContainer(
  parentComponent: ?React$Component<any, any>,
  children: ReactNodeList,
  container: DOMContainer,
  forceHydrate: boolean,
  callback: ?Function,
) {
  let root: RootType = (container._reactRootContainer: any);
  let fiberRoot;
  if (!root) {
    // Initial mount
    root = container._reactRootContainer = legacyCreateRootFromDOMContainer(
      container,
      forceHydrate,
    );
    fiberRoot = root._internalRoot;
    // Initial mount should not be batched.
    unbatchedUpdates(() => {
      updateContainer(children, fiberRoot, parentComponent, callback);
    });
  }
  return getPublicRootInstance(fiberRoot);
}
```

3. legacyCreateRootFromDOMContainer

这个函数先是清除了container DOM节点中的所有子元素，然后用这个处理过的dom节点对象，创建root对象。

```js
// 去除dev的条件判断和不相关的判断代码
// 其中，关于hydrate的属性值均为false

function legacyCreateRootFromDOMContainer(
  container: DOMContainer,
  forceHydrate: boolean,
): RootType {
  const shouldHydrate =
    forceHydrate || shouldHydrateDueToLegacyHeuristic(container);
  // First clear any existing content.
  if (!shouldHydrate) {
    let warned = false;
    let rootSibling;
    while ((rootSibling = container.lastChild)) {
      container.removeChild(rootSibling);
    }
  }

  return createLegacyRoot(
    container,
    shouldHydrate
      ? {
          hydrate: true,
        }
      : undefined,
  );
}
```

4. createLegacyRoot

注意这个函数调用ReactDOMBlockingRoot函数使用`new`关键字的，也就是说这里会返回一个对象，一直返回到`legacyRenderSubtreeIntoContainer`函数中，成为root。

```js
  function createLegacyRoot(
    container: DOMContainer,
    options?: RootOptions,
  ): RootType {
    return new ReactDOMBlockingRoot(container, LegacyRoot, options);
  }
```

5. ReactDOMBlockingRoot

关于命名有个疑问，为啥要带Blocking？

legacyRenderSubtreeIntoContainer函数中的root对象是下面这种结构的：

```js
{
  _internalRoot: {
    // 这里的内容详见第9个函数处
  }
}
```

```js
  function ReactDOMBlockingRoot(
    container: DOMContainer,
    tag: RootTag,
    options: void | RootOptions,
  ) {
    this._internalRoot = createRootImpl(container, tag, options);
  }
```

6. createRootImpl

和hydrate相关的都是false。

```js
  function createRootImpl(
    container: DOMContainer,
    tag: RootTag,
    options: void | RootOptions,
  ) {
    // Tag is either LegacyRoot or Concurrent Root
    const hydrate = options != null && options.hydrate === true;
    const hydrationCallbacks =
      (options != null && options.hydrationOptions) || null;
    const root = createContainer(container, tag, hydrate, hydrationCallbacks);
    // 这里root.current表示的是一个Fiber对象
    // 其中包含的属性见 函数12
    markContainerAsRoot(root.current, container);
    if (hydrate && tag !== LegacyRoot) {
      const doc =
        container.nodeType === DOCUMENT_NODE
          ? container
          : container.ownerDocument;
      eagerlyTrapReplayableEvents(doc);
    }
    return root;
  }
```

7. createContainer

不太明白为啥要把`createFiberRoot`函数放进`createContainer`中。。。直接调不香么(手动扶额)

返回的是带有Fiber对象的FiberRoot对象

```js
  function createContainer(
    containerInfo: Container,
    tag: RootTag,
    hydrate: boolean,
    hydrationCallbacks: null | SuspenseHydrationCallbacks,
  ): OpaqueRoot {
    return createFiberRoot(containerInfo, tag, hydrate, hydrationCallbacks);
  }
```

8. createFiberRoot

到这里`containerInfo`依然是dom节点对象。

```js
  function createFiberRoot(
    containerInfo: any,
    tag: RootTag,
    hydrate: boolean,
    hydrationCallbacks: null | SuspenseHydrationCallbacks,
  ): FiberRoot {
    const root: FiberRoot = (new FiberRootNode(containerInfo, tag, hydrate): any);
    if (enableSuspenseCallback) {
      root.hydrationCallbacks = hydrationCallbacks;
    }

    // Cyclic construction. This cheats the type system right now because
    // stateNode is any.
    const uninitializedFiber = createHostRootFiber(tag);
    root.current = uninitializedFiber;
    uninitializedFiber.stateNode = root;

    initializeUpdateQueue(uninitializedFiber);

    return root;
  }
```

9. FiberRootNode

这里创建的是FiberRoot

```js
  function FiberRootNode(containerInfo, tag, hydrate) {
    this.tag = tag;
    this.current = null;
    this.containerInfo = containerInfo;
    this.pendingChildren = null;
    this.pingCache = null;
    this.finishedExpirationTime = NoWork;
    this.finishedWork = null;
    this.timeoutHandle = noTimeout;
    this.context = null;
    this.pendingContext = null;
    this.hydrate = hydrate;
    this.callbackNode = null;
    this.callbackPriority = NoPriority;
    this.firstPendingTime = NoWork;
    this.firstSuspendedTime = NoWork;
    this.lastSuspendedTime = NoWork;
    this.nextKnownPendingLevel = NoWork;
    this.lastPingedTime = NoWork;
    this.lastExpiredTime = NoWork;

    if (enableSchedulerTracing) {
      this.interactionThreadID = unstable_getThreadID();
      this.memoizedInteractions = new Set();
      this.pendingInteractionMap = new Map();
    }
    if (enableSuspenseCallback) {
      this.hydrationCallbacks = null;
    }
  }
```

10.  createHostRootFiber

```js
createHostRootFiber(tag: RootTag): Fiber {
  let mode;
  if (tag === ConcurrentRoot) {
    mode = ConcurrentMode | BlockingMode | StrictMode;
  } else if (tag === BlockingRoot) {
    mode = BlockingMode | StrictMode;
  } else {
    // 会到这个分支
    mode = NoMode;
  }

  if (enableProfilerTimer && isDevToolsPresent) {
    // Always collect profile timings when DevTools are present.
    // This enables DevTools to start capturing timing at any point–
    // Without some nodes in the tree having empty base times.
    mode |= ProfileMode;
  }
  // 这里的 HostRoot = 3
  return createFiber(HostRoot, null, null, mode);
}
```

11. createFiber

```js
  const createFiber = function(
    tag: WorkTag,
    pendingProps: mixed,
    key: null | string,
    mode: TypeOfMode,
  ): Fiber {
    return new FiberNode(tag, pendingProps, key, mode);
  };
```

12. FiberNode

这里创建的是Fiber

```js
// 去除dev的条件判断和不相关的判断代码

  function FiberNode(
    tag: WorkTag,
    pendingProps: mixed,
    key: null | string,
    mode: TypeOfMode,
  ) {
    // Instance
    this.tag = tag;
    this.key = key;
    this.elementType = null;
    this.type = null;
    this.stateNode = null;

    // Fiber
    this.return = null;
    this.child = null;
    this.sibling = null;
    this.index = 0;

    this.ref = null;

    this.pendingProps = pendingProps;
    this.memoizedProps = null;
    this.updateQueue = null;
    this.memoizedState = null;
    this.dependencies = null;

    this.mode = mode;

    // Effects
    this.effectTag = NoEffect;
    this.nextEffect = null;

    this.firstEffect = null;
    this.lastEffect = null;

    this.expirationTime = NoWork;
    this.childExpirationTime = NoWork;

    this.alternate = null;

    if (enableProfilerTimer) {
      this.actualDuration = Number.NaN;
      this.actualStartTime = Number.NaN;
      this.selfBaseDuration = Number.NaN;
      this.treeBaseDuration = Number.NaN;

      // It's okay to replace the initial doubles with smis after initialization.
      // This won't trigger the performance cliff mentioned above,
      // and it simplifies other profiler code (including DevTools).
      this.actualDuration = 0;
      this.actualStartTime = -1;
      this.selfBaseDuration = 0;
      this.treeBaseDuration = 0;
    }

    if (enableUserTimingAPI) {
      this._debugID = debugCounter++;
      this._debugIsCurrentlyTiming = false;
    }
  }
```

13. markContainerAsRoot

hostRoot是Container DOM对象对应的Fiber对象

node是Container DOM对象

这个函数的作用是给Container DOM对象添加一个属性，属性值是Container DOM对象对应的Fiber对象，通过Fiber对象的stateNode属性能够拿到对应的FiberRoot对象

```js
  function markContainerAsRoot(hostRoot, node) {
    node[internalContainerInstanceKey] = hostRoot;
  }
```

14. updateContainer

到此开始创建更新。

- expirationTime：过期时间的创建是个重点
- context：不太重要，可忽略
- update: 是用来标记React应用中需要更新的地点
- enqueueUpdate: 在同一个节点上会产生多个更新，这个用来对更新排序
- scheduleWork: 开始进入任务调度，根据优先级来确定更新的先后顺序

```js
// 去除dev的条件判断和不相关的判断代码

export function updateContainer(
  element: ReactNodeList,
  container: OpaqueRoot,
  parentComponent: ?React$Component<any, any>,
  callback: ?Function,
): ExpirationTime {
  const suspenseConfig = requestCurrentSuspenseConfig();
  const expirationTime = computeExpirationForFiber(
    currentTime,
    current,
    suspenseConfig,
  );


  const context = getContextForSubtree(parentComponent);
  if (container.context === null) {
    container.context = context;
  } else {
    container.pendingContext = context;
  }

  const update = createUpdate(expirationTime, suspenseConfig);
  // Caution: React DevTools currently depends on this property
  // being called "element".
  update.payload = {element};

  callback = callback === undefined ? null : callback;
  if (callback !== null) {
    update.callback = callback;
  }

  enqueueUpdate(current, update);
  scheduleWork(current, expirationTime);

  return expirationTime;
}
```

15. createUpdate

```js
// 去除dev的条件判断和不相关的判断代码

  function createUpdate(
    expirationTime: ExpirationTime,
    suspenseConfig: null | SuspenseConfig,
  ): Update<*> {
    let update: Update<*> = {
      expirationTime,
      suspenseConfig,

      tag: UpdateState,
      payload: null,
      callback: null,

      next: (null: any),
    };
    update.next = update;

    return update;
  }
```

16. getPublicRootInstance

会走第一个分支，返回null

```js

  function getPublicRootInstance(
    container: OpaqueRoot,
  ): React$Component<any, any> | PublicInstance | null {
    const containerFiber = container.current;
    if (!containerFiber.child) {
      return null;
    }
    switch (containerFiber.child.tag) {
      case HostComponent:
        return getPublicInstance(containerFiber.child.stateNode);
      default:
        return containerFiber.child.stateNode;
    }
  }
```


### 函数所在文件路径

![ReactDOM.render函数路径图](/images/react/ReactDOM.render函数路径图.png)