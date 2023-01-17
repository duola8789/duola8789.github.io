---
title: React基础01 入门
top: false
date: 2017-03-23 17:43:13
updated: 2019-07-17 18:27:51
tags:
- 生命周期
categories: React
---

React入门笔记。

<!-- more -->

## 简介

React是一个用于构建用户界面的JavaScript库，主要用于构建UI，可以认为React是MVC中的 V（视图）。React起源于Facebook的内部项目，用来架设Instagram的网站，并于2013年5月开源。

## 特点

1. 声明式设计 − React采用声明范式，可以轻松描述应用。
2. 高效 − React通过对DOM的模拟，最大限度地减少与DOM的交互。
3. 灵活 − React可以与已知的库或框架很好地配合。
4. JSX − JSX是JavaScript语法的扩展。React开发不一定使用JSX，但建议使用。
5. 组件 − 通过React构建组件，使得代码更加容易得到复用，能够很好的应用在大项目的开发中。
6. 单向响应的数据流 − React实现了单向响应的数据流，减少了重复代码，这也是它为什么比传统数据绑定更简单。


## ReactJS的背景和原理

在Web开发中，我们总需要将变化的数据实时反应到UI上，这时就需要对DOM进行操作。而复杂或频繁的DOM操作通常是性能瓶颈产生的原因（如何进行高性能的复杂DOM操作通常是衡量一个前端开发人员技能的重要指标）。

React为此引入了**虚拟DOM**（Virtual DOM）的机制：在浏览器端用Javascript实现了一套DOM API。基于React进行开发时所有的DOM构造都是通过虚拟DOM进行，每当数据变化时，React都会重新构建整个DOM树，然后React将当前整个DOM树和上一次的DOM树进行对比，得到DOM结构的区别，然后仅仅将需要变化的部分进行实际的浏览器DOM更新。

而且React能够批处理虚拟DOM的刷新，在一个事件循环（Event Loop）内的两次数据变化会被合并，例如你连续的先将节点内容从A变成B，然后又从B变成A，React会认为UI不发生任何变化，而如果通过手动控制，这种逻辑通常是极其复杂的。

尽管每一次都需要构造完整的虚拟DOM树，但是因为虚拟DOM是内存数据，性能是极高的，而对实际DOM进行操作的仅仅是Diff部分，因而能达到提高性能的目的。

这样，在保证性能的同时，开发者将不再需要关注某个数据的变化如何更新到一个或多个具体的DOM元素，而只需要关心在任意一个数据状态下，整个界面是如何Render的。

服务器端Render的纯Web页面那么应该知道，服务器端所要做的就是根据数据Render出HTML送到浏览器端。如果这时因为用户的一个点击需要改变某个状态文字，那么也是通过刷新整个页面来完成的。服务器端并不需要知道是哪一小段HTML发生了变化，而只需要根据数据刷新整个页面。

换句话说，任何UI的变化都是通过**整体刷新**来完成的。而React将这种开发模式以高性能的方式带到了前端，每做一点界面的更新，你都可以认为刷新了整个页面。至于如何进行局部更新以保证性能，则是React框架要完成的事情。

## 组件化

虚拟DOM(virtual-dom)不仅带来了简单的UI开发逻辑，同时也带来了组件化开发的思想。

所谓组件，即封装起来的具有独立功能的UI部件。React推荐以组件的方式去重新思考UI构成，将UI上每一个功能相对独立的模块定义成组件，然后将小的组件通过组合或者嵌套的方式构成大的组件，最终完成整体UI的构建。例如，Facebook的instagram.com整站都采用了React来开发，整个页面就是一个大的组件，其中包含了嵌套的大量其它组件。

在React中，按照界面模块自然划分的方式来组织和编写代码，整个UI是一个通过小组件构成的大组件，每个组件只关心自己部分的逻辑，彼此独立。

React认为一个组件应该具有如下特征：

（1）可组合（Composeable）：一个组件易于和其它组件一起使用，或者嵌套在另一个组件内部。如果一个组件内部创建了另一个组件，那么说父组件拥有它创建的子组件，通过这个特性，一个复杂的UI可以拆分成多个简单的UI组件；

（2）可重用（Reusable）：每个组件都是具有独立功能的，它可以被使用在多个UI场景；

