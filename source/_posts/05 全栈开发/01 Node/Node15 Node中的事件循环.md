---
title: Node15 Node中的事件循环
top: false
date: 2019-10-20 17:32:23  
updated: 2019-10-20 17:32:25
tags:
- 事件循环
- Event Loop
categories: Node
---

Node中的事件循环，与浏览器中的事件循环，还是有一些不同的。

<!-- more -->

## 事件循环

Node.js是单线程的语言，是通过**事件循环**处理非阻塞I/O操作的，Node会将这些操作转移到系统内核中，内核会在后台处理多种操作。当其中一个操作完成的时候，内核将通知Node将对应的回调函数加入轮询队列中。

Node的I/O处理使用了自己设计的基于事件驱动的跨平台抽象层libuv，它封装了不同操作系统的一些底层特性，对外提供统一的API，事件循环也是有libuv负责

Node中的每次事件循环都包含了6个阶段：

![](http://image.oldzhou.cn/FsuRUsI8Z65WUJoCaoRUrT2o7r2c)

（1）**timers阶段**：这个阶段执行Timer（`setTimeout`、`setInterval`）的回调函数

（2）I/O回调阶段：执行一些系统调用错误的回调（比如网络通信的错误回调函数）

（3）idle，prepare阶段：仅供Node内部使用

（4）**poll（轮询）阶段**：获取新的I/O事件，执行I/O相关的回调函数，适当的条件下将Node阻塞在这里

（5）**check阶段**：执行`setImmediate()`的回调函数

（6）close callbacks阶段：执行一些准备关闭的回调函数，比如执行Socket的`close`事件回调

重点关注`timers`、`poll`和`check`三个阶段，日常开发中的绝大部分异步任务都是在这三个阶段处理的。

## `timers`阶段

在这个阶段，Node会检查有无**超时的**Timer，如果有则把其回调函数压入timer的任务队列中等待执行。

同浏览器环境一样，Node并不能保证Timer在预设时间到了就会立即执行，因为Node对timer的过期检查不一定靠谱，它会受到系统的影响，比如下面的代码`setTimeout`和`setImmediate`的执行顺序是不确定的：

```JS
setTimeout(() => {
  console.log('timeout')
}, 0)

setImmediate(() => {
  console.log('immediate')
})
```
但是如果在一个I/O回调中，那一定是`setImmediate`先执行，因为`poll`阶段后面就是`check`阶段。

## `poll`阶段

这个阶段主要有两个功能：

1. 处理poll队列的事件
2. 如果有超时的timer，则执行timer的回调函数

在这个阶段，Event Loop会**同步执行poll**队列中的回调函数，直到队列为空，然后Event Loop会去检查`check`队列中有无预设的`setImmediate()`：

1. 有预设的`setImmediate()`，Event Loop将结束`poll`阶段进入`check`阶段，并执行`check`阶段的任务队列
2. 如果没有预设的`setImmediate()`，Event Loop检查timer队列是否为空，如果timer非空，则Event Loop开始**下一轮事件循环**
3. 如果timer队列也为空，那么Event Loop将**阻塞**在该阶段等待。

## `check`阶段

`setImmediate()`的回调函数会被加入`check`队列中

## `process.nextTick()`

从语义角度来看，`setImmediate`应该与`process.nextTick()`名字调换。`process.nextTick()`会在各个阶段之间进行，准确的说，是在当前阶段的尾部执行。一旦执行就要直到nextTick队列被清空，才会进入到下一个事件阶段。

**`nextTick`会在异步任务之前执行**。

如果递归调用，会导致Event Loop卡死。

## 与浏览器事件循环的差异

浏览器环境下，微任务Microtask的任务队列是在每个宏任务Macrotask任务执行完成后执行：

![](http://image.oldzhou.cn/Fktc_7XdXtIvBlBhwEeqskpmk8Ot)

在Node中，Microtask会在事件循环的各个阶段之间执行，也就是在一个阶段执行完毕，就回去执行Microtask队列的任务。

![](http://image.oldzhou.cn/FvvCKKRXrM1WjR8e9oe0PICPQZ9N)

## 总结

Node.js的事件循环分为了六个阶段，其中常用的是`timers`、`poll`和`check`阶段

Event Loop在每个阶段都有一个任务队列，当执行到某个阶段时将执行该阶段的任务队列，知道队列清空才会进入下一个阶段。

当所有阶段被顺序执行一次后，事件循环就完成了一个Tick。

浏览器环境和Node环境下，Microtask任务队列的执行时机不同：浏览器的Microtask在事件循环的Macrotask执行完成后执行，Node中Microtask会在各个循环阶段之间执行。

## 练习1

下面的代码在浏览器和Node环境下执行的结果各是什么：

```JS
setTimeout(() => {
  console.log('timer1')
  Promise.resolve().then(function() {
    console.log('promise1')
  })
}, 0)

setTimeout(() => {
  console.log('timer2')
  Promise.resolve().then(function() {
    console.log('promise2')
  })
}, 0)
```

浏览器环境下：

```
timer1 
promise1
timer2
promise2
```

Node环境下：

```
timer1
timer2
promise1
promise2
```

浏览器环境下比较好理解了，每次执行完一次宏任务，都要去检查并执行微任务队列。

![](http://image.oldzhou.cn/FpZSAEmHJnuc1aJwUuDoS1aIsH0n)

在Node环境下，在`timer`阶段，先执行`timer1`后将`promsie1`放到微任务队列，由于Node中的微任务队列是在各个阶段之间执行的，所以此时不会执行微任务队列，而是继续执行第二个`timer2`，所以两个`setTimeout`先后执行，执行完成后在会执行为微任务。

![](http://image.oldzhou.cn/FroBUsdEOgcaYS5P2jNE_to0Zh5v)

## 练习2

```JS
process.nextTick(function A() {
  console.log(1);
  process.nextTick(function B() {
    console.log(2);
  });
});

setTimeout(function timeout() {
  console.log('TIMEOUT FIRED');
})
```

结果是：

```
1
2
TIMEOUT FIRED
```

这是因为`process.nextTick`会在每个阶段之间进行，也可以理解为在所有阶段之前进行，它会在所有异步任务之前进行，而且其队列清空之前会持续执行。

## 练习3

```JS
setImmediate(function A() {
  console.log(1);
  setImmediate(function B() {
    console.log(2);
  });
});

setTimeout(function timeout() {
  console.log('TIMEOUT FIRED');
}, 0);
```

上面代码中，`1`和`TIMEOUT FIRED`哪个先执行是不确定的，运行结果可能是`1--TIMEOUT FIRED--2`，也可能是`TIMEOUT FIRED--1--2`。

但是如果放在了一个I/O回调中，执行顺序就是确定的：

```JS
const fs = require('fs');

fs.readFile('readme.md', err => {
  if (err) {
    console.log(err);
    return;
  }
  setImmediate(function A() {
    console.log(1);
    setImmediate(function B() {
      console.log(2);
    });
  });

  setTimeout(function timeout() {
    console.log('TIMEOUT FIRED');
  }, 0);
});
```

在一个I/O回调中，那一定是`setImmediate`先执行，因为`poll`阶段后面就是`check`阶段。



## 参考

- [深入理解js事件循环机制（Node.js篇）@lunnelv](http://lynnelv.github.io/js-event-loop-nodejs)
- [Node.js 事件循环，定时器和 process.nextTick()@Node.js文档](https://nodejs.org/zh-cn/docs/guides/event-loop-timers-and-nexttick/)
- [JavaScript 运行机制详解：再谈Event Loop@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)
