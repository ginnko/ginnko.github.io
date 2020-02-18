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

再之后将进入调度过程。进入调度之后，不论是通过setState还是ReactDOM.render，在创建完更新后，它们的后续操作都是由调度器管理的，和创建更新调用的哪个api没有任何关系。

<!-- more -->

### 重要概念

1. ReactRoot

这个对象的主要用途是创建FiberRoot

2. FiberRoot

FiberRoot是一个特殊的Fiber对象，它用来：

  - 作为整个应用的起点
  - 包含应用挂在的目标节点（DOM节点对象，比如#root）
  - 记录整个应用更新过程的各种信息

FiberRoot各个属性的含义参见BaseFiberRootProperties，后续遇到再详细分析和补充。

3. Fiber

一个ReactElement对象对应一个Fiber对象，它的作用包括：

  - 记录节点的各种状态：比如class component中state，props都是记录在Fiber对象上的然后在Fiber更新后，才会更新到this.state或this.props上。**简单的说就是更新先记录在Fiber上，再挂到this上，这也为没有this的function component实现Hook提供了基础。**
  - 串联起整个应用形成树结构，其组织方式和对应关系如下图
    - ReactElement使用**props.children**组成树结构
    - Fiber使用**return**，**child**，**sibling**组成树结构

  ![ReactElement-tree-vs-Fiber-tree](/images/react/ReactElement-tree-vs-Fiber-tree.png)

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

这里是FiberRoot的属性定义，待用到再返回头来看

文件地址：packages/react-reconciler/src/ReactFiberRoot.js

```js
  type BaseFiberRootProperties = {|
    // The type of root (legacy, batched, concurrent, etc.)
    tag: RootTag,

    // Any additional information from the host associated with this root.
    containerInfo: any,
    // Used only by persistent updates.
    pendingChildren: any,
    // The currently active root fiber. This is the mutable root of the tree.
    current: Fiber,

    pingCache:
      | WeakMap<Thenable, Set<ExpirationTime>>
      | Map<Thenable, Set<ExpirationTime>>
      | null,

    finishedExpirationTime: ExpirationTime,
    // A finished work-in-progress HostRoot that's ready to be committed.
    finishedWork: Fiber | null,
    // Timeout handle returned by setTimeout. Used to cancel a pending timeout, if
    // it's superseded by a new one.
    timeoutHandle: TimeoutHandle | NoTimeout,
    // Top context object, used by renderSubtreeIntoContainer
    context: Object | null,
    pendingContext: Object | null,
    // Determines if we should attempt to hydrate on the initial mount
    +hydrate: boolean,
    // Node returned by Scheduler.scheduleCallback
    callbackNode: *,
    // Expiration of the callback associated with this root
    callbackExpirationTime: ExpirationTime,
    // Priority of the callback associated with this root
    callbackPriority: ReactPriorityLevel,
    // The earliest pending expiration time that exists in the tree
    firstPendingTime: ExpirationTime,
    // The earliest suspended expiration time that exists in the tree
    firstSuspendedTime: ExpirationTime,
    // The latest suspended expiration time that exists in the tree
    lastSuspendedTime: ExpirationTime,
    // The next known expiration time after the suspended range
    nextKnownPendingLevel: ExpirationTime,
    // The latest time at which a suspended component pinged the root to
    // render again
    lastPingedTime: ExpirationTime,
    lastExpiredTime: ExpirationTime,
  |};
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

这里是FiberNode的类型定义

```js
  type Fiber = {|
    // These first fields are conceptually members of an Instance. This used to
    // be split into a separate type and intersected with the other Fiber fields,
    // but until Flow fixes its intersection bugs, we've merged them into a
    // single type.

    // An Instance is shared between all versions of a component. We can easily
    // break this out into a separate object to avoid copying so much to the
    // alternate versions of the tree. We put this on a single object for now to
    // minimize the number of objects created during the initial render.

    // Tag identifying the type of fiber.
    tag: WorkTag,

    // Unique identifier of this child.
    key: null | string,

    // The value of element.type which is used to preserve the identity during
    // reconciliation of this child.
    elementType: any,

    // The resolved function/class/ associated with this fiber.
    type: any,

    // The local state associated with this fiber.
    stateNode: any,

    // Conceptual aliases
    // parent : Instance -> return The parent happens to be the same as the
    // return fiber since we've merged the fiber and instance.

    // Remaining fields belong to Fiber

    // The Fiber to return to after finishing processing this one.
    // This is effectively the parent, but there can be multiple parents (two)
    // so this is only the parent of the thing we're currently processing.
    // It is conceptually the same as the return address of a stack frame.
    return: Fiber | null,

    // Singly Linked List Tree Structure.
    child: Fiber | null,
    sibling: Fiber | null,
    index: number,

    // The ref last used to attach this node.
    // I'll avoid adding an owner field for prod and model that as functions.
    ref:
      | null
      | (((handle: mixed) => void) & {_stringRef: ?string, ...})
      | RefObject,

    // Input is the data coming into process this fiber. Arguments. Props.
    pendingProps: any, // This type will be more specific once we overload the tag.
    memoizedProps: any, // The props used to create the output.

    // A queue of state updates and callbacks.
    updateQueue: UpdateQueue<any> | null,

    // The state used to create the output
    memoizedState: any,

    // Dependencies (contexts, events) for this fiber, if it has any
    dependencies: Dependencies | null,

    // Bitfield that describes properties about the fiber and its subtree. E.g.
    // the ConcurrentMode flag indicates whether the subtree should be async-by-
    // default. When a fiber is created, it inherits the mode of its
    // parent. Additional flags can be set at creation time, but after that the
    // value should remain unchanged throughout the fiber's lifetime, particularly
    // before its child fibers are created.
    mode: TypeOfMode,

    // Effect
    effectTag: SideEffectTag,

    // Singly linked list fast path to the next fiber with side-effects.
    nextEffect: Fiber | null,

    // The first and last fiber with side-effect within this subtree. This allows
    // us to reuse a slice of the linked list when we reuse the work done within
    // this fiber.
    firstEffect: Fiber | null,
    lastEffect: Fiber | null,

    // Represents a time in the future by which this work should be completed.
    // Does not include work found in its subtree.
    expirationTime: ExpirationTime,

    // This is used to quickly determine if a subtree has no pending changes.
    childExpirationTime: ExpirationTime,

    // This is a pooled version of a Fiber. Every fiber that gets updated will
    // eventually have a pair. There are cases when we can clean up pairs to save
    // memory if we need to.
    alternate: Fiber | null,

    // Time spent rendering this Fiber and its descendants for the current update.
    // This tells us how well the tree makes use of sCU for memoization.
    // It is reset to 0 each time we render and only updated when we don't bailout.
    // This field is only set when the enableProfilerTimer flag is enabled.
    actualDuration?: number,

    // If the Fiber is currently active in the "render" phase,
    // This marks the time at which the work began.
    // This field is only set when the enableProfilerTimer flag is enabled.
    actualStartTime?: number,

    // Duration of the most recent render time for this Fiber.
    // This value is not updated when we bailout for memoization purposes.
    // This field is only set when the enableProfilerTimer flag is enabled.
    selfBaseDuration?: number,

    // Sum of base times for all descendants of this Fiber.
    // This value bubbles up during the "complete" phase.
    // This field is only set when the enableProfilerTimer flag is enabled.
    treeBaseDuration?: number,

    // Conceptual aliases
    // workInProgress : Fiber ->  alternate The alternate used for reuse happens
    // to be the same as work in progress.
    // __DEV__ only
    _debugID?: number,
    _debugSource?: Source | null,
    _debugOwner?: Fiber | null,
    _debugIsCurrentlyTiming?: boolean,
    _debugNeedsRemount?: boolean,

    // Used to verify that the order of hooks does not change between renders.
    _debugHookTypes?: Array<HookType> | null,
  |};