（3）可维护（Maintainable）：每个小的组件仅仅包含自身的逻辑，更容易被理解和维护；

## JSX语法


```HTML
<script type="text/babel">
  const test = ["xx", "yy", "zz"];
  ReactDOM.render( 
    <div> 
      <h1> Hello, world! </h1>
      {
        test.map(function(name){
          return  <div>nice! {name}</div>
        })
      } 
      <p> test </p>
      <a>{test[1]}</a> 
    /div>,
    document.getElementById('example2')
  );
</script>
```
上面代码体现了JSX的基本语法规则：遇到HTML标签（以`<`开头），就用HTML规则解析；遇到代码块（以 `{`开头），就用JavaScript规则解析。上面代码的运行结果如下：

```TEXT
Hello, world!

nice! xx
nice! yy
nice! zz
test

yy
```

JSX允许直接在模板插入JavaScript变量。如果这个变量是一个数组，则会展开这个数组的所有成员

```HTML
<script type="text/babel">
const test=[
  <h1>this is h1</h1>,
  <h2>this is h2</h2>
];
ReactDOM.render(
  <div>{test}</div>,
  document.getElementById('example2')
);
  
//this is h1
//this is h2
</script>
```

在 JSX 中不能使用`if else`语句，但可以使用三元运算表达式来替代。


```HTML
<script type="text/babel">
const test=[
  <h1>this is h1</h1>,
  <h2>this is h2</h2>
];
const a = true;

ReactDOM.render(
  <div>
    {a ? test : 'oops'}
  </div>,
  document.getElementById('example2')
);
</script>
```

React推荐使用内联样式。可以将样式作为一个对象，插入到模板中并且`{}`括起来


```HTML
<script type="text/babel">
let myStyle={
  "width": "120px",
  "background": "#9f9f9f"
};

ReactDOM.render(
  <div style={myStyle}>
    { a ? test : 'oops' }
  </div>,
  document.getElementById('example')
);
</script>
```
注释需要写在花括号中

```HTML
<script type="text/babel">
const test=[
  <h1>this is h1</h1>,
  <h2>this is h2{/* test */}</h2>
];
</script>
```

## 组件

```HTML
<script type="text/babel">
  const MyComponent = React.createClass({
    render: function() {
      return <div>Hello, {this.props.title}</div>
    }
  });
  
  ReactDOM.render(
   <MyComponent title="周杰伦" />,
   document.getElementById("example")
   )
</script>
```

新的React版本不在推荐使用`React.createClass()`来创建组件，而是使用ES6的`class`和`extends`创建组件：

```
export default class MyComponent extends React.Component {
  render() {
    return (
      <h1> Hello {this.props.title}</h1>
    );
  }
}
```

或者使用纯函数式的组件：

```
const MyComponent = (props) => {
  return (
    <div>
      <h1>Hello {props.title}</h1>
    </div>
  );
};
```

如果仅使用ES5的语法，需要单独引入一个第三方的工具库createReactClass来代替原来的`React.createClass()`方法创建组件：

```
var createReactClass = require('create-react-class');

var MyComponent = createReactClass({
  render: function() {
    return <h1>Hello, {this.props.name}</h1>;
  }
});
```

组件本身的使用和直接使用HTML标签是非常相似的。但是要注意，组件类的第一个字母必须大写，否则会报错。另外，组件类只能包含一个顶层标签，否则也会报错。

组件的用法与原生的HTML标签完全一致，可以任意加入属性，比如`<Hello title="周杰伦" />`，就是 `Hello`组件加入一个`title`属性，值为`周杰伦`。组件的属性可以在组件类的`this.props`对象上获取。

添加组件属性，有一个地方需要注意，就是`class`属性需要写成`className`，`for`属性需要写成`htmlFor`，这是因为`class`和`for`是JavaScript的保留字。

组件元素的属性可以完全是用户自定义的属性，而DOM元素的属性必须是标签自带属性，使用自定属性必须加上`data`前缀。

## Props

`this.props`对象的属性与组件的属性一一对应，但是有一个**例外**，就是`this.props.children`属性。它表示组件的所有子节点。


