---
title: React提高11 组件间通讯
top: false
date: 2019-10-24 14:23:17
updated: 2019-10-24 14:23:19
tags:
- Redux
- Context
- Prop
categories: React
---

React与Vue一样，都是通过组件化的思想来开发各个模块，这就必然面临着组件间通信的问题。项目中的组件间关系包括如下几种：

1. 父子组件之间
2. 爷孙组件之间（就是跨级的父子组件）
3. 兄弟组件之间

各种组件间通信的方法都有不同的适用场景，下面分别学习一下。Demo中都使用了React Hooks API，顺道可以复习一下Hooks的基本用法。


<!-- more -->

## 通过传递`props`

React中，父组件向子组件传递数据是基础的组件间通信的方式，是通过`props`完成的。这种方式一般适用于父子组件之间通信。

父组件向子组件传递`props`进行通信，子组件向父组件通信则需要调用父组件通过`props`传递给子组件的函数（下面代码中的`changeMsg`），这个函数作用于是组父组件自身。子组件调用该函数，将子组件想要传递的信息作为参数，传递给父组件
``
```
const Child1 = props => {
  return (
    <div className={style['child-container']}>
      <h4 onClick={() => props.changeMsg('By Child1')}>Child1 -- {props.msg}</h4>
      <Grandson1 {...props} />
    </div>
  );
};

const Parent = () => {
  const [msg, setMsg] = useState('start');

  useEffect(() => {
    setTimeout(() => {
      setMsg('end');
    }, 2000);
  }, []);

  return (
    <div className={style['parent-container']}>
      <h3>Parent -- {msg}</h3>
      <Child1 msg={msg} changeMsg={newMsg => setMsg(newMsg)} />
    </div>
  );
};

export default function () {
  return (
    <div>
      <h2>Demo14 -- 组件间通信</h2>
      <Parent />
    </div>
  );
}
```

这种方式也适用于爷孙组件之间进行通信，但是需要通过子组件作为周转，在子组件中，使用`...`运算符将父组件的全部的`props`传递给更深层次的孙组件。孙子组件想爷爷组件传递消息也是通过`...`传递的函数进行的。

```
const Child1 = props => {
  return (
    <div className={style['child-container']}>
      <h4 onClick={() => props.changeMsg('By Child1')}>Child1 -- {props.msg}</h4>
      <Grandson1 {...props} />
    </div>
  );
};
```

至于兄弟组件之间，也可以使用这个方式进行通讯，但是必须借由相同的父组件，`Child1`将数据传递给`Parent`，再通过`Parent`将数据由`props`传递给`Child2`，组件比较多的时候不便于管理和维护。

## 观察者模式

可以引入第三方的观察者模式，来实现组件间的通信，这种方式适用于上面的所有情况。

观察者模式是一种比较常见的设计模式，观察者模式也叫做『发布-订阅』模式，发布者发布事件，订阅者监听事件并作出反应。

面试的时候也经常遇到手写建议的观察者模式的情况，下面是一种简易的实现：

```JS
export class EventEmitter {
  constructor() {
    this.events = {};
  }

  on(name, cb) {
    if (!this.events[name]) {
      this.events[name] = [];
    }
    this.events[name].push(cb);
  }

  emit(name, ...params) {
    if (!this.events[name]) {
      return;
    }
    this.events[name].forEach(cb => cb(...params));
  }

  off(name) {
    if (!this.events[name]) {
      return;
    }
    this.events[name] = [];
  }
}
```

我们在`Child2`中订阅`msg`事件，并且在订阅的回调函数中修改其内部对象：

```
const Child2 = () => {
  const [msg, setMsg] = useState('Child2');

  useEffect(() => {
    event.on('msg', newMsg => setMsg(newMsg));
  }, []);

  return (
    <div className={style['child-container']}>
      <h4>Child2 -- {msg}</h4>
    </div>
  );
};
```

然后在`Child1`中发布`msg`事件，发布的时候就可以将数据从`Child1`传递到`Child2`中（实际上发布订阅模式就是将函数进行传递，在上面的例子中，我们是将`setMsg`函数作为参数传递到了订阅的回调函数中）

```
const Child1 = props => {
  useEffect(() => {
    setTimeout(() => {
      event.emit('msg', 'Message from Child1');
    }, 3000);
  }, []);

  return (
    <div className={style['child-container']}>
      <h4 onClick={() => props.changeMsg('By Child1')}>Child1 -- {props.msg}</h4>
      <Grandson1 {...props} />
    </div>
  );
};
```

## Redux

利用Redux框架可以管理复杂的组件间通讯的状况，它可以解决文章一开始提出的多种情况的组件间通讯。

