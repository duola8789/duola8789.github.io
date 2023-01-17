---
title: React提高06 Render Props
top: false
date: 2019-01-25 16:07:43
updated: 2019-03-19 14:43:49
tags:
- React
- 坑
categories: React
---

上一篇笔记学习了通过高阶组件实现React的代码复用，它也有着一些缺点。

除了HOC之外，还有没有别的方法呢？来，学习一下Render Props。

<!-- more -->

## HOC

HOC的出现是为了实现代码的复用，是一种React中的编程范式，基本的形式是：

```
function HOCFactory(WrappedComponent, ...args) {
  return class HOC extends React.Component {
    render(){
      return <WrappedComponent {...this.props} />
    }
  }
}
```
代码通过一个类似装饰器技术（可以参考《16 ES6标准入门(Decorator)》这篇笔记）共享，接受一个基础组件作为参数，返回了一个新的组件

```JS
const ResultComponent = HOC(BaseComponent)
```

## 一个HOC的例子

[Demo的Github地址在这里](https://github.com/duola8789/react-learning/blob/master/src/components/demo6/index.js)。

一个响应鼠标事件的HOC的例子：

```
const withMouseHOC = Component => {
  return class extends React.Component {
    state = { x: 0, y: 0 }
    
    handleMouseMove = e => {
      this.setState({
        x: e.clientX,
        y: e.clientY
      })
    }
    
    render() {
      return (
        <div onMouseMove={this.handleMouseMove.bind(this)}
          <Component {...this.props} mouse={this.state} />
        </div>
      )
    }
  }
}

class App extends React.Component {
  render() {
    const {x, y} = this.props.mouse;
    return (
      <div>
        <h1>The mouse position is ({x}, {y})</h1>
      </div>
    )
  }
}

// 也可以写成纯函数式组件
const App = props => {
  const { x, y } = props.mouse;
  return (
    <div>
      <h1>The mouse position is ({x}, {y}) </h1>
    </div>
  )
}

const AppWithMouse = withMouseHOC(App);

ReactDOM.render(<AppWithMouse />, document.getElementById('root')
```

## HOC的问题

HOC存在着几个问题：

（1）存在着多个HOC时，不知道`props`从何而来

（2）名字冲突，如果多个HOC使用了同名的`prop`，它们将发生冲突并彼此覆盖，React不会发出警告

（3）HOC使用的是静态组合而不是动态组合（结果组件被创建时发生的组合），不能再Render中调用HOC，不能使用React的生命周期

## Render Props

于是出现了另一门技术来实现代码复用，可以规避上面出现的问题

那什么是Render Prop呢？一个Render Prop是一个类型为函数的`prop`，将可复用组件的`state`作为参数传递给这个函数`prop`，返回对应的HTML模板（我理解，这也是“Render Prop”的意思吧，一个可以作为render函数返回HTML模板的`prop`）

将上面的例子改写成Render Prop的形式

```
// 用一个普通组件来共享代码
class Mouse extends React.Component {
  state = { x: 0, y: 0 }
  
  handleMouseMove = e => {
    this.setState({
      x: e.clientX,
      y: e.clientY
    })
  }
  
  // 在render函数中，利用prop.render来进行渲染
  render() {
    return (
      <div onMouseMove={this.handleMouseMove.bind(this)}
        {this.prop.render(this.state)}
        </div>
      )
    }
}

const AppWithMouse = () => {
  // 给组件的render的prop传入了一个函数
  return (
    <div>
      <Mouse render={({x, y}) => (
        <h1>The mouse position is ({x}, {y}) </h1>
      )}
    </div>
  )
}

ReactDOM.render(<AppWithMouse />, document.getElementById('root')
```

> 使用的窍门：将一个返回HTML函数作为名为`render`的`prop`，传给复用组件，复用组件中调用`this.props.render(this.state)`，渲染个性组件，最终返回最终组件。

有了Render Prop，我们可以使用一个`prop`去进行渲染，它解决了HOC的问题

（1）足够直接，可以通过Render Prop传入的函数的参数列表，有哪些`state`和`prop`可以使用

（2）不会有变量名的冲突，因为不会有任何的自动的属性合并

（3）组合模型是动态的，每次组合都是在`render`内部，可以利用React生命周期

此外，由于Render Prop仅仅是一个函数，所以不会带来过多的复杂的编程范式，更加简洁

## 用Render Prop代替HOC？

技术实现上，可以使用Render Prop代替HOC，例如可以用一个一般的、具有Render Prop的`<Mouse>`组件实现的`witchMouse`的HOC：

```
const withMouse = Component => {
  return class extends React.Component {
    render() {
      return (
        <Mouse render={mouse => (
          <Component {...this.props} mouse={mouse} />
        )} />
      )
    }
  }
}
```
但是我现在也没有具体的实践经验，是否在各种复杂的场下，Render Prop代替HOC都是更优的，还是需要实践慢慢总结证明。

## 与HOC的实现方式的区别

假设包含能够复用的逻辑的公共组件是A，需要继承的个性组件是B，HOC的实现是将用一个函数的形式，B为参数，作为`render`的内容组合至A中，返回包含B的A，也就是最终的组件：

```
const HOCFactory = B => {
  return <A />
}
// B组合至A中返回
class A extends from React.Component {
  render() {
    return <B />
  }
}
```

而Render Props方式的实现是，在A中渲染的是`this.props.render`函数的返回值，最终的组件形成是在使用A时，为A传入一个`render`参数，它的返回值就是B的内容（通过Prop传进A的内部进行渲染）

```
// A 
class A extends from React.Component {
  render() {
    return (
      <A>
        {this.props.render(this.state)}
      </A>
    )
  }
}

// 最终组件
class App extends from React.Component {
  render() {
    return (
      <A render={(state) => (<B>)} />
    )
  }
}
```


## 参考
- [[译]使用Render props吧！@掘金](https://juejin.im/post/5a3087746fb9a0450c4963a5)
- [React中的Render Props@知乎](https://zhuanlan.zhihu.com/p/31267131)
