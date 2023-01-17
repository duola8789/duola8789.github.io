---
title: HTML+CSS40 CSS matrix函数
top: false
date: 2019-12-02 11:18:26
updated: 2019-12-02 11:18:34
tags:
- matrix
categories: HTML+CSS
---

[`matrix()`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/transform-function/matrix)是CSS的`transform`的一个基础属性，用它可以实现很多高级、复杂的效果，实际上`transfrom`的`translate`、`rotate`等都是在`matrix`的基础上实现的简化版的语法。

<!-- more -->

## 线性代数基础

了解和使用必须熟悉线性代数的向量和矩阵知识，当初学习的线性代数的课程早就还给老师了，因为不知道有什么用，如果知道今天会用到，当初一定会好好学习线性代数、高数等课程。

（1）向量

向量用来描述方向和大小，一般使用笛卡尔坐标系来进行向量的描述，比如向量`(2, 1)`和`(3, 3)`在坐标系中表示如下：

![](http://image.oldzhou.cn/Fuii4qe4aanaEgMR4h8Y4nskb4a8)

向量的加法、减法和乘法都是将对应位置的坐标进行运算：


```
(8, 2) + (2, 6) = (8 + 2, 2 + 6) = (10, 8)

(8, 2) - (2, 6) = (8 - 2, 2 - 6) = (6, -4)

(8, 2) * (2, 6) = (8 * 2, 2 * 6) = (16, 12)
```

（2）矩阵

矩阵是高等代数学中的常见工具，将数字按照长方阵列进行排列，便于进行统计分析等高等数学运算，一个`2 X 3`的矩阵就是说这个矩阵有2行3列

当相同规模的矩阵之间进行相加时，是对应的位置两两相加：

![](http://image.oldzhou.cn/FoHAFgxwwAIWHMLYHFqzMprN9fLc)

矩阵相乘时，会将前一个矩阵每行对应的位置与后一个矩阵每行对应位置分别相乘，然后将结果相加，得到的矩阵的行数等于左边矩阵的行数，列数等于右边矩阵的列数，例如

![](http://image.oldzhou.cn/FqjfweKUIK7171DpdqO7WkROIu6S)

## `matrix`方法

CSS3中的`transfrom`的`matrix()`方法写法如下：

```
transform: matrix(a,b,c,d,e,f);
```

一共有六个参数，它对应了一个`3*3`的矩阵，书写方向是竖向的：

![](http://image.oldzhou.cn/FkRSeyfwqrZwP1D9n4PtkQm3XfcN)

在进行坐标变换时（2D坐标），假设目标点的坐标为`(x, y)`，那么转换公式如下：

![](http://image.oldzhou.cn/FsOndG5AmvR5PgS9ppePKJDRW8XN)

得到的新的矩阵的第一行和第二行就是转换后的横纵坐标。

要注意的是`transfrom-origin`这个属性是变形的原点，它会影响到点的坐标的计算，我们一帮都会将它设置为`0 0`


## 例子

假设有这样一个元素，CSS属性如下：

```CSS
#transformedObject {
  position: absolute;
  left: 0px;
  top: 0px;
  width: 200px;
  height: 80px;
}
```

此时页面效果如下：

![](http://image.oldzhou.cn/FkhaQFcGrMTDf9FHfAsSSfQOZXC-)

我们对这个元素进行`transform: matrix()`的变换：

```CSS
#transformedObject {
  position: absolute;
  left: 0px;
  top: 0px;
  width: 200px;
  height: 80px;
  transform: matrix(0.9, -0.05, -0.375, 1.375, 220, 20);
  transform-origin: 0 0;
}
```

> 注意我们设置了`transform-origin`


进行`matrix`运算的原则就和上面提到的一样，以`(200, 80)`的运算过程为例：

![](http://image.oldzhou.cn/FomsUaZ_jD9vkNJ45CLVBVCWwxge)

应用变换后的效果如下：

![](http://image.oldzhou.cn/Fo0nNS15QT17Hl-svG7sVaR8KLyc)

## `matrix`与其他属性的关系

`matrix`是最基本的变化，它有六个参数，这六个参数给定特殊的值，就可以实现`translate`/`rotate`等特殊的效果：

![](http://image.oldzhou.cn/FhqFDZ7z_5mXmmmmVVW4LgTjFlhM)

以`translate`距离，当坐标点为`(10px, 10px)`，进行`translate(50px 50px)`的变化时，结果的坐标会是`(60px, 60px)`

如果使用`matrix()`进行变化，按照上图，需要给出的参数就是`matrix(1, 0, 0, 1, 50, 50)`，进行运算的过程：

```
1 0 50     10     1 * 10 + 0 * 10 + 50 * 1     60
0 1 50  *  10  =  0 * 10 + 1 * 10 + 50 * 1  =  60
0 0 1      1      0 * 10 + 0 * 10 + 1  * 1     1
```

结果是相同的。

对`matrix`有了了解之后，也就可以使用`matrix`同时进行复杂的、复合的变化，从而能够实现更复杂的动画。

## 参考

- [matrix()@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/transform-function/matrix)
- [【css基础】如何理解transform的matrix()用法@掘金](https://juejin.im/post/5d0ba96df265da1ba252659b)
- [理解CSS3 transform中的Matrix(矩阵)@张鑫旭的个人主页](https://www.zhangxinxu.com/wordpress/2012/06/css3-transform-matrix-%E7%9F%A9%E9%98%B5/)
