---
title: 09 ES6标准入门(Promise对象)
top: false
date: 2018-04-17 14:14:38
updated: 2019-04-11 10:44:14
tags:
- Promise
- es6
categories: ES6标准入门
---

学习Promise的笔记。

<!-- more -->

## 含义

Promise是异步编程的一种解决方案，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。

从语法上说，Promise是一个对象，从它可以获取异步操作的消息。Promise提供统一的API，各种异步操作都可以用同样的方法进行处理。

`Promise`对象有以下两个特点。

（1）对象的状态不受外界影响。

Promise对象代表一个异步操作，有三种状态：`pending`（进行中）、`fulfilled`（已成功）和 `rejected`（已失败）。==只有异步操作的结果，可以决定当前是哪一种状态==，任何其他操作都无法改变这个状态。

（2）==一旦状态改变，就不会再变，任何时候都可以得到这个结果==。
    
Promise对象的状态改变，只有两种可能：从`pending`变为`fulfilled`和从`pending`变为`rejected`。只要这两种情况发生，状态就==凝固==了，不会再变了，会一直保持这个结果，这时就称为 `resolved`（已定型）。如果改变已经发生了，再对`Promise`对象添加回调函数，也会立即得到这个结果。

Promise也有一些缺点：

1. 无法取消Promise，一旦新建它就会立即执行，无法中途取消
2. 如果不设置回调函数，Promise内部抛出的错误，不会反应到外部
3. 当处于`pending`状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。

## 基本用法

`Promise`对象是一个构造函数，用来生成`Promise`实例。

```JS
const promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```
`Promise`构造函数接受一个函数作为参数，该函数的两个参数分别是`resolve`和`reject`。它们是两个函数，由JavaScript引擎提供，不用自己部署。

- `resolve`函数的作用是，将Promise对象的状态从“未完成”变为“成功”（即从`pending`变为`resolved`），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；
- `reject` 函数的作用是，将Promise对象的状态从“未完成”变为“失败”（即从`pending`变为`rejected`），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。

`Promise`实例生成以后，可以用`then`方法分别指定`resolved`状态和`rejected`状态的回调函数。

```JS
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```
`then`方法可以接受两个回调函数作为参数
- 第一个回调函数是Promise对象的状态变为`resolved`时调用
- 第二个回调函数是Promise对象的状态变为`rejected`时调用（第二个函数是可选的，不一定要提供。这两个函数都接受Promise对象传出的值作为参数）。

一个简单的例子：

```JS
function timeout(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, ms, 'done');
  });
}

timeout(100).then((value) => {
  console.log(value);
});
```
Promise 新建后就会立即执行。

```JS
let promise = new Promise(function(resolve, reject) {
  console.log('Promise');
  resolve();
});

promise.then(function() {
  console.log('resolved.');
});

console.log('Hi!');

// Promise
// Hi!
// resolved
```
如果调用`resolve`函数和`reject`函数时带有参数，那么它们的参数会被传递给回调函数。`reject`函数的参数通常是`Error`对象的实例，表示抛出的错误；`resolve`函数的参数除了正常的值以外，还可能是另一个`Promise`实例，比如像下面这样

```JS
const p1 = new Promise(function (resolve, reject) {
  // ...
});

const p2 = new Promise(function (resolve, reject) {
  // ...
  resolve(p1);
})
```
这时`p1`的状态就会传递给`p2`，也就是说，`p1`的状态决定了`p2`的状态。如果`p1`的状态是`pending`，那么`p2`的回调函数就会等待`p1`的状态改变；如果`p1`的状态已经是`resolved`或者`rejected`，那么`p2`的回调函数将会立刻执行。

```JS
const p1 = new Promise(function (resolve, reject) {
  setTimeout(() => reject(new Error('fail')), 3000)
})

const p2 = new Promise(function (resolve, reject) {
  setTimeout(() => resolve(p1), 1000)
})

p2.then(result => console.log(result))
  .catch(error => console.log(error))
// Error: fail
```
上面的`p2`的状态在1秒之后改变，`resolve`方法返回的是`p1`，所以`p2`的状态决定于`p1`，在`p1`状态确定后`p2`的`then`和`catch`才会被执行

注意，==调用`resolve`或`reject`并不会终结`Promise`的参数函数的执行==。