```
class NotesList extends React.Component {
  render() {
    return (
      <div>
        {this.props.children}
      </div>
    );
  }
}

// 函数式组件
const NotesList = (props) => {
  return (
    <div>
      {props.children}
    </div>
  );
};

ReactDOM.render(
  <NotesList>
    <p>Hello</p>
    <p>world</p>
  </NotesList>,
  document.body
);
```
上面的`NoteList`组件有两个`<p>`子节点，它们都可以通过`this.props.children`读取，运行结果如下。

```TEXT
hello
world
```

`this.props.children`的值有三种可能：

- 如果当前组件没有子节点，它就是`undefined`;
- 如果有一个子节点，数据类型是`object`；
- 如果有多个子节点，数据类型就是`array`。

所以，处理`this.props.children`的时候要小心。

React提供一个静态方法`React.Children`来处理`this.props.children`：

```JS
React.Children.map(this.props.children, v => v)
```

使用`React.Children.map`来遍历子节点，不用担心`this.props.children`的数据类型是`undefined`还是`object`。

## `propTypes`

自React v15.5起，`React.prototypes`已经移入到了另一个包`prop-types`中，需要单独引入这个库：

```
import PropTypes from 'prop-types';

class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

Greeting.propTypes = {
  name: PropTypes.string
};
```

指定参数的默认值只需要指定组件的`defaultProps`属性即可：

```
// 指定 props 的默认值：
Greeting.defaultProps = {
  name: 'Stranger'
};

// 如果使用了fransform-class-properties的Babel工具，可以在类中生命静态属性
class Greeting extends React.Component {
  static defaultProps = {
    name: 'stranger'
  }

  render() {
    return (
      <div>Hello, {this.props.name}</div>
    )
  }
}
```

如果使用了ES5环境下使用的`createReactClass`来创建组件，需要使用`getDefaultProps`函数来设置属性的默认值：

```JS
var Greeting = createReactClass({
  getDefaultProps: function() {
    return {
      name: 'Mary'
    };
  },
});
```

