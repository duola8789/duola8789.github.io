---
title: React提高07 Context
top: false
date: 2019-01-25 16:08:43
updated: 2019-03-19 15:30:23
tags:
- React
- 坑
categories: React
---

跨组件数据传递和兄弟组件数据传递，一直是一个比较让人头痛的问题，Redux和Mobx都是很好的解决方法。

如果不使用第三方的框架，React提供了Context API来实现组件树传递数据。

<!-- more -->

## 简介

React Context API提供了一种通过组件树传递数据的方法，而不必在每个级别通过`props`属性一层一层传递。

Context提供了一个在组件树内可被视为“全局”的数据，当一些数据需要在不同的嵌套级别上被许多组件访问时，可以考虑使用Context。

请谨慎使用Context，它使组件重用更加困难。

## 旧的API

之前的使用方法是：

在上层组件（提供者）添加一个方法`getChildContext`，返回要传递的`context`对象

```
class MessageList extends React.Component {
  getChildContext() {
    return {color: "purple"};
  }
  render() {
    return (
      <Button></Button>
    )
  }
}
// 还要提前声明传递的数据的类型：
MessageList.childContextTypes = {
  color: PropTypes.string
};
```
在下层组件（使用者）中直接使用`this.context`获取属性

```
class Button extends React.Component {
  render() {
    return (
      <button>
        {this.context.color}
      </button>
    );
  }
}

// 同样要提前声明传递的数据的类型
Button.contextTypes = {
  color: PropTypes.string
};
```
它的问题是，这样的写法不符合React的声明式的写法，并且在Context值更新后，顶层组件组件向目标组件`props`透传的过程中，如果中间某个组件的`shouldComponentUpdate`返回了`false`，因为无法在继续出发底层组件的`render`，新的`Context`将无法达到目标组件。


因此React提出了新的Context API

## 新的API

新的API采用了声明式的写法，也解决了穿透`shouldComponentUpdate`的问题

看一个例子：

```
import React from "react";
import { render } from "react-dom";

// 初始化一个Context
const ThemeContext = React.createContext();

// 父组件
class FatherComponent extends React.Component {
  state = {
    color: 'blur'
  };
  
  changeColor = color => {
    this.setState({
      color
    })
  };
  
  render() {
    return (
      // XXXContext.Provider作为顶层组件接受名为value的Prop
      <ThemeContext.Provider
        value={{ 
          color: this.state.color,
          changeColor: this.changeColor
        }}
      >
    )
  }
}

// 后辈组件（目标组件）
class App extends React.Component {
  render() {
    <FatherComponent>
      <ThemeContext.Consumer>
        // XXXContext.Consumer作为目标组件接受参数，必须是一个函数
        {context => (
          <ChildComponent
            color={context.color}
            changeColor={context.changeColor}
          >
        )}
      </ThemeContext.Consumer>
    </FatherComponent>
  }
}
```
新的Context API由三个部分组成：

（1）`React.createContext`用于初始化一个Context

（2）`xxxContext.Provider`作为顶层组件，接受一个`value`的`prop`，可以接受任意需要被放入`context`中的数据和函数（注意，这个Prop的名字必须是`value`，这是React要求的，并且内部应该以对象的形式传入数据）

（3）`xxxContext.Consumer`作为目标组件，可以出现在`xxxContext.Provider`之后的组件树的任意位置，接受Child组件，这里的Child组件必须是一个函数，参数就是`context`，用来接受从顶层组件传来的Context，返回值是后辈组件。

## 例子

有这样三层组件：

父组件：

```
class Demo7 extends Component {
  state = { count: 0 };
  render() {
    return (
      <div>
        <h1>这是老子</h1>
        <button onClick={() => this.setState({ count: this.state.count + 1})}> + </button>
        <button onClick={() => this.setState({ count: this.state.count - 1})}> - </button>
        <Child />
      </div>
    )
  }
}
```

儿子组件：

```
class Child extends Component {
  render() {
    return (
      <div>
        <h2>这是儿子</h2>
        <Grandson />
      </div>
    )
  }
}
```
孙子组件

```
const Grandson = () => (
  <div>
    <h3>这是孙子</h3>
  </div>
);
```

现在，我们想让父组件的`count`属性能够被孙子组件获取到，按照原来的做法，需要一层一层传递进去：

```
const Grandson = ({ count }) => (
  <div style={{"border": "1px solid #fff", "width": "50%", "margin": "0 auto"}}>
    <h3>这是孙子</h3>
    <p>老子给的数：{count}</p>
  </div>
);

class Child extends Component {
  render() {
    return (
      <div style={{"border": "1px solid #fff", "width": "50%", "margin": "0 auto"}}>
        <h2>这是儿子</h2>
        <Grandson count={this.props.count} />
      </div>
    )
  }
}

export default class Demo7 extends Component {
  state = { count: 0 };

  render() {
    return (
      <div style={{"border": "1px solid #fff"}}>
        <h1>这是老子</h1>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}> +</button>
        <button onClick={() => this.setState({ count: this.state.count - 1 })}> -</button>
        <Child count={this.state.count} />
      </div>
    )
  }
}
```
如果改为使用Context来实现呢？那么就需要在父组件增加一个Provider的架子，将后续的组件都装进去，在孙子组件使用Consumer的架子（是一个函数，返回值是原本的结构，但是通过函数的参数来获取要传递的跨级的数据）


```
// 初始化一个Context
const Context = React.createContext();

const Grandson = () => (
  <Context.Consumer>
    {context => (
      <div style={{ "border": "1px solid #fff", "width": "50%", "margin": "0 auto" }}>
        <h3>这是孙子</h3>
        <p>老子给的数：{context.count}</p>
      </div>
    )}
  </Context.Consumer>

);

class Child extends Component {
  render() {
    return (
      <div style={{ "border": "1px solid #fff", "width": "50%", "margin": "0 auto" }}>
        <h2>这是儿子</h2>
        <Grandson />
      </div>
    )
  }
}

export default class Demo7 extends Component {
  state = { count: 0 };

  render() {
    return (
      <Context.Provider value={{count: this.state.count}}>
        <div style={{ "border": "1px solid #fff" }}>
          <h1>这是老子</h1>
          <button onClick={() => this.setState({ count: this.state.count + 1 })}> +</button>
          <button onClick={() => this.setState({ count: this.state.count - 1 })}> -</button>
          <Child />
        </div>
      </Context.Provider>
    )
  }
}
```
[Demo地址在这里](https://github.com/duola8789/react-learning/blob/master/src/components/demo7/index.js)。



## 参考

- [上下文(Context)@React中文文档](https://react.css88.com/docs/context.html)
- [如何解读 react 16.3 引入的新 context api？@知乎](https://www.zhihu.com/question/267168180/answer/319754359)



