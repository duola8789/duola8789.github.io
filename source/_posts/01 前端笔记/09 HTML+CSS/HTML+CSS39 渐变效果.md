---
title: HTML+CSS39 渐变效果
top: false
date: 2019-10-21 14:49:47
updated: 2019-10-21 14:49:49
tags:
- 渐变效果
- linear-gradient
- radial-gradient
categories: HTML+CSS
---

CSS中的渐变效果，学习笔记。

<!-- more -->

## `linear-gradient`

```
linear-gradient( 
  [ <angle> | to <side-or-corner> ,]? <color-stop> [, <color-stop>]+ )
  \---------------------------------/ \----------------------------/
    Definition of the gradient line        List of color stops  
```

- `<angle>| to <side-or-corner>`用来描述渐变发生的方向或角度的。未指定时，默认是由上至下进行渐变。
- `<color-stop>`由一个`<color>`值组成，并且跟随着一个可选的终点位置（可以是一个百分比值或者是沿着渐变轴的`<length>`）。取值如：`#FF837E 80%`或者`blue 30px`。

它分为两组参数，第一组参数用来描述渐变线的起点位置，`to top`这样的值会转换为0度，`to left`会被转换为270度。第二组参数可以包含多个值，用逗号分隔，每个值都是一个色值，后面跟随一个可选的终点位置，终点位置可以是百分比也可以是绝对值

