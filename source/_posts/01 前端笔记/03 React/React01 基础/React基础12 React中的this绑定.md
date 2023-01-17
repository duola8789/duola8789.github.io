---
title: React基础12 React中的this绑定
top: false
date: 2019-05-12 23:13:33 
updated: 2019-05-12 23:13:34
tags:
- bind
- this
categories: React
---

React组件中的this绑定到底是怎么一回事？学习一下。

<!-- more -->

## 为什么要`bind(this)`

React的Class组件中常常会用到`bind(this)`来绑定`this`，但是究竟为什么要这么做呢？难道Class中的方法拿不到实例的`this`吗？

来试验一下：

```JS
class Person {
  constructor(name) {
    this.name = name;
  }
  say() {
    console.log('my name is ' + this.name);
  }
}
```

上面代码的`Person`类中，`say`方法去获取了`this`，然后当我们实例化一个对象时，并调用`say()`方法时，结果是：

```JS
let p1 = new Person('Jay');
p1.say();
// my name is Jay
```

结果说明在Class的方法中，是可以直接通过`this`获得实例对象的。所以实际上在React的Class中直接使用`this`是没有问题的，例如在生命周期函数或者`render`中。但是在`render`函数中的JSX模板中的事件处理函数，这里面的调用的方法的`this`就不会指向组件实例：

```
export default class Demo5 extends Component {
  handleClick() {
    console.log(this);
  }

  render() {
    return (
      <div onClick={this.handleClick}>
        Hello
      </div>
    );
  }
}
```
上面的`handleClick`函数作为JSX中的事件处理函数，其中的`this`不会指向组件实例，而是指向了事件响应的上下文环境（非严格环境下是`window`，严格环境下是`undefined`），而类声明和类表达式的主体以**严格模式**执行（主要包括构造函数、静态方法和原型方法以及`getter`和`setter`函数），所以`this`指向`undefined`


## 为什么`this`不能指向组件实例

实际上这与React和JSX语法没有关系，是JavaScript的`this`绑定机制导致了上述情况的发生。要明确的是，函数内部的`this`取决于该函数被调用时的上下文环境。

### 默认绑定

```JS
function display(){
 console.log(this); // 'this' 将指向全局变量
}

display(); 
```

在上面的情况下，`display`方法中的`this`在非严格模式下指向`window`，在严格模式下指向`undefined`

### 隐式绑定

```JS
const obj = {
 name: 'Saurabh',
 display: function(){
   console.log(this.name); // 'this' 指向 obj
  }
};

obj.display(); // Saurabh 
```

在上面的情况下，通过`obj`对象来调用这个函数时，`display`内部的`this`指向了`obj`。

但是如果将这个函数赋值给其他变量，并且通过这个变量去调用该函数时，在`display`中获得`this`就不同了：

```JS
var name = 'hello';
const outer = obj.display;
outer(); // 'hello'
```

我们调用`outer`时，并没有指定一个具体的上下文对象，这个时候`this`值与默认绑定的结果是相同的，在非严格模式下指向`window`，在严格模式下指向`undefined`

在将一个方法以回调的形式传递给另外一个函数，或者像`setTimeout`这样的内置JavaScript函数时，就可以依照上面的过程进行判断

例如我们自定义一个`setTimeout`方法并调用，预测一下会发生什么：

```JS
//setTimeout 的虚拟实现
function setTimeout(callback, delay){
  //等待 'delay' 数个毫秒

   callback();
}

setTimeout(obj.display, 1000);
```
在调用`setTimeout`时，函数内部将`obj.display`赋值给参数`callback`:

```JS
callback = obj.display;
```
在一段时间后调用这个方法时，调用的实际上是`callback()`，而这种调用会让`display`方法丢失上下文，其中的`this`会退回至默认绑定，指向全局变量

```JS
var name = "uh oh! global";
setTimeout( obj.display, 1000 );

// uh oh! global
```
### 显示绑定

为了避免上面的情况，可以使用`bind`来显式的为方法绑定上下文：

```JS
var name = "uh oh! global";
var outerDisplay = obj.display.bind(obj); 
outerDisplay();

// Saurabh
```

绑定了`this`后，在调用对应的方法也能够渠道我们绑定的上下文。同理，粗若将`obj.display`作为`callback`参数传递给函数，`display`中的`this`也会正确指向`obj`

### React编译时的处理

明确了隐式绑定时，将方法作为参数传递给另一个函数时会导致该方法的上下文丢失。而在React的类的写法中，JSX的事件处理程序的`this`会丢失，就是因为这个原因。

