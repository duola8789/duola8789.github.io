---
title: HTML+CSS38 两列布局
top: false
date: 2019-10-18 10:19:22
updated: 2019-10-18 10:19:24
tags:
- 两列布局
categories: HTML+CSS
---

面试时经常遇到一个比较基础的问题，如何实现一列定宽、一列自适应的两列布局？我实际工作中一般都会使用`flex`来进行布局，但是有的时候想简单一点，就直接用`float`布局，结果阴沟里翻了船，手写代码除了错误。所以需要好好总结一下，都有哪些常用的方式。

<!-- more -->

## 准备工作

HTML结构：

```HTML
<body>
<div class="left left1">固定宽度200px</div>
<div class="right right1">
  <p>宽度自适应</p>
</div>
</body>
```
基础样式：

```CSS
* {
  margin: 0;
  padding: 0;
}

.left {
  width: 200px;
  height: 100px;
  background: darkcyan;
}

.right {
  height: 200px;
  background: red;
}
```

此时布局如下：

![](http://image.oldzhou.cn/FgtYHROyghwTcG2l67JbSWorBazn)

最终效果：

![](http://image.oldzhou.cn/Fr7zAHm0XIzxWBKqB7Cm6e_M2zHw)

## 1 `float` + `margin`

最基础的`float`布局就可以实现，要注意的是，默认的块级元素会充满整个父容器的宽度，我们就是利用这一点，来实现`right`的自适应宽度。

这种方法需要让右侧元素的外边距等于左侧元素的宽度。

```CSS
.left1 {
  float: left
}

.right1 {
  margin-left: 200px;
}
```

## 2 `float` + BFC

`float`元素的宽度会自动缩减为能容纳内部元素的最小宽度。上一种方法需要将`right`的`margin-left`设定为`left`的宽度，如果`left`的宽度不固定就无能为力。

其实可以利用BFC来实现，通过开启右侧元素的`overflow`可以出发BFC块级格式上下文，触发BFC之后，元素内部布局不再受到外部布局的影响。这样即使左侧元素不固定，也可以实现。

```CSS
.left1 {
  float: left
}

.right1 {
  overflow: hidden;
}
```

## 3 绝对定位 + `margin`

原理与第一种方式类似。

```CSS
.left1 {
  position: absolute;
  left: 0;
  top: 0;
}

.right1 {
  margin-left: 200px;
}
```

## 3 `flex`布局

最常用的布局方式之一了。

我原来都习惯将`flex-grow`单独写，其实可以直接为子元素设置`flex`属性，它是`flex-grow`、`flex-shrink`和`flex-basis`的缩写，默认取值为`0 1 auto`，所以左侧元素可以不进行设置，右侧元素直接设置定为`flex: 1`，这样右侧元素会自动充满空间。

```CSS
body {
  display: flex;
}

.left1 {
}

.right1 {
  flex: 1;
}
```

**要注意的是`flex-basis`设定的宽度，比直接设定`width`有更高的优先级**。


## 4 `inline-block` + `calc`

`inline-block`后，两个块级元素会并排排列，默认宽度都缩减为最小宽度，所以利用`calc`计算出右侧元素的宽度，就可以充满剩余空间。

要注意的是，HTML中的空格也会占据一定空间，利用`calc`计算时会出现误差，所以需要分别设定`font-size`，让空格不占据空间。


```CSS
body {
  font-size: 0;
}

.left1 {
  display: inline-block;
  font-size: 14px;
}

.right1 {
  display: inline-block;
  width: calc( 100% - 200px);
  font-size: 14px;
}
```

## 5 `grid`

`grid`实际上是最适合布局的方案了，但是还是需要考虑兼容性的。

`fr`是网格布局提供的关键字，来指明元素占据剩余空间的比例。

```CSS
body {
  display: grid;
  grid-template-columns: 200px 1fr;
}
```

此外，还可以使用`table`布局，但是在2019年，实现想不到使用`table`布局的理由了。

## 参考

- [Flex 布局教程：语法篇@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)
- [flex-basis@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/flex-basis)
- [CSS Grid 网格布局教程@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2019/03/grid-layout-tutorial.html)

