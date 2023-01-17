---
title: Koa01 中间件
top: false
date: 2019-05-05 16:50:20
updated: 2019-05-05 16:50:25
tags:
- Koa
- 中间件
categories: Koa
---

Koa 的最大特色，也是最重要的一个设计，就是中间件（middleware）。基本上，Koa 所有的功能都是通过中间件实现的。

<!-- more -->

## 概念

Koa中间件的最大特色就是中间件（middleware）的设计。

中间件是一个函数，它处在HTTP Request和HTTP Response中间，用来实现某种中间功能，通过`app.use()`来加载中间件。

```JS
const Koa = require('koa');

const app = new Koa();

app.use(async (ctx) => {
  ctx.response.body = 'GO'
});

app.listen(8080, () => {
  console.log('app is listening 8080...');
});
```

## 中间件的执行顺序

多个中间件会形成栈结构，以**先进后出**的顺序执行：

1. 最外层的中间件首先执行
2. 代用`next`函数，把执行权交给下一个中间件
3. ......
4. 最内层的中间件最后执行
5. 执行结束后，把执行权交回上一层的中间件
6. ......
7. 最外层的中间件收回执行权后，执行`next`函数后面的代码

看下面的例子：

```JS
app.use(async (ctx, next) => {
  console.log(1-1);
  ctx.response.body = 'GO';
  next();
  console.log(1-2);
});

app.use(async (ctx, next) => {
  console.log(2-1);
  next();
  console.log(2-2);
});

app.use(async (ctx, next) => {
  console.log(3-1);
  next();
  console.log(3-2);
});

app.listen(8080, () => {
  console.log('app is listening 8080...');
});
```
执行结果是：

```TEXT
1-1
2-1
3-1
3-2
2-2
1-2
```
这种先进后出的加载模型也可以叫做洋葱圈的模型：