在编译JSX的过程中，事件处理程序会作为属性值被放置在一个对象中，调用时会识别为函数调用模式，上下文丢失。

举个例子，下面的Class组件：

```
export default class Demo extends Component {
  handleClick() {
    console.log(this);
  }

  render() {
    this.handleClick();
    
    return (
      <div onClick={this.handleClick}>
        Hello
      </div>
    );
  }
}
```

编译完是下面这样的：

```JS
const Demo = function (_Component) {
  _inherits(Deom, _Component);
  function Demo() {
    _classCallCheck(this, Demo);
    return  _possibleConstructorReturn(this, _Component.apply(this, arguments));
  }
  Demo.prototype.handleClick = function handleClick() {
    console.log(this);
  }
  Demo.prototype.render = function render() {
    this.handleClick();
    return __WEBPACK_IMPORTED_MODULE_0_react___default.a.createElement(
      'div',
      { onClick: this.handleClick },
      'hello',
    )
  }
  return Demo;
}(__WEBPACK_IMPORTED_MODULE_0_react__["Component"])
```

很明显的看到，当编译完成后，JSX中的事件处理程序`handleClick`是放置在`{ onClick: this.handleClick }`中，当点击事件触发时实际上调用的是`onClick`，和隐式调用中的结果一样，上下文丢失了，`this`指向`undefined`。

而如果我们在JSX中使用`bind`绑定`this`：

```HTML
<div onClick={this.handleClick.bind(this)}>Hello</div>
```

那么编译后就变成了`{ onClick: this.handleClick.bind(this) }`，这就显式的绑定了`this`。

## 解决方法

（1）React官方首推的一种方法就是使用实验性的语法`public class fileds`，可以使用class fields正确的绑定回调函数：

```
export default class Demo extends Component {
  handleClick = () => {
    console.log(this);
  };

  render() {
    return (
      <div onClick={this.handleClick}>
        Hello
      </div>
    );
  }
}
```

上面的写法会会保证`handleClick`内的`this`被正确绑定，要注意的是这是一个实验性的语法，但是Create React App默认启用了这个语法。

如果没有使用Create React App的话需要手动开启这个语法，使用Babel的`transform-class-properties `或者`enable stage-2 in Babel`这两项功能

（2）第种方式是在JSX的回调函数中使用箭头函数：

```
export default class Demo extends Component {
  handleClick() {
    console.log(this);
  }

  render() {
    return (
      <div onClick={(e) => this.handleClick(e)}>
        Hello
      </div>
    );
  }
}
```

这种语法的主要问题还是来自于性能的担忧，因为每次渲染组件时都会创建不同的回调函数。大多数情况时没有什么问题，但是如果该回调函数作为prop传入子组件时，这些组件可能会进行额外的重新渲染。

（3）第三种方式是在JSX的回调函数中使用`bind`进行绑定：

```
export default class Demo extends Component {
  handleClick() {
    console.log(this);
  }

  render() {
    return (
      <div onClick={this.handleClick.bind(this)}>
        Hello
      </div>
    );
  }
}
```

这种方法的问题和上一种是类似的，有可能带来性能的问题。

（4）第四种方式是在构造函数`constructor`中进行显示的绑定：

```
export default class Demo extends Component {
  constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    console.log(this);
  }

  render() {
    return (
      <div onClick={this.handleClick}>
        Hello
      </div>
    );
  }
}
```

这样就可以避免第二种方法可能带来的性能问题，而且很直观，但问题是增加了代码数量，而且为了绑定`this`必须声明`constructor`，在`constructor`中还不能忘记`super`，有点麻烦

所以，从性能角度上考虑，如果开启了对应的`public class fileds`语法（使用了Create React App），那么建议使用第一种方式，否则的话建议使用最后一种方式。

## 参考

- [事件处理@React](https://react.docschina.org/docs/handling-events.html?no-cache=1)
- [react为什么需要手动绑定方法？@React教程中文网](http://www.reactpeixun.com/reactganhuo/2017-08-08/315.html)
- [[译] 为什么需要在 React 类组件中为事件处理程序绑定 this@掘金](https://juejin.im/post/5afa6e2f6fb9a07aa2137f51)
- [React與bind this的一些心得@medium](https://medium.com/reactmaker/react-%E8%88%87-bind-this-%E7%9A%84%E4%B8%80%E4%BA%9B%E5%BF%83%E5%BE%97-323c8d3d395d)
