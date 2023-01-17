---
title: HTML+CSS34 1px边框问题的解决方案
top: false
date: 2019-07-11 10:46:17
updated: 2019-07-11 10:46:17
tags:
- border
- 移动端
categories: HTML+CSS
---

1px边框问题的解决方案的总结

<!-- more -->

## 现象及原因

在移动端开发时，设计图中的`1px`的边框，如果我们直接在CSS中设置边框的宽度为`1px`，实际上在设备上显示的并不是`1px`。这是因为不同的手机有着不同的像素密度，即`window.devicePixelRatio`属性，它反应的是物理像素与逻辑像素的比值，IPhone6的dpr是2，也就是说，对于IPhone6来说，CSS的`1px`显示时会显示为`2px`的像素

当设置为1px的边框时，移动端的显示：

![](http://image.oldzhou.cn/Frjko4_G7H4orzsylSV3vouXS-WZ)

当设置为0.5px的边框时，移动端的显示：

![](http://image.oldzhou.cn/FjYLG_kAnrcb3IvMZXGJA2GerTjF)

PC端`1px`的边框：

![](http://image.oldzhou.cn/Fp_tnwxaHdjmJ9ZTbGt6aNF2lBVP)


很明显看出，移动端`1px`的边框更粗

## 媒体查询

要想解决这个问题，必须针对不同dpr的设备进行不同的处理，完全不必使用JavaScript，而是可以通过媒体查询实现

```CSS
@media screen and (min-device-pixel-ratio: 2),  (-webkit-min-device-pixel-ratio: 2){
  /* 2倍屏 */
}

@media screen and (min-device-pixel-ratio: 3),  (-webkit-min-device-pixel-ratio: 3){
  /* 3倍屏 */
}
```

下面的各种处理方法，都是通过媒体查询进行分类处理，只列出2倍屏的处理方法，都省略了媒体查询的代码

## 解决方法1：伪元素 + `tranform: scaleY`

这种方法是比较常用，兼容性也比较好的，利用高度为`1px`的伪元素来模拟边框，在媒体查询中利用`tranform: scaleY`来进行缩放，需要设置`transform: origin(0, 0)`保证缩放时伪元素距离父元素的距离

```CSS
h1 {
  position: relative;
}

h1:after {
  content: '';
  display: block;
  width: 100%;
  height: 1px;
  position: absolute;
  left: 0;
  bottom: 0;
  background: red;
  transform: scaleY(1);
  transform-origin: 0 0;
}

@media screen and (min-device-pixel-ratio: 2),  (-webkit-min-device-pixel-ratio: 2) {
  h1:after {
    transform: scaleY(0.5);
  }
}
```

- 优点：兼容性好，边框圆角也可以实现
- 缺点：代码量比较大，占据了伪元素，会引起冲突

## 解决方法2：`0.5px`边框

直接将`border`设置为`0.5px`

```CSS
.h1 {
  border-bottom: 0.5px solid #000
}
```

- 优点：简单直接
- 缺点：兼容性太差，安卓和低版本的IOS都不支持

## 解决方法3：伪元素 + `liner-gradient` + `sacle`

同样是利用伪元素实现，但是使用了`liner-gradient`来模拟边框，实际上和第一种方法思路是相同的


```CSS
h1:after {
  display: block;
  content: '';
  height: 1px;
  background: linear-gradient(0, #fff, #000);
}

@media screen and (min-device-pixel-ratio: 2),  (-webkit-min-device-pixel-ratio: 2) {
  h1:after {
    transform: scaleY(0.5);
  }
}
```

## 解决方法4：通过`viewport`实现

可以使用JavaScript来读取`window.devicePixelRatio`，根据读取到的值来对`<meat>`的`viewport`进行改写，当dpr为2时，将页面缩放到原来的一半

```JS
const dpr = window.devicePixelRatio;

// 创建meta视口标签
const meta = document.createElement('meta') 

// 设置name为viewport
meta.setAttribute('name', 'viewport');

// 动态初始缩放、最大缩放、最小缩放比例
meta.setAttribute(
  'content', 
  `width=device-width, user-scalable=no, initial-scale=${1/dpr}, maximum-scale=${1/dpr}, minimum-scale=${1/dpr}`
) 
```

这样做可以直接使用`px`来定义各处的尺寸，但是就不单单针对边框了，而是针对所有了，需要整体考虑


## 解决方法4：通过图片模拟

可以使用`border-image` 或者`background-image`来加载预先设置好的边框图片

这种方法并不是很理想，因为修改颜色的时候都需要替换图片，并且如果边框有圆角的话也需要对图片特殊处理。

## 总结

从兼容性、针对性以及可维护性，还是使用伪元素 + `tranform: scaleY`的第一种方法最为推荐，其他的方法要不是很麻烦，要不是原理相同。有一些文章介绍了七八种，有的能实现，但是实施起来很麻烦，有的根本就不能实现在移动端实现`1px`，也不知道是我水平太低，还是作者根本没有亲自试验。

## 参考

- [1px边框解决方案总结@掘金](https://juejin.im/post/5af136b8f265da0b7a20a40e)
- [7 种方法解决移动端 Retina 屏幕 1px 边框问题@掘金](https://juejin.im/entry/584e427361ff4b006cd22c7c)
- [移动端1px实现@知乎](https://zhuanlan.zhihu.com/p/34931318)
