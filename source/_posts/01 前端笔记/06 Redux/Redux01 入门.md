---
title: Redux01 入门
top: false
date: 2019-03-05 15:43:29
updated: 2019-03-18 15:11:26 
tags:
- React
- Redux
- flux
categories: Redux
---

Redux入门学习。

<!-- more -->

Redux是一个可预测状态的JavaScript容器

## 是否需要使用Redux

Redux的出现是为了解决React的两个问题：

```
1. 代码结构
2. 组件之间的通信
```

是否使用Redux，取决于React应用是否存在上面的问题需要解决，一般来说，如果UI层比较简单，没有很多的交互，那么Redux就不是必须的，但是如果你面临的是下面的某些场景，那么就有必要考虑使用Redux了：

```
1. 用户的使用方式复杂
2. 不同身份的用户有不同的使用方式（比如普通用户和管理员）
3. 多个用户可以写作
4. 与服务器大量交互，或者使用了Websocket
5. view需要从多个来源获取数据
```

上面的情况才是Redux的适用场景：多交互、多数据源

从组件角度来看，如果组件有一下需求，可以考虑使用Redux：

```
1. 某个组件的状态需要共享
2. 某个状态需要在任何地方都能够拿到（全局状态）
3. 一个组件需要改变全局状态
4. 一个组件需要改变另一个组件的状态
```

Redux提供了一个统一的位置和机制来管理状态、组织代码结构，是Web架构的一种解决方案

## 设计思想

Redux的设计思想：

```
1. Web应用是一个状态机，视图与状态是一一对应的
2. 所有的状态保存在一个对象里面
```

这是flux单向数据流图：

```
                 _________               ____________               ___________
                |         |             |            |             |           |
                | Action  |------------▶| Dispatcher |------------▶| callbacks |
                |_________|             |____________|             |___________|
                     ▲                                                   |
                     |                                                   |
                     |                                                   |
 _________       ____|_____                                          ____▼____
|         |◀----|  Action  |                                        |         |
| Web API |     | Creators |                                        |  Store  |
|_________|----▶|__________|                                        |_________|
                     ▲                                                   |
                     |                                                   |
                 ____|________           ____________                ____▼____
                |   User       |         |   React   |              | Change  |
                | interactions |◀--------|   Views   |◀-------------| events  |
                |______________|         |___________|              |_________|
```
其中的每一个步骤和概念都是下面要介绍的一个部分，其实没有什么太多新的东西，更多是一种编程的范式和思想。

在flux的流程中，flux确保所有Action首先通过一个Dispatcher发送给Store，由Reducer计算后通知所有的监听器

## 概念

首先创建一个React应用，然后安装Redux

```
npm install redux --save
```
### Store

Store是保存数据的地方，可以把它看成一个容器。==整个应用应该只有一个Stroe==。


```JS
import { createStore } from 'redux';
const store = createStore(fn);
```

Redux提供了`createStore`函数，来生成Stroe，这个函数接受另一个函数作为参数，这个函数就是下面要提到的reducer，返回新生成的Stroe对象。

往`createStore`传Reducer的过程就是给Redux绑定Action处理函数（也就是Reducer）的过程

### State

Store对象包含所有数据，想要得到某一时刻（状态）下的数据，新需要对Store生成快照，生成的快照数据就是State

当前时刻的State，可以通过`Store.getState`获得：

```JS
import { createStore } from 'redux';
const store = createStore(fn);

const state = store.getState();
```
Redux中，一个State对应一个View，只要State相同，View就相同，反之亦然。

### Action

State的变化就会导致View的变化，但用户接触不到State，用户通过View发出通知，告诉State要发生变化了，这个通知就是Action。