![](http://image.oldzhou.cn/Fs1zX-0ZXEY4vx-PVTNp7PQ03G08)


CSS规范规定，如果某个色标的位置值比整个列表在它之前的位置都要小，则改色标的位置会被设置为它前面的额所有色标位置值的最大值，所以可以使用`0`来指代前面的颜色的位置最大值

（1）默认效果


```CSS
.container {
  background: linear-gradient(white, black);
}
```

![](http://image.oldzhou.cn/FhIcB6-IDDT17cS_K3Fcp6W-KB1Y)

（2）从左到右的渐变：

```CSS
.container {
  background: linear-gradient(to right, white, black);
  /* 或者 */
  background: linear-gradient(90deg, white, black);
}
```

（3）从左到右渐变，渐变位置发生在中间的一小部分（20%左右）

```CSS
.container {
  background: linear-gradient(to right, white 40%, black 60%);
}
```

![](http://image.oldzhou.cn/Fu0XwQfjTicPykicc9kyfk3Tpi9Y)

（4）突然变色

如果多个色标具有相同的位置，他们会产生一个无限小的过渡区域，过渡的起止色分别是第一个和最后一个指定值。从效果上看，颜色会在那个位置突然变化，而不是一个平滑的渐变过程。

```CSS
.container {
  background: linear-gradient(to right, white 50%, black 50%, black 100%);
  /* 或者 */
  background: linear-gradient(to right, white 50%, 0, black 100%);
}
```

![](http://image.oldzhou.cn/FqKuYGOuDywcAigGEhv0x6i1KegA)

（5）边框渐变

三种渐变都属于CSS数据类型中的`<gradient>`类型，而`<gradient>`数据类型又是`<image>`数据类型的子类型。所以有时能应用到`<image>`的地方同样也可以应用到`<gradient>`，比如我们可以利用`border-image`属性来实现边框的渐变。如下面的效果：

```CSS
.container {
  border: 10px solid transparent;
  border-image: linear-gradient(to right, white, black) 10;
}
```

![](http://image.oldzhou.cn/FhiwUsH02vLM-2AWcZEkQV6-dHik)

（6）条纹进度条（竖直）


```CSS
.container {
  width: 500px;
  height: 20px;
  border-radius: 5px;
}

.container:after {
  content: '';
  display: block;
  width: inherit;
  height: inherit;
  border-radius: inherit;
  background: linear-gradient(90deg, black 50%, darkgoldenrod 0);
  background-size: 20;
  animation: loading 10s linear infinite;
}

@keyframes loading {
  to {
    background-position: 100% 0;
  }
}
```

![](http://image.oldzhou.cn/FvfGizAT795CYaX8dkenO_Oi7GTe)

（7）条纹进度条（倾斜）

倾斜的话，除了修改角度之外，还需要细化色阶值，让色块充满`backgroun-size`的范围内，只需要修改：

```CSS
.container:after {
   background: linear-gradient(45deg, black 0, black 25%, darkgoldenrod 0, darkgoldenrod 50%,
      black 0, black 75%,  darkgoldenrod 0, darkgoldenrod 20px);
}
```

![](http://image.oldzhou.cn/FpKtPOvn9ApVWOMLA_0MBTWPE1oZ)


（7）桌布效果

使用`backgroun-image`绘制多个背景图像，进行叠加：

```CSS
.container {
  width: 210px;
  height: 210px;
  background-color: #FFF;
  background-image: linear-gradient(90deg, black 50%, transparent 0),
      linear-gradient(black 50%, transparent 0);
  background-size: 20px 20px;
}
```


![](http://image.oldzhou.cn/FpKtPOvn9ApVWOMLA_0MBTWPE1oZ)

## 径向渐变

```
radial-gradient(
  [ [ circle || <length> ]                         [ at <position> ]? , |
    [ ellipse || [ <length> | <percentage> ]{2} ]  [ at <position> ]? , |
    [ [ circle | ellipse ] || <extent-keyword> ]   [ at <position> ]? , |
    at <position> ,
  ]?
  <color-stop> [ , <color-stop> ]+
)
```

可以简化为：

```
radial-gradient(shape size position, color-stop[...,color-stop]);
```

- `position`，代表径向渐变的圆心位置，默认值是中心点
- `shape`，径向渐变的形状，可以为`circle`或者`ellipse`（默认）
- `size`，代表径向渐变范围的的小，`shape`为`ellipse`需要指定两个值，比如`20%, 30%`，前者代表相对元素宽度的`20%`，后者代表相对高度的`30%`，也可以用关键字

![](http://image.oldzhou.cn/FqT4bGqpRzsnLxEw5VK-R-gStDSU)


（1）中间到四周渐变：

```CSS
.container {
  background: radial-gradient(white, black)
}
```

![](http://image.oldzhou.cn/FuR5dOkUSLPEiwQ8UGLXjhlqcVNd)\


（2）半椭圆形渐变，由左侧中心点到四周

```CSS
.container {
  background: radial-gradient(ellipse 100% 50% at left center, white, black)
}
```

![](http://image.oldzhou.cn/FuR5dOkUSLPEiwQ8UGLXjhlqcVNd)

（3）从左上角到右下角的发散式渐变

```CSS
.container {
  background: radial-gradient(circle farthest-corner at left top, white, black)
}
```

![](http://image.oldzhou.cn/Fh7ewMwcRG12FHBck06DiWIeLr56)

（4）指定颜色渐变范围

```CSS
.container {
  background: radial-gradient(ellipse 50% 30%, white 30%, black 70%)
}
```

![](http://image.oldzhou.cn/FnBNQFH-Dj1fNe7Vd9Wz4qWhTPRT)

## 重复渐变

重复渐变分为两种：线性重复渐变和径向重复渐变，语法都与普通的渐变相同。

[`repeating-linear-gradient`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/repeating-linear-gradient)类似`linear-gradient`，并且采用相同的参数，但是它会在所有方向上重复渐变以覆盖其整个容器。

其实`repeating-linear-gradient`的效果大多数时候可以用`linear-gradient` + `background-repeat` + `background-size`实现，上面的倾斜的进度条用`repeating-linear-gradient`也可以直接实现，关键点就是色阶值改用固定尺寸而非百分比，不再需要`background-size`，因为`repeating-linear-gradient`会自动充满容器了

```CSS
.container {
  width: 500px;
  height: 20px;
  border-radius: 5px;
}

.container:after {
  content: '';
  display: block;
  width: inherit;
  height: inherit;
  border-radius: inherit;
  background: repeating-linear-gradient(45deg, black 0, black 10px, darkgoldenrod 0, darkgoldenrod 20px);
  animation: loading 10s linear infinite;
}

@keyframes loading {
  to {
    background-position: 100% 0;
  }
}
```

更多的渐变效果可以直接Copy这个[网站](https://leaverou.github.io/css3patterns/#)。

## 参考

- [聊一聊CSS3的渐变——gradient@掘金](https://juejin.im/post/5bc2fb09f265da0ac55e75e7)
- [条纹背景@You-need-to-know-css](https://lhammer.cn/You-need-to-know-css/#/zh-cn/stripes-background?id=%e8%bf%9b%e5%ba%a6%e6%9d%a1)
- [linear-gradient@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/linear-gradient)
- [repeating-linear-gradient@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/repeating-linear-gradient)
- [10 You-need-to-know-css@不负好时光](https://duola8789.github.io/2019/06/30/03%20%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/10%20You-need-to-know-css/)