一般来说，调用`resolve`或`reject`以后，`Promise`的使命就完成了，后继操作应该放到`then`方法里面，而不应该直接写在`resolve`或`reject`的后面。所以，最好在它们前面加上`return`语句，这样就不会有意外。

```JS
new Promise((resolve, reject) => {
  return resolve(1);
  // 后面的语句不会执行
  console.log(2);
})
```
## `Promise.prototype.then()`

`then`方法是定义在原型对象`Promise.prototype`上的。它的作用是为`Promise`实例添加状态改变时的回调函数。

`then`方法返回的是一个==新的==`Promise`实例（注意，不是原来那个`Promise`实例）。因此可以采用链式写法，即`then`方法后面再调用另一个`then`方法。

```JS
getJSON("/posts.json").then(function(json) {
  return json.post;
}).then(function(post) {
  // ...
});
```
采用链式的`then`，可以指定一组按照次序调用的回调函数。这时，前一个回调函数，有可能返回的还是一个`Promise`对象（即有异步操作），这时后一个回调函数，就会等待该`Promise`对象的状态发生变化，才会被调用。

```JS
getJSON("/post/1.json").then(function(post) {
  return getJSON(post.commentURL);
}).then(function funcA(comments) {
  console.log("resolved: ", comments);
}, function funcB(err){
  console.log("rejected: ", err);
});
```

上面代码中，第一个`then`方法指定的回调函数，返回的是另一个`Promise`对象。这时，第二个`then`方法指定的回调函数，就会等待这个新的`Promise`对象状态发生变化。如果变为`resolved`，就调用`funcA`，如果状态变为 `rejected`，就调用`funcB`。

## `Promise.prototype.catch()`

`Promise.prototype.catch`方法是`.then(null, rejection)`的别名，用于指定发生错误时的回调函数。

```JS
getJSON('/posts.json').then(function(posts) {
  // ...
}).catch(function(error) {
  // 处理 getJSON 和 前一个回调函数运行时发生的错误
  console.log('发生错误！', error);
});
```
上面代码中，`getJSON`方法返回一个`Promise`对象，如果该对象状态变为`resolved`，则会调用`then`方法指定的回调函数；如果异步操作抛出错误，状态就会变为`rejected`，就会调用`catch`方法指定的回调函数，处理这个错误。

另外，`then`方法指定的回调函数，如果运行中抛出错误，也会被catch方法捕获。

如果`Promise` 状态已经变成`resolved`，再抛出错误是无效的。

`Promise`对象的错误具有“冒泡”性质，==会一直向后传递==，直到被捕获为止。也就是说，错误总是会被下一个`catch`语句捕获。

```JS
getJSON('/post/1.json').then(function(post) {
  return getJSON(post.commentURL);
}).then(function(comments) {
  // some code
}).catch(function(error) {
  // 处理前面三个Promise产生的错误
});
```
==一般来说，不要在`then`方法里面定义`Reject`状态的回调函数（即 `then` 的第二个参数），总是使用`catch`方法。==

```JS
// bad
promise.then(function(data) {
    // success
  }, function(err) {
    // error
  });

// good
promise.then(function(data) { //cb
    // success
  }).catch(function(err) {
    // error
  });
```
上面代码中，第二种写法要好于第一种写法，理由是==第二种写法可以捕获前面`then`方法执行中的错误==，也更接近同步的写法（`try/catch`）。因此，建议总是使用`catch`方法，而不使用`then`方法的第二个参数。

如果没有使用`catch`方法指定错误处理的回调函数，`Promise`对象抛出的错误不会传递到外层代码，即不会有任何反应。

```JS
const someAsyncThing = function() {
  return new Promise(function(resolve, reject) {
    // 下面一行会报错，因为x没有声明
    resolve(x + 2);
  });
};

someAsyncThing().then(function() {
  console.log('everything is great');
});

setTimeout(() => { console.log(123) }, 2000);
// Uncaught (in promise) ReferenceError: x is not defined
// 123
```
上面代码中，`someAsyncThing`函数产生的`Promise`对象，内部有语法错误。浏览器运行到这一行，会打印出错误提示`ReferenceError: x is not defined`，但是不会退出进程、终止脚本执行，2 秒之后还是会输出`123`。

这就是说，`Promise`内部的错误不会影响到`Promise`外部的代码，通俗的说法就是**Promise 会吃掉错误**。

一般总是建议，`Promise`对象后面要跟`catch`方法，这样可以处理`Promise`内部发生的错误。`catch`方法返回的还是一个`Promise`对象，因此后面还可以接着调用`then`方法。

