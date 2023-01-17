---
title: React提高01 SetState的执行时机
top: false
date: 2019-07-24 11:08:00
updated: 2019-07-24 11:08:00
tags: 
- setState
categories: React
---

React的`setState`并不保证是同步执行的，但是也不一定就是异步执行的，准确的说是利用了队列来模拟异步执行，**并没有用到任务的异步API**。[这篇文章](https://zhuanlan.zhihu.com/p/57748690)分析了`setState`的执行机制，帮助我理解`setState`的执行时机有很大帮助。

<!-- more -->

## 执行时机

先放下结论：

1. `setState`的实现并没有涉及到任何的异步API
2. 真正更新组件`state`的是`flushBatchedUpdates`函数，而`setState`不一定会调用这个函数，有可能多次`setState`调用一次这个函数
3. `setState`会不会立刻更新`state`取决于调用`setState`时是不是已经处于批量更新事务中。
4. 组件的生命周期函数和绑定的事件回调函数都是在批量更新事务中执行的。


## 延迟调用

在`componentDidMount`生命周期中执行`setState`：

```
export default class Index extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      val: 0
    };
  }

  componentDidMount() {
    this.setState({ val: this.state.val + 1 });
    console.log('第一次 ', this.state.val);
  }

  render() {
    return (
      <div>
        {this.state.val}
      </div>
    );
  }
}
```

执行的结果是什么？

控制台会打印`第一次 0`，页面上显示`1`。这是因为`componentDidMount`生命周期是批量更新事务，在批量更新事务中调用`setState`不会立即执行，而是放到队列中等待批量更新事务结束后统一执行。

所以生命周期中打印出来的是`0`，而`render`中上一个生命周期的更新已经结束，所以页面显示`1`

同理，在两个顺序的生命周期中调用`setState`，第二个生命周期的`state`就是已经更新过的`state`：

```
export default class Index extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      val: 0
    };
  }

  componentWillMount() {
    this.setState({ val: this.state.val + 1 });
    console.log('第一次 ', this.state.val);
  }

  componentDidMount() {
    this.setState({ val: this.state.val + 1 });
    console.log('第二次 ', this.state.val);
  }


  render() {
    return (
      <div>
        {this.state.val}
      </div>
    );
  }
}
```

结果：

```
第一次 0
第二次 1

// 页面 2
```

## 立刻调用

如果要在同一个批量更新事务中立刻获得更新后的`state`，就要将相关操作放到`setState`的回调函数中：


```
export default class Index extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      val: 0
    };
  }

  componentDidMount() {
    this.setState({ val: this.state.val + 1 }, () => {
      console.log('第一次 ', this.state.val);
    });
  }

  render() {
    return (
      <div>
        {this.state.val}
      </div>
    );
  }
}
```

结果：

```
第一次 1

// 页面 1
```

## 例子

看一个更复杂一些的例子：


```
export default class Index extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      val: 0
    };
  }

  componentDidMount() {
    this.setState({ val: this.state.val + 1 });
    console.log('第一次 ', this.state.val);

    this.setState({ val: this.state.val + 1 });
    console.log('第二次 ', this.state.val);

    setTimeout(() => {
      console.log('第三次 ', this.state.val);
      
      this.setState({ val: this.state.val + 1 });
      console.log('第四次 ', this.state.val);

      this.setState({ val: this.state.val + 1 });
      console.log('第五次 ', this.state.val);
    }, 0);
  }


  render() {
    return (
      <div>
        {this.state.val}
      </div>
    );
  }
}
```

执行的结果：

```
第一次  0
第二次  0
第三次  1
第四次  2
第五次  3

// 页面 3
```

为什么前两次的结果都是`0`呢？因为前面提过了，生命周期是批量更新事务，`setState`会在更新事务结束后统一执行，所以打印的都是`0`

这两次更新后`state`的结果是`1`，并非是`2`，因为这两次调用时`this.state.val`都是`0`，相当于执行`this.setState({ val: 1 })`两次，所以第三次打印的结果是`1`

而`setTimeout`是异步任务，`setState`并非在批量更新事务中，`setState`会立即执行。所以后两次的`console.log`结果可以立刻获取到更新后的`state`

[详细的源码解读](https://zhuanlan.zhihu.com/p/57748690)放到以后慢慢看吧。

## 参考

- [你真的了解 setState 吗？@知乎](https://zhuanlan.zhihu.com/p/57748690)

