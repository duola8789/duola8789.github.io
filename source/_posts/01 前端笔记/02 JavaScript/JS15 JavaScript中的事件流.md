---
title: JS15 JavaScript中的事件流
top: false
date: 2017-06-21 10:08:04
updated: 2020-06-07 20:22:03
tags:
- 事件流
- addEventListener

categories: JavaScript
---

JavaScript中的事件流

<!-- more -->

# 什么是事件流

当点击页面的某个按钮时，单击事件并不仅仅发生在按钮上。换句话说，在单击按钮的同时，也点击了按钮的容器元素，甚至也单机了整个页面。

**事件流描述的是从页面中接受时间的顺序**，IE和Netscape提出了几乎相反的事件流的概念：

1. IE的事件流是事件冒泡流
2. Netscape是事件捕获流

![](http://image.oldzhou.cn/18-11-26/58945493.jpg)

**大多是情况下都是将事件处理程序添加到事件冒泡阶段**

# 事件冒泡

IE的事件流叫做事件冒泡（Event Bubbling），即事件开始时由最具体的元素（嵌套层次最深的节点）接受，然后逐级向上传播，直至文档根节点。

以下面的HTML页面为例：

```HTML
<!DOCTYPE html>
<html>
  <body>
    <div id="myDiv">Click Me</div>
  </body>
</html>
```
当点击了`<div>`元素，`click`事件会按照如下顺序传播

```
graph LR
div-->body
body-->html
html-->document
```

事件从我们单击的元素开始，沿着DOM树向上传播，在每一级节点上都会发生，直至传播到`document`对象

![](http://image.oldzhou.cn/18-11-26/32733667.jpg)

# 事件捕获

Netscape提出的另一种事件流是事件捕获（Event Capturing），即不太具体的节点应该更早接收到事件，最具体的节点应该最后接收到事件。事件捕获的用于在于事件到达预订目标之前捕获它。

仍然以上面的页面作为例子，在事件捕获的状态下，点击`<div>`元素，`click`事件会按照相反的顺序传播：

```
graph LR
document-->html
html-->body
body-->div
```
事件捕获过程中，`document`对象最先接收到`click`事件，然后事件沿DOM树向下，直至传播到事件的实际目标，即`<div>`元素

![](http://image.oldzhou.cn/18-11-26/42852881.jpg)

# DOM事件流

==DOM2级事件规定的事件流包括了三个阶段==：

1. 事件捕获阶段
2. 处于目标阶段
3. 事件冒泡阶段

首先发生的事件捕获，为截获事件提供了机会。然后是实际的目标接收到事件，最后一个阶段是冒泡阶段，可以在这个阶段对事件作出相应。

以前面的HTML页面为例，单击`<div>`元素会按照下图顺序触发事件

![](http://image.oldzhou.cn/18-11-26/92970597.jpg)

在DOM事件流中，实际的目标`<div>`元素在捕获阶段不会接收到事件，这意味着在捕获阶段事件从`document`到`html`再到`body`后就停止了。下一个阶段是“处于目标”阶段，事件在`<div>`上发生。然后冒泡阶段发生，事件有传播回`document`

**在事件处理中，事件处于目标阶段被看成冒泡阶段的一部分**。

**为了最大限度的兼容，大多是情况下都是将事件处理程序添加到事件冒泡阶段。不是特别需要，不建议在事件捕获阶段注册事件处理程序**

# DOM2级事件处理程序

DOM2级事件定义了对两个方法：

1. `addEventListener`：添加事件处理程序
2. `removeEventListener`：删除事件处理程序

所有DOM节点都包括这两个方法，它们都接受3个参数，前两个参数是要处理的事件名和作为事件处理程序的函数。

第三个参数是一个布尔值，如果是`true`，表示在捕获阶段调用事件处理程序，如果是`false`，表示在冒泡阶段调用事件处理程序。默认为`false`

同一个`EventTarget`注册了多个相同的`EventListener`，那么重复的实例会被抛弃。所以这么做不会使得`EventListener被调用两次，也不需要用`removeEventListener`手动清除多余的`EventListener`，前提是`options`中的`capture`的参数值一致。

看一个例子：

```HTML
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>事件冒泡</title>
  <style>
    #outer {
      width: 500px;
      height: 300px;
      background: darkcyan;
    }
    #inner {
      width: 300px;
      height: 150px;
      background: chocolate;
    }
  </style>
</head>
<body>
  <div id="outer">
    Parent
    <div id="inner">Child</div>
  </div>
</body>
<script>
  const outer = document.getElementById('outer'),
    inner = document.getElementById('inner');

  outer.addEventListener('click', function () {
    console.log('parent')
  });

  inner.addEventListener('click', function () {
    console.log('child')
  });
</script>
</html>
```
![](http://image.oldzhou.cn/18-12-7/25637854.jpg)

而点击`child`，事件在冒泡阶段触发，首先被触发的是`child`本身，打印了`child`，然后向上层冒泡，触发`parent`的事件，打印出`parent`

将`addEventListener`的第三个参数改变为`true`，则事件触发由冒泡阶段转为捕获阶段，此时，当点击`child`，事件在捕获阶段的传播从`document`开始，向下传播到`parent`，然后再到`child`本身，所以会先打印`parent`，然后才打印出`child`

当冒泡阶段和捕获阶段都会调用事件处理程序时，事件按DOM事件流的顺序执行时间处理程序

![](http://image.oldzhou.cn/18-11-26/92970597.jpg)

当事件处于目标阶段时，事件调用顺序决定于绑定事件的书写顺序。

```JS
outer.addEventListener('click', function() {
  console.log('parent冒泡')
}, false);
outer.addEventListener('click', function() {
  console.log('parent捕获')
}, true);
inner.addEventListener('click', function() {
  console.log('child冒泡')
}, false);
inner.addEventListener('click', function() {
  console.log('child捕获')
}, true);
```

上面的代码，由于`child`是处于目标阶段的事件，执行的顺序和我们书写的顺序有关，因此是`'child冒泡'`在前，而`'child捕获'`在后。而`parent`的事件就会按照DOM事件流顺序执行，即先`'parent捕获'`后`'parent冒泡'`

因此，这种情况下的调用顺序是：

```
graph LR
parent捕获-->child冒泡
child冒泡-->child捕获
child捕获-->parent冒泡
```

如果我们改动一下书写顺序：

```JS
outer.addEventListener('click', function() {
  console.log('parent冒泡')
}, false);
outer.addEventListener('click', function() {
  console.log('parent捕获')
}, true);
inner.addEventListener('click', function() {
  console.log('child捕获')
}, true);
inner.addEventListener('click', function() {
  console.log('child冒泡')
}, false);
```

那么调用顺序就变成了：

```
graph LR
parent捕获-->child捕获
child捕获-->child冒泡
child冒泡-->parent冒泡
```
# `addEventListenr`的第三个参数

（2020.06.07更新）

在旧版本的DOM规定中，`addEventListener`的第三个参数如上所述，是一个布尔值，表示是否在捕获阶段调用事件处理程序。但是在新版本浏览器中支持第三个参数传入一个对象，它可以包含各种属性：

- `capture`，用来表示`listener`回调函数是否在捕获阶段时触发
- `once`，表示`listener`在添加后最多调用一次，如果是`true`，`listener`会在被调用之后自动移除
- `passive`，如果设置为`true`，表示`listener`永远不会调用`preventDefault()`，如果`listener`中调用了`preventDefault()`客户端会忽略并抛出控制台警告，利用这个属性可以避免在某些事件中调用`preventDefault()`从而导致客户端公洞处理期间性能降低，导致滚屏卡顿

某些版本的浏览器可能不支持第三个参数为对象的语法，需要对这种情况进行检测，原理就是为检测的`options`中的某个属性设置一个全局的变量，来表示这个属性是否被支持，而这个变量的赋值是在`try...catch`中对`options`的被检测属性的`getter`中完成的，具体代码如下：

```JS
var passiveSupported = false;

try {
  var options = Object.defineProperty({}, "passive", {
    get: function() {
      passiveSupported = true;
    }
  });

  window.addEventListener("test", null, options);
} catch(err) {}
```

# 事件委托

事件委托就是把一个元素响应事件的函数委托到父层或者更外层元素上，真正绑定事件的是外层元素，当事件响应到需要绑定的元素上时，会通过事件冒泡机制从而触发它的外层元素的绑定事件上，然后在外层元素上去执行函数。

事件委托的优点：

1. 提高性能
2. 动态绑定事件（不必再为动态生成的、需要绑定事件的元素逐一绑定事件）

这样的一个例子：鼠标放到`li上`，让对应的`li`背景变红：

```HTML
<body>
  <ul id="ul">
    <li>item1</li>
    <li>item2</li>
    <li>item3</li>
  </ul>
</body>
<script>
  document.querySelector('#ul').addEventListener('mouseover', function (e) {
    if(e.target.tagName.toLowerCase() === 'li') {
      e.target.style.backgroundColor = 'red'
    }
  });

  document.querySelector('#ul').addEventListener('mouseout', function (e) {
    if(e.target.tagName.toLowerCase() === 'li') {
      e.target.style.backgroundColor = '#fff';
    }
  });
</script>
</html>
```
> `target`属性可返回事件的目标节点（触发该事件的节点），如生成事件的元素、文档或窗口。

这样避免了在所有的`li`上绑定事件，而且如果是动态生成的节点，直接在`li`上绑定事件就需要再次绑定，而事件委托就不需要。

利用jquery也可以实现：

```JS
$("ul").on("click","li",function(){
  // some code here
})
```
其中`on`方法的第二个参数是一个jQuery选择器，用于指定哪些后代元素可以触发绑定的事件。如果该参数为`null`或被省略，则表示当前元素自身绑定事件(实际触发者也可能是后代元素，只要事件流能到达当前元素即可)。

# 关于事件委托的一个问题

例如，当我们在`ul`上绑定了点击事件，想要点击`<li>`触发，但是`<li>`中间有着`<span>`，点击事件实际目标就是`<span>`，对`e.target.tagName`进行判断是否等于`LI`是不成立的，导致事件委托不能生效

解决方法是，当点击到`<span>`时，需要向外层递归寻找`<li>`标签，直到找到`<li>`为止，或者找到外层的`<ul>`为止

```JS
const ul = document.querySelector('ul');

ul.addEventListener('click', (e) = > {
  let target = e.target;
  while (target.tagName !== 'LI') {
    target = target.parentNode;
    if (target === ul) {
      target = null;
      break
    }
  }
  if (target.tagName.toLowerCase() === 'li') {
    console.log(e.target.innerHTML)
  }
})
```

所以或许用其他的选择器代替标签选择器是一个好的方式。

# 阻止事件冒泡

## `event.stopPropagation()`

可以使用`event.stopPropagation()`来阻止事件继续传播，在上面的`parent`和`child`的例子中，如果将`child`绑定的事件改为下面的代码：

```JS
outer.addEventListener('click', function() {
  console.log('parent冒泡')
}, false);
inner.addEventListener('click', function() {
  console.log('child冒泡');
  e.stopPropagation()
}, false);
```

这样再次点击`child`，就不会触发`parent`的`click`事件

## `e.target`

也可以通过`e.target`来阻止事件冒泡

```JS
document.body.addEventListener('click', e = > {
  // 通过e.target判断阻止冒泡
  if (e.target && e.target.matches('a')) {
    return;
  }
})
```

## `return false`

JS的原生的事件绑定中，`return false`是不能阻止事件冒泡的，它可以用来阻止浏览器的默认事件，作用于`event.preventDefault()`相同

在jQuery绑定的事件中使用`return false`，既阻止默认行为又停止冒泡


# 参考

- 《JavaScript高级程序设计》第十三章
- [EventTarget.addEventListener()@MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget/addEventListener/)
- [JavaScript事件流@掘金](https://juejin.im/entry/5826ba9d0ce4630056f85e07)
- [JS中事件冒泡与捕获@segmentfault](https://note.youdao.com/)
- [JavaScript事件委托详解@知乎](https://zhuanlan.zhihu.com/p/26536815)
- [事件委托@简书](https://www.jianshu.com/p/ac47521806d2)
