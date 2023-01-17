---
title: HTML+CSS33 关于居中的总结
top: false
date: 2019-06-20 19:45:15
updated: 2019-06-20 19:45:18
tags:
- CSS
- 居中
categories: HTML+CSS
---

关于CSS中水平、垂直居中的总结

<!-- more -->

## 准备工作

HTML结构：

```HTML
<div class="container">
  <span class="text">你好</span>
</div>
```

公共样式：

```CSS
.container {
  width: 300px;
  height: 300px;
  background: royalblue;
}
.text {
  background: darkgoldenrod;
}
```

## 水平居中的方法

（1）使用`text-align: center`

对于行内元素，可以直接使用`text-align: center`来水平居中

```CSS
.container {
  text-align: center;
}
```

（2）使用`margin: 0 auto`

对于宽度确定的块级元素，可以使用`margin: 0 auto`来水平居中，原理和后面的绝对定位加`margin: auto`的原理类似，后面单独结合总结。


```CSS
.text {
  display: block;
  width: 100px;
  margin: 0 auto;
}
```

（3）flex布局

如果不考虑兼容性，很完美的方案：

```CSS
.container {
  display: flex;
  justify-content: center;
}
```

（4）grid布局

显然兼容性有着问题的方案：

```CSS
.container {
  display: grid;
  justify-content: center;
}
```

（5）绝对定位+移动

父元素相对定位，子元素绝对定位，并且使用`transfrom`移动自身宽度的`-50%`（如果子元素的高度已知也可以通过`margin`向左移动自身高度的一半

```CSS
.container {
  position: relative;
}
.text {
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
}
```
（6）绝对定位和`margin: auto`

```CSS
.container {
  position: relative;
}

.text {
  position: absolute;
  left: 0;
  right: 0;
  width: 100px;
  margin: auto;
}
```

一定要指定子元素的宽度，否则子元素就会充满父元素

## 垂直居中的方法



（1）`vertical-align: middle`

针对行内元素，并且需要有一个兄弟行内元素，撑满父元素：

```CSS
.container:after {
  content: '';
  display: inline-block;
  height: 100%;
  background: aliceblue;
  vertical-align: middle;
}
.text {
  vertical-align: middle;
}
```

（2）`line-height` === `height`

让父元素的`height`和`line-height`相等，一般适用于文字的居中

```CSS
.container {
  line-height: 300px;
}
```
（3）`padding`

适用于父元素高度不确定，或者说父元素高度是根据子元素高度确定的情况：

```CSS
/* container 不能指定 height */
.container {
  padding: 20px 0;
}
.text {
  background: darkgoldenrod;
}
```

（4）绝对定位+移动

父元素相对定位，子元素绝对定位，并且使用`transfrom`移动自身高度的`-50%`（如果子元素的高度已知也可以通过`margin`向上移动自身高度的一半

```CSS
.container {
  position: relative;
}

.text {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
}
```
（5）flex布局

如果不考虑兼容性，很完美的方案：

```CSS
.container {
  display: flex;
  align-items: center;
}
```

（6）grid布局

显然兼容性有着问题的方案：

```CSS
.container {
  display: grid;
  align-items: center;
}
```

（7）`display: table-cell` + `vertical-align: middle`

```CSS
.container {
  display: table-cell;
  align-items: middle;
}
```

平时没怎么用，不知道有什么缺点呢~

（8）绝对定位和`margin: auto`

```CSS
.container {
  position: relative;
}

.text {
  position: absolute;
  top: 0;
  bottom: 0;
  margin: auto;
  height: 30px;
}
```

一定要指定子元素的高度，否则子元素就会充满父元素。


## 利用`margin: auto`居中的原理

```CSS
.father {
  position: relative;
  width: 500px;
  height: 500px;
}
.child {
  position: absolute;
  left: 0;
  top: 0;
  bottom: 0;
  right: 0;
  margin: auto;
  width: 100px;
  height: 50px;
}
```

- 优点：兼容性好
- 缺点：子元素需要指定宽高（否则会充满父元素）

原理：以水平方向举例，垂直方向相同。

当绝对定位的元素只定义了一个方向属性时（比如`left`），并且元素没有利用`width`指定宽度，宽度是`0`；当同时指定了相反的两个方向的属性时且相等（`left`和`right`都为`0`），此时如果没有指定`width`，则宽度会充满父元素，自定宽度会沿着`left`摆放。

此时指定`marign`：

1. 如果一侧定值，一侧`auto`，`auto`为剩余空间大小
2. 如果两侧均是`auto`，则平分剩余空间，所以就居中了。

块级元素利用`margin: 0 auto`水平居中的原理也是这样。

## 参考

- [小tip: margin:auto实现绝对定位元素的水平垂直居中](https://www.zhangxinxu.com/wordpress/2013/11/margin-auto-absolute-%E7%BB%9D%E5%AF%B9%E5%AE%9A%E4%BD%8D-%E6%B0%B4%E5%B9%B3%E5%9E%82%E7%9B%B4%E5%B1%85%E4%B8%AD/)
