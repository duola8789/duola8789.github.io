---
title: React基础13 遇到的坑
top: false
date: 2019-01-22 18:02:20
updated: 2019-01-22 18:02:20
tags:
- React
- 坑
categories: React
---

![](/images/react01.jpg)

总结整理了一下一年前使用React开发测评平台时的经验。

总共使用React开发了一个项目，用了1个多月，学到的东西比这一年都多，值得好好总结。

<!-- more -->

## 定时任务中的`setState`

如果定时任务触发时，组件已经被销毁，会给出警告

```BASH
setState(...): Can only update a mounted or mounting component.
```

虽然只是一个warning，但是还是证明写的代码不规范，不一定什么时候就埋坑了。

这个问题实质是：`setState`在异步的`callback`里执行，而这个时候由于返回上一页，组件已经被销毁了。

用`isMounted`方法做判断是官方不推荐的方法，而且我也不知道怎么实现。

真正的解决方法应该是在`componentWillUnmount`中将事件清除或者变量设置为`null`。

对于`setInterval`来说需要在`componentWillUnmount`中`clear`

## `bind`在`addEventListener`中的使用

在组件中添加了`scroll`事件：

```JS
window.addEventListener('scroll', this.windowScroll.bind(this));
```

在`componentWillUnmount`中想要清除绑定的事件：

```JS
window.removeEventListener('scroll', this.windowScroll.bind(this));
```
这样做是不会生效的。

因为`bind(this)`方法总会返回一个新的函数，所以在`removeEventListener`时，移除的是一个不存在的、新的函数。

解决方法是在`constructor`里面对`windowScroll`一次性绑定`this`。（这样`this.windowScroll`变量指向的就是同一个`bind`之后的方法了。

```JS
export default class Overview extends React.Component {
  constructor(props) {
    super(props);
    this.windowScroll = this.windowScroll.bind(this)
  }

  windowScroll(e) {
    // do something
  }

  componentDidMount() {
    window.addEventListener('scroll', this.windowScroll);
  }

  componentWillUnmount() {
    window.removeEventListener('scroll', this.windowScroll);
  }

  // ... 
}
```

## `globalStore`中的方法在组件中不能直接调用

```JS
const { globalStore } = this.props;
const { setBtnStatus } = globalStore;

globalStore.setBtnStatus(); // OK
setBtnStatus(); // 报错
```
这是因为`this`指向调用者，前者的`this`指向`globalStore`，后者指向`window`

## `checkbox`值不能正确重置

在不同题目之间跳转时`checkbox`值不会清除，原因是在跳转时即使`input`所在的组件被重新`render`，但是如果`input`本身的`key`没有变化，React就认为这个组件整体没有变化，不会重新渲染，只会对`input`局部渲染，所以`input`的`value`的值就不会重置。

==只有`key`值变化，React才会认为组件整体变化，整体渲染==

解决方法是给`input`加上`key`的属性，根据页面变化，强制重新渲染，然后在`componentWillReceiveProps`里面对选中的答案状态进行重置

并且，不能在`componentWillUpdate`和`componentShouldUpdate`里面对`state`的值进行控制，会造成死循环

（2017.07.14更新）

当时查资料的时候对这里理解的不全面，原文的意思是如果`key`值太简单，例如只有数字序号作为`key`，当项目发生变化，`key`值可能不变，React可能会认为是同一个组件而不进行渲染。所以上面提到的：

> 只有`key`值变化，React才会认为组件整体变化，整体渲染

不够准确，应该==将`key`值独一无二化，例如用`ID`来标识`key`值==，这样`key`值不重复，就不会发生不渲染的情况。

（2019.01.22更新）

有两个问题：

（1）React中的`key`并不要求全局唯一，因为`key`的作用域是==当前列表内==，同一个列表内唯一即可，不同列表、不同组件间都不需要考虑这个问题。

（2）第二个是，`key`值是加可以在包含`input`的组件的（由于当时的组件划分的不合理所以只能加在`input`上）。

当组件上没有`key`时，传入的`Prop`发生变化，React会寻求复用，保存组件状态，不会触发`Mount`系列事件，只会触发`Update`系列事件;

而如果增加了`key`，当传入的`Prop`发生变化也会导致组件重新渲染，所有状态重置

因此就有两个处理方法：一个就是不为组件增加`key`，而是在更新周期的`componentWillReceiveProps`中寻求状态重置，另一种就是为组件增加`key`，有React自动完成重置

