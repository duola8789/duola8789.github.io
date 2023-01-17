---
title: React基础11 React中初始化state的两种方法
top: false
date: 2019-05-12 17:43:13
updated: 2019-05-12 17:43:16
tags:
- state
- constructor
categories: React
---

在React的组件中可以在两个位置来初始化`state`：（1）在组件的`constructor`中；（2）直接在`class`中利用属性赋值的方式

<!-- more -->

## 在`constructor`中

在`constructor`中初始化`state`如下所示：

```JS
class App extends React.Component {
  constructor(props) {
    // 必须在这里通过super调用父类的constructor
    super(props);

    // 给state赋值，可以使用props
    this.state = {
      loggedIn: false,
      currentState: "not-panic",

      // 在使用props为state赋值时要格外仔细
      someInitialValue: this.props.initialValue
    }
  }

  render() {
    // whatever you like
  }
}
```

当一个class组件创建之后，`constructor`会首先被调用，所以在`construcot`中可以来初始化所有值，包括`state`。class实例在内存中已经被创建，所以可以使用`this`来为`state`赋值

要注意的是，在`constructor`中可以不使用`this.setState`来为`state`直接赋值，除此之外其他位置都不能这样做：

```JS
// 除了在constructor中，其他位置都不要这样做
this.state.a = 123
```

只有通过`setState`才能通知React，我们修改了数据，React需要重新渲染组件。

还有要注意的是，在`constructor`中不要忘记使用`super(props)`来调用父类的`constructor`。默认的`constructor`（当创建一个class时，如果我们没有显式的声明`constructor`，JS会默认提供一个）会自动调用`super`，将将所有的参数传入

## 使用`prop`来初始化`state`

大多数情况下都不要使用`prop`来为State的初始化赋值，因为这会让你的数据来源不唯一，这常常会导致Bug。数据源唯一是最佳实践。

当组件的`prop`发生改变，组件会重渲染，所以没有必要将`prop`复制为`state`来保证`porp`永远是最新的值。


```
// 不要这样做
class BadExample extends Component {
  state = {
    data: props.data
  }

  componentDidUpdate(oldProps) {
    // 复制了prop的值给state之后
    // 必须保证在props更新时，state的值也随之更新
    if(oldProps.data !== this.props.data) {
      // This triggers an unnecessary re-render
      this.setState({
        data: this.props.data
      });
    }
  }

  render() {
    return (
      <div>
        The data: {this.state.data}
      </div>
    )
  }
}

// 正确的做法：
class GoodExample extends Component {
  render() {
    return (
      <div>
        The data: {this.props.data}
      </div>
    )
  }  
}
```

## `constructor`是必须的吗？

并一定必须显式的定义`constructor`，因为JS会提供默认的`constructor`。比如下面这个例子：

```JS
class Parent { 
  constructor(arg) { 
    console.log('constructing Parent with', arg)
  } 
}

class Child extends Parent {}

new Child(5);
// constructing Parent with 5
```

当创建`Child`的实例时，控制台会打印出`constructing Parent with 5`，虽然`Child`类并没有显式的定义`constructor`，也没有显式的通过`super(props)`调用父类。当我们没有定义自己的`constructor`时，JS会自动完成`super`的步骤。


## 直接在Class中定义

第二种初始化`state`的方法就是直接在Class的内部，使用Class的属性来定义：

```JS
class App extends React.Component {
  state = {
    loggedIn: false,
    currentState: "not-panic",
    someDefaultThing: this.props.whatever
  }

  render() {
    // whatever you like
  }
}
```

比如使用`construcot`更加得直接、简介。要注意的是：

- 没有定义`constructor`
- `state`属性是直接引用的，并不是通过`this.state`来引用的
- `state`的作用域是在Class内部，并不是一个方法的内部
- 仍然可以使用`this.props`和`this.context`
- `state`是class的实例属性，并不是静态属性，不需要添加`static`关键字（就像为`static propTypes {...}`）


## 哪种更好

习惯使用哪种，就使用哪种。

class属性的方式看起来更加简单，不再需要额外的模板代码（`constructor`），也不需要提醒自己调用`super(props)`

有的时候需要在`constructor`中处理事件处理函数（为函数绑定`this`，就像：

```JS
class Thing extends React.Component {
  constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick(event) {
    // do stuff
  }
}
```

可以通过Class属性的另外一种形式来替代上面的形式，可以让一个属性等于一个箭头函数，箭头函数获得了class实例的`this`，所以不再需要在`constructor`中显式的去绑定：

```JS
class Thing extends React.Component {
  handleClick = (event) => {
    // do stuff
  }
}
```

## 参考
- [Where to Initialize State in React@Dave Ceddia](https://daveceddia.com/where-initialize-state-react/)








