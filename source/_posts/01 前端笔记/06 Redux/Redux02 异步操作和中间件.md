---
title: Redux02 异步操作和中间件
top: false
date: 2019-03-07 10:00:42
updated: 2019-03-18 15:26:12
tags:
- React
- Redux
- 异步操作
categories: Redux
---

学习Redux中间件的概念，以及使用Redux中间件完成异步操作的方法。

<!-- more -->

## 同步和异步流程

先来复习一下Redux的基本流程：

```
1. 用户发出Action

2. Store自动调用Reducer，计算返回一个新的State

3. Store就会调用监听函数

4. 监听函数listener中重新渲染View
```

在第1、2步之间有一个问题，之前考虑的情况都是在Action发出之后，Reducer立刻计算出State，这是一个同步的过程。如果在Action发出之后，过一段时间再执行Reducer，这是异步过程：

```
1. 用户发出Action

1.5 异步操作（等待一段时间）

2. Store自动调用Reducer，计算返回一个新的State

3. Store就会调用监听函数

4. 监听函数listener中重新渲染View
```

现在我们希望的是在异步操作结束后，自动执行Reducer，这就要用到中间件（middleware）

## 中间件的概念

什么是中间件？中间件（middleware）是一种很常见、也很强大的模式，被广泛应用在Express、Koa、Redux等类库和框架当中。

简单来说，中间件就是在调用目标函数之前，可以随意插入其他函数预先对数据进行处理、过滤，在这个过程里面你可以打印数据、或者停止往下执行中间件等。数据就像水流一样经过中间件的层层的处理、过滤，最终到达目标函数。


```
// 中间件可以把 A 发送数据到 B 的形式从
// A -----> B
// 变成:
// A ---> middleware 1 ---> middleware 2 ---> middleware 3 --> ... ---> B
```

具体到Redux来看，如果要实现中间件，最合适环节就是在发送Action的环节，即使用中间件包裹`store.dispatch`来添加功能，比如要增加打印功能，将Action和State打印出来，我们就可以编写这样一个中间件：

```JS
const next = store.dispatch;

store.dispatch = function (action) {
  console.log('action: ', action);
  next(action);
  console.log('next state: ', store.getState())
};
```
中间件对`store.dispatch`进行了改造，在发出Action和执行Reducer之间添加了其他功能。但是实际上中间件的写法不是这样的。

在Redux中，中间件是纯函数，有明确的使用方法，并且要严格的遵循以下格式：

```JS
var anyMiddleware = function ({ dispatch, getState }) {
  return function(next) {
    return function (action) {
      // 你的中间件业务相关代码
    }
  }
}
```

中间件由三个嵌套的函数构成（会依次调用）：

（1）第一层向其余两层提供分发函数`dispatch`和`getState`函数

（2）第二层提供`next`函数，它允许你显示的将处理过的输入传递给下一个中间件或Redux（这样Redux才能调用所有reducer）。实际上`next`作为参数，就是通过`componse`传入的下一个要执行的函数，通过`next(action)`就将`action`传递给了下一中间件

（3）第三层提供从上一个中间件或者从`dispatch`传递过来的Action，这个Action可以调用下一个中间件（让Action继续流动）或者以想要的方式处理action


所以一个Log的中间件应该这样写：

```JS
function logMiddleware ({ dispatch, getState }) {
  return function(next) {
    return function (action) {
      console.log('logMiddleware action received:', action)
      return next(action)
    }
  }
}
```
`next(action)`就是继续传递Action，如果不进行这一步，所有的Action都会被丢弃。

## 中间件的用法