`prop-types`提供了多种验证器，[可以参考文档来使用](https://react.docschina.org/docs/typechecking-with-proptypes.html/)。

## `refs`

根据React的设计，所有的DOM变动，都先在虚拟DOM上发生，然后再将实际发生变动的部分，反映在真实 DOM上，这种算法叫做DOM diff，它可以极大提高网页的性能表现。但是有些时候必须获取真实的DOM，比如：

- 管理焦点，文本选择或媒体播放。
- 触发强制动画。
- 集成第三方 DOM 库。

需要在虚拟DOM上插入`ref`属性，这样通过`this.ref.[refName]`就能获取到真实的DOM节点。要注意的是， 由于`this.refs.[refName]`属性获取的是真实DOM，所以必须等到真实DOM插入文档以后，才能使用这个属性，否则会报错。

```
class Greeting extends React.Component {
  componentWillMount() {
    console.log(this.refs.myRef, 'componentWillMount');
  }
  
  componentDidMount() {
    console.log(this.refs.myRef, 'componentDidMount');
  }
  
  render() {
    return (
      <div ref="myRef">Hello</div>
    )
  }
}

// undefined, componentWillMount
// <div>Hello</div>, componentDidMount
```

但是在React 16.3版本以后，上面这种方式已经不再被推荐了，引入了`React.createRef`这个API，来创建`ref`属性

```
class Greeting extends React.Component {
  myRef = React.createRef();

  render() {
    return (
      <div ref={this.myRef}>Hello</div>
    );
  }
}
```

如果使用的较早版本的React，无法使用`React.createRef`这个API时，应该使用回调函数形式的`refs`来代替：

```
class Greeting extends React.Component {
  componentDidMount() {
    console.log(this.myRef);
  }

  render() {
    return (
      <div ref={el => { this.myRef = el }}>Hello</div>
    );
  }
}
```

这时候引用DOM元素实在回调函数中将DOM元素赋值给实例属性，通过引用实例属性来引用到React的。


**要注意，避免使用`refs`来做任何可以通过声明式实现来完成的事情**。比如，避免在`Dialog`组件里暴露`open()`和`close()`方法，最好传递`isOpen`属性。一定不要过度使用`refs`，如果一定要通过`refs`来让触发某个组件的功能，那么应该再反思一下组件的`state`属性是否可以被提升到父组件中。

另外，**不能在函数组件上使用`ref`属性**，应为它们没有实例，可以在函数组件的内部的DOM元素或者class组件上使用`ref`属性。

## `state`

`state`组件内部的状态，它是一个对象，完全又组件自身来控制。改变它的方法应该是使用`setState`方法，当它的值改变后，React会自动调用`render`方法，重新渲染组件。

注意不能直接修改`state`的属性值，这不会让组件重新渲染。

定义`state`有几种方法，当在使用ES5环境下的`createReactClass`来创建组件时，需要使用`getInitialState`方法来定义`state`：

```JS
var Counter = createReactClass({
  getInitialState: function() {
    return {count: this.props.initialCount};
  },
  // ...
});
```
而ES6的class组件中有以下几种方法：

（1）在`constructor`中：

```JS
class Greeting extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      x: 1
    }
  }
}
```

这种方式是比较主流的方式，要注意的是，如果显式的声明了`constructor`方法，必须调用`super`来获取`this`对象

（2）直接在`class`中定义

另外一种方式是直接定义在`class`中，效果与定义在`constructor`上一样的，都是定义的类的实例属性。这是ES7的提案，有一定的兼容性问题，最好配合Babel的转换使用。

```JS
class Greeting extends React.Component {
  state = {
    x: 1
  }
}
```

（3）使用`useState`在函数组件中定义

这是React16.8中新提出的Hooks API，它让函数式组件也可以拥有了自己的`state`（以前是不行的）：

```
const Greeting = () => {
  const [name] = useState('Jay');
  return (
    <h1>Hello, {name}</h1>
  );
};
```

具体关于Hooks API的使用[参考文档](https://react.docschina.org/docs/hooks-state.html)。

关于`state`有几个要注意的点：

（1）由于`componentWillMount`生命周期在React 16.8版本中已经被标记为不推荐的方法，有可能在随后的版本中被放弃，所以不要在这个生命周期中初始化`state`，应该在上面提到的方法中进行初始化。

（2）不要直接修改`state`，因为这样不会重新渲染组件。

（3）`state`的更新**可能是异步**的，React出于性能的考虑，可能会把多个`state`调用合并为一个调用，因此`this.state`可能会一步更新，不要依赖它们的值来更新下一个状态。

比如，下面的代码不会更新`state.count`的值

```JS
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment,
});
```

正确的做法是让`setState`接受一个函数，这个函数的第一个参数是上一个`state`值，第二个参数是更新时的`props`值：


```JS
this.setState((state, props) => {
  return {
    count: state.count + props.increment
  };
 });
```

## 生命周期

当前React16.8版本的生命周期：

![](http://image.oldzhou.cn/FveZx7Xv8MyhxPx0JfpMHugViFy0)

组件的生命周期分成三个状态

1. Mounting：创建阶段
2. Updating：更新阶段
3. Unmounting：移除阶段

（1）在创建阶段涉及到的生命周期及在该周期内可以完成的功能如下：

```JS
constructor(){
  // 初始化 state
}

getDgetDerivedStateFromProps(nextProps, prevState) {
  // 根据 props 来更新 state
  // 当 props 改变时，获取外部数据
}

// 此生命周期在V16.3版本中已被标记为不安全，所以不再推荐使用！！！
componentWillMount() {
  // 初始化 state
  // 获取外部数据
  // 添加事件订阅
}

render() {
}

componentDidMount {
  // 获取外部数据
  // 添加事件订阅
}
```

（2）在更新阶段涉及到的生命周期及在该周期内可以完成的功能如下：

```JS
getDgetDerivedStateFromProps(nextProps, prevState) {
  // 根据 props 来更新 state
  // 当 props 改变时，获取外部数据
}

// 此生命周期在V16.3版本中已被标记为不安全，所以不再推荐使用！！！
componentWillReceiveProps() {
  // 根据 props 来更新 state
  // 当 props 改变时，执行相关操作
  // 当 props 改变时，获取外部数据
}

shouldComponentUpdate(nextProps, nextState) {
  // 判断组件是否应该继续更新
}

// 此生命周期在V16.3版本中已被标记为不安全，所以不再推荐使用！！！
componentWillUpdate(nextProps, nextState) {
  // 执行外部回调
  // 在组件更新之前读取 DOM 节点
}

render() {
}

getSnapshotBeforeUpdate(prevProps, prevState) {
  // 在组件更新之前读取 DOM 节点
}

componentDidUpdate(prevProps, prevState, snapshot) {
  // 执行外部回调
  // 当 props 改变时，执行相关操作、 
}
```

（3）销毁阶段

```JS
componentWillUnmount() {
  // 取消事件订阅
}
```

（4）其他

```JS
componentDidCatch() {
  // 捕获子组件的异常
}
```

综上，React16时期，推荐使用的生命周期如下：


```JS
import React from 'react'

export default class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    // 初始化state方式（1）
    this.state = {

    }
  }
    
  static defaultProps = {

  }

  // 初始化state方式（2）
  state = {

  }
  
  static getDerivedStateFromProps(props, state) {
    return state
  }
  
  componentDidCatch(error, info) {

  }
  
  render() {

  }
  
  componentDidMount() {

  }
  
  shouldComponentUpdate(nextProps, nextState) {
    
  }
  
  getSnapshotBeforeUpdate(prevProps, prevState) {

  }
  
  componentDidUpdate(prevProps, prevState, snapshot) {

  }
  
  componentWillUnmount() {

  }
}
```


## 网络请求

组件的数据来源，通常是通过网络请求从服务器获取，一般网络请求都会放在`componentWillMount`和`componentDidMount`生命周期内。 

原来我有一个误解，以为如果网络请求放在`componentWillMount`内，获取到数据后组件才会`render`，只会执行一次操作，而网络请求放在`componentDidMount`中，组件会以空数据首先渲染一次，然后再根据网络请求的结果更新组件，多了一次组件更新过程。

但是实际上，由于`componentWillMount`和`render`的时间间隔是非常短的，所以网络请求即使放在`componentWillMount`中，组件也会执行两次渲染过程，基本上无法做到性能优化的。

而且由于`componentWillMount`已经被被标记为不安全的方法，可能在未来的某个版本中被丢弃，所以网络请求还是应该放在`componentDidMount`中，在网络请求的回调函数或者`then`方法中，通过`setState`重新渲染UI。


## Babel编译JSX文件

实际生产环境中，不会将JSX文件的转换放在浏览器端进行，需要在上线之前对JSX文件进行预编译。

一般情况下使用类似Create-React-App这样的脚手架工具，会配置好Babel进行预编译。如果需要手动配置的话可以按照下面的步骤：

（1）在项目中安装Babel

```BASH
npm install  --save-dev babel-cli
```

（2）在项目中安装Babel需要的转码规则：

```BASH
# ES2015转码规则
$ npm install --save-dev babel-preset-es2015

# react转码规则
$ npm install --save-dev babel-preset-react

# ES7不同阶段语法提案的转码规则（共有4个阶段），选装一个
$ npm install --save-dev babel-preset-stage-0
$ npm install --save-dev babel-preset-stage-1
$ npm install --save-dev babel-preset-stage-2
$ npm install --save-dev babel-preset-stage-3
```

（3）配置Babel

新建配置文件文件`.babelrc`，存放在项目的根目录下。使用Babel必须配置这个文件，根据安装的转码规则，加入配置文件：

```JS
{
  "presets": [
    "es2015",
    "react", 
    "stage-2"
  ],
  "plugins": []
}
```

（4）在`package.json`,中增加`bulid`指令：

```JS
{
  // ...
  "scripts": {
    "build": "babel src -d lib" // 也可以自定义其他对应的 babel 命令
  },
}
```

转码时执行下面命令即可

```BASH
npm run bulid
```

上面的命令是直接将`src`文件夹里的文件输出到`lib`文件夹中


## 参考

- [React 入门实例教程@阮一峰的网络日志](https://note.youdao.com/)
- [Babel 入门教程@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2016/01/babel.html)
- [使用 PropTypes 进行类型检查@React](https://react.docschina.org/docs/typechecking-with-proptypes.html/)
- [Refs and the DOM@React](https://react.docschina.org/docs/refs-and-the-dom.html)
- [State & 生命周期@React](https://react.docschina.org/docs/state-and-lifecycle.html)
- [关于React v16.3 新生命周期@掘金](https://juejin.im/post/5aca20c96fb9a028d700e1ce#heading-0)
