---
title: HTML+CSS01 CSS技巧
top: false
date: 2019-06-30 08:58:00
updated: 2019-08-13 16:39:12
tags:
- CSS
categories: HTML+CSS
---

各种CSS效果

<!-- more -->

# 透明边框

默认情况下，背景的颜色会延伸至边框下层，所以如果边框设置为透明色，会被背景色覆盖掉。

可以设置CSS3的属性[`background-clip`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-clip)设置元素的背景（背景图片或颜色）是否延伸到边框下面。

`background-clip`取值有四个：

- `border-box`: 背景延伸至边框外沿（但是在边框下层）
- `padding-box`: 背景延伸至内边距（`padding`）外沿，不会绘制到边框外
- `content-box`: 背景延伸至内容区（`content box`）外沿
- `text`：背景剪裁成为文字的前景色。（只有Chrome支持，需加`-webkit-`前缀）

![](http://image.oldzhou.cn/FpKGstS3QYG5aEbxFSCWYfrP7VlE)

所以将`background-clip`设置为`padding-box`就可以实现透明边框。

> 参考：
> - [半透明边框@You-need-to-know-css](https://lhammer.cn/You-need-to-know-css/#/zh-cn/translucent-borders)
> - [background-clip@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-clip)

# 多重边框

## `box-shadow`

`box-shadow`用来产生阴影效果，如果只给出两个数值，那么浏览器解析为`x`方向偏移量和`y`方向偏移量；如果给出第三个值，将被解释为模糊半径的大小；如果给出第四个值，将被监事未扩展半径的大小。

另外两个属性是`inset`属性（声明式）和颜色值。

一直对模糊半径和扩展半径有一些糊涂，从[下面这个例子](https://juejin.im/post/5ce290cc6fb9a07ea8039d83)了解`box-shadow`的生成过程

首先定义出`box-shadow`属性：

```CSS
.box {
  width: 300px;
  height: 200px;
  background-color: yellowgreen;
  box-shadow: 30px 50px 20px #000;
}
```
（1）首先在`box`元素生成一个同`box`大小、形状完全相同的阴影，颜色为设置的颜色`#000`

![](http://image.oldzhou.cn/FsV6IWhYI7_Q1bgL6TF7OHma9E0Z)

此时，阴影在元素的上层

（2）根据`offset-x`和`offset-y`，将阴影从上层向右移动`30px`，向上移动`50px`

![](http://image.oldzhou.cn/FrF8Xmp5uh3UukOsaYQiaLaG9vwo)

（3）然后根据设置的`20px`的模糊半径，然后在四个方向上，每个方向的阴影边缘为中心，向两侧各扩展`10px`的区域，作为高斯模糊的半径

以右侧为例：

![](http://image.oldzhou.cn/FkPYqMg8GKKPhsJ9sajTyZhABVEw)

（4）最后一步，将阴影与元素重叠的区域剪裁掉，如下图：

![](http://image.oldzhou.cn/FtG6m6JEX1SUeZn6p7SgdTuJpcvq)

（5）得到最终效果：

![](http://image.oldzhou.cn/Fo2YmjK0hJYLXI_R1LH5Ll4iZXE7)

从上面的例子看出，模糊半径是是以阴影边缘为中心，向两侧扩展一半，作为半径，进行高斯模糊。只能取正值。

扩展半径，是将阴影的面积增大，在它取值为`0`时，阴影面积是等于元素的面积的，它不等于`0`时，取值是四个方向增大或者减小的尺寸

```CSS
.inner {
  box-shadow: 30px 50px 0 30px #000;
}
```

![](http://image.oldzhou.cn/Fn4GZlUhMnmHGxpUWPlUJd0Xx6Q9)

上图的虚线尺寸就是增加的扩展半径`30px`，当设置了扩展半径，模糊半径也会从增大后的阴影边缘向两侧模糊。

可以通过将`x-offset`、`y-offset`、模糊半径都设为`0`，设置不同尺寸的扩展半径，来实现多重边框的效果：

```CSS
.inner {
  box-shadow: 0 0 0 5px darkgoldenrod, 0 0 0 10px darkolivegreen, 0 0 0 15px red;
}
```

![](http://image.oldzhou.cn/FswhNgLNUD7PxJLo2woa0kCXqvXa)

优点是很容易实现2条以上的边框，且可以实现圆角边框，缺点是无法实现非实线的边框。

## `outline` + `outline-offset`

`outline`可以设置一个或者或者多个单独轮廓属性，轮廓不占据空间（`box-shadow`和`border`都占据空间）

`outline-offset`用来设置一个`outline`与一个元素边缘或者边框的间隙

将两者结合，就可以实现多种边框，并且可以实现`box-shadow`无法实现的，非实线的多重边框

```CSS
.inner2 {
  border: aqua dotted 5px;
  outline: red dotted 5px;
  outline-offset: -15px;
}
```

![](http://image.oldzhou.cn/FrXgZTN2joXiIvqj7YStdIl2XUA2)

优点是可以实现非实线的边框，缺点是实现2条以上的边框不方便，且无法实现圆角边框。

> 参考
> - [多重边框@You-need-to-know-css](https://lhammer.cn/You-need-to-know-css/#/zh-cn/multiple-borders)
> - [box-shadow@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/box-shadow)
> - [box-shadow详解@掘金](https://juejin.im/post/5ce290cc6fb9a07ea8039d83)
> - [outline@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/outline)
> - [outline-offset@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/outline-offset)

# 边框内圆角

要实现的效果：

![](http://image.oldzhou.cn/FluW1mYFqLlYY8D1bBtJLU4cJHmL)

边框外围没有圆角，内部有圆角，所以单独使用`border-radius`是不行的。上一部分内容中，我们可以发现，`box-shadow`是跟随边框有圆角的，而`outline`则是没有圆角的，所以我们可以使用三者的组合实现边框内圆角。

利用`border-radius`设置圆角，假设为`r`，但是不设置`border`，设置`outline`为`w`，这样外边框是没有圆角的，


```CSS
.inner {
  background: darkgray;
  border-radius: 5px;
  outline: 5px solid darkgreen;
}
```
这时候效果是：

![](http://image.oldzhou.cn/FpHhAgjiahF6PeuxsF_XaqwZb0-i)

背景会由于`border-radius`的设置出现圆角，然后我们用`box-shadow`将这部分补上，只设置其扩展半径，颜色与`outline`的颜色相同

```CSS
.inner {
  background: darkgray;
  border-radius: 5px;
  outline: 5px solid darkgreen;
  box-shadow: 0 0 0 5px darkgreen;
}
```
扩展半径的取值范围见下图：

![](http://image.oldzhou.cn/FgsHRYQ-1rkRVFYStGFTgFUGFWCl)

复习一下扩展半径的值：

![](http://image.oldzhou.cn/Fn4GZlUhMnmHGxpUWPlUJd0Xx6Q9)

所以这里取值最大值不能超过`outline`的宽度`w`，而最小宽度根据勾股定理不能小于`$\sqrt{2}r - r$`，即`$\sqrt{2-1}r$`，在我们上面的设置中，扩展半径的取值范围是`[2.07, 5)`


> 参考：
> - [边框内圆角@You-need-to-know-css](https://lhammer.cn/You-need-to-know-css/#/zh-cn/inner-rounding)

# 背景定位

## [`background-position`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-position)

![](http://image.oldzhou.cn/FrnFBglSd0nXpgo1VAOqqMONVf8R)

我比较熟悉的背景定位是使用`background-position`，但是都是只指定了两个属性值：

```CSS
.backgroundPosition {
  background-position: right bottom;
}
```

有三个点要注意：

1. `background-position`指定的位置是相对于由`background-origin`定义的位置图层的，默认定位于元素的左上角
2. `background-position`在一个方向上，可以指定一个值（例如上面），也可以指定两个值，可以在`right`后面跟一个数值，表示相对于边缘的位置
3. 如果指定的是百分比，`0%`代表图片的左（上）边界和容器的左（上）边界重合，`100%`代表图片的右（下）边界与容器的右（下）边界重合，`50%`代表图片的中心与容器的中心重合

```CSS
.inner {
  width: 50%;
  height: 200px;
  margin: 0 auto;
  padding: 20px;
  background: #fff url('../assets/images/css-tricks.png') no-repeat;
  background-size: 50px;CSS
}

.backgroundPosition {
  background-position: right 20px bottom 20px;
}
```
另外，这个属性是可以融合到`background`属性中的：

```CSS
.backgroundPosition {
 background: #fff url('../assets/images/css-tricks.png') no-repeat right 20px bottom 20px;
}
```

## [`background-origin`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-origin)

刚才也提到了，`background-position`指定的位置是相对于由`background-origin`定义的位置图层的。`background-origin`规定了背景图片的属性的原点位置与容器的关系（与`background-clip`类似）

可以取值有：

- `border-box`: 图片边界与`border`重合
- `padding-box`: 图片边界与`padding`区域重合
- `content-box`: 图片边界与`content`区域重合

![](http://image.oldzhou.cn/Fk6jnaJ2R33O7q7j93Z-lMh4gnZ0)

所以，可以取值`content-box`，借用`padding`定位实现。

```CSS
.inner {
  padding: 20px;
  border: 10px solid red;
  background: #fff url('../assets/images/css-tricks.png') no-repeat right bottom;
  background-origin: content-box;
}
```

> 参考：
> - [背景定位@You-need-to-know-css](https://lhammer.cn/You-need-to-know-css/#/zh-cn/extended-bg-position)
> - [background-position@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-position)
> - [background-origin@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-origin)

# 条纹进度条

可以使用`background`的`repeating-linear-gradient`属性，来生成一个静态的进度条，再结合动画属性，实现动态的进度条样式。

![](http://image.oldzhou.cn/FhYUUEkaUI0ZKKsS-9Uz_s4ItDWm)

[`linear-gradient`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/linear-gradient)可以实现线形重复渐变效果。

[`repeating-linear-gradient`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/repeating-linear-gradient)类似`linear-gradient`，并且采用相同的参数，但是它会在所有方向上重复渐变以覆盖其整个容器。

它分为两组参数，第一组参数用来描述渐变线的起点位置，`to top`这样的值会转换为0度，`to left`会被转换为270度。第二组参数可以包含多个值，用逗号分隔，每个值都是一个色值，后面跟随一个可选的终点位置，终点位置可以是百分比也可以是绝对值

```CSS
/* 一个倾斜45度的重复线性渐变,
   从蓝色开始渐变到红色 */
linear-gradient(45deg, blue, red);
```

![](http://image.oldzhou.cn/FrfiO8WADiybGVVVZEaqCzegt6Pt)

```CSS
/* 一个从右下角到左上角的重复线性渐变,
   从蓝色开始渐变到红色 */
linear-gradient(to left top, blue, red);
```

![](http://image.oldzhou.cn/Fk3UAB64mjf_QRIBHnCjpd9OJSxM)

```CSS
/* 一个由下至上的重复线性渐变,
   从蓝色开始，40%后变绿，
   最后渐变到红色 */
linear-gradient(0deg, blue, green 40%, red);
```

![](http://image.oldzhou.cn/FmQdq_CuWwjIoiPrNLZDG7ygKc3y)

要想实现突然变色，而非渐变色，需要为两个颜色制定同一个终点位置，那么这两个颜色会产生一个无线小的过渡区域，从效果上看，颜色会在那个位置突然变化，而不是一个平滑的过程。

```CSS
background: linear-gradient(90deg, blue 50%, red 50%);
```

![](http://image.oldzhou.cn/Fu8bj-9f2y3nll4eB0pa23e7-WIX)

上面的写法的可维护性不太好，因为每次修改尺寸时都需要修改两处，但是CSS规范规定，如果某个色标的位置值比整个列表在它之前的位置都要小，则改色标的位置会被设置为它前面的额所有色标位置值的最大值

所以上面的代码也可以改为：

```CSS
background: linear-gradient(90deg, blue 50%, red 0);
```

色块的大小是可以通过`background-size`来控制的：

```CSS
background: linear-gradient(90deg, blue 50%, red 0);
background-size: 16px;
```

![](http://image.oldzhou.cn/FmbtnK0ucf0g_dQjb2KsLOB1S6d2)

在就将角度倾斜，通过`animation`控制`background-position`，就可以实现动起来的效果了：

```CSS
.inner:after {
  background: linear-gradient(90deg, blue 50%, red 0);
  background-size: 16px;
  animation: loading 10s linear infinite;
}
@keyframes loading {
  to {
    background-position: 100% 0 ;
  }
}
```

这样可以实现水平方向条纹的横向移动

![](http://image.oldzhou.cn/FvfGizAT795CYaX8dkenO_Oi7GTe)

但是如果要是倾斜45度就出现问题了

```CSS
.inner:after {
  background: linear-gradient(45deg, blue 50%, red 0);
  background-size: 16px;
  animation: loading 10s linear infinite;
}
```

![](http://image.oldzhou.cn/Fgpe2VMO_K0CPIc8uLoeG-Nr6KAA)

这个时候应该改为使用`repeating-linear-gradient`属性，在所有方向上重复渐变以覆盖其整个容器，使用这个属性时，色标值需要写完整：

```CSS
background: repeating-linear-gradient(90deg, blue 0 ,blue 50%, red 50%, red 100%);
```
所以这样可以实现斜着45度的进度条：

```CSS
.inner:after {
  background: repeating-linear-gradient(45deg, red 0, red 25%, blue 0, blue 50%,
  red 0, red 75%, blue 0, blue 100%);
  background-size: 16px;
  animation: loading 10s linear infinite;
}
```

（2019-10.21更新）之前理解有错误，其实`repeating-linear-gradient`就等于`linear-gradient` + `background-repeat` + `background-size`，因为之前用的是百分比来确定色款其实体现不出来，上面的代码不用`repeating-linear-gradient`而是直接使用`linear-gradient`也是可以直接实现的。

如果改用`px`，就可以舍弃`background-size`，直接使用`repeating-linear-gradient`实现：

```CSS
.inner:after {
  background: repeating-linear-gradient(45deg, black 0, black 5px, darkgoldenrod 0, darkgoldenrod 10px);
}
```

> 参考：
> - [条纹背景@You-need-to-know-css](https://lhammer.cn/You-need-to-know-css/#/zh-cn/stripes-background?id=%e8%bf%9b%e5%ba%a6%e6%9d%a1)
> - [linear-gradient@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/linear-gradient)
> - [repeating-linear-gradient@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/repeating-linear-gradient)
> - [聊一聊CSS3的渐变——gradient@掘金](https://juejin.im/post/5bc2fb09f265da0ac55e75e7)

# 不规则卡片

想要实现这样的卡片：

![](http://image.oldzhou.cn/Fnbau00ivj7GkoEPuD4nv8plYK6u)

上面的不规则椭圆切口使用`background`的[`radial-gradient`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/radial-gradient)实现。

它的第一个参数定义圆形渐变的中心点，默认为背景的中心，第二个参数定义渐变的形状，可以取值`circle`（圆形）和`ellipse`（椭圆），这两个参数可以通过`at`关键字合并为一个参数，比如`circle at 50% -5px`

后面的参数和线性填充一样，表示某个确定位置的固定色值，形式为`color1 position1, color2 position2`，这个位置值是相对虚拟渐变射线的位置值，如果是圆形可以理解为半径

所以这样的一个椭圆缺口可以通过下面实现：

```CSS
.top {
  background: radial-gradient(circle at 50% -5px, transparent 20px, #73d2d3 21px );
}
```

第一个参数定义了在`50%, -5px`的横纵坐标位置的一个圆形填充，如果不指定位置如下：

![](http://image.oldzhou.cn/Ft5HZrTncSbiqJMnmXAne6e-onT2)

后面的色值参数实际上省略了一部分，完整的应该是：


```CSS
.top {
  background:  radial-gradient(circle at 50% -5px,transparent 0, transparent 20px, #73d2d3 21px, #73d2d3 100% );
}
```

定义了在`20px`范围内是透明色，在`21px`到填充满都是蓝色，之所以大了`1px`是为了让边缘平滑。这样切口实现。

下面实现上下两个色块的交界处的波浪线，可以利用伪元素加上样式为`dotted`的`border`实现：

```CSS
.top:after {
  content: '';
  display: block;
  position: absolute;
  left: 0;
  bottom: -3px;
  width: 100%;
  border-bottom: #f7f7f7 6px dotted;
}
```

`dotted`的边框，显示为一系列圆点。标准中没有定义两点之间的间隔大小，视不同实现而定。圆点半径是`border-width`计算值的一半。

![](http://image.oldzhou.cn/FuAr2s9nK8fiPG_G5luffh0pN815)

> 参考：
> - [不规则卡片@You-need-to-know-css](https://lhammer.cn/You-need-to-know-css/#/zh-cn/stripes-background?id=%e4%b8%8d%e8%a7%84%e5%88%99%e5%8d%a1%e7%89%87)
> - [radial-gradient()@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/radial-gradient)
> - [border-style@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-style)

# 圆和椭圆

一般我们会使用`border-radius`实现圆角效果，但是实际上使用`border-radius`可以实现圆、椭圆、半圆等各种圆形的效果。

我们常用的可能就是`border-radius: 5px`，但是实际上它是一个缩写，它的完整写法是：

```TEXT
border-radius: 5px 5px 5px 5px / 5px 5px 5px 5px
```

这八个取值以`/`进行分割，每一组四个值，分别代表左上、右上、右下、左下四个角的圆角半径，第一组值是水平方向半径，第二组是垂直方向的半径


```
border-radius: 左上角水平圆角半径大小 右上角水平圆角半径大小 右下角水平圆角半径大小 左下角水平圆角半径大小
               / 左上角垂直圆角半径大小 右上角垂直圆角半径大小 右下角垂直圆角半径大小 左下角垂直圆角半径大小
```

理解了完整的取值，就可以通过设置不同的值来实现不同的圆形。以半圆为例：

![](http://image.oldzhou.cn/FjzAnPWXdFuA7jhf7_TME52N7FPx)

首先矩形的宽度应该是高度的2倍，上图的半圆应该设置的是左上和右上的圆角，而且水平半径应该都是矩形宽度的一半，垂直半径应该等于矩形高度，所以：

```CSS
.semicircle {
  width: 200px;
  height: 100px;
  border-radius: 50% 50% 0 0 / 100% 100% 0 0;
}
```
`border-radius`的百分比是相对于`offsetWidth`和`offsetHeight`进行设置的，及包括了`content`、`padding`以及`border`的宽度。

更多的实现起来也就不难了，无非是各种组合。

![](http://image.oldzhou.cn/FoVIGXokiPMOlo0NCPkoEfgpEZ_k)

> 参考
> - [圆与椭圆@You-need-to-know-css](https://lhammer.cn/You-need-to-know-css/#/zh-cn/ellipse)
> - [秋月何时了，CSS3 border-radius知多少@鑫空间鑫生活](https://www.zhangxinxu.com/wordpress/2015/11/css3-border-radius-tips/)
> - [border-radius@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-radius)

# 四边形

![](http://image.oldzhou.cn/FlxewVHNLJBvvy_PXSu1Z4ob5dWP)

实现一个四边形我首先想到的方案是使用矩形+三角形拼接而成，三角形使用`border`来实现：

```CSS
.parallel1 {
  position: relative;
  width: 100px;
  height: 100px;
  background: #FFF;
}

.parallel1:before {
  content: '';
  display: block;
  position: absolute;
  left: 0;
  top: 0;
  width: 0;
  height: 0;
  transform: translateX(-100%);
  border: 50px solid transparent;
  border-right-color: #FFF;
  border-bottom-color: #FFF;
}

.parallel1:after {
  content: '';
  display: block;
  position: absolute;
  right: 0;
  top: 0;
  width: 0;
  height: 0;
  transform: translateX(100%);
  border: 50px solid transparent;
  border-left-color: #FFF;
  border-top-color: #FFF;
}
```
能够实现，但是代码量是在是不少，而且还占据了两个伪元素。其实有两种更简单的方法，一种是使用`transform: skew`，`skew`的意思就是偏离、扭转，它定义了在2D平面上一个对象的歪斜变换，参数可以使一个也可以是两个，当是两个的时候，分别表示沿着横坐标、从坐标扭曲元素的角度。

它的效果可以看下面的三幅图：

`transform: skewX(30deg)`

![](http://image.oldzhou.cn/FgEYRszHBiQ9RCUNpnI5-YZDBHmb)


`transform: skewY(30deg)`

![](http://image.oldzhou.cn/FiwuEwmAMoxE3y52uMbkP77c1rma)

`transform: skew(30deg)`

![](http://image.oldzhou.cn/FpGOAUJylHDQ7Or8diFZGHXRhGcu)


所以用它来实现四边形就很简单了：

```CSS
.parallel2 {
  width: 200px;
  height: 100px;
  background: #FFF;
  transform: skewX(-45deg);
}
```
还有一种方法是使用`clip-path`，它可以用来创建一个只有元素的部分区域显示的剪切区域，区域内的部分显示，区域外的隐藏（这个属性替代了已经弃用的`clip`属性）

它的可能取值有：

- `clip-source`，用一个SVG的URL来表示剪切元素的路径，比如`clip-path: url(resources.svg#c1);`
- `geometry-box`，为基本形状提供相应的参考矿框盒，可以取值有`margin-box`、`border-box`、`padding-box`、`content-box`、`fill-box`等
- `basic-shape`，可以用一些预置的形状来进行剪切，比如`circle`、`epplipse`、`polygon`

用这个属性与使用SVG是很类似的，可以实现多种多样的形状，使用`clie-path: polygon(A, B, C, D)`就可以实现各种四边形。其中A/B/C/D分别是四个点的坐标值，其中横坐标在前，纵坐标在后：

```CSS
.parallel3 {
  width: 300px;
  height: 100px;
  background: #FFF;
  clip-path: polygon(33% 0, 100% 0, 66% 100%, 0 100%);
}
```

可以实现类似菱形的四边形：

```CSS
.parallel4 {
  width: 100px;
  height: 100px;
  background: #FFF;
  clip-path: polygon(50% 0, 100% 50%, 50% 100%, 0 50%);
}
```

也可以实现一个三角形：

```CSS
.parallel5 {
  width: 100px;
  height: 100px;
  background: #FFF;
  clip-path: polygon(0 100%, 100% 100%, 100% 0);
  transition: clip-path 0.5s;
}
```

![](http://image.oldzhou.cn/Fjpiu_n8Qytes9AF5NsW0zfyeHET)

简直太强大了。

> 参考
> - [parallel四边形@You-need-to-know-css](https://lhammer.cn/You-need-to-know-css/#/zh-cn/parallelogram)
> - [skew()@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/transform-function/skew)
> - [clip-path@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/clip-path)
> - [css3中-webkit-transform 的 skew 如何使用？@知乎](https://www.zhihu.com/question/21725826/answer/19247715)

# 切角效果

想实现下面这两种切角效果

![](http://image.oldzhou.cn/FrlcN9GSAToCVBLlQWLdyxFwxuKa)

有三种方法，实际上前面都介绍过，先看实现直角切角效果

（1）首先可以使用`linear-gradient`，首先定义一个方向的直线渐变：

```CSS
.corner1 {
  background: linear-gradient(45deg, transparent 12px, #FFF 13px) left bottom no-repeat；
  background-size: 51%  51%;
}
```

要注意的是，这个切角是定义的左下角的位置，首先这个「左下角」是由`background-position`定义的，只不过简写在`background`属性中

还要注意的是角度，只需要记住`to top`是`0deg`，`to right`是`90deg`，计算角度时需要从镜像的起始点出发计算

![](http://image.oldzhou.cn/FmvTrDRTVMCA2XEplQu01ZB91opA)

然后定义了`background-size: 51%  51%;`相当于只定义了左下角四分之一的背景填充：

![](http://image.oldzhou.cn/Fob1EZjScaYgxzMjH7_BNuFYCxut)

之所以不用`50%`而是`51%`，是为了防止有些浏览器在渲染时出现缝隙的问题，和`border-radius`设置为`51%`是一样的道理。

所以按照同理，将另外三个角补充完整即可：

```CSS
.corner1 {
  background: linear-gradient(45deg, transparent 12px, #FFF 13px) left bottom no-repeat,
              linear-gradient(135deg, transparent 12px, #FFF 13px) left top no-repeat,
              linear-gradient(-45deg, transparent 12px, #FFF 13px) right bottom no-repeat,
              linear-gradient(-135deg, transparent 12px, #FFF 13px) right top no-repeat;
  background-size: 51%  51%;
}
```

（2）也可以使用之前一节学习过的`clip-path`来实现，将这个切过角的形状作为一个多边形，将各个坐标点描绘出来即可（当然也可以使用内联SVG）

在描述各点坐标时，可以使用`calc`这个属性来计算，小心点一点别把横纵坐标搞错就行：


```CSS
.corner2 {
  clip-path: polygon(0 12px, 12px 0, calc(100% - 12px) 0, 100% 12px,
                     100% calc(100% - 12px), calc(100% - 12px) 100%, 12px 100%, 0 calc(100% - 12px));
  }
```

（3）圆角切角可以使用`radial-gradient`实现，原理与使用`linear-gradient`类似，只是需要额外指定径向渐变的圆心位置：


```CSS
.corner3 {
  background: radial-gradient(circle at bottom left, transparent 12px, #FFF 13px) bottom left no-repeat,
              radial-gradient(circle at top left, transparent 12px, #FFF 13px) top left no-repeat,
              radial-gradient(circle at bottom right, transparent 12px, #FFF 13px) bottom right no-repeat,
              radial-gradient(circle at top right, transparent 12px, #FFF 13px) top right no-repeat;
  background-size: 51%  51%;
  }
```

> 参考
> - [切角效果@You-need-to-know-css](https://lhammer.cn/You-need-to-know-css/#/zh-cn/bevel-corners)
> - [linear-gradient()@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/linear-gradient)

# 使用一个`div`实现进度条

```CSS
.bar {
  --c: #5b8b7b;
  --h: 80px;
  --p: 50%;
  width: 800px;
  height: var(--h);
  border-radius: calc(var(--h) / 2);
  background-color: lightgray;
  background-image: linear-gradient(var(--c), var(--c));
  background-repeat: no-repeat;
  background-size: var(--p) 100%;
}
```
效果：

![](http://image.oldzhou.cn/FishqPKefkv-UFpfufwRMw-8IS6I)

为了使完成部分也出现圆角，可以添加一个`radial-gradient`，利用它可以实现一个圆球：

```CSS
.circle {
  background-image: radial-gradient(closest-side circle at 50% 50%, red, red 100%, yellow);
  background-repeat: no-repeat;
}
```

![](http://image.oldzhou.cn/FggrhOtmhulYd1Gnj_dG7McX9YX3)

综合起来：

```CSS
.bar {
  --c: #5b8b7b;
  --h: 80px;
  --p: 60%;
  width: 800px;
  height: var(--h);
  border-radius: calc(var(--h) / 2);
  background-color: lightgray;
  background-image: radial-gradient(closest-side circle at var(--p) center, var(--c), var(--c) 100%, transparent),
                      linear-gradient(var(--c), var(--c));
  background-repeat: no-repeat;
  background-size: 100%, var(--p);
}
```
注意，`background-size`两个取值之间有一个逗号，意味着是对多个背景的设置，一个背景的设置中如果提供了两个数值时，第一个将作为宽度，第二个作为高度，如果只提供了一个数值，那么它将作为宽度值大小，高度值会被设为`auto`，所以完整的写法是：

```
background-size: 100% auto, var(--p) auto;
```

实现的效果：

![](http://image.oldzhou.cn/Fsj-gmkm0X6fDXcs-3u-aFKTRdrn)

> 参考
> - [你未必知道的49个CSS知识点@掘金](https://juejin.im/post/5d3eca78e51d4561cb5dde12#heading-37)
> - [linear-gradient()@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/linear-gradient)
> - [background-size@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-size)
> - [radial-gradient()@MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/radial-gradient)

# 空心圆环

想到了四种方法来实现空心圆环：

（1）使用标签嵌套（或者伪元素）

```CSS
.circle {
  position: relative;
  width: 150px;
  height: 150px;
  background: red;
  border-radius: 50%;
}

.circle::after {
  content: '';
  display: block;
  position: absolute;
  left: 50%;
  top: 50%;
  width: 130px;
  height: 130px;
  transform: translate(-50%, -50%);
  background: darkcyan;
  border-radius: 50%;
}
```

使用标签嵌套和微元素原理一样，缺点是需要设置背景色与整体的背景色一致，并非完全的空心圆环

（2）使用`border-radius`

```CSS
.circle {
  width: 150px;
  height: 150px;
  background: transparent;
  border: 10px solid red;
  border-radius: 50%;
}
```

元素本身是透明背景，边框有颜色


（3）使用`box-shadow`

```CSS
.circle {
  margin-left: 10px;
  width: 130px;
  height: 130px;
  background: transparent;
  border-radius: 50%;
  box-shadow: 0 0 0 10px red;
}
```

与上一种方案类似

（4）使用`radial-gradient`

```CSS
.outer4 {
  width: 150px;
  height: 150px;
  background: radial-gradient(circle closest-side, transparent 65px, red 65px);
  border-radius: 50%;
}
```

（5）`outline`

使用`outline`是无法实现的，因为`border-radius`不能影响`outline`
