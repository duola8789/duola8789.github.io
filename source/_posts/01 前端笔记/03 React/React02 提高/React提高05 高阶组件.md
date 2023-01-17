---
title: React提高05 高阶组件
top: false
date: 2019-01-25 16:06:43
updated: 2019-01-25 16:06:43
tags:
- HOC
- 高阶组件
categories: React
---

如何更好的实现代码复用，又能够保证一定的灵活性、可维护性和可读性，需要程序员自身的技巧和能力，也需要框架更好和合理的设计实现和编程范式。

高阶组件就是React实现代码复用的一种方法。

<!-- more -->

## 什么是高阶组件

高阶组件（简称HOC）的目的就是实现代码的复用，它不是React的API，而是根据React的特性形成的一种开发范式。它接受一个组件作为参数并返回一个新的组件

```
function HOCFactory(WrappedComponent, ...args) {
  return class HOC extends React.Component {
    render(){
      return <WrappedComponent {...this.props} />
    }
  }
}
```
HOC中并没有修改输入的组件，也没有通过继承来扩展组件，而是通过==组合的方式==来达到扩展的目的

即：传入`HOCFactory`中的`WrappedComponent`是一个有个性的组件，而HOC中返回的`class`是有公共特性的组件，通过传入`args`一些配置参数，返回的就是这个特性组件和公共组件的组合组件


## HOC可以做什么

- 代码复用，代码模块化
- 增删改`props`
- 渲染劫持


（1）代码复用，代码模块化

看这样的一个例子：

加载数据、刷新数据的行为很常见，现在把这种逻辑抽离到高阶组件当中去。完成高阶组件`loadAndRefresh`，它具有以下功能：

```
class Post extends Component {
  render () {
    return (
      <div>
        <p>{this.props.content}</p>
        <button onClick={() => this.props.refresh()}>刷新</button>
      </div>
    )
  }
}

Post = loadAndRefresh('/post')(Post)
```

高阶组件`loadAndRefresh`接受一个`url`作为参数，然后返回一个接受组件作为参数的函数，这个新函数返回一个新的组件。新的组件渲染的时候会给传入的组件设置`content`和`refresh`作为`props`。

`getData(url)`的返回Promise，它返回一个字符串的文本，你需要通过`content`的`props`把它传给被包裹的组件。组件在第一次加载还有`refresh`的时候会去服务器取数据。

另外，组件在加载数据的时候，`content`显示`数据加载中...`。而且，所有传给`loadAndRefresh`返回的组件的`props`要原封不动传给被包裹的组件。

最后一句话，`loadAndRefresh`返回的组件就是返回的新组件`Post`，被包裹的就是传入的原来的`Post`，原封不动就是指需要将`this.props`完全传递，利用了对象解构的语法


```
const getData = url => new Promise((resolve) => {
  setTimeout(resolve, 2000, Date.now())
});

class Post extends Component {
  render () {
    return (
      <div>
        <p>{this.props.content}</p>
        <button onClick={() => this.props.refresh()}>刷新</button>
      </div>
    )
  }
}

const loadAndRefresh = url => (Wrapper) => {
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.state = {
        content: ''
      }
    };
    componentDidMount () {
      this.refresh()
    };
    async refresh() {
      this.setState({
        content: '数据加载中...',
      });
      const content = await getData(url);
      this.setState({
        content,
      });
    };
    render () {
      return (
        <Wrapper content={this.state.content} refresh={this.refresh.bind(this)} {...this.props}/>
      )
    }
  }
};

Post = loadAndRefresh('/post')(Post);

export default class PCHeader extends Component {
  render () {
    return (
      <Post />
    )
  }
}
```
上面的高阶组件中，接受了`Post`作为个性组件，而HOC中的公共组件部分实现的就是抽离出来的获取、刷新数据的逻辑（它也是一个组件）。

`props`的传递是通过在组件上，利用对象的扩展，将所有`prop`传入

（2）增删改`props`

HOC组件可以对传入的`props`进行修改或者添加

HOC组件会在原始组件的基础上增加一些扩展功能使用的`props`，这些`props`不应该传入到原始组件，一般会这样处理：

```
function control(wrappedComponent) {
  return class Control extends React.Component {
    render(){
      let props = {
        ...this.props,
        message: "You are under control"
      };
      return <wrappedComponent {...props} />
    }
  }
}
```
（3）渲染劫持

可以在HOC中控制是否渲染（这里控制的组件整体是否被渲染，而非组建内部的细节），无法在HOC中控制渲染细节

比如，组件在`data`没有加载完的时候加载`LOADING`，可以在HOC中这样实现：

```
function loading(wrappedComponent) {
  return class Loading extends React.Component {
    render(){
      if(!this.props.data) {
        return <div>LOADING...</div>
      }
      return <wrappedComponent {...props} />
    }
  }
}
```

## 注意事项

### 不要在`render`方法里面调用HOC方法

在`render`里面每次调用HOC都会返回一个新的`class`，重新渲染会让性能损耗加大。

### 拷贝静态方法

有的时候在组件的`class`包装的静态方法，通过HOC返回的包装后的组件就没有这些静态方法。

为了保持组件使用的一致性，一般会把这些静态方法拷贝到包装后的组件上

```JS
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  // Must know exactly which method(s) to copy :(
  Enhance.staticMethod = WrappedComponent.staticMethod;
  return Enhance;
}
```


## 例子

### logger和debugger

官网的例子，可以用来监控父级组件传入的`props`的改变：

```
function logProps(WrappedComponent) {
  return class extends React.Component {
    componentWillReceiveProps(nextProps) {
      console.log(`WrappedComponent: ${WrappedComponent.displayName}, Current props: `, this.props);
      console.log(`WrappedComponent: ${WrappedComponent.displayName}, Next props: `, nextProps);
    }
    render() {
      // Wraps the input component in a container, without mutating it. Good!
      return <WrappedComponent {...this.props} />;
    }
  }
}
```

### 页面权限管理

可以使用HOC对组件进行包裹，当组件加载的时候，首先检验用户是否有对应的权限，如果有的话就渲染页面，如果没有就跳走


## 参考
- [Higher-Order Components@React](https://reactjs.org/docs/higher-order-components.html)
- [深入React高阶组件(HOC)@掘金](https://juejin.im/post/5adddc57f265da0b8635de56)
- [React HOC高阶组件详解@掘金](https://juejin.im/post/5bbb1cd5f265da0af1615718)
