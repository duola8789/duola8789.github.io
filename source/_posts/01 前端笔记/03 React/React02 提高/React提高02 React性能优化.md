---
title: React提高02 React性能优化
top: false
date: 2019-01-25 16:03:43
updated: 2019-01-25 16:03:43
tags:
- React
- 坑
categories: React
---

以前面试的时候，遇到过如何进行React性能优化的问题。总结了一下。

但是在实际工作中，确实没有遇到过React性能的问题，还是业务场景不复杂。希望以后能有机会实践。

<!-- more -->


## PureRenderMixin

因为react的`diff`是在某一个根节点发生变化的时候，调用所有节点进行`render`，再对生成的虚拟DOM进行对比，如果不变则不对真实DOM进行更新。这就导致了性能的浪费。

所以优化针对两方面：
1. 拆分组件，有利于组件复用和优化
2. 避免不必要的`render`

避免不要的`render`，主要基于`componentShouldUpdate(nextProps, nextState)`生命周期函数，该函数默认返回`true`，所以一旦`prop`或`state`有任何变化都会引起重新`render`

React的官方解决方案是 PureRenderMixin

ES5的写法：

```
var PureRenderMixin = require('react-addons-pure-render-mixin');
React.createClass({
  mixins: [PureRenderMixin],

  render: function() {
    return <div className={this.props.className}>foo</div>;
  }
});
```

## pure-render-decorator

ES7装饰器的写法：

```
import pureRender from "pure-render-decorator"
...

@pureRender
class Person  extends Component {
  render() {
    console.log("我re-render了");
    const {name,age} = this.props;

      return (
        <div>
          <span>姓名:</span>
          <span>{name}</span>
          <span> age:</span>
          <span>{age}</span>
        </div>
      )
  }
}
```
pureRender很简单，就是把传进来的`component`的`shouldComponentUpdate`给重写掉了，原来的`shouldComponentUpdate`，无论怎样都是`return ture`，现在不了，我要用`shallowCompare`比一比，`shallowCompare`代码及其简单

```
function shallowCompare(instance, nextProps, nextState) {
  return !shallowEqual(instance.props, nextProps) || !shallowEqual(instance.state, nextState);
}
```

## PureComponent

React在`15.3.0`里面加入了了`React.PureComponent` - 一个可继承的新的基础类, 用来替换`react-addons-pure-render-mixin`。用法：

```
class CounterButton extends React.PureComponent {
  constructor(props) {
    super(props);
    this.state = {count: 1};
  }

  render() {
    return (
      <button
        color={this.props.color}
        onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
}
```
要注意的是，这种比较只是浅比较，在多层嵌套的对象中比较会失败

## immutable.js

Immutable Data 就是一旦创建，就不能再被更改的数据。对 Immutable 对象的任何修改或添加删除操作都会返回一个新的 Immutable 对象。

Immutable 实现的原理是 `Persistent Data Structure`（持久化数据结构），也就是使用旧数据创建新数据时，要保证旧数据同时可用且不变。同时为了避免 `deepCopy` 把所有节点都复制一遍带来的性能损耗，Immutable 使用了`Structural Sharing`（结构共享），即如果对象树中一个节点发生变化，只修改这个节点和受它影响的父节点，其它节点则进行共享。请看下面动画：

![image](https://segmentfault.com/img/bVsXeZ)

Immutable 则提供了简洁高效的判断数据是否变化的方法，只需 === 和 is 比较就能知道是否需要执行 render()，而这个操作几乎 0 成本，所以可以极大提高性能。修改后的 `shouldComponentUpdate` 是这样的：


```
import { is } from 'immutable';

shouldComponentUpdate: (nextProps = {}, nextState = {}) => {
  const thisProps = this.props || {}, thisState = this.state || {};

  if (Object.keys(thisProps).length !== Object.keys(nextProps).length ||
      Object.keys(thisState).length !== Object.keys(nextState).length) {
    return true;
  }

  for (const key in nextProps) {
    if (!is(thisProps[key], nextProps[key])) {
      return true;
    }
  }

  for (const key in nextState) {
    if (thisState[key] !== nextState[key] || !is(thisState[key], nextState[key])) {
      return true;
    }
  }
  return false;
}
```
## 无状态组件

react官方还在0.14版本中加入了无状态组件

这种组件没有状态，没有生命周期，只是简单的接受 props 渲染生成 DOM 结构。无状态组件非常简单，开销很低，如果可能的话尽量使用无状态组件。比如使用箭头函数定义：


```
// es6
const HelloMessage = (props) => <div> Hello {props.name}</div>;
render(<HelloMessage name="John" />, mountNode);
```
## 参考
- https://segmentfault.com/a/1190000007811296
- https://segmentfault.com/a/1190000010438089