常用的中间件都有现成的，不用我们自行编写，只需要直接引用别人写好的模块即可，比如上面的打印日志的中间件，就可以使用现成的[redux-logger](https://github.com/LogRocket/redux-logger)模块：

```JS
import { applyMiddleware, createStore } from 'redux';
import createLogger from 'redux-logger';
const logger = createLogger();

const store = createStore(
  reducer,
  applyMiddleware(logger)
);
```
使用的时候首先通过redux-logger提供的生成方法`createLogger`创建一个中间件实例`logger`，然后将它放在Redux提供的`applyMiddleware`方法中，放到`createStore`方法中（由于`createStore`方法可以接受应用的初始状态作为第二个参数，这个时候`applyMiddleware`方法就是第三个参数了）

有的中间件有次序要求，必须放在何时的位置才能正确输出，使用之前要查看文档。

## `applyMiddleware()`

`applyMiddleware()`是Redux的原生方法，会将所有中间件组成一个数组，依次执行，下面是它的源码：


```JS
export default function applyMiddleware(...middlewares) {
  return (createStore) => (reducer, preloadedState, enhancer) => {
    var store = createStore(reducer, preloadedState, enhancer);
    var dispatch = store.dispatch;
    var chain = [];

    var middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    };
    chain = middlewares.map(middleware => middleware(middlewareAPI));
    dispatch = compose(...chain)(store.dispatch);

    return {...store, dispatch}
  }
}
```

`applyMiddleware`可以接受多个中间件作为参数，全部放进了数组`chain`中，每个中间件接受Store的`dispatch`和`getState`函数作为命名参数，返回一个函数。该函数会被传入称为`next`的下一个中间件的`dispatch`方法，并返回一个接受Action的新函数，这个函数可以直接调用`next(action)`。这个过程是通过`compose`方法完成的。

多个中间件形成了一个调用链，调用链中的最后一个中间件会接受真实Store的`dispatch`作为`next`参数，并借此结束调用链。

```JS
// 中间件函数的函数签名
({ getState, dispatch }) => next => action
```

## `compose()`

`compose(...functions)`的功能是从右到左来组合多个函数，这是函数式编程的方法，其中每个函数的返回值作为参数提供给左边的函数：

```JS
compose(funcA, funcB, funcC); 
// 同等于
funcA(funcB(funcC()))
```
关于`compose`方法，以前做过一道练习题[《前端练习17 函数式编程的compose函数》](https://blog.csdn.net/duola8789/article/details/85039248)，手写简易的`compose`方法。


```JS
const store = createStore(
  reducer,
  applyMiddleware(thunk, promise, logger)
);
```

## 异步操作的基本思路

处理异步操作需要使用中间件。

同步操作只要发出一种Action即可，异步操作的差别是要发出三种Action

```
- 操作发起时的Action
- 操作成功时的Action
- 操作失败时的Action
```
以向服务器取出数据为例，三种Action有两种不同的写法：

```JS
// 写法一，名称相同，参数不同
{ type: 'FETCH_POSTS' }
{ type: 'FETCH_POSTS', status: 'error', error: 'Oops' }
{ type: 'FETCH_POSTS', status: 'success', respose: {} }

// 写法二， 名称不同
{ type: 'FETCH_POSTS' }
{ type: 'FETCH_POSTS_FAILURE', error: 'Oops' }
{ type: 'FETCH_POSTS_SUCCESS', respose: {} }
```

除了Action种类不同，异步操作的State也要进行改造，反映不同的操作状态，例如：

```JS
const state = {
  // ...
  isFetching: true,
  didInvalidate: true,
  lastUpdated: 'xxxxxxx'
}
```

State中的属性`isFetching`表示是否正在抓取数据，`didInvalidate`表示是否正过期，`lastUpdated`表示上一次更新事件。

现在整个异步操作的思路就很清晰了：

```
1. 操作开始，发出一个Action，触发State更新为“正在操作”状态，View重新渲染
2. 操作结束，再次发出一个Action，触发State更新为“操作结束”状态，View再次重新渲染
```

## redux-thunk中间件

异步操作至少要发出两个Action，用户操作触发第一个Action，这个和同步操作一样，标识着异步操作的开始，现在要做的是在异步操作结束时，自动发送第二个Action

奥妙就在Action Creator中，需要对其进行改造。我们有一个组件，点击按钮后会发出一个Ajax请求，将返回的结果填充在视图中，按钮的点击事件如下：

```JS
sendQuestion() {
  const question = this.state.questionInput;

  // Action Creator1
  const requestPost = (question) => ({ type: 'SEND_QUESTION', status: 'sending...', question });
  
  // Action Creator2
  const receivePost = (answer) => ({ type: 'RECEIVE_ANSWER', status: '', answer });
  
   // Action Creator3
  const actionCreator = () => (dispatch, getState) => {
    dispatch(requestPost(question));
    // 重置输入框
    this.setState({
      questionInput: ''
    });
    return Request.demo2.getAnswer({ question })
      .then(res => dispatch(receivePost(res)))
  };
  store.dispatch(actionCreator())
}
```

其中最关键的就是`actionCreator`，它的返回值是一个函数，这个函数执行时，会先发出一个Action`requestPost`（由Action Creator生成）并进行其他同步操作，然后进行异步操作`Request.demo2.getAnswer({ question })`，在异步操作的回调函数中发出第二个Action`actionCreator`（由Action Creator2生成）。

上面的代码中，有几点要注意：

（1）完成异步操作的Action Creator`actionCreator`返回的是一个函数，普通的Action Creator返回的是Action对象

（2）返回的这个函数参数是`dispatch`和`getState`这两个Redux方法，普通的Action Creator参数是Action的内容。

（3）在返回的函数中，先发出的Action`dispatch(requestPost(question))`表示操作开始

（4）异步操作结束后，在发出的Action`dispatch(receivePost(res))`表示操作结束

第二点中，返回函数的两个Redux方法是执行时由函数的执行者传进去的，函数的执行者是谁呢？就是中间件[redux-thunk](https://github.com/reduxjs/redux-thunk)

为什么要使用redux-thunk？因为Action是由`store.dispatch`发出的，这个方法接受的参数是一个对象，而我们的Action Creator返回的是一个函数，使用redux-thunk对`store.dispatch`进行改造，改造后在执行Action Creator返回的函数时就传入了`dispatch`和`getState`两个参数

```JS
import { createStore, applyMiddleware } from "redux";
import reducer from "./reducers/index";

// 使用thunk中间件，使dispatch可以接受函数作为参数（默认只能接受Action对象作为参数）
import thunk from 'redux-thunk';

// 创建Store
const store = createStore(reducer, applyMiddleware(thunk));

export default store;
```

因此，异步操作的第一种解决方案就是，==编写一个返回函数的Action Creator，然后使用redux-thunk中间件改造`store.dispatch`==


## redux-promise中间件

在上面的Action Creator返回了一个函数，也可以返回其他值，另一种异步操作的解决方案，就是让Action Creator返回一个Promise对象

这需要使用redux-promise中间件

```JS
import { createStore, applyMiddleware } from "redux";
import reducer from "./reducers/index";

// 使用redux-promise中间件，使dispatch可以接受Promise作为参数
import promiseMiddleware from 'redux-promise'

// 创建Store
const store = createStore(reducer, applyMiddleware(promiseMiddleware));

export default store;
```

来看一下它的源码：

```JS
import isPromise from 'is-promise';
import { isFSA } from 'flux-standard-action';

export default function promiseMiddleware({ dispatch }) {
  return next => action => {
    if (!isFSA(action)) {
      return isPromise(action)
        ? action.then(dispatch)
        : next(action);
    }

    return isPromise(action.payload)
      ? action.payload.then(
          result => dispatch({ ...action, payload: result }),
          error => {
            dispatch({ ...action, payload: error, error: true });
            return Promise.reject(error);
          }
        )
      : next(action);
  };
}
```

如果Action本身是一个Promise，它resolve后的值是一个Action对象，会被`dispatch`方法送出，`reject`后不会有任何动作，如果Action本身不是一个Promise对象，而Action对象的`payload`属性是一个Promise对象，那么无论其resolve或reject，`dispatch`都会发出Action



所以有两种写法，一种是让Action本身返回一个Promise对象：

```JS
sendQuestion() {
  const question = this.state.questionInput;
  
  // Action Creator1
  const requestPost = (question) => ({ type: 'SEND_QUESTION', status: 'sending...', question });
  
  // Action Creator2
  const receivePost = async () => ({
    type: 'RECEIVE_ANSWER',
    status: '',
    answer: await Request.demo2.getAnswer({ question })
  });
  
  store.dispatch(requestPost(question));
  
  // 重置输入框
  this.setState({
    questionInput: ''
  });
  
  store.dispatch(receivePost());
}
```

更常见的是第二种写法，一般会配合[redux-action](https://github.com/redux-utilities/redux-actions)中间件使用。

redux-action中`createAction`的用法：

```JS
const a = createAction('test1', () => 10);
a(); 
// {type: "test1", payload: 10}


const b = createAction('test2');
b(100); 
// {type: "test2", payload: 100}
```
使用redux-action将上面的写法改为：

```JS
// 使用redux-promise中间件解决异步操作第二种写法
sendQuestion() {
  const question = this.state.questionInput;
  // Action Creator1
  const requestPost = (question) => ({type: 'SEND_QUESTION', status: 'sending...', question});
  // 发出同步Action
  store.dispatch(requestPost(question));
  // 重置输入框
  this.setState({
    questionInput: ''
  });
  // 发出异步Action
  store.dispatch(
    createAction('RECEIVE_ANSWER')(
      // Promise的then函数返回值才是createAction的第二个参数
      Request.demo2.getAnswer({question}).then(v => ({
          status: '',
          answer: v
        })
      )
    )
  );
}
```
注意，createAction的第二个参数实际上就是向要发送的Action的`payload`属性值，这里必须是一个Promise对象。（在reducer里面也必须从`action.payload`属性中获取对应的值）

明显，使用redux-promise的代码量更小一些，但是也因此失去了一定的灵活度，它的同步Action是脱离在异步操作之外单独存在的（即无法在一个Action Creator完成多个`dispatch`动作）

其他的比较热门的解决方案还有[redux-promise-middleware](https://github.com/pburtchaell/redux-promise-middleware)（感觉像是前两者的一个集合）、[redux-action-tools](https://github.com/kpaxqin/redux-action-tools)、[redux-saga](https://github.com/redux-saga/redux-saga)，可以学习[这篇文章](https://segmentfault.com/a/1190000007248878#articleHeader7)的讲解。

## 总结

学习Redux的异步操作和中间件之后，最大的体会就是太繁琐了，各种解决方案太多了。如果是复杂的项目中，有着复杂的业务逻辑，使用Redux会是一个很麻烦的事情。

以前在做一个React项目时，项目组选型使用的Mobx，当时没觉得有好用（当然也有用的比较浅的原因），但是仅仅是学习Redux，就发现Mobx或者是Vuex真的比Redux好上手太多了，Redux的函数式编程的思想带来的难度不仅是阅读、学习的难度，更是过多的范式代码带来的苦恼。

我认为会经久流传的解决方案一定会在可阅读性、可维护性以及入手难度上取得一个比较好的平衡，除非它是为了解决一些别人无法解决的问题而提出的，是一个时间段内近乎唯一的解决方案，但我感觉Redux好像并不是这样。



## 参考

- [Redux 入门教程（二）：中间件与异步操作](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_two_async_operations.html)
- [compose@Redux中文文档](https://www.redux.org.cn/docs/api/compose.html)
- [applyMiddleware@Redux中文文档](https://www.redux.org.cn/docs/api/applyMiddleware.html)
- [redux-tutorial-cn@github](https://github.com/react-guide/redux-tutorial-cn/blob/master/07_dispatch-async-action-1.js)
- [Redux异步方案选型@segmentfault](https://segmentfault.com/a/1190000007248878#articleHeader7)


