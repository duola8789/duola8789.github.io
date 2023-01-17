---
title: React提高04 对虚拟DOM和加载过程的理解
top: false
date: 2019-01-25 16:05:43
updated: 2019-05-11 22:11:14
tags:
- React
- 坑
categories: React
---

也是面试时比较常遇到的React的问题之一。了解了之后还是能够加深对React的理解。

<!-- more -->

![image](http://upload-images.jianshu.io/upload_images/1814354-4bf62e54553a32b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

## React虚拟DOM的理解

（1）用JavaScript对象结构表示DOM树的结构；然后用这个树构建一个真正的 DOM 树，插到文档当中

（2）当状态变更的时候，重新构造一棵新的对象树。然后用新的树和旧的树进行比较，记录两棵树差异

（3）把2所记录的差异应用到步骤1所构建的真正的DOM树上，视图就更新了

虚拟DOM本质上就是在JS和DOM之间做了一个缓存。可以类比CPU和硬盘，既然硬盘这么慢，我们就在它们之间加个缓存：既然DOM这么慢，我们就在JS和DOM之间加个缓存。CPU（JS）只操作内存（虚拟DOM），最后的时候再把变更写入硬盘（DOM）。

虚拟DOM是用JS的对象结构模拟出HTML中DOM的结构，批量的增删改查，由于直接操作的是JS对象，所以速度要比操作真实DOM要快，最后更新到真正的DOM中

虚拟DOM构建的对象，除了DOM相关属性，还包括了React自身需要的属性，比如`ref`，`key`等，大概如下结构：

```JS
{
  type: 'div',
  props: {
    className: 'xxx',
    children: [{
      type: 'span',
      props: {
        children: ['CLICK ME']
      },
      ref: key:
    }, {
      type: Form,
      props: {
        children: []
      },
      ref: key:
    }] | Element
  }
  ref: 'xxx',
  key: 'xxx'
}
```

## React何时将虚拟DOM渲染为真实DOM

`render`这个步骤就是React组件挂载的步骤

> React组件挂载：将组件渲染，并构建DOM元素然后插入页面的过程

`render`的步骤是创建虚拟DOM，挂载组件，

在`render`之后，将这个虚拟DOM树真正渲染成一个DOM树，插入了页面，可以认为是在`componentDidMount`这个步骤完成的，该方法被调用时，已经渲染出真实的DOM

在组件存在期，`componentDidUpdate`与`componentDidMount`类似，在组件被重新渲染后，此方法被调用，真实DOM已经渲染完成

## React不能setState的步骤

`shouldComponentUpdate`和`componentWillUpdate`就会造成循环调用，使得浏览器内存占满后崩溃

## 对于`setState`的理解

`setState`是一个异步方法，一个生命周期内的所有`setState`方法会合并操作。

在各个生命周期中执行`setState`：

（1）在`componentWillMount`执行`setState`是无意义的，应该将这里为`state`的赋值放到`constructor`中直接作为`state`的初始值。

这是因为，组件直挂再一次，在`componentWillMount`里面`setState`会但是仅仅会更新`state`一次，而且会和`constructor`中的初始化`state`合并执行。所以这是无意义的`setState`

（2）在`componentDidMount`中执行`setState`会导致组件在初始化的时候就触发了更新，渲染了两遍，应该尽量避免。

有一些场景，比如在组件DOM渲染完成后获得DOM元素位置或者宽高等等设置为`state`，会不得在`componentDidMount`之后`setState`，但是除了这些必要的时候，都应该尽量避免在`componentDidMount`里`setState`。

（3）在`componentWillUnmount`中执行`setState`不会更新`state`，是不生效而且无意义的。

（4）**禁止**在`shouldComponentUpdate`和`componentWillUpdate`中调用 `setState`，这会造成循环调用，直至耗光浏览器内存后崩溃。

在`shouldComponentUpdate`和`componentWillUpdate`调用`setState`会再次触发这两个函数，然后在两个函数又触发了`setState`，然后再次触发这两个函数，这样就进入了一个不停`setState`然后不停触发组件更新的死循环里，会导致浏览器内存耗光然后崩溃。

（5）在`componentDidUpdate`中执行`setState同样会导致组件刚刚完成更新又要再更新，进入死循环。

但是在某些特殊情况下，比如说`state`或者`props`变化触发了DOM变化，需要重新获取DOM元素宽高时然后更新某个`state`的时候，就不得不在这个函数里使用`setState`了。此时一定要给`setState`设置一个**前提条件**，以避免出现循环渲染的问题。

```JS
componentWillUpdate(nextProps, nextState, nextContext) {
  if (this.state.count !== 3333) {
    this.setState({
      count: 3333
    });
  }
}
```

因此，如非必须，应该尽量避免在本函数里`setState`。

（5）在`componentWillReceiveProps`中可以`setState`，不会造成二次渲染。由于只有`props`的变化才会触发`componentWillReceiveProps`事件，因为在这个事件里`setState`不会造成不停触发组件更新的死循环，可以放心在这个函数里`setState`。

![WX20180507-180427.jpg](http://image.oldzhou.cn/WX20180507-180427.jpg)

## 为什么虚拟DOM比原生DOM性能更高

React的基本思维模式是每次有变动就整个重新渲染整个应用，相当于设置 innerHTML，只不过它设置的是内存里面 Virtual-DOM，而不是真实的 DOM。

React厉害的地方并不是说它比DOM快(这句话本来就是错的)，而是说不管你数据怎么变化，我都可以以最小的代价来更新DOM。方法就是在内存里面用新的数据刷新一个虚拟的DOM数，然以后新旧DOM树进行比较，找出差异，再将差异更新到真正的DOM 树上。

原生DOM性能低，因为DOM的规范迫使浏览器在实现的时候为每一个DOM元素添加了非常多的属性，然而这其中很多我们都用不到

而React对虚拟DOM的属性进行了精简，非常轻量化，并且使用了diff的算法，比较前后差异，最后只把变化的部分一次性应用到真实的DOM树

> React的变动检查由于是DOM结构层面的，即使是全新的数据，只要最后渲染结果没变，那么就不需要做无用功。

更正确的说法应该是：虚拟DOM不一定比原生DOM性能高，但是让开发者更省心的更新数据。

> 首先, 虚拟DOM并没有比直接原生操作更快, 所谓"快"是有条件的
比如点赞, 数字+1, 直接操作DOM会更快。
> 
> 如果你能自己捋请规则, 每回手动操作DOM的时候, 都只改动应该改变的, 那DOM操作永远比虚拟DOM快。
> 
> 但如果你的改动勾连的地方很多，而且要保持状态，那虚拟DOM的自动diff无疑会让你省更多心。
> 
> 比如一个列表, 列表项有点赞等状态, 回复数量等信息, 有动态新增, 有动态加载, 这时候你要直接操作DOM会很繁琐。
> 
> 虚拟DOM的核心在于diff，它自动帮你计算那些应该调整，然后只修改该修改的区域, 省下的不是运行速度这种小速度，而是开发速度/维护速度/逻辑简练程度等"总体速度"

## diff算法

比较两棵DOM树的差异是Virtual DOM算法最核心的部分，这也是所谓的Virtual DOM 的diff算法。两个树的完全的diff算法是一个时间复杂度为`O(n^3)`的问题。但是在前端当中，你很少会跨越层级地移动DOM元素。所以Virtual DOM只会对同一个层级的元素进行对比：

![6d64b0b7889e7f020bb020aea5947a09_hd.jpg](http://image.oldzhou.cn/6d64b0b7889e7f020bb020aea5947a09_hd.jpg)

上面的`div`只会和同一层级的`div`对比，第二层级的只会跟第二层级对比。这样算法复杂度就可以达到`O(n)`。

## 参考
- [网上都说操作真实 DOM 慢，但测试结果却比 React 更快，为什么？ - 尤雨溪的回答@知乎](https://www.zhihu.com/question/31809713/answer/53544875) 
- [react里面的virtual DOM的效率为什么比直接操作DOM更快呢@segmentfault](https://segmentfault.com/q/1010000009401008?_ea=1922362/*&^%$)
- [如何理解虚拟DOM? - 戴嘉华的回答@知乎](https://www.zhihu.com/question/29504639/answer/73607810)
- [React组件生命周期函数里setState调用分析@Simona&Peter](http://varnull.cn/set-state-in-react-component-life-cycle/)