![](http://image.oldzhou.cn/68747470733a2f2f7261772e6769746875622e636f6d2f66656e676d6b322f6b6f612d67756964652f6d61737465722f6f6e696f6e2e706e67.png)

如果中间件内部没有调用`next`函数，那么执行权就不会传递下去。

## 异步中间件

当中间件中包含异步操作时，中间件应该写成Async函数：

```JS
app.use(async (ctx, next) => {
  ctx.response.body = await fse.readFile('../demo3/test.txt', 'utf-8');
});
```

注意，如果调用`next`，必须等待完成

```JS
app.use(async (ctx, next) => {
  console.log(1);
  next();
});

app.use((ctx) => {
  ctx.response.body = await fse.readFile('../demo3/test.txt', 'utf-8');
});
```

如果是上面的形式，返回的`body`中将没有任何内容，这是因为Koa在Promise链被机械系了之后就结束了请求，这意味着我们在设置`ctx.response.body`之前，`response`就已经被发送了给客户端，正确的做法应该是在第一个中间件的`next`之前添加`await`：

```JS
app.use(async (ctx, next) => {
  console.log(1);
  await next();
});

app.use((ctx) => {
  ctx.response.body = await fse.readFile('../demo3/test.txt', 'utf-8');
});
```

当使用纯粹的Promise（不使用Async/Await）应该写成这样：

```JS
app.use((ctx, next) => {
  ctx.status = 200
  console.log('Setting status')
  // need to return here, not using async-await
  return next()
})
```


## 处理中间件中的错误

为了方便处理错误，最好使用`try...catch`将其捕获。但是为每个中间件写`try...catch`太麻烦，可以让最外层的中间件负责所有中间件的错误处理

```JS
const handler = async (ctx, next) => {
  try {
    await next();
  } catch (e) {
    ctx.response.status = e.statusCode || e.status || 500;
    ctx.response.body = {
      message: e.message
    }
  }
};

app.use(handler);

app.use(async (ctx, next) => {
  ctx.response.body = await fse.readFile('../demo3/test.txt', 'utf-8');
  await next();
});

app.use((ctx) => {
  ctx.throw(500)
});
```

运行中，没有被`catch`的错误都会触发Koa的`error`时间，监听这个事件，也可以处理错误：

```JS
const main = ctx => {
  ctx.throw(500);
};

app.on('error', (err, ctx) =>
  console.error('server error', err);
);
```
但是如果错误被`catch`捕获，就不会触发`error`事件，这时候必须调用`ctx.app.emit()`手动释放`error`事件，才能使监听函数生效。

```JS
// demos/18.js`
const handler = async (ctx, next) => {
  try {
    await next();
  } catch (err) {
    ctx.app.emit('error', err, ctx);
  }
};

const main = ctx => {
  ctx.throw(500);
};

app.on('error', function(err) {
  console.log('logging error ', err.message);
  console.log(err);
});
```

## 中间件的开发

### generator中间件的开发

Koa1中的异步流程控制使用的是Generator函数，所以Koa1的中间件都是基于generator的。

Generator中间件返回的是`function *(){}`函数：

```JS
function log(ctx) {
  console.log(ctx.method, ctx.header.host + ctx.url);
}

module.exports = function () {
  return function* f(next) {

    // 执行中间件的操作
    log(this);

    if (next) {
      yield next;
    }
  }
};
```
Generator中间件在Koa1中可以直接使用，在Koa2中需要使用koa-convert转换后进行使用

```JS
app.use(convert(loggerGenerator()));
```

### Async中间件的开发

```JS
function log(ctx) {
  console.log(2, ctx.method, ctx.header.host + ctx.url);
}

module.exports = function () {
  return async function f(ctx, next) {
    // 执行中间件的操作
    log(ctx);

    if (next) {
      await next();
    }
  }
};
```
Async中间件在Koa2中可以直接使用：

```JS
app.use(loggerAsync());
```

## 中间件引擎

### 简单实现

Koa中的中间件的加载和解析主要是通过Koa的中间件引擎`koa-compose`模块来实现的，也是Koa实现**洋葱模型**的核心引擎。

```JS
const Koa = require('koa');
let app = new Koa();

const middleware1 = async (ctx, next) => { 
  console.log(1); 
  await next();  
  console.log(6);   
}

const middleware2 = async (ctx, next) => { 
  console.log(2); 
  await next();  
  console.log(5);   
}

const middleware3 = async (ctx, next) => { 
  console.log(3); 
  await next();  
  console.log(4);   
}

app.use(middleware1);
app.use(middleware2);
app.use(middleware3);
app.use(async(ctx, next) => {
  ctx.body = 'hello world'
})

app.listen(3001)

// 启动访问浏览器
// 控制台会出现以下结果
// 1
// 2
// 3
// 4
// 5
// 6
```

上面`await next`前后的操作，很像数据结构的一种场景——栈，先进后出，并且各个中间件有着统一的上下文，便于管理、操作数据，所以Koa的中间件具有以下特性：

- 有统一的上下文对象`context`
- 执行顺序先进后出
- 通过`next`来控制先进后出的机制
- 有提前结束机制

可以简单的用Promise来实现

```JS
/**
 * Created By zh on 2019-05-05
 */
// 所以 Koa 的中间件具有以下特性：
// 1 有统一的上下文对象 context
// 2 执行顺序先进后出
// 3 通过 next 来控制先进后出的机制
// 4 有提前结束机制
// 可以使用 Promise 来做一个简单的实现

let context = {
  data: []
};

class MyKoa {
  constructor() {
    this.middlewares = [];
    this.context = {
      data: []
    }
  }
  use(middleware) {
    this.middlewares.push(middleware)
  }
  compose() {
    let count = -1;
    const dispatch = () => {
      count += 1;
      return Promise.resolve(this.middlewares[count](this.context, async () => {
        if (this.middlewares.length - 1  === count) {
          return Promise.resolve()
        }
        return dispatch()
      }))
    };
    return dispatch().then(() => {
      console.log('end');
      console.log('context = ', this.context);
    });
  }
}

async function middleware1(ctx, next) {
  console.log('action 001');
  ctx.data.push(1);
  await next();
  console.log('cation 006');
  ctx.data.push(6)
}

async function middleware2(ctx, next) {
  console.log('action 002');
  ctx.data.push(2);
  await next();
  console.log('cation 005');
  ctx.data.push(5)
}

async function middleware3(ctx, next) {
  console.log('action 003');
  ctx.data.push(3);
  await next();
  console.log('cation 004');
  ctx.data.push(4)
}

const app = new MyKoa();

app.use(middleware1);
app.use(middleware2);
app.use(middleware3);

app.compose();

// action 001
// action 002
// action 003
// cation 004
// cation 005
// cation 006
// end
// context =  { data: [ 1, 2, 3, 4, 5, 6 ] }
```

### 源码解读

核心原理就如同上面的`compose`方法所示，洋葱模型的先进后出顺序，对应`Promise.resolve`的前后操作，来看一下`koa-compose`的源码：


```JS   
const compose = middleware => {
  if (!Array.isArray(middleware)) {
    throw new TypeError('Middleware stack must be an array')
  }
  for (const fn of middleware) {
    if (typeof fn !== 'function') {
      throw new TypeError('Middleware must be composed of functions')
    }
  }
  return function (context, next) {
    let index = -1;
    return dispatch(0);

    function dispatch(i) {
      console.log(index, 888);
      console.log(i, 999);
      // 理论上 i 会大于 index，因为每次执行一次都会把 i 递增，
      // 如果相等或者小于，则说明 next() 执行了多次
      if (i <= index) {
        return Promise.reject(new Error('next() called multiple times'));
      }
      index = i;
      // 取到当前的中间件
      let fn = middleware[i];
      if (i === middleware.length) {
        fn = next;
      }
      if (!fn) {
        return Promise.resolve();
      }
      try {
        return Promise.resolve(fn(context, function () {
          return dispatch(i + 1);
        }))
      } catch (err) {
        return Promise.reject(err);
      }
    }
  }
};
```

一个中间件中是不能够调用两次`next`，这是通过`if (i <= index)`这条代码来判断的，我想了好一会，才理解这是什么原理。先把它放在这里，把主题逻辑理清楚再回过头说它。

`compose`返回了一个匿名函数，匿名函数里定义了`dispatch`函数，并传入`0`作为初始函数。

在`dispatch`函数中，`i`用于标识当前的中间件的下标（中间件通过`use`方法收集到了`middleware`这个数组中）。

然后判断`next`是否在一个中间件中多次调用（暂时略过），然后将当前的`i`赋值给`index`，`index`的唯一的作用也是用来记录当前中间件的下标，判断`next`方法调用的次数，后面再说。

接下来对`fn`赋值，获得中间件，在定义中间件时传入了两个参数，第一个就是上下文对象`ctx`，第二个参数是用来控制流程的`next`方法，这个`next`方法中我们通过执行`dispatch(i + 1)`来递归调用，执行下一个中间件。

这也是为什么我们在自己编写中间件时需要手动执行`await next()`，只有执行了`next`函数，才能正确的执行下一个中间件

在多个中间件级联执行时，第一个中间件需要等待第二个中间件返回一个resolved的Promise，也就是在`await next()`后才能继续执行剩余代码，第二个中间件同样需要等待下一个中间件返回resolved的Promise，这样就实现了洋葱圈模型的执行顺序。

所以如果要写一个Koa2的插件就应该如同上面说的一样：

```JS
async function koaMiddleware(ctx, next){
  try{
    // do something
    await next()
    // do something
  }
.catch(err){
    // handle err
  }
}
```
使用时：

```JS
app.use(koaMiddleware)
```

### 多次`next`的判定

虽然只有一行代码用来判断如果在一个中间件中执行了多次`next`方法，却真让我想了好一会才理解，还是我太笨了。


```JS
if (i <= index) return Promise.reject(new Error('next() called multiple times'))
```

正常情况下，`index`必然会小于`i`，在执行`dispatch(i+1)`时，实际上可以认为将当前中间件改变为了下一个中间件，每一个中间件都有着自己的闭包作用域，闭包中的`i`是固定的，而`index`是在闭包之外的变量，当执行到下一个中间件时`index`的值会发生改变。

如果执行了两次`next`，每个中间件的`i`是固定的，但是`index`一直在增大，出现了`i <= index`的情况，拿下面的情况举例吧：

有两个中间件：

```JS
async function middleware1(ctx, next) {
  console.log('action 001');
  await next();
  console.log('action 004');
}

async function middleware2(ctx, next) {
  console.log('action 002');
  await next();
  console.log('action 003');
}
```
同时我们在`dispatch`中打印出`index`和`i`的值：

```JS
function dispatch(i) {
  console.log('index: ', index);
  console.log('i: ', i);
  // ...
}
```
在正常情况下打印的结果：

```TEXT
index:  -1
i:  0
action 001

index:  0
i:  1
action 002

index:  1
i:  2
action 003
action 004
```

如果在第一个中间件中执行两个`next`，执行结果：

```TEXT
index:  -1
i:  0
action 001

index:  0
i:  1
action 002

index:  1
i:  2
action 003

index:  2
i:  1
something wrong--  Error: next() called multiple times
    at dispatch (/Users/duola/projects/node-learning/demo4/koa-compose.js:33:31)
    at /Users/duola/projects/node-learning/demo4/koa-compose.js:46:18
    at middleware1 (/Users/duola/projects/node-learning/demo4/koa-compose.js:87:9)
    at <anonymous>
    at process._tickCallback (internal/process/next_tick.js:188:7)
```

执行时，进入“洋葱圈”的过程是不变的，但是在执行完`action3`之后，在第一个中间件中再次执行了`next`，而在第一个中间件中，`i`是一个闭包中固定的值，为`0`，所以在执行的`dispatch`是`dispatch(1)`，在执行完`action3`之后，`index`已经变成了`2`，这时候在判断时，`i <= index`相当于`1 <= 2`是成立的，抛出了错误。


## 参考

- [Koa 框架教程@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2017/08/koa.html)
- [掌握 Koa 中间件@Joe's Blog](https://hijiangtao.github.io/2017/11/10/Mastering-Koa-Middleware/)
- [koa中间件开发和使用@Koa2进阶学习笔记](https://chenshenhai.github.io/koa2-note/note/start/middleware.html)
- [深入理解 Koa2 中间件机制@掘金](https://juejin.im/post/5a5f5a126fb9a01cb0495b4c)
