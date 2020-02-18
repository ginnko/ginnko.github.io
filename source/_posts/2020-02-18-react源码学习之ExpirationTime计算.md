---
layout: post
title: react源码学习之ExpirationTime计算
date: 2020-02-18
tag: 
- react
- ExpirationTime
---

基于16.12.0。

本篇主要记录：

1. expirationTime的创建代码整理
2. 两次放大时间间隙原因的猜想

有一个不明白的，计算过程中使用到了四个常量参数，这些常量的值是如何确定的？目前没有找到这方面的资料。。。

```js
export const LOW_PRIORITY_EXPIRATION = 5000;
export const LOW_PRIORITY_BATCH_SIZE = 250;

export const HIGH_PRIORITY_EXPIRATION = __DEV__ ? 500 : 150;
export const HIGH_PRIORITY_BATCH_SIZE = 100;
```

<!-- more -->

### ExpirationTime的作用

为防止某个update因为低优先级额的原因一直被打断而始终未能执行，当时间到了ExpirationTime，如果某个update还未执行，React将会强制执行该update。

### ExpirationTime的种类

1. sync模式：优先级最高，创建完成后立即更新到DOM中（创建即更新的流程）
2. 异步模式：会进行调度，可能会被中断，根据优先级（分为Interactive和普通异步两种，Interactive一般由事件触发）更新
3. 指定context模式，外部强制更新（还不清楚这是个啥）
4. suspense模式（还不清楚这是个啥）

### ExpirationTime的计算过程

这里想单独说下`msToExpirationTime`函数和`ceiling`函数。

`msToExpirationTime`函数是用来计算currentTime的，参数来源是`now()`的执行结果，其内部封装了`Performance.now()`，查文档看到新的浏览器才支持，这个api相对于页面加载开始计时，在数量级上更精确，返回浮点数形式的ms，这些特点符合react要求精确的场景，同时能满足react使用32位整数处理的要求。因为这个api返回的是浮点数形式的ms，`msToExpirationTime`中取整操作用来将expiration time取整是为了数学上的方便这样的猜想感觉更靠谱了，虽然能抹除10ms的差距，但感觉并非是主要目的。

`ceiling`函数才是主要放大时间区间的地方，低优先级的两个update之间的先后时间差不大于25个expiration time单位，就被视为同一批次更新，高优先级的在生产环境下是10个expiration time单位。（在网上看到的参考资料，都说这里是25ms，感觉他们说的不对，从单位上就说不通，如果换成ms应该是250ms的时间差。）

关于ExpirationTime的纯计算代码如下：

```js
const MAX_SIGNED_31_BIT_INT = 1073741823; // 单位是expiration time单位

const Sync = MAX_SIGNED_31_BIT_INT; // 单位是expiration time单位
const Batched = Sync - 1; // 单位是expiration time单位

const UNIT_SIZE = 10; // 感觉这里的单位是 10ms/expiration time单位
const MAGIC_NUMBER_OFFSET = Batched - 1; // 单位是expiration time单位

// 这个函数是将ms转换为expiration time
// 1单位的expiration time表示10ms
function msToExpirationTime(ms: number): ExpirationTime {
  // Always add an offset so that we don't clash with the magic number for NoWork.
  // 这里是怕出现结果为零的情况
  // 按位或是舍弃小数取整
  return MAGIC_NUMBER_OFFSET - ((ms / UNIT_SIZE) | 0);
}

// 这个函数是将expiration time 转换为ms
function expirationTimeToMs(expirationTime: ExpirationTime): number {
  return (MAGIC_NUMBER_OFFSET - expirationTime) * UNIT_SIZE;
}

// 这个函数是将
// expiration time值相近的时间按precision来归类
// 这里的相近表示差值没有大于等于1个precision的计算结果都是相同的
function ceiling(num: number, precision: number): number {
  return (((num / precision) | 0) + 1) * precision;
}

// 这个函数是实际计算expiration time的地方
function computeExpirationBucket(
  currentTime,
  expirationInMs,
  bucketSizeMs,
): ExpirationTime {
  return (
    MAGIC_NUMBER_OFFSET -
    ceiling(
      MAGIC_NUMBER_OFFSET - currentTime + expirationInMs / UNIT_SIZE,
      bucketSizeMs / UNIT_SIZE,
    )
  );
}

// 这里是用来处理异步模式中的低优先级的部分
// 一个疑问是LOW_PRIORITY_EXPIRATION和LOW_PRIORITY_BATCH_SIZE是如何确定的
// 没有找到相关资料
// 猜测和页面的更新频率有关
const LOW_PRIORITY_EXPIRATION = 5000;
const LOW_PRIORITY_BATCH_SIZE = 250;

function computeAsyncExpiration(
  currentTime: ExpirationTime,
): ExpirationTime {
  return computeExpirationBucket(
    currentTime,
    LOW_PRIORITY_EXPIRATION,
    LOW_PRIORITY_BATCH_SIZE,
  );
}

// 这里是用来处理异步模式中高优先级的部分
// 同样不明白HIGH_PRIORITY_EXPIRATION和HIGH_PRIORITY_BATCH_SIZE是怎么确定出来的
const HIGH_PRIORITY_EXPIRATION = __DEV__ ? 500 : 150;
const HIGH_PRIORITY_BATCH_SIZE = 100;

function computeInteractiveExpiration(currentTime: ExpirationTime) {
  return computeExpirationBucket(
    currentTime,
    HIGH_PRIORITY_EXPIRATION,
    HIGH_PRIORITY_BATCH_SIZE,
  );
}
```