前者在逻辑不复杂的情况下是可以使用的，但是如果逻辑比较复杂就会导致大量的逻辑和函数在`componentWillReceiveProps`中，而后者就一劳永逸了，直接销毁了组件并重建，在`componentDidMount`中处理重置好，可以参考[这篇文章](http://taobaofed.org/blog/2016/08/25/react-key/)。

其实在Vue中也是相同的原理。

## `Prop`的类型验证和默认值

```JS
export default class Question extends React.Component {
  // 类型验证
  static propTypes = {
    finishBtnDisabled:  PropTypes.bool
  };

  // Prop默认值
  static defaultProps = {
    finishBtnDisabled : false,
  };
}
```

## 对Mobx的Store中的变量赋值

```JS
let { spendMinute } = timeStore;
spendMinute = 100;
```

这样是不行的，这是声明了新的变量，有了两种方法:

（1）直接引用Store中的变量，这种方法在Mobx的严格模式下第一种方法是不允许的

```JS
timeStore.spendMinute = 100
```
（2）引用Store中的方法，对变量赋值（推荐）


```JS
// Store中
export default class Mark {
  @observable spendMinute = 0;
  @action changeSpendMinute(minute) {
    this.spendMinute = minute
  }
}

// 组件中
@observer
export default class Overview extends React.Component {
  changeMinute() {
    const { globalStore } = this.props;
    globalStore.changeSpendMinute(100);
  }
}
```
## `<div>`的`blur`事件

会遇到这样的需求：当标签失去焦点时，将菜单隐藏并触发一些操作，直觉就想到使用`focus`和`blur`事件。

但是这两个事件只对`form`表单控件有效，但是对于`<span>`、`<div>`等普通元素并不生效

解决方法就是设置这些元素的`tabindex`属性，就可以触发焦点事件了。

```HTML
<div onBlur={this.blurHandler.bind(this)} 
     onFocus={this.focusHandler.bind(this)} 
     tabIndex="1">
  blur
</div>
```


此外，如果希望点击出现的菜单本身不会在点击自己时，因为`blur`事件消失，需要将菜单放入被点击事项的子元素中。

## DOM事件传参

DOM事件传参，事件对象作为最后一个参数并传入到事件处理程序中

```HTML
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```
与Vue中是不同的。

Vue中的点击事件传参时，需要手动将`$event`传入，否则事件处理程序无法访问事件对象。不传参时，事件处理程序默认的参数就是事件对象。

## React中的`this`

React组件中的`this`都指向了组件本身，但是为了接受客户端的响应而添加的回调函数，直接添加到了`window`对象上，再这个函数里面用到的`this`就指向了`window`而非React组件。

```JS
export default class PCIndex extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      updateCounter: 0,
    }
  };
  async markPhoto(index) {
    // ...
    
    window.teacherMark = async function(result) {
      // ...
      this.setState({
        updateCounter: (++this.state.updateCounter) 
      });
    };
  }
```
上面的`setState`会报错（可怕的是当时使用了Mobx，直接对组件内的属性赋值`this.updateCounter++`，没有报错而是直接无效）

解决方法有两个，一个是将组件的`this`缓存下来：

```JS
async markPhoto(index) {
  const self = this;
  // ...
  window.teacherMark = async function(result) {
    // ...
    self.setState({
      updateCounter: (++this.state.updateCounter) 
    });
  };
}
```
第二种方法就是改用箭头函数：
```JS
async markPhoto(index) {
  const self = this;
  // ...
  window.teacherMark = result => {
    // ...
    self.setState({
      updateCounter: (++this.state.updateCounter) 
    });
  };
}
```
对于`this`的指向，一定要谨慎！

## 循环的问题

其实这个问题和React关系不大，还是自己太菜。

有这样的一段代码，要求根据数组成员的某些属性筛选数后，创建一个新的数组，当时的做法啊是：
```JS
const questions = [
  { hasMarked: true}, 
  { hasMarked: false}, 
  { hasMarked: true}
];
let markParamAnswer = [];

questions.map((question, index) => {
  if (question.hasMarked) {
    markParamAnswer[index] = {
      "answerId": "123",
      "teacherName": "123",
      "score": -1,
      "comment": "111111"
    };
  }
});
console.log(markParamAnswer[1]); // undefined
console.log(markParamAnswer); // [{...}, empty, {...}]
```
但由于间隔着遍历导致数组，导致结果会形成带有空位的数组，在后面处理的时候出现了bug

当时的改进方案是：

```JS
const questions = [
  { hasMarked: true}, 
  {  hasMarked: false}, 
  {  hasMarked: true}
];
let markParamAnswer = [];
let i = 0;

questions.map((question, index) => {
  if (question.hasMarked) {
    markParamAnswer[index] = {
      "answerId": "123",
      "teacherName": "123",
      "score": -1,
      "comment": "111111"
    };
    i++
  }
});
console.log(markParamAnswer); // [{...}, {...}]
```

但是现在来看，当时还是太菜，这就是没有code review的缘故，没人指出你的代码有多烂，只能靠自己回过头再看看，发现自己菜的一比。

可以直接用`push`就行了（2018-11-22）：

```JS
questions.forEach((question, index) => {
  if (question.hasMarked) {
    markParamAnswer.push({
      "answerId": "123",
      "teacherName": "123",
      "score": -1,
      "comment": "111111"
    });
  }
});
```
如果数据量不大（因为会遍历两次）的时候可以写成函数式的，更清晰（2019-01-22）：

```JS
let markParamAnswer = questions.filter(question => question.hasMarked).map(v => {
  return {
    "answerId": "123",
    "teacherName": "123",
    "score": -1,
    "comment": "111111"
  }
});
```


## 参考

- [React setState can only update a mounted or mounting component@stackoverflow](https://stackoverflow.com/questions/35345338/react-setstate-can-only-update-a-mounted-or-mounting-component)
- [react native Warning: setState(...): Can only update a mounted or mounting component.@QCSDN](http://blog.csdn.net/tujiaw/article/details/58238975)
- [React Checkbox Stays Checked Even After Component is Unmounted@stackoverflow](https://stackoverflow.com/questions/43481466/react-checkbox-stays-checked-even-after-component-is-unmounted)
- [React 实践心得：key 属性的原理和用法](http://taobaofed.org/blog/2016/08/25/react-key/)