之前[学习过Redux](https://duola8789.github.io/2019/03/05/01%20%E5%89%8D%E7%AB%AF%E7%AC%94%E8%AE%B0/06%20Redux/Redux01%20%E5%85%A5%E9%97%A8/)，但是实际工作中没有用过，总是忘，但是这次回顾还是顺利的多，只不过没有实际经验，Redux的实际维护缺乏太多工程实践了。

下面的Demo使用了[React-Redux](https://duola8789.github.io/2019/03/18/01%20%E5%89%8D%E7%AB%AF%E7%AC%94%E8%AE%B0/06%20Redux/Redux03%20React-Redux/)，它提供了很多遍历的API，更加方便

首先改造了根组件：

```
import React, { useState, useEffect } from 'react';
import { connect, Provider } from 'react-redux';
import store from '@/store/';

const mapStateToProps = (state) => ({ rootMsg: state.reducer14.msg });
const mapDispatchToProps = { changeMsg: where => ({ type: 'changeMsg', payload: { where }}) };

const Demo14 = ({ rootMsg, changeMsg }) => {
  return (
    <div>
      <h2 onClick={() => changeMsg('Root')}>Demo14 -- 组件间通信 -- {rootMsg} </h2>
      <Parent />
    </div>
  );
};

const Demo14Container = connect(mapStateToProps, mapDispatchToProps)(Demo14);

export default function () {
  return (
    <Provider store={store}>
      <Demo14Container />
    </Provider>
  );
}
```

通过`connect`方法生成一个新的注入了Redux相关方法的组件，通过`mapStateToProps`把`Store`中的`state`映射为UI组件的`prop`，通过`mapDispatchToProps`将`dispatch`也映射为`props`的方法（有没有很熟悉的感觉，Vuex的`mapState`/`mapMutations`），通过`<Provider>`注入`store`。

同样，也通过`connect`方法生成包装后的`Grandson2Container`组件：

```
const Grandson2 = ({ rootMsg, changeMsg }) => {
  return (
    <div className={style['grandson-container']}>
      <h5 onClick={() => changeMsg('Grandson1')}>Grandson1 -- {rootMsg}</h5>
    </div>
  );
};

const Grandson2Container = connect(mapStateToProps, mapDispatchToProps)(Grandson2);
```

这样，根组件和`Grandson2`就通过Redux实现了通信，兄弟组件之间也同样可以实现。

## Content API

之前学习过[React的Content API](https://duola8789.github.io/2019/01/25/01%20%E5%89%8D%E7%AB%AF%E7%AC%94%E8%AE%B0/03%20React/React02%20%E6%8F%90%E9%AB%98/React%E6%8F%90%E9%AB%9807%20Context/)，这个API也可以实现组件间通讯的功能。

这个API有过较大的变化，Demo中使用的都是新的API。要注意的是，官方文档也提到，使用Context要谨慎，因为它会使组件重用更加困难。


```
// 初始化一个 Context
const Context = React.createContext();

const Grandson3 = () => {
  return (
    <Context.Consumer>
      {context => (
        <div className={style['grandson-container']}>
          <h5 onClick={() => context.changeMsg('By Grandson3')}>Grandson3 -- {context.msgToGrand3}</h5>
        </div>
      )}
    </Context.Consumer>
  );
};

const Child3 = () => {
  const [msg] = useState('Child3');

  return (
    <div className={style['child-container']}>
      <h4>Child3 -- {msg}</h4>
      <Grandson3 />
    </div>
  );
};

const Parent = () => {
  const [msg, setMsg] = useState('start');

  return (
    <Context.Provider value={{
      msgToGrand3: msg,
      changeMsg: newMsg => setMsg(newMsg)
    }}>
      <div className={style['parent-container']}>
        <h3>Parent -- {msg}</h3>
        <Child3 />
      </div>
    </Context.Provider>
  );
};

export default Parent;
```

这种方式首先通过`React.createContext`初始化一个`xxxContext`，然后`xxxContext.Provider`作为顶层组件，接受一个`value`作为`prop`，将数据以对象的形式传入（包括方法），`xxxContext.Consumer`作为目标组件，其内部通过一个函数返回需要接受数据的组件，这个函数的参数就是`context`，它就是从`Provider`中传过来的数据。

这种方式在有多个数据需要传递时，不太容易管理。


还有就是Context结合Hooks，可以实现Redux类似的数据共享的作用，可以参考[这篇文章](https://fed.taobao.org/blog/2019/05/17/use-the-react-hooks-instead-of-the-redux/)。

## 参考

- [React 组件间通讯@淘宝FED](https://fed.taobao.org/blog/2016/11/18/react-components-communication/)
- [React提高07 Context@不负好时光](https://duola8789.github.io/2019/01/25/01%20%E5%89%8D%E7%AB%AF%E7%AC%94%E8%AE%B0/03%20React/React02%20%E6%8F%90%E9%AB%98/React%E6%8F%90%E9%AB%9807%20Context/)
- [Redux01 入门@不负好时光](https://duola8789.github.io/2019/03/05/01%20%E5%89%8D%E7%AB%AF%E7%AC%94%E8%AE%B0/06%20Redux/Redux01%20%E5%85%A5%E9%97%A8/)