下面是具体的代码流程

1. updateContainer

创建expirationTime的过程发生在updateContainer函数中。

```js
  // 已删除测试代码和其他与创建expirationTime的代码
  function updateContainer(
    element: ReactNodeList,
    container: OpaqueRoot,
    parentComponent: ?React$Component<any, any>,
    callback: ?Function,
  ): ExpirationTime {
    const current = container.current;
    const currentTime = requestCurrentTimeForUpdate();

    const suspenseConfig = requestCurrentSuspenseConfig();
    const expirationTime = computeExpirationForFiber(
      currentTime,
      current,
      suspenseConfig,
    );
  }
```

2. requestCurrentTimeForUpdate

这里先不看细节，总之就是调用`msToExpirationTime(now())`获取当前时间。

```js
  function requestCurrentTimeForUpdate() {
    if ((executionContext & (RenderContext | CommitContext)) !== NoContext) {
      // We're inside React, so it's fine to read the actual time.
      return msToExpirationTime(now());
    }
    // We're not inside React, so we may be in the middle of a browser event.
    if (currentEventTime !== NoWork) {
      // Use the same start time for all updates until we enter React again.
      return currentEventTime;
    }
    // This is the first update since React yielded. Compute a new start time.
    currentEventTime = msToExpirationTime(now());
    return currentEventTime;
  }
```

3. requestCurrentSuspenseConfig

直接过

4. computeExpirationForFiber

来到了实际给fiber计算expiration time的地方，也是大概区分下不同模式的计算地方，具体看不明白。。。

这里要记录下按位与和按位或的神奇用法。

```js
  function computeExpirationForFiber(
    currentTime: ExpirationTime,
    fiber: Fiber,
    suspenseConfig: null | SuspenseConfig,
  ): ExpirationTime {
    const mode = fiber.mode;
    // sync模式
    if ((mode & BlockingMode) === NoMode) {
      return Sync;
    }

    const priorityLevel = getCurrentPriorityLevel();
    if ((mode & ConcurrentMode) === NoMode) {
      return priorityLevel === ImmediatePriority ? Sync : Batched;
    }

    // 这里对应指定context模式，也就是外部强制更新
    if ((executionContext & RenderContext) !== NoContext) {
      // Use whatever time we're already rendering
      return renderExpirationTime;
    }

    let expirationTime;
    // 这里对应suspense模式
    if (suspenseConfig !== null) {
      // Compute an expiration time based on the Suspense timeout.
      expirationTime = computeSuspenseExpiration(
        currentTime,
        suspenseConfig.timeoutMs | 0 || LOW_PRIORITY_EXPIRATION,
      );
    } else {
      // 这里对应异步模式
      // Compute an expiration time based on the Scheduler priority.
      switch (priorityLevel) {
        case ImmediatePriority:
          expirationTime = Sync;
          break;
        case UserBlockingPriority:
          expirationTime = computeInteractiveExpiration(currentTime);
          break;
        case NormalPriority:
        case LowPriority: 
          expirationTime = computeAsyncExpiration(currentTime);
          break;
        case IdlePriority:
          expirationTime = Idle;
          break;
        default:
          invariant(false, 'Expected a valid priority level');
      }
    }

    // If we're in the middle of rendering a tree, do not update at the same
    // expiration time that is already rendering.
    if (workInProgressRoot !== null && expirationTime === renderExpirationTime) {
      // This is a trick to move this update into a separate batch
      expirationTime -= 1;
    }

    return expirationTime;
  }
```

### 参考

1. https://stackoverflow.com/questions/30795525/performance-now-vs-date-now
2. https://react.jokcy.me/book/update/expiration-time.html