---
title: HTML+CSS37 镂空遮罩
top: false
date: 2019-08-19 11:27:15
updated: 2019-08-19 11:27:18
tags:
- 镂空遮罩
- canvas
- svg
categories: HTML+CSS
---

镂空遮罩这样实现。

<!-- more -->

前一阵子面试被问题到这个问题，突然懵逼了，脑子一片空白，以前知道这种效果，比如[什么值得买](https://post.smzdm.com)的改版引导页面：

![](http://image.oldzhou.cn/FqOtRoAHGqyejr0VBUJAYImhY0lX)

当时再紧张也应该打出一种实现方法，就是什么值得买这种使用图片实现

![](http://image.oldzhou.cn/FhNFuij_H3lB86wXy2mfwN-y1SlH)

它首先加了一个半透明的黑色蒙层（`background-color: rgba(0,0,0,.8)`）然后添加提前制作好的图片作为子元素，然后通过决定定位，让图片与被遮盖的部分的定位相同，制造出一种假的镂空的效果

虽然这种方式处理定位有一些麻烦，并且不适合页面有滚动的情况，滚动的时候可能出现错位。

但是当时怎么也应该答出这种方式，但是确实一面试就紧张，脑子不转了，就想着添加一个伪元素，但是不知道怎么穿透。

回来查了一些资料，找到了几种实现的方法

首先准备好要被遮罩的DOM结构：


```HTML
<div class="outer">
  <div class="content">
    <p>这是要露出来的字</p>
    <p>这是要露出来的字</p>
    <p>这是要露出来的字</p>
  </div>
  <div class="inner"></div>
</div>
```


以及样式：

```CSS
.outer {
  position: relative;
  margin: 20px 0;
  height: 500px;
  background: darksalmon;
  overflow: hidden;
}

.content {
  width: 200px;
  height: 80px;
  color: #FFF;
  line-height: 1.5;
  background: #5b8b7b;
  margin: 100px 0 0 100px;
}
```

此时的效果：

![](http://image.oldzhou.cn/FviU5TmChy5ynOP4_eAkKdWkA2O3)

要实现的效果：

![](http://image.oldzhou.cn/FvB1mCAzIugS2TamhPQfv4rysoAR)


## 透明边框

中间的镂空部分为实际的`width`和`height`，为完全透明的背景，而四周半透明的遮罩使用`rgba`的`border`来实现

```CSS
.inner {
  position: absolute;
  left: 0;
  top: 0;
  box-sizing: content-box;
  width: 200px;
  height: 80px;
  border-color: rgba(0, 0, 0, 0.5);
  border-style: solid;
  border-width: 100px 1500px 1500px 100px;
  background: transparent;
}
```

## 透明轮廓

使用边框的地方，大多数时候都可以使用轮廓`outline`来替代，实际上没有什么不同，只是要注意，`outline`是不占据文档流空间的，所以定位方式与使用`border`时不同

```CSS
.inner2 {
  position: absolute;
  left: 100px;
  top: 100px;
  box-sizing: content-box;
  width: 200px;
  height: 80px;
  outline: rgba(0, 0, 0, 0.5) 1500px solid;
  background: transparent;
}
```

## 透明阴影

还可以使用透明阴影实现，主要利用了阴影的第四个扩展半径这个参数

```CSS
.inner3 {
  position: absolute;
  left: 100px;
  top: 100px;
  box-sizing: content-box;
  width: 200px;
  height: 80px;
  box-shadow: rgba(0, 0, 0, 0.5) 0 0 0 1500px;
  background: transparent;
}
```

## 使用Canvas实现

可以使用强大的Canvas实现，当然使用Cavnas就需要使用脚本来编写了，虽然有些复杂，但是使用灵活，能够适应各种不同的要求，比如同时镂空多个等等。

使用Canvas也有两种方式来实现，第一种方式是使用`clearRect`方法，比较简单：

```JS
const canvas = document.querySelector('#canvas');
const ctx = canvas.getContext('2d');

ctx.fillStyle = 'rgba(0, 0, 0, 0.5)';

ctx.fillRect(0, 0, 1500, 1500);
ctx.clearRect(100, 100, 200, 80);
```

另一种方式是自己通过`path`直接画出这样的一个形状，这里就需要介绍一下**非零环绕规则**

![](http://image.oldzhou.cn/Fj7l9oJEwZDT4cDr_0E9plY9x0Q2)

所以在画外围的半透明矩形时顺时针，那么里面镂空的矩形就需要逆时针：


```JS
const canvas = document.querySelector('#canvas2');
const ctx = canvas.getContext('2d');

// 外围
ctx.moveTo(0, 0);
ctx.lineTo(1500, 0);
ctx.lineTo(1500, 1500);
ctx.lineTo(0, 1500);
ctx.closePath();

// 内层
ctx.moveTo(300, 100);
ctx.lineTo(100, 100);
ctx.lineTo(100, 180);
ctx.lineTo(300, 180);
ctx.closePath();

ctx.fillStyle = 'rgba(0, 0, 0, 0.5)';
ctx.fill();
```

## 使用SVG实现

我对SVG基本上是不了解的，直接复制修改了一段代码



```HTML
<svg class="svg" width="1500" height="500">
  <defs>
    <mask id="myMask">
      <rect x="0" y="0" width="100%" height="100%" style="stroke:none; fill: #ccc"></rect>
      <rect width="200" height="80" x="100" y="100" style="fill: #000"></rect>
    </mask>
  </defs>
  <rect x="0" y="0" width="100%" height="100%" style="stroke: none; fill: rgba(0, 0, 0, 0.6); mask: url(#myMask)"></rect>
</svg>
```

也不是很复杂。

 
## 总结

- 如果页面布局尺寸都是固定的，可以使用CSS的三种方法实现
- 如果实现效果比较复杂，可以使用Canvas或者SVG实现
- 如果要偷懒，可以让UI出一张图片实现

## 参考

- [浏览器中遮罩层镂空效果的多种实现方法@CSDN](https://blog.csdn.net/xjun0812/article/details/51207177)
- [canvas-------绘制镂空的正方形+非零环绕规则@CSDN](https://blog.csdn.net/scwMason/article/details/82829964)
- [HTML5 canvas clearRect() 方法@W3Sschool](https://www.w3school.com.cn/tags/canvas_clearrect.asp)