Action是一个对象，必须包含一个名为`type`的属性，用来标识Action的名称，其他属性可以自由设置，可以参考这个[规范](https://github.com/redux-utilities/flux-standard-action)进行设置。

```JS
const action = {
  type: 'ADD_TODO',
  payload: 'Learn Redux'
}
```
可以这样理解，Action描述当前发生的事情，它是唯一改变State的方法，将数据从View运送到Store。

### Action Creator

View要发送多少种消息，就有多少种Action，如果都手写会很麻烦。可以定义一个函数来生成Action，这个函数就是Action Creator

```JS
const ADD_TODO = '添加TODO';

funciton addTodo (text) {
  return  {
    type: ADD_TODO,
    text
  }
}
```

### Dispatch

有了Action，还需要一种行为将Action由View传递到Store，这种行为就是dispatch，`store.dispatch`是View发出Action的唯一方法


```JS
import { createStore } from 'redux';
const store = createStore(fn);

const action = {
  type: '添加TODO',
  payload: 'Learn Redux'
}

store.dispatch(action)
```
`store.dispatch`接受一个Action对象作为参数，并将它发送给Store，结合上面的Action Creator，可以改写为

```JS
store.dispatch(addTodo('Learn Redux'))
```

### Reducer

Store收到Action后，需要对Action进行处理，返回一个新的State，这样View才会发生变化，这种State的计算过程就是Reducer

Reducer是一个函数，它接受当前State和一个Action作为参数，返回一个新的State：

```JS
const reducer = function (state, action) {
  // ...
  return new_state;
};
```
`store.dispatch`会自动触发Reducer的自动执行（因为在使用`CreateStore`时将Reducer作为参数传递给了Store）

```JS
import { createStore } from 'redux';
const store = createStore(reducer);
```

==Reducer函数最重要的特征是，它是一个纯函数==，也就是说只要是同样的输入，必定得到同样的输出。

纯函数必须遵守以下的约束：

```
- 不得改写参数
- 不得调用系统I/O的API
- 不能调用Date.now()或者Math.random()等不纯的方法，因为每次会得到不同的结果
```

Reducer函数中不能改变State对象，必须返回一个==全新的对象==，参考下面的写法：

```JS
// State 是一个对象
function reducer(state, action) {
  return Object.assign({}, state, { thingToChange });
  // 或者
  return { ...state, ...newState };
}

// State 是一个数组
function reducer(state, action) {
  return [...state, newItem];
}
```

使用上面的ES7的对象展开进行拷贝时，只是浅拷贝，如果数据结构更复杂或者是嵌套的，那在处理State更新的时候可能要考虑一些不同的做法，可以考虑使用[ImmutableJS](https://immutable-js.github.io/immutable-js/)，Redux对此是全无预设方式的，记住它只是一个状态的容器。


任何时候，与某个View对应的State总是一个不变的对象

这里有个常见模式：在Reducer里用`switch`来响应对应的Action ：

```JS
const reducer3 = function (state = {}, action) {
  console.log('reducer_3 was called with state', state, 'and action', action)

  switch (action.type) {
    case 'SAY_SOMETHING':
      return {
        ...state,
        message: action.value
      };
    default:
      return state;
  }
};
```
用`switch`的时候， ==永远==不要忘记放个`default`来返回State，否则，你的Reducer可能会返回`undefined`（等于你的State就丢了）


### Subscribe

现在，用户在View层，通过`dispatch`发出了一个Action到Stroe，触发了对应的Reducer返回了一个新的State，但是这个State和View之间还需要关联起来，才能让视图进行封信

通过`store.subscribe`可以设置监听函数，一旦State发生变化，设置的监听函数就会自动执行

```JS
import { createStore } from 'redux';
const store = createStore(reducer);

store.subscribe(listener);
```

所以需要吧View的更新函数（对于React就是组件的`render`方法或`setState`方法）放到`listener`，就可以实现View根据State对象的变化而自动更新渲染。

`store.subscribe`返回一个函数，调用这个函数可以解除监听

```JS
let unsubscribe = store.subscribe(() =>
  console.log(store.getState())
);

unsubscribe();
```

## Store的实现

Store设对象提供了三种基本方法：

```
- store.getState()
- store.dispatch()
- store.subscribe()
```

Store对象是由Redux提供的`createStore`方法创造的，这个方法除了接受一个Reducer作为第一个参数外，还接受第二个参数，表示State的初始状态，==这个状态会覆盖Reducer函数的默认参数==

下面是`createStore`的简单实现，利用了闭包的原理（也证明，如果声明两个Store对象，其中保存的State对象是相互独立的）

```JS
const createStore = reducer => {
  let state = reducer(undefined, {});
  let listeners = [];

  const getState = () => state;

  const dispatch = action => {
    reducer(state, action);
    listeners.forEach(fn => {
      fn()
    })
  };

  const subscribe = listener => {
    listeners.push(listener);
    return () => {
      listeners.filter(fn => fn !== listener)
    }
  };

  return {
    getState,
    dispatch,
    subscribe
  }
};
```

## Reducer的拆分

Reducer函数负责生成State对象，但是由于整个应用只有一个State对象，包含所有数据，对于大型应用来说，这个State对象会很大

看这个例子：

```JS
const chatReducer = (state = defaultState, action = {}) => {
  const { type, payload } = action;
  switch (type) {
    case ADD_CHAT:
      return Object.assign({}, state, {
        chatLog: state.chatLog.concat(payload)
      });
    case CHANGE_STATUS:
      return Object.assign({}, state, {
        statusMessage: payload
      });
    case CHANGE_USERNAME:
      return Object.assign({}, state, {
        userName: payload
      });
    default: return state;
  }
};
```
三种Action分别改变State的三个属性：

```
- ADD_CHAT：chatLog属性
- CHANGE_STATUS：statusMessage属性
- CHANGE_USERNAME：userName属性
```

三个属性之间没有联系，因此可以将Reducer函数拆分，不同的函数负责处理不同的属性（即部分state），然后再合并为一个大的Reducer即可

```JS
const chatReducer = (state = defaultState, action = {}) => {
  return {
    chatLog: chatLog(state.chatLog, action),
    statusMessage: statusMessage(state.statusMessage, action),
    userName: userName(state.userName, action)
  }
};
```
拆分后，每一个小的函数负责修改对应的属性，返回值都是完整的State对象

> 一开始我的理解有一点问题，以为返回一个对象，对象的内容是方法，根据传入的`action.type`的值来确定执行哪个方法。并不是这样，返回的对象的每一个内容都是State对象的一个属性，当一个Action发送到Stroe后，对象中的每一个子Reducer函数都会被执行一次，这样才能返回完整的State对象
> 
> 这个时候，每个属性对应的子Reducer内部还是要根据`action.type`来判断具体逻辑的，否则会应用在所有属性上

这种拆分与React应用的结构相吻合，一个React根组件由很多子组件构成，子组件与Reducer完全可以对应。

Redux提供了`combineReducers`方法，用于Reducer的拆分，只要定义各个子Reducer函数，然后调用这个方法，将它们合成一个大的Reducer：

```JS
mport { combineReducers } from 'redux';

const chatReducer = combineReducers({
  chatLog,
  statusMessage,
  userName
})

export default chatReducer;
```
`combineReducers`产生一个整体的Reducer函数，根据State的key，分别执行子Reducer，并将返回的记过合并成为一个大的State对象

combineReducers的简单实现，注意返回的应该是一个函数（与常规的Reducer相同），给每个子Reducer传递的第一个参数不应该是整个State对象，而是对应的子对象`State[key]`

```JS
const combineReducer = reducers => (state = {}, action) => {
  Object.keys(reducers).reduce((newState, key) => {
    newState[key] = reducers[key](state[key], action);
    return newState
  }, {})
}
```
可以把所有子Reducer放在一个文件里面，然后统一引入

```JS
import { combineReducers } from 'redux'
import * as reducers from './reducers'

const reducer = combineReducers(reducers)
```

## 工作流程

之前接触过Vuex，感觉里面一些概念是非常类似的，首先来看一下Redux的工作流程

![bg2016091802.jpg](http://image.oldzhou.cn/bg2016091802.jpg)

```
// 1. 用户发出Action
store.dispatch(action)

// 2. Store自动调用Reducer，并传入两个参数，当前State和收到的Action，返回一个新的State
let newState = reducer(previousState, atcion)

// 3. 如果返回的新的State发生了变化，Store就会调用监听函数
store.subscribe(listener)

// 4. 监听函数listener中可以通过getState获取最新的State，在这里可以触发重新渲染View
function listener () {
  const newState = store.getState();
  component.setState(newState)
}
```

## 实例：计数器

我将阮一峰老师的例子稍微改写了一下，只是形式上采用了Class组件的形式

首先在`demo1.js`组件中，引入Redux

```JS
const store = createStore(reducer);
```
`reducer`是从`./reducers/index`导出的一个经由`combineReducers`合成的Reducer（`reducer2`没什么用，只是为了练习而引入的）

```JS
import { combineReducers } from 'redux'

const reducer1 = (state = 0, action) => {
  console.log('reducer1 was called with state', state, 'and action', action);
  switch (action.type) {
    case 'INCREMENT': {
      return +state + 1
    }
    case 'DECREMENT': {
      return +state - 1
    }
    case 'CHANGE': {
      return +action.payload
    }
    default: {
      return +state
    }
  }
};

const reducer2 = (state = { test: 'go' }, action) => {
  if (action.type === 'INCREMENT') {
    console.log('reducer2 was called with state', state, 'and action', action);
  }
  return state
};

const reducer = combineReducers({
  val1: reducer1,
  val2: reducer2
});

export default reducer
```
在`<Demo1>`中，`<Count>`是一个函数式组件，只负责表现，具体的逻辑在`<Demo1>`中，通过`store.getState`来为`this.state`中的变量赋值，当用户点击按钮而使用`dispatch(action)`发出了Action，State变化，会触发监听函数变化，监听函数中再来调用`setState`来触发视图的更新

```
const Count = ({ value, onIncrement, onDecrement, onChange }) => {
  return (
    <div>
      <button onClick={ onIncrement }>+</button>
      <span>Now, the count is { value }</span>
      <button onClick={ onDecrement }>-</button>
      <input type="number" onInput={ onChange } />
    </div>
  )
};

export default class Demo1 extends Component {
  constructor(props) {
    super(props);
    this.state = {
      val1: store.getState().val1,
      val2: store.getState().val2
    }
  }

  render() {
    const value = this.state.val1;
    const value2 = this.state.val2;

    const ACTIONS = {
      INCREMENT: 'INCREMENT',
      DECREMENT: 'DECREMENT',
      CHANGE: 'CHANGE'
    };

    store.subscribe(() => {
      this.setState({
        val1: store.getState().val1
      })
    });

    return (
      <Count value={ value }
             value2={ value2 }
             onIncrement={ () => store.dispatch({ type: ACTIONS.INCREMENT }) }
             onDecrement={ () => store.dispatch({ type: ACTIONS.DECREMENT }) }
             onChange={ (e) => store.dispatch({
               type: ACTIONS.CHANGE,
               payload: e.target.value
             }) } />
    )
  }
};
```

## 参考

- [Redux 入门教程（一）：基本用法@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html)
- [redux-tutorial-cn@github](https://github.com/react-guide/redux-tutorial-cn)
