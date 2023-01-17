---
title: JS56 Generator 函数的异步应用
top: false
date: 2019-01-22 10:36:14
updated: 2019-01-22 10:36:14
tags:
- Generator
- ES6
- 异步
categories: JavaScript
---

![](/images/JS56.jpg)

Generator函数实现异步编程，利用的是协程的思想。Generator函数可以将异步流程表示的很简洁，但是流程管理不方便，有两种方式进行Generator函数的自动流程化管理，一种是利用Thunk函数，另外一种是使用Promise对象，二者结合起来就是co模块。

<!-- more -->

## 传统方法

所谓异步，简单说就是一个任务不是连续完成的，被分成了两段，先执行第一段，然后转而执行其他任务，等做好了准备，再回过头执行第二段。

在JavaScript中，ES6之前，实现的异步编程的方法有四种：

1. 回调函数
2. 事件监听
3. 发布/订阅
4. Promise

### 回调函数

所谓回调函数，就是把任务的第二段单独写在一个函数里面，等到回过头重新执行这个任务的时候，就直接调用这个函数。

第二段所需要信息和错误对象，都必须通过参数的形式传递给回调函数，这是因为程序分为两段执行，当第一段执行后，==任务所在的上下文环境就已经结束了==。在这之后的任务信息和抛出的错误，原来的上下文环境已经无法捕获，所以只能当做参数传入。

### Promise

回调函数本身没有问题，但是当多个回调函数存在的时候，会出现“回调地狱”，形成强耦合，只要有一个操作需要修改，它的上下层函数都要跟着修改

```JS
fs.readFile(fileA, 'utf-8', function (err, data) {
  fs.readFile(fileB, 'utf-8', function (err, data) {
    // ...
  });
});
```
Promise对象就是为了解决这个问题而提出的，它不是新的语法功能，而是一种新的写法，将回调函数的嵌套改为链式调用：

```JS
var readFile = require('fs-readfile-promise');

readFile(fileA)
.then(function (data) {
  console.log(data.toString());
})
.then(function () {
  return readFile(fileB);
})
.catch(function (err) {
  console.log(err);
});
```
Promise的问题是代码冗余，很多的`then`导致语义不清楚

## Generator函数

### 协程的 Generator 函数实现

Generator函数实现异步编程，利用的是协程的思想：

1. 协程`A`开始执行
2. 协程`A`执行到一半，进入暂停，执行权转移到协程`B`
3. 一段时间后，协程`B`交换执行权
4. 协程`A`恢复恢复执行

协程A就是异步任务，分为了多段执行

```JS
function* asyncJob() {
  // ...其他代码
  var f = yield readFile(fileA);
  // ...其他代码
}
```
上面的`asyncJob`就是一个协程，关键就在于`yield`命令，当程序执行到此处，`asyncJob`将执行权交给其他协程

整个Generator函数就是一个异步任务的容器，程序需要暂停的地方都需要使用`yidld`表达式

Generator之所以能够成为异步编程的旺盛解决方法，除了可以暂停执行和恢复执行之外，还因为Generator函数体内外的==数据交换和错误处理机制==。

