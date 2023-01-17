---
title: JS语言理解14 JS中的事件循环（Event Loop
top: false
date: 2017-12-06 19:19:04
updated: 2019-10-29 12:52:39
tags:
- 队列
- Event Loop
- MarcoTask
- MicroTask
categories: JavaScript
---
JS中的事件队列（Event Loop）学习笔记及练习。

<!-- more -->

## 同步和异步

首先要明确：

**JS是单线程语言**。

也就是说，JS一次只能做一件事情。

CPU处理指令速度非常快，远比磁盘I/O和网络I/O速度快，所以一些CPU直接执行的任务就成了优先执行主线任务（即同步任务），然后需要I/O返回数据的任务就成了等待被执行的任务（即异步任务）

- 同步任务（Asynchrono）：在主线程上排队执行的任务，前一个任务执行完毕，才能执行后一个任务；
- 异步任务（Synchrono）：不进入主线程、而进入“任务队列”（task queue）的任务，只有“任务队列”通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。

所以：**当要主线程任务完成会后，就会去读取异步任务的“任务队列”，这就是JavaScript的运行机制**

## Microtasks和Macrotasks

具体到异步任务的任务队列，又分为宏任务（Microtasks）和微任务（Macrotasks）

属于微任务的任务有：

- `Process.nextTick`
- `Promise`
- `Object.observe`（已被废弃）
- `MutationObserver`

属宏任务的任务有：

- `setTimeout`
- `setInterval`
- `setImmediate`
- `MessageChannel`
- `I/O`
- `UI渲染`

具体的执行顺序：

（1）代码开始第一次循环，执行所有主线程的同步任务，遇到异步函数，分别添加到微任务队列和宏任务队列。

（2）所有同步任务执行完成后，开始执行异步任务。

（3）首先执行微任务队列中的全部任务，在执行过程中，如果遇到新的微任务，那么会**加入到当前的微任务队列中，继续执行，直到所有的微任务执行完毕**

（4）微任务执行完成后，开始执行宏任务中的任务，在执行过程中，如果遇到微任务，会将微任务将入到微任务队列，优先执行微任务队列中的任务，微任务执行完成后返回继续执行宏任务

（5）直到所有宏任务执行完毕。

也就是说，**JavaScript在执行完主线程的同步任务后，开始执行异步任务。首先执行异步任务中的微任务队列，然后执行宏任务。在执行过程中，每次执行宏任务之前都会检查微任务队列，如果微任务队列未清空，则总会优先执行微任务**。

## 异步中的异步

面试题一般都会在异步中再次遇到异步的问题上搞事情，我比较容易犯糊涂的有下面两点。

（1）在微任务中又遇到了微任务，举例子来说明吧：

```JS
console.log(1);

setTimeout(() => {
  console.log(2);
});

Promise.resolve().then(() => {
  console.log(3);
  process.nextTick(() => {
    console.log(4);
  })
});
```

按照上面的分析，首先打印出`1`，然后将`console.log(2)`放到宏任务的队列，在然后将`console.log(3)`和`process.nextTick`放入了微任务队列：

![](http://image.oldzhou.cn/FvFPJrqlOCzMaqZhgjKXanYKZQR4)

执行完成同步任务后，首先执行微任务队列，打印出`3`之后，遇到了另外一个微任务`process.nextTick`，所以正确的顺序是，将`process.nextTick`中的代码`conosle.log(4)`**再次加入微任务队列**：

![](http://image.oldzhou.cn/Flu3jvAxoUwLMprikd87JZ3hHKML)

然后继续执行微任务队列，打印`4`，此时微任务队列已经清空，这个时候才会去执行宏任务，打印`2·

所以，正确的打印顺序是`1` → `3` → `4` → `2`

（2）在宏任务又遇到了微任务

```JS
console.log(1);

setTimeout(() => {
  console.log(2);
  Promise.resolve().then(() => {
    console.log(3);
  });
});

setTimeout(() => {
  console.log(4);
});
```

同样首先先打印了`1`，然后将`console.log(2)`以及`Promise.resolve()`后面这一堆都加入了宏任务队列，然后将`console.log(4)`也加入宏任务队列

![](http://image.oldzhou.cn/FqAC_k1q6layQhk7jboZq6W3zFFa)

由于此时没有微任务，开始执行宏任务队列，首先打印了`2`，然后执行`Promise.resovle`的代码，由于这是一个微任务，所以会将`console.log(3)`加入了微任务队列

![](http://image.oldzhou.cn/Fo1mY5lRR5mOx7zoii5DaYU0aOVn)

此时还未执行的任务中，由于`console.log(3)`是微任务，所以会优先执行，所以会先打印`3`，最后打印`4`

所以，正确的打印顺序是`1` → `2` → `3` → `4`

> 写Dome的时候，发现浏览器环境（Chrome 75）与Node（10.16）执行处的结果并不完全相同，Node环境本身执行的结果也不相同，大部分时间结果是`1243`，不知道为何，因为对Node的时间循环并不了解，留下疑问（2019-07-07）

## 多线程的浏览器内核

JavaScript是单线程的语言，但是浏览器的内核是多线程的，一个浏览器通常有以下几个常驻的线程：

- 渲染引擎线程，负责页面的渲染（Chrome/Safari/Opera用的是Webkit引擎）
- JS引擎线程，负责JS的解析和执行（Chrome用的是V8）
- 定时触发器线程，处理定时事件，比如`setTimeout`、`setInterval`
- 事件触发线程，处理DOM事件
- 异步HTTP请求线程，处理HTTP请求

要注意的是，**渲染线程和JS线程是互斥的**（不能同时进行）。渲染线程在执行任务的时候，JS引擎会被挂起，因为JS可以操作DOM，如果在渲染过程中JS处理了DOM，浏览器就无法处理了。

JS引擎可以说是JS的虚拟机，负责JS代码的解析和执行，通常包含以下几个步骤：

- 词法分析，将源代码分解为有意义的粉刺
- 语法分析，用语法分析器将分词解析成为语法树
- 代码生成，生成及其能运行的代码
- 代码执行

前面提到JavaScript是单线程的，是因为浏览器在运行时只开启了一个JS引擎来解析和执行JavaScript，这是为了避免两个线程同时操作DOM导致的互斥。

所以，**虽然JavaScript是单线程的，但是浏览器内核不是单线程的，一些I/O操作、定时器的计时、DOM的事件监听等都是由浏览器提供的其他线程完成的**。

## 事件循环对页面渲染的影响

UI线程负责页面的渲染和交互，它与JS线程一样，都受到浏览器的统一调度。

![](http://image.oldzhou.cn/FudbKAurvDAYl3XAQ1ws99MQRR4Y)

浏览器每秒会插入60个渲染帧，也就是说每16ms需要完成一次渲染。每次渲染过程中JS线程和UI线程是互斥的，如果存在一个任务16ms内未能结束，那么UI线程就无法顺利完成渲染，页面就会掉帧，给人卡顿的感觉。

[有的文章](https://harttle.land/2019/01/16/how-eventloop-affects-rendering.html)是从浏览器渲染的角度来理解事件循环对页面渲染的影响，我想也可以直接从宏任务和微任务的角度来理解。

每次Event Loop的最后，会有一个UI Render步骤，也就是更新DOM。

可以认为**UI渲染和I/O操作是MarcoTask**，所以**当页面有一个非常耗时的同步任务时，JavaScript线程会首先执行这个同步任务，然后才会去执行UI渲染**。即使将这个同步任务利用`Promise`放到MicroTask中，UI渲染仍然会被卡顿，因为JavaScript会在执行完成同步任务后首先清空MicroTask的队列，才会去执行MaroTask中的UI渲染和I/O操作的回调函数

看一个例子：

```HTML
<div id="target"></div>
<button id="button">click</button>
<script>
function changeText() {
  document.querySelector('#target').textContent = '555';
}

function calculate() {
  const start = Date.now();
  while(Date.now() - start < 3000) {}
  console.log('done')
}

document.querySelector('#button').addEventListener('click', () => {
  changeText();
  calculate();
});
</script>
```

`calculate`是一个耗时3s的同步任务，当点击`button`后，页面会立刻渲染出`555`吗？何时打印`done`的提示？

分析一下，当点击`button`后，在同步任务的队列中添加了`changeText`和`changeText`两个函数，依次执行，首先执行`changeText`，将`target`的文本值改变为`555`，但是这是改变并不会立刻显示在屏幕上，这是一个由UI渲染负责的过程，前面提到了，这是一个MacroTask，所以会将它放到MacroTask的队列中：

![](http://image.oldzhou.cn/FsDvK058fJe660xu-Z4VeF5WSqxr)

JS线程会继续执行`calculate`，3S后执行完毕后会打印`done`的提示，然后才会去执行宏任务中的UI渲染任务，此时页面才会渲染处`555`

这也就是如果JavaScript中大量的纯计算部分，会导致UI卡顿的原因。一般来说，为了解决这个卡顿，除了将计算放到服务端来做，如果是偶尔的计算任务，可以放到`setTimeout`中，避免页面的卡顿。如下进行改造：

```JS
document.querySelector('#button').addEventListener('click', () => {
  changeText();
  setTimeout(() => { calculate() })
});
```
这是，当执行完成`changText`后，遇到了`setTimeout`后，会将里面的回调函数加入到MacroTask中：

![](http://image.oldzhou.cn/FsqQwib7f4-w01r_KchsO0Gq2UPG)

这时耗时操作会在UI渲染后才开始执行，所以在执行完成`changeText`后页面立刻渲染`555`，然后才开始执行耗时操作。

如果将耗时操作放到Promise中行不行呢？答案是不行的，因为放到Promise的回调函数中会加到MircoTask队列中，JavaScript线程在同步任务执行完成后，会首先清空微任务后，才会去执行宏任务，这样耗时操作仍然在UI渲染前执行：

![](http://image.oldzhou.cn/Ftmes4QGBemFkoxKOaXrooGdc6kD)

还有一种避免因为类似计算的耗时函数导致页面卡顿，也可以将这部分操作放到[WebWorker](http://www.ruanyifeng.com/blog/2018/07/web-worker.html)中完成，相当于在主线程之外，创建了另外的Worker线程在后台进行计算，当Worker线程完成任务后再把结果返给主线程。


```JS
const worker = new Worker('worker.js');

worker.addEventListener('message', e => {
  if (e.type === 'calculate') {
    console.log(e.data);
  }
});

function changeText() {
  document.querySelector('#target').textContent = '555';
}

document.querySelector('#button').addEventListener('click', () => {
  changeText();
  worker.postMessage({type: 'calculate', data: {time: 3000}});
});
```

在`worker.js`中耗时耗时3s的计算，完成后将结果返回给主线程：

```JS
// worker.js
const events = {
  calculate({ time }) {
    const start = Date.now();
    while(Date.now() - start < time) {}
    return 'done'
  }
};

self.addEventListener('message', e => {
  const { type, data, origin } = e;
  const result = events[type](data);
  self.postMessage({ type: 'calculate', data: result }, origin);
});
```

## 练习

在面试中经常会遇到考察输出顺序的题目。

第一题

```JS
setTimeout(function() {
  console.log(4)
}, 0);
new Promise(function(resolve) {
  console.log(1)
  for (var i = 0; i < 10000; i++) {
    i == 9999 && resolve()
  }
  console.log(2)
}).then(function() {
  console.log(5)
});
requestAnimationFrame(function () {
  console.log(6)
})
console.log(3);
```

之所以`6`在`4`前面，因为`setTimeOut`即便第二个参数是0，但是HTML5标准规定了其最小值不能低于`4`毫秒，并且浏览器设置的最短间隔都在`10`毫秒左右，而`requestAnimationFrame`采用系统时间间隔，浏览器自动确定刷新频率，优先执行

> 但是今天在重温这道题目时，发现浏览（Chrome 75）的执行结果有时是`4`在`6`的前面，有时是`6`在`4`的前面，实际上这个顺序也是不确定的，不知道为何，存疑（2019-07-07）

第二题

```JS
console.log('start')
const interval = setInterval(() => {
  console.log('setInterval')
}, 0)
setTimeout(() => {
  console.log('setTimeout 1')
  Promise.resolve().then(() => {
    console.log('promise 3')
  }).then(() => {
    console.log('promise 4')
  }).then(() => {
    setTimeout(() => {
      console.log('setTimeout 2')
      Promise.resolve().then(() => {
        console.log('promise 5')
      }).then(() => {
        console.log('promise 6')
      }).then(() => {
        clearInterval(interval)
      })
    }, 0)
  })
}, 0)
Promise.resolve().then(() => {
  console.log('promise 1')
}).then(() => {
  console.log('promise 2')
})
```

第三题

```JS
console.log('start');
setTimeout(() => { console.log('s1') }, 0);
new Promise((resolve) => {
  console.log('p1');
  resolve()
}).then(v => {
  console.log('t1');
  setTimeout(() => { console.log('s2') }, 0);
  new Promise((resolve) => {
    console.log('p2');
    resolve()
  }).then(v => {
    console.log('t2')
  });
  console.log('t3');
  setTimeout(() => { console.log('s3') }, 0);
});
console.log('end');
```

第四题

面试快手时遇到的，对于在执行微任务时又遇到微任务，突然又糊涂了，傻逼一个。

```JS
console.log(1);

setTimeout(() => {
  console.log(2);
});

process.nextTick(() => {
  console.log(3);
});

setImmediate(() => {
  console.log(4);
});

new Promise(resolve => {
  console.log(5);
  resolve();
  console.log(6);
}).then(() => {
  console.log(7);
});

Promise.resolve().then(() => {
  console.log(8);
  process.nextTick(() => {
    console.log(9);
  })
})
```

第五题，结合了DOM操作。

执行下面的代码，执行后，5S内点击两下，过了5S后，再点击两下，整个过程的输出结果是什么？

```JS
setTimeout(function () {
  for (let i = 0; i < 100000000; i++) {}
  console.log('timer a');
}, 0);

for (let j = 0; j < 5; j++) {
  console.log(j);
}

setTimeout(function () {
  console.log('timer b');
}, 0);

function waitFiveSeconds() {
  let now = (new Date()).getTime();
  while (((new Date()).getTime() - now) < 5000) {}
  console.log('finished waiting');
}

document.addEventListener('click', function () {
  console.log('click');
});

console.log('click begin');
waitFiveSeconds();
```

首先这段代码会分别将两个`setTimeout`的回调函数放到异步任务的队列中，然后执行`j`循环的同步任务，当执行到`waitFiveSeconds`时，它是一个耗时5s的同步任务，在这期间JS引擎执行，渲染引擎会被挂起，也就是说页面的交互、渲染都会被卡顿。此时点击页面，点击事件不会立即执行。由于DOM事件时**同步任务**，所以点击事件的回调函数会被放到同步任务的任务队列末尾。

当5s后，JS引擎执行耗时任务完成，会首先检查同步任务队列，然后才会去执行异步任务的队列。所以会先打印两次`click`，然后才会打印定时器的操作。

```
0
1
2
3
4
5
click begin
finished waiting
click
click
timer a
timer b
```

5s后由于JS引擎已经空闲，所以点击会立刻执行。

```
click
click
```


## 参考

- [event loop js事件循环 microtask macrotask@CSDN](http://blog.csdn.net/sjn0503/article/details/76087631)
- [Promise的队列与setTimeout的队列有何关联？@知乎](https://www.zhihu.com/question/36972010)
- [详细解析JavaScript中的异步机制@掘金](https://juejin.im/post/5ccee3d16fb9a03204595cdd)
- [Web Worker 使用教程@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2018/07/web-worker.html)