另外，==注意`catch`和`then`一样，也会返回一个可链式操作的新的Promise对象==

## `Promise.prototype.finally()`

`finally`方法用于指定不管`Promise`对象最后状态如何，都会执行的操作。该方法是ES2018引入标准的。

```JS
promise.then(result => {···})
  .catch(error => {···})
  .finally(() => {···});
```

`finally`方法的回调函数**不接受任何参数**，这意味着没有办法知道，前面的`Promise`状态到底是`fulfilled`还是`rejected`。这表明，`finally`方法里面的操作，应该是与状态无关的，不依赖于`Promise`的执行结果。

可以手写一个简单的`finally`方法（美团点评面试）：


```JS
Promise.prototype.finally = Promise.prototype.finally || function (callback) {
  if (Object.prototype.toString.call(this) !== '[object Promise]') {
    return
  }
  return Promise.resolve(
    this.then(() => callback()).catch(() => callback())
  )
}
```

## `Promise.all()`

`Promise.all`方法用于将多个`Promise`实例，包装成一个新的`Promise`实例。

```JS
const p = Promise.all([p1, p2, p3]);
```

`Promise.all`方法接受一个数组作为参数，`p1`、`p2`、`p3`都是Promise实例，如果不是，就会先调用`Promise.resolve`方法，将参数转为Promise实例，再进一步处理。（`Promise.all`方法的参数可以不是数组，但必须具有Iterator接口，且返回的每个成员都是Promise 实例。）

> **数组中的各个Promise实例同时开始**

`p`的状态由`p1`、`p2`、`p3`决定，分成两种情况。

（1）只有`p1`、`p2`、`p3`的状态都变成`fulfilled`，`p`的状态才会变成`fulfilled`，此时`p1`、`p2`、`p3`的返回值==按顺序==组成一个数组，传递给`p`的回调函数。

（2）只要`p1`、`p2`、`p3`之中有一个被`rejected`，`p`的状态就变成`rejected`，此时第一个被`reject`的实例的返回值，会传递给`p`的回调函数。

注意，如果作为参数的Promise实例，自己定义了`catch`方法，那么它一旦被`rejected`，并==不会触发==`Promise.all()`的`catch`方法。

```JS
const p1 = new Promise((resolve, reject) => {
  resolve('hello');
}).then(result => result).catch(e => e);

const p2 = new Promise((resolve, reject) => {
  throw new Error('报错了');
}).then(result => result).catch(e => e);

Promise.all([p1, p2]).then(result => console.log(result))
  .catch(e => console.log(e));
// ["hello", Error: 报错了]
```

上面代码中，`p1`会`resolved`，`p2`首先会`rejected`，但是`p2`有自己的`catch`方法，该方法返回的是一个新的Promise实例，`p2`指向的实际上是这个实例。

该实例执行完`catch`方法后，也会变成`resolved`，导致`Promise.all()`方法参数里面的两个实例都会`resolved`，因此会调用`then`方法指定的回调函数，而不会调用`catch`方法指定的回调函数。

注意，这样每个`promise`实例都要有自己的`then`方法，并且有返回值，才能被 `all` 方法的`then`接住

如果`p2`没有自己的`catch`方法，就会调用`Promise.all()`的`catch`方法。

同样可以手写一个`Promise.all`方法（滴滴面试）：

```JS
Promise.all = Promise.all || function (promiseAll) {
  let total = promiseAll.length;
  
  // 需要提前将结果数组的长度预定好，因为需要按顺序存入Promise结果
  let result = new Array(total);
  let doneCount = 0;
  
  return new Promise((resolve, reject) => {
    promiseAll.forEach(promise => {
      promise.then((value, index) => {
        result[index] = value;
        doneCount++;
        if(doneCount === total) {
          resolve(result)
        }
      }).catch(err => {
        reject(err)
      })
    })
  })
}
```

## `Promise.race()`

`Promise.race`方法同样是将多个`Promise`实例，包装成一个新的`Promise`实例。

```JS
const p = Promise.race([p1, p2, p3]);
```
上面代码中，只要`p1`、`p2`、`p3`之中有一个实例率先改变状态，`p`的状态就跟着改变。那个率先改变的`Promise`实例的返回值，就传递给`p`的回调函数。其余的`Promise`就不再改变了

> **数组中的各个Promise实例同时开始**

下面是一个例子，如果指定时间内没有获得结果，就将`Promise`的状态变为`reject`，否则变为`resolve`。

```JS
const p = Promise.race([
  fetch('/resource-that-may-take-a-while'),
  new Promise(function (resolve, reject) {
    setTimeout(() => reject(new Error('request timeout')), 5000)
  })
]);

p.then(console.log).catch(console.error);
```

手写一个`Promise.race`的实现：

```JS
Promise.race2 = Promise.race2 || function (promiseArr) {
  return new Promise((resolve, reject) => {
    promiseArr.forEach(promise => {
      promise.then(v => resolve(v)).catch(err => reject(err))
    })
  })
}
```

## `Promise.resolve()`

`Promise.resolve`方法可以将现有对象转为Promise对象

```JS
const jsPromise = Promise.resolve($.ajax('/whatever.json'));
```

上面代码将 jQuery生成的`deferred`对象，转为一个新的`Promise`对象。

`Promise.resolve`等价于下面的写法

```JS
Promise.resolve('foo')
// 等价于
new Promise(resolve => resolve('foo'))
```

`Promise.resolve`方法的参数分成四种情况

（1）参数是一个Promise实例
    
`Promise.resolve`将不做任何修改、原封不动地返回这个实例。

（2）参数是一个`thenable`对象

`thenable`对象指的是具有`then`方法的对象，`Promise.resolve`方法会将这个对象转为Promise对象，然后就立即执行改对象的`then`方法。

```JS
let thenable = {
  then: function(resolve, reject) {
    resolve(42);
  }
};
let p1 = Promise.resolve(thenable);
p1.then(function(value) {
  console.log(value);  // 42
});
```
（3）参数不是具有`then`方法的对象，或根本就不是对象

如果参数是一个原始值，或者是一个不具有`then`方法的对象，则`Promise.resolve`方法返回一个新的Promise对象，状态为`resolved`。

```JS
const p = Promise.resolve('Hello');
    
p.then(function (s){
  console.log(s)
});
// Hello
```
    
（4）不带有任何参数

`Promise.resolve`方法允许调用时不带参数，直接返回一个`resolved`状态的Promise对象。所以，==如果希望得到一个Promise对象，比较方便的方法就是直接调用`Promise.resolve`方法==
    
```JS
const p = Promise.resolve();
p.then(function () {
  // ...
});
```

## `Promise.reject()`

`Promise.reject(reason)`方法也会返回一个新的Promise实例，该实例的状态为`rejected`。

```JS
const p = Promise.reject('出错了');
// 等同于
const p = new Promise((resolve, reject) => reject('出错了'))

p.then(null, function (s) {
  console.log(s)
});
// 出错了
```

## Promise的错误处理

最好使用`catch`代替`then`里面的第二个参数来捕获错误，因为这样就可以==捕获`then`的第一个参数中发生的错误==

同时，`catch`会检测的区域是==整个==`promise`链上之前每个地方的（`then`和其他异步操作），如果它前面还有另一个`catch`，则从那个`catch`后面开始

还有就是，`catch`也会返回一个可链式操作的新Promise对象，所以如果在一个`catch`中抛出一个错误，也会被下一个`catch`捕获

如果多层嵌套时，如果内层的错误在内层有`catch`捕获，那么就不会被外层的`catch`捕获到，如果内层没有被捕获，则会“冒泡”到外层的`catch`


如果Promise的错误没有被处理，那么可以通过`unhandledrejection`来统一捕获未==处理的Promise错误==（美团面试）

注意，有两个关键词：

（1）一个是==未处理==的，如果被`catch`处理了，则不会被`unhandledrejection`捕获

（2）另一个是Pormise错误，必须是在Promise链路上发生的错误，否则也不会被捕获

使用：