上一篇笔记详细学习过Generator的基础知识了，[看这里](https://duola8789.github.io/2019/01/18/01%20%E5%89%8D%E7%AB%AF%E7%AC%94%E8%AE%B0/02%20JavaScript/JS55%20Generator%E5%87%BD%E6%95%B0/)。

### 异步任务的封装

看一个例子：

```JS
var fetch = require('node-fetch');

function* gen(){
  var url = 'https://api.github.com/users/github';
  var result = yield fetch(url); // fetch返回的是Promise对象
  console.log(result.bio);
}
```
执行折断代码的方法：

```JS
var g = gen();
var result = g.next();

result.value.then(function(data){
  return data.json();
}).then(function(data){
  g.next(data);
});
```
Generator函数将异步流程表示的很简洁，但是流程管理不方便，即何时执行第一段、何时执行第二段

## Thunk

### 参数的求值策略

求值策略关注的是函数的参数到底应该何时求值

有两种求值策略，一种是传值调用，即在进入函数体之前就进行计算，另一种是传名调用，即只将表达式传入函数体，旨在用到的时候求值。

传值调用有可能造成性能的浪费。

### Thunk函数的含义

编译器的传名调用，是将参数放到一个临时函数中，再将这个函数传入函数体，这个临时函数叫做Thunk函数

```JS
var thunk = function () {
  return x + 5;
};

function f(thunk) {
  return thunk() * 2;
}
```
Thunk函数是传名调用的一种实现，用来替换某个表达式

### JavaScript中的Thunk的函数

JavaScript中的Thunk函数替换的不是表达式，而是多参数函数，将多参数函数替换为只接受一个回调函数作为参数的==单参数函数==。

```JS
// 正常版本的readFile（多参数版本）
fs.readFile(fileName, callback);

// Thunk版本的readFile（单参数版本）
const Thunk = fileName => {
  return function(callback) {
    fs.readFile(fileName, callback);
  }
}

const readFileThunk = Thunk(fileName);
readFileThunk(callback)
```
任何函数，只要参数有回调函数，就能写成Thunk函数的形式，简单的Thunk函数转换器：

```JS
const Thunk = function (fn) {
  return function(...args) {
    return fucntion(callback) {
      return fn.call(this, ...args, callback)
    }
  }
}
```
使用：

```JS
const readFileThunk = Thunk(fs.readFile);
readFileThunk(fileA)(callback)
```

## Generator函数的流程管理

Thunk函数可以用于Generator函数的自动流程管理，下面的Generator函数中封装了两个异步操作：

```JS
var fs = require('fs');
var thunkify = require('thunkify');
var readFileThunk = thunkify(fs.readFile);

var gen = function* (){
  var r1 = yield readFileThunk('/etc/fstab');
  console.log(r1.toString());
  var r2 = yield readFileThunk('/etc/shells');
  console.log(r2.toString());
};
```
在使用Thunk函数管理之前，看一下如何手动执行上面这个函数：

```JS
const g = gen();

const r1 = g.next();
r1.value(funciton(err, data) {
  if (err) {
    throw err;
  }
  const r2 = g.next(data);
  r2.value(function(err, data) {
    if (err) {
      throw err;
    }
    g.next(data)
  })
})
```
为什么能够在`r1.value`里面传入一个函数呢？`r1.value`是第一个`yield`的结果，也就是`readFileThunk('/etc/fstab')`的结果，它是一个Thunk化的函数，返回值仍是一个函数，参数是回调函数：

```JS
// 相当于

const thunk1 = r1.value;
thunk1(function(err, data){
  // ...
}))
```
通过上面的代码可以发现，Generator函数的执行过程，就是将同一个回调函数返回传入`next`方法返回值的`value`属性。

这使得我们可以通过递归来自动完成这个过程

### Thunk函数的自动流程化管理

Thunk函数的真正的威力，就在于可以==自动执行==Generator函数。下面是一个基于Thunk函数的Generator执行器

```JS
function run(fn) {
  const gen = fn();
  
  function next(err, data) {
    const result = gen.next(data);
    if (result.done) {
      return;
    }
    result.value(next)
  }
}
```
有了这个执行器执行Generator函数的时候，不管内部有多少个异步操作，直接将Generator函数传入`run`函数即可（但是前提==每一个异步操作都要是Thunk函数==）

```JS
var g = function* (){
  var f1 = yield readFileThunk('fileA');
  var f2 = yield readFileThunk('fileB');
  // ...
  var fn = yield readFileThunk('fileN');
};

run(g);
```
Thunk函数并不是Generator函数自动执行的唯一方案，因为自动执行的关键是，必须==有一种机制，自动控制Generator函数的流程，接受和交换程序的执行权==。

Promise对象也可以代替回调函数做到这一点。

## co模块

co模块让你不用编写Generator函数的执行器：

```JS
const co = require('co');

var gen = function* () {
  var f1 = yield readFile('/etc/fstab');
  var f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};

co(gen).then(() => {console.log('执行完成')})
```
`co`函数返回一个Promise对象，当Generator函数执行完，可以用`then`方法添加回调函数。

### co模块的原理

co模块将两种自动执行器（Thunk函数和Promise对象）包装成一个模块，使用co模块的前提条件是，Generator函数的`yield`命令后面，只能是Thunk函数或者Promise对象（或者数组或对象的成员全都是Promise对象）

### 基于Promise对象的自动执行

同样的例子，将`fs`模块的`readFile`方法包装成为一个Promise对象：

```JS
const fs = require('fs');

const readFile = function (fileName) {
  return new Promise((resolve, reject) => {
    fs.readFile(fileName, (err, data) => {
      if (err) {
        reject(err)
      }
      resolve(data)
    })
  })
};

const gen = function* () {
  const f1 = yield readFile('/etc/filaA');
  const f2 = yield readFile('/etc/filaB');
  console.log(f1.toString());
  console.log(f2.toString());
}
```
然后手动执行上面的函数：

```JS
var g = gen();

g.next().value.then(function(data){
  g.next(data).value.then(function(data){
    g.next(data);
  });
});
```
实际上手动执行就是用`then`方法，层层添加回调函数（原理和前面的基于Thunk函数的自动执行器类似）：

```JS
function run(gen){
  var g = gen();

  function next(data){
    var result = g.next(data);
    if (result.done) return result.value;
    result.value.then(function(data){
      next(data);
    });
  }

  next();
}

run(gen);
```
### co模块的源码

```JS
function co(gen) {
  var ctx = this;

  // 接受Generator函数作为参数，返回一个Promise对象
  return new Promise(function(resolve, reject) {
    // 检查参数gen是否为Generator函数。
    // 如果是，就执行该函数，得到一个内部指针对象
    // 如果不是就返回，并将Promise对象的状态改为resolved。
    if (typeof gen === 'function') {
      gen = gen.call(ctx);
    }
    if (!gen || typeof gen.next !== 'function') {
      return resolve(gen);
    }
    
    // 将Generator函数的内部指针对象的next方法，包装成onFulfilled函数。
    // 这主要是为了能够捕捉抛出的错误。
    onFulfilled();
    function onFulfilled(res) {
      var ret;
      try {
        ret = gen.next(res);
      } catch (e) {
        return reject(e);
      }
      next(ret);
    }
    
    // next函数，它会反复调用自身
    function next(ret) {
      // 检查是否为Generator函数最后一步，是的话就返回最终结果
      if (ret.done) {
        return resolve(ret.value);
      }
      
      // 将返回结果转换为Promise对象
      var value = toPromise.call(ctx, ret.value);
      
      // 确保每一步的返回值，是 Promise 对象。
      if (value && isPromise(value)) {
        // 使用then方法，为返回值加上回调函数
        // 然后通过onFulfilled函数再次调用next函数
        return value.then(onFulfilled, onRejected);
      }
      
      // 在参数不符合要求的情况下（参数非 Thunk 函数和 Promise 对象）
      // 将Promise对象的状态改为rejected，从而终止执行。
      return onRejected(
        new TypeError(
          'You may only yield a function, promise, generator, array, or object, '
          + 'but the following object was passed: "'
          + String(ret.value)
          + '"'
        )
      );
    }
  });
}
```

## 参考
- [Generator函数的异步应用@ECMAScript6 入门](http://es6.ruanyifeng.com/#docs/generator-async)