```

13. initializeUpdateQueue

这里完成了updatequeue的初始化

```js
  function initializeUpdateQueue<State>(fiber: Fiber): void {
    const queue: UpdateQueue<State> = {
      baseState: fiber.memoizedState,
      baseQueue: null,
      shared: {
        pending: null,
      },
      effects: null,
    };
    fiber.updateQueue = queue;
  }
```

14. markContainerAsRoot

hostRoot是Container DOM对象对应的Fiber对象

node是Container DOM对象

这个函数的作用是给Container DOM对象添加一个属性，属性值是Container DOM对象对应的Fiber对象，通过Fiber对象的stateNode属性能够拿到对应的FiberRoot对象

```js
  function markContainerAsRoot(hostRoot, node) {
    node[internalContainerInstanceKey] = hostRoot;
  }
```

15. updateContainer

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

16. createUpdate

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

update的类型定义如下：

```js
  type Update<State> = {|
    expirationTime: ExpirationTime,
    suspenseConfig: null | SuspenseConfig,

    tag: 0 | 1 | 2 | 3,
    payload: any,
    callback: (() => mixed) | null,

    next: Update<State>,

    // DEV only
    priority?: ReactPriorityLevel,
  |};
```

updateQueue的类型定义如下：

```js
  type UpdateQueue<State> = {|
    baseState: State,
    baseQueue: Update<State> | null,
    shared: SharedQueue<State>,
    effects: Array<Update<State>> | null,
  |};
```

17. getPublicRootInstance

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

### 参考

1. https://react.jokcy.me/book/api/react-structure.html