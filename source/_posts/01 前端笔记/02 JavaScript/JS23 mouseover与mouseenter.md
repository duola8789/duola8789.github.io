---
title: JS23 mouseover与mouseenter
top: false
date: 2017-08-25 18:02:04
updated: 2019-10-21 20:00:59
tags:
- mouseover
- mouseout
- mouseenter
- mouseleave
- contains
categories: JavaScript
---

`mouseover`/`mouseout`与`mouseenter`/`mouseleave`的区别以及模拟。

<!-- more -->

## 区别

`mouseover`和`mouseout`成对使用，**事件会冒泡**，当鼠标指针穿过被选元素子元素时，也会触发事件。

`mouseenter`和`mouseleave`成对使用，事件不会冒泡，只有在鼠标指针穿过被选元素时，才会触发`mouseleave`事件。

也就是说，假设把`mouseover`/`mouseout`事件绑定到了父元素，那么它其中的任何子元素只要发生了`mouseover`/`mouseout`事件，同时也会触发父元素的`mouseover`/`mouseout`事件，父元素的父元素也会触发……然后一直向上，就像池塘的气泡一样一直往上冒。

而`mouseenter`/`mouseleave`不会发生事件冒泡，属于传统思维上的鼠标进出。

## 用`mouseover`模拟`mouseenter`

```HTML
<div id="container">
  <div id="child1"></div>
  <div id="child2"></div>
</div>
```
`mouseover`因为具有冒泡的性质，在子元素内移动的时候父元素的`mouseover`会被频繁触发。为了避免这个状况，可以使用`mouseenter`代替。

也可以使用`mouseover`结合`relatedTarget`模拟`mouseenter`的效果，对于`mouseover`事件来说，`relatedTarget`是鼠标移动目标节点时所离开的那个节点，对于`mouseout`事件时，改属性是离开目标时鼠标指针进入的节点

对于上面`container`的`mouseover`事件，它的`relatedTarget`值可能是：

1. `container`的父元素（即`body`），对应从外界移入`container`
2. `conainer`元素本身，对应从其子元素上移入`container`
3. 子元素本身，对应从`child1`移入`child2`

要模拟`mouseenter`，`relatedTarget`值只能是第一种情况，但是我们不能这样直接判断，这是因为有可能`container`的父元素不一定在各个方向包裹`container`，也就是说我们要判断的父元素可能是祖先元素的某一个，也就无法准确判断`relatedTarget`是哪个元素，所以需要通过排除2、3是更好的选择

```JS
const container = document.querySelector('#container'),
  child1 = document.querySelector('#child1'),
  child2 = document.querySelector('#child2'), 

function simulateEnter(parent, e) {
  return (e.relatedTarget !== e.target) && (!parent.contains(e.relatedTarget))

}

container.addEventListener('mouseover', e => {
  if (simulateEnter(container, e)) {
    console.log(123)
  }
});
```

判断父子节点包含关系使用了[`ele.contains`这个API](https://developer.mozilla.org/zh-CN/docs/Web/API/Node/contains)，用来表示传入的节点是否为改节点的后代节点。

用`mouseout`模拟`mouseleave`原理是一样的。

可以直接封装为一个函数，注意`this`就是`container`，也就是我们绑定事件的对象，可以使用`e.currentTarget`来代替

```JS
function simulateEnterOrLeave(cb) {
  return function (e) {
    const { relatedTarget } = e;
    if(relatedTarget !== this && !this.contains(relatedTarget)) {
      cb.apply(this, arguments);
    }
  }
}

function log() {
  console.log(123);
}

container.addEventListener('mouseout', simulateEnterOrLeave(log));
```

## 参考

- [mouseenter、mouseover；mouseleave、mouseout的区别@bobscript](https://blog.bobscript.com/2014/01/24/mouseenter-mouseover-mouseleave-mouseout.html)
- [mouseenter与mouseover为何这般纠缠不清？@IMWeb前端博客](https://imweb.io/topic/5a2bb4cfa192c3b460fce2b9)