```JS
window.addEventListener('unhandledrejection', e =>{
  console.log(e)
})
```
事件对象`e`是[`PromiseRejectionEvent`事件](https://developer.mozilla.org/zh-CN/docs/Web/API/PromiseRejectionEvent)，有两个属性:

- `promise`：被rejected的`Promise`
- `reason`：被rejected的原因

```JS
const p1 = new Promise((resolve) = > {
  a();
  setTimeout((a) = > {
    resolve(a)
  }, 1000, 'ok1')
});

p1.then(v = > {
  console.log(v)
});

window.addEventListener('unhandledrejection', e = > {
  console.log(e.reason)
  e.preventDefault()
})
// ReferenceError: a is not defined
```

可以通过`e.preventDefault()`来将错误拦截到此为止。

还可以监听`rejectionhandled`事件，当一个Promise发生错误，最初未被处理，稍后被处理的情况

```JS
const p1 = new Promise((resolve) => {
  a();
  setTimeout((a) => {
    resolve(a)
  }, 1000, 'ok1')
});

p1.then(v = > {
  console.log(v)
});

setTimeout(() => {
  p1.catch (v => {
    console.log(v, 'rejection')
  })
}, 1000);

window.addEventListener('unhandledrejection', e => {
  console.log(e.reason, 'unhandledrejection');
  e.preventDefault()
});
window.addEventListener('rejectionhandled', e => {
  console.log(e.reason, 'unhandledrejection')
});

// ReferenceError: a is not defined  "unhandledrejection"

// ReferenceError: a is not defined  "rejection"
```

## 实例：图片加载

一个例子，根据图片加载状态执行异步操作

```JS
const preLoadImage = (path) = > {
  return new Promise(((resolve, reject) = > {
    let img = new Image();
    img.src = path;
    img.className = 'image';
    img.onload = () = > resolve(img);
    img.onerror = () = > reject(new Error('出错了'));
  }))
};
preLoadImage('../demo03-上传图片预览/default.png')
  .then((img) = > {
    document.querySelector('#div').appendChild(img);
  }).catch ((err) = > {
    console.log(err)
  })
```

## 手动实现Promise

看了一些[参考文章](https://juejin.im/post/5a30193051882503dc53af3c)，还是迷迷糊糊，有机会要重新看（2019-04-11）

首先建立一个构造函数

```JS
function MyPromise(fn) {
  // 省略非 new 实例化方式处理
  // 省略 fn 非函数异常处理
  
  // promise 状态变量
  // 0 - pending
  // 1 - resolved
  // 2 - rejected
  this._state = 0;
  
  // promise 执行结果
  this._value = null;
  
  //  then 方法注册的回调函数
  this._deferreds = [];
  
  // 立即执行 fn 函数，调用私有方法 resolve 和 reject
  try {
    fn(
      value => {
        resolve(this, value)
      },
      reason => {
        reject(this, reason)
      }
    )
  } catch (e) {
    reject(this, e)
  }
}
```

然后来看`resolve`函数，它的目的主要是用来将promise实例的状态由pending改为resolved，它接受了两个参数，第一个参数是当前的promise实例，第二个参数是promise的执行结果

`resolve`函数中要处理的情况还是比较复杂的，主要是根据`value`的类型，这里只处理了`value`为promise和普通对象的情况，为`thanable`对象和函数的情况省略没有处理。

```JS
/**
 * 用来改变 promise 状态。
 * @param promise promise实例
 * @param value promise的执行结果
 * @returns {*}
 */
function resolve(promise, value) {
  // 非 pending 状态不可改变
  if (promise._state !== 0) {
    return;
  }
  // 如果 promise 和 x 指向同一对象，以 TypeError 为据因拒绝执行 promise
  if (value === promise) {
    return reject(promise, new TypeError('A Promise cannot be resolved with itself'))
  }
  // 如果 value 为 Promise，则使 promise 接受 value 的状态
  if (isPromise(value)) {
    const deferreds = promise._deferreds;
    if (value._state === 0) {
      // value 为 pending 状态
      // 将 promise._deferreds 传递给 value.deferreds
      // 这样，当 value 不为 pending 状态后，可以抛弃之前的 promise，以 value 作为当前的 promise 执行 then 注册函数
      value._deferreds.push(...deferreds)
    } else if (deferreds.length > 0) {
      // value 为非 pending 状态
      // 使用 value 作为当前的 promise ，执行 then 注册回调处理
      for (let i = 0; i < deferreds.length; i++) {
        // handleResolved 是实际处理
        handleResolved(value, deferreds[i])
      }
      // 清空回调函数队列
      value._deferreds = []
    }
    return;
  }
```
其中用到一个工具函数`isPromise`用来判断对象是否是一个Promise对象：

```JS
function isPromise(value) {
  return value && Object.prototype.toString.call(value) === '[object Promise]' && value.then
}
```

最后实际执行调用的是`handleResolved`函数，它不光在`resolve`函数中调用，在其他地方也被调用，它的主要目的有两个，一个是实现了当`then`注册函数为空时的透传功能，另外就是根据promise的状态来判断调用`onResolved`或`onRejected`

要注意的是需要保证异步调用，防止调用顺序错乱，使用了`asyncFn`函数来模拟异步执行。

```JS
// 根据 promise 当前状态判断调用 onResolved 或 onRejected
// 处理 then 注册回调为空的情形
// 维护 then 链式调用
function handleResolved(promise, deferred) {
  asyncFn(function () {
      const cb = promise._state === 1 ? deferred.onResolved : deferred.onRejected;
      let res;
      // 使用 deferred.promise 作为当前 promise 结合 value 调用后续处理函数继续往后执行，实现值穿透空处理函数往后传递。
      if (!cb) {
        if (promise._state === 1) {
          resolve(deferred.promise, promise._value)
        } else {
          reject(deferred.promise, promise._value)
        }
        return;
      }
      try {
        // 根据状态调用 then 中注册的 onResolved 或 onRejected 函数
        res = cb(promise._value);
      } catch (e) {
        reject(deferred.promise, e)
      }
      resolve(deferred.promise, res)
    }
  )
}

// 模拟异步执行函数
function asyncFn() {
  if (process && typeof process === 'object' && typeof (process.nextTick) === 'function') {
    return process.nextTick
  } else if (typeof setImmediate === 'function') {
    return setImmediate
  }
  return setTimeout
}
```

上面的函数中，之所以`deferred`对象之所以有`onResolved`和`onRejected`对应的方法，是因为我们在`then`函数的处理中进行了封装，下面看一下在原型上定义的`then`方法：

```JS
MyPromise.prototype.then = function (onResolved, onRejected) {
  // 实例化空 promise 对象用来返回（保持then链式调用)
  const res = new Promise(function () {
  });
  
  // 使用 onResolved，onRejected 实例化处理对象 Handler
  const deferred = new Handler(onResolved, onRejected, res);
  
  // 当前状态为 pendding，存储延迟处理对象
  if (this._state === 0) {
    this._deferreds.push(deferred);
    // 返回新 promise 对象，维持链式调用
    return res;
  }
  
  // 当前 promise 状态不为 pending
  // 调用 handleResolved 执行 onResolved 或 onRejected 回调
  handleResolved(this, deferred);
  
  // 返回新 promise 对象，维持链式调用
  return res;
}

// 封装存储 onResolved、onRejected 函数和新生成 promise 对象
function Handler(onResolved, onRejected, promise) {
  this.onResolved = typeof onResolved === 'function' ? onResolved : null;
  this.onRejected = typeof onRejected === 'function' ? onRejected : null;
  this.promise = promise;
}
```
之所以没有直接返回`this`，而是返回了一个新的Promise对象来实现链式调用，看下面的代码

```JS
var promise2 = promise1.then(function (value) {
  return Promise.reject(3)
})
```
假如`then`函数执行返回`this`调用对象本身，那么`promise2 === promise1`，`promise2`状态也应该等于`promise1`同为`resolved`。而`onResolved`回调中返回状态为`rejected`对象。考虑到Promise状态一旦`resolved`或`rejected`就不能再迁移，所以这里`promise2`也没办法转为回调函数返回的`rejected`状态，产生矛盾。

剩下的就是`reject`函数，简单得多：

```JS
function reject(promise, reason) {
  // 非 pending 状态不可变
  if (promise._state !== 0) {
    return
  }
  // 改变 promise 内部状态为 rejected
  promise._state = 2;
  promise._value = reason;
  // 判断是否存在 then 注册回调函数，如果存在则依次执行
  if (promise._deferreds.length > 0) {
    for (let i = 0; i < promise._deferreds.length; i++) {
      handleResolved(promise, promise._deferreds)
    }
    promise._deferreds = []
  }
}
```
感觉现在大概能够明白这个意思了，但是要是自己实现可能还是要费点劲，考虑不了太多的情况，还是能力差。

有时间还是要来回顾，不断整理自己的笔记。

## 参考
- [then or catch@Promise迷你书](http://liubin.org/promises-book/#then-or-catch)
- [PromiseRejectionEvent@MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/PromiseRejectionEvent)
- [unhandledrejection@MDN](https://developer.mozilla.org/zh-CN/docs/Web/Events/unhandledrejection)
- [解读Promise内部实现原理@掘金](https://juejin.im/post/5a30193051882503dc53af3c)
