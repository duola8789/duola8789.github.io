---
title: Vue源码03 Vue中nextTick的实现
top: false
date: 2018-12-12 15:43:54
updated: 2019-10-30 11:12:00
tags:
- 源码
- nextTick
categories: Vue
---

Vue中nextTick的实现。

<!-- more -->

## 使用`nextTick`

Vue中的`nextTick`的作用是，在**DOM更新完毕之后**执行一个回调。

```JS
// 修改数据
vm.msg = 'Hello'

// DOM 还没有更新

Vue.nextTick(function () {
  // DOM 更新了
})
```

通过`nextTick`，我们可以操作更新后的DOM。

## MutationObserver

MutationObserver是HTML5新增的属性，可以用来监听DOM的修改（包括节点属性、文本内容、子节点的改动）：


```JS
var observer = new MutationObserver(function(){
  //这里是回调函数
  console.log('DOM被修改了');
});

var article = document.querySelector('article');
observer.observer(article);
```

但是可惜的是，Vue并不是通过这个API实现的（曾经使用过这个API，但是并不是利用它来检测DOM的更新，并且在V2.5版本中将这个API的使用从源代码中移除了）

那么Vue究竟是如何实现的检测DOM更新的呢？

## 异步任务

我们都知道，JavaScript是一个单线程的语言，当要主线程任务完成会后，就会去读取异步任务的“任务队列”，这就是JavaScript的运行机制。具体到异步任务的任务队列，又分为宏任务（Microtasks）和微任务（Macrotasks）。在每次Event Loop的最后，最后一个UI Render的步骤，可以把它理解为一个宏任务，在这个步骤中，会更新DOM。

例如下面的代码：

```JS
for ( let i = 0; i < 100; i++) {
  dom.style.left = i + 'px';
}
```

浏览器并不会将DOM更新100次，而是在执行完同步任务（`for`循环）后才会去MacroTask队列中执行UI Render，这样只会执行一次更新。如果出现动画的效果可以使用`requestAnimationFrame`：

```JS
let i = 0;
(function changeLeft() {
  if (i < 100) {
    requestAnimationFrame(() => {
      dom.style.left = i + 'px';
      i++;
      changeLeft()
    })
  }
})()
```

## Vue是如何更新DOM的

实际上，Vue并没有检测Dom的更新，而是利用了JS的事件队列的机制来实现`nextTick`，将更新DOM的操作放在了`nextTick`中完成。

当我们在Vue中改变了某个响应式数据，它的`setter`函数会通知闭包中的`Dep`，`Dep`则会调用它管理的所有`watcher`对象，触发所有`watcher`对象的`update`方法，在`update`方法中对DOM进行更改

```JS
update() {
  /* istanbul ignore else */
  if (this.lazy) {
    this.dirty = true
  } else if (this.sync) {
    /*同步则执行run直接渲染视图*/
    this.run()
  } else {
    /*异步推送到观察者队列中，下一个tick时调用。*/
    queueWatcher(this)
  }
}
```

可以发现，Vue默认是异步进行DOM更新的，在异步更新时会调用`queueWatcher`函数：

```JS
/* 将一个观察者对象push进观察者队列，在队列中已经存在相同的id则该观察者对象将被跳过，除非它是在队列被刷新时推送 */
export function queueWatcher(watcher: Watcher) {
  /*获取watcher的id*/
  const id = watcher.id
  /*检验id是否存在，已经存在则直接跳过，不存在则标记哈希表has，用于下次检验*/
  if (has[id] == null) {
    has[id] = true
    if (!flushing) {
      /*如果没有flush掉，直接push到队列中即可*/
      queue.push(watcher)
    } else {
      // if already flushing, splice the watcher based on its id
      // if already past its id, it will be run next immediately.
      let i = queue.length - 1
      while (i >= 0 && queue[i].id > watcher.id) {
        i--
      }
      queue.splice(Math.max(i, index) + 1, 0, watcher)
    }
    // queue the flush
    if (!waiting) {
      waiting = true
      nextTick(flushSchedulerQueue)
    }
  }
}
```

`watcher`对象并没有立即更新DOM，而是被`push`进入了队列`queue`中，此时处于`waiting`状态，如果有其他的数据更新，其对应的`watcher`对象也会被推入这个队列中。等到下一个Tick运行时，这些`watcher`对象才会被从队列中遍历去除，并且更新视图。

可以发现，Vue触发的DOM更新也是通过`nextTick`实现的。

```JS
// Here we have async deferring wrappers using both microtasks and (macro) tasks.
// In < 2.4 we used microtasks everywhere, but there are some scenarios where
// microtasks have too high a priority and fire in between supposedly
// sequential events (e.g. #4521, #6690) or even between bubbling of the same
// event (#6566). However, using (macro) tasks everywhere also has subtle problems
// when state is changed right before repaint (e.g. #6813, out-in transitions).
// Here we use microtask by default, but expose a way to force (macro) task when
// needed (e.g. in event handlers attached by v-on).
let microTimerFunc
let macroTimerFunc
let useMacroTask = false

// Determine (macro) task defer implementation.
// Technically setImmediate should be the ideal choice, but it's only available
// in IE. The only polyfill that consistently queues the callback after all DOM
// events triggered in the same loop is by using MessageChannel.
/* istanbul ignore if */
if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  macroTimerFunc = () => {
    setImmediate(flushCallbacks)
  }
} else if (typeof MessageChannel !== 'undefined' && (
  isNative(MessageChannel) ||
  // PhantomJS
  MessageChannel.toString() === '[object MessageChannelConstructor]'
)) {
  const channel = new MessageChannel()
  const port = channel.port2
  channel.port1.onmessage = flushCallbacks
  macroTimerFunc = () => {
    port.postMessage(1)
  }
} else {
  /* istanbul ignore next */
  macroTimerFunc = () => {
    setTimeout(flushCallbacks, 0)
  }
}

// Determine microtask defer implementation.
/* istanbul ignore next, $flow-disable-line */
if (typeof Promise !== 'undefined' && isNative(Promise)) {
  const p = Promise.resolve()
  microTimerFunc = () => {
    p.then(flushCallbacks)
    // in problematic UIWebViews, Promise.then doesn't completely break, but
    // it can get stuck in a weird state where callbacks are pushed into the
    // microtask queue but the queue isn't being flushed, until the browser
    // needs to do some other work, e.g. handle a timer. Therefore we can
    // "force" the microtask queue to be flushed by adding an empty timer.
    if (isIOS) setTimeout(noop)
  }
} else {
  // fallback to macro
  microTimerFunc = macroTimerFunc
}

/**
 * Wrap a function so that if any code inside triggers state change,
 * the changes are queued using a (macro) task instead of a microtask.
 */
export function withMacroTask (fn: Function): Function {
  return fn._withTask || (fn._withTask = function () {
    useMacroTask = true
    const res = fn.apply(null, arguments)
    useMacroTask = false
    return res
  })
}

export function nextTick (cb?: Function, ctx?: Object) {
  let _resolve
  callbacks.push(() => {
    if (cb) {
      try {
        cb.call(ctx)
      } catch (e) {
        handleError(e, ctx, 'nextTick')
      }
    } else if (_resolve) {
      _resolve(ctx)
    }
  })
  if (!pending) {
    pending = true
    if (useMacroTask) {
      macroTimerFunc()
    } else {
      microTimerFunc()
    }
  }
  // $flow-disable-line
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(resolve => {
      _resolve = resolve
    })
  }
}

```

`nextTick`返回了一个`queueNextTick`函数，传入的`cb`会被`callbacks`存放起来，然后执行`timerFunc`，其中的`pendding`是为了保证在下一个`tick`之前只执行一次。

其中，`timeFunc`是Vue通过不同的方案构造的一个一步函数，在V2.5版本后，Vue移除了上面代码中的MutationOberver方法，首选的方案就是利用Promise，当不支持Promise时，Vue的降级方案就是使用Macro Task，降级方案依次是`setImmediate`→`MessageChannel`→`setTimeout`

> `setImmediate`是仅有高版本IE和Edge以及NodeJS支持的特性，用来把一些需要长时间运行的操作放在一个回调函数里,在浏览器完成后面的其他语句后,就立刻执行这个回调函数，属于Marco Task。
> 
> MessageChannel用来实现通道通信，在跨页面的通信总结时再仔细研究一下
> 
> 最后的兜底方案就是`setTimeout`

## 机制解析

通过这套机制，在我们响应式的更改数据后，DOM的更新默认放到Promise构造的MicroTask队列中，这时事件队列的状态：

![](http://image.oldzhou.cn/Fs4EVCGKq37SJG8EN6Jx96Xyg8nF)

这时候，DOM更新后，在Tick的一轮结束会进行UI的Rerender，我们可以再屏幕上看到更新后的数据

但是如果在对`vm.message`赋值后，立刻去取DOM中的数据，是拿不到更新后的数据的，因为如上图所示，Vue更新DOM是在异步任务中，所以可以使用`Vue.nextTick`，将取DOM数据的操作同样放到微任务队列，在更新DOM后，所可以可以拿到

![](http://image.oldzhou.cn/FkSDE4VrKaLRVXaED81H-NU8_ZnQ)

## 参考

- [LearnVue@github](https://github.com/answershuto/learnVue/blob/master/docs/Vue.js%E5%BC%82%E6%AD%A5%E6%9B%B4%E6%96%B0DOM%E7%AD%96%E7%95%A5%E5%8F%8AnextTick.MarkDown)
- [全面解析Vue.nextTick实现原理@掘金](https://juejin.im/entry/5aced80b518825482e39441e)
- [Vue源码中的nextTick的实现逻辑@segmentfault](https://segmentfault.com/a/1190000015680531)
