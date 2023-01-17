---
title: HTML+CSS07 瀑布流布局
top: false
date: 2019-08-13 10:40:58
updated: 2019-08-13 10:41:01
tags:
- 瀑布流
- 多列布局
categories: HTML+CSS
---

使用多列布局和Flex布局实现瀑布流的Demo及笔记。

<!-- more -->

## 预备知识：Column布局

CSS Columns（多列布局）是一种定义了多栏布局的模块，它能够表现出将内容在列之间怎么流动的以及间隙和分割线怎么使用。浏览器的兼容性还是不错的，生产环境建议`-moz`和`-webkit`前缀以及不加前缀三种形式共同使用

![](http://image.oldzhou.cn/Fg6svKPz1Nq7U1DV2G_e8HYvl6mh)

### `columns`

CSS多列布局可以轻松的实现多列的瀑布流布局，它的两个关键属性是`column-count`和`cloumn-width`

`column-count`设置多列布局得的列数，取值：

- `auto`，由其他CSS属性来决定列的数量，比如`column-width`
- `<number>`，用来描述元素内容被划分的列数，如果`column-width`也设置为非零值，**此参数仅表示所允许的最大列数**

`column-width`用来设定最佳列宽，取值：

- `auto`，由其他CSS属性来决定列的数量，比如`column-count`
- `<length>`，用来描述最理想的列宽，实际列宽可能更宽（用来填充可用空间）也可能更窄（当可用空间比指明的列宽更窄），如果`column-width`也设置为非零值，**此参数仅表示所允许的最大列数**

因为这两个属性值没有重叠，可以使用`columns`来进行简写：
```
column-count: 5       →     columns: 5
column-width: 200px   →     columns: 200px
↓
columns:  5 200px
```

### 高度

一般来说，多列布局的高度是由浏览器自动确定的，但是也可以通过设置`height`或者`max-height`来限制列的高度。这样在生成新的一列之前都会仅允许增加到这个高度，剩下的内容会放置到下一列中。（这样会令我们的设置的`column-count`失效）

未设置高度：

![](http://image.oldzhou.cn/Fg6svKPz1Nq7U1DV2G_e8HYvl6mh)

设置最大高度后，列数超出了设置的`column-count: 4`：

![](http://image.oldzhou.cn/Firn_-dzIdvZLIQuAOxAbjPFHYrq)

### 列间隙

`column-gap`属性来设置列之间的间隔，取值：

- `normal`，多列布局时默认值为`1em`，其他类型布局时默认值为`0`
- `<length>`，非负整数
- `<percentage>`，基于父元素宽度的百分比

### 列分割线

可以通过`column-rule`属性来为多列布局时各列之间添加分割线，它与`border`属性类似，也是一个缩写属性，它是由下面三个属性构成：

- `column-rule-width`，指定分割线宽度，可以设定具体数值，也可以在`thin`、`medium`、`thick`之间取值
- `column-rule-style`，指定分割线的样式，取值与`border-style`类似
- `column-rule-color`，指定分割线的颜色

添加这个属性后`column-rule: thick inset blue`：

![](http://image.oldzhou.cn/FsrZxoEEV2kcqePVjRzZ5kj-cqSK)

## 使用多列布局实现瀑布流

使用多列布局实现瀑布流就十分简单了：

DOM结构直接用一个多列布局的容器，内部防止排列的元素即可：

```HTML
<div class="container">
  <div class="img-container"><img src="./images/1.jpg" alt=""></div>
  <div class="img-container"><img src="./images/2.jpg" alt=""></div>
  <div class="img-container"><img src="./images/2.jpg" alt=""></div>
  <div class="img-container"><img src="./images/2.jpg" alt=""></div>
  <div class="img-container"><img src="./images/1.jpg" alt=""></div>
  <div class="img-container"><img src="./images/1.jpg" alt=""></div>
  <div class="img-container"><img src="./images/2.jpg" alt=""></div>
  <div class="img-container"><img src="./images/1.jpg" alt=""></div>
  <div class="img-container"><img src="./images/1.jpg" alt=""></div>
  <div class="img-container"><img src="./images/2.jpg" alt=""></div>
  <div class="img-container"><img src="./images/2.jpg" alt=""></div>
  <div class="img-container"><img src="./images/2.jpg" alt=""></div>
  <div class="img-container"><img src="./images/1.jpg" alt=""></div>
  <div class="img-container"><img src="./images/1.jpg" alt=""></div>
  <div class="img-container"><img src="./images/2.jpg" alt=""></div>
  <div class="img-container"><img src="./images/1.jpg" alt=""></div>
  <div class="img-container"><img src="./images/1.jpg" alt=""></div>
  <div class="img-container"><img src="./images/2.jpg" alt=""></div>
  <div class="img-container"><img src="./images/1.jpg" alt=""></div>
</div>
```
关键是设置`container`的多列布局的相关属性：

```CSS
.container {
  width: 1000px;
  column-count: 4;
  column-gap: 1em;
  column-width: 220px;
}
.img-container {
  margin-bottom: 10px;;
  break-inside: avoid;
}
```

[完整代码在这里](https://github.com/duola8789/demos/blob/master/demo36-%E7%80%91%E5%B8%83%E6%B5%81/%E7%80%91%E5%B8%83%E6%B5%81-multi-colunm.html)。

此外，需要在内部元素上添加一个[`break-inside`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/break-inside)属性，用来防止多列布局中内容以外中断，取值：

- `auto`，不强制也不禁止元素内的页、列中断
- `avoid`，避免元素内的分页符、列中断以及区域中断
- `aviod-page`，避免元素内的分页符
- `aviod-column`，避免元素内的列中断
- `aviod-region`，避免元素内的区域中断

实现的效果：

![](http://image.oldzhou.cn/FhcCTH3Wze-ztE8ORE-cgqar3gtq)

多列布局来实现瀑布流布局的优点是灵活简单，并且不需要JavaScript参与，且DOM结构很简单，缺点除了需要考虑兼容性之外，我没发现缺点

## 使用Flex布局实现瀑布流

之前面试的时候被问到，如何实现瀑布流，我当时并不知道多列布局，自然而然就想到使用Flex实现。Flex完全可以实现，但是DOM结构会稍微复杂一些

首先需要让父容器成为Flex容器，再使用4个`<div>`充当列，然后向每个列中分别添加元素即可（网上有一些方案认为列元素内部还需要使用Flex布局，然后`flex-direction`为`column`，我觉得没有必要，自然向下排列就可以了吧）

```HTML
<div class="container">
  <div class="col">
    <div class="img-container"><img src="./images/1.jpg" alt=""></div>
    <div class="img-container"><img src="./images/2.jpg" alt=""></div>
    <div class="img-container"><img src="./images/2.jpg" alt=""></div>
  </div>
  <div class="col">
    <div class="img-container"><img src="./images/2.jpg" alt=""></div>
    <div class="img-container"><img src="./images/2.jpg" alt=""></div>
    <div class="img-container"><img src="./images/1.jpg" alt=""></div>
  </div>
  <div class="col">
    <div class="img-container"><img src="./images/1.jpg" alt=""></div>
    <div class="img-container"><img src="./images/2.jpg" alt=""></div>
    <div class="img-container"><img src="./images/1.jpg" alt=""></div>
  </div>
  <div class="col">
    <div class="img-container"><img src="./images/2.jpg" alt=""></div>
    <div class="img-container"><img src="./images/1.jpg" alt=""></div>
    <div class="img-container"><img src="./images/1.jpg" alt=""></div>
  </div>
</div>
```

CSS设置并不复杂：

```CSS
.container {
  display: flex;
  width: 1000px;
  justify-content: space-around;
}
```

[完整代码在这里](https://github.com/duola8789/demos/blob/master/demo36-%E7%80%91%E5%B8%83%E6%B5%81/%E7%80%91%E5%B8%83%E6%B5%81-Flex.html)。

实现的效果：

![](http://image.oldzhou.cn/FgfjfdpECc-xjxdpsTJ1hAAYGTRZ)

这种方法优点是兼容性好，缺点主要是DOM结构复杂一些，而且需要手动的将获取的元素插入到不同的列中，还需要综合考虑各列的高度，放着某一列高度超出其他列太多。

## 参考

- [使用CSS的多列布局@MDN](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Using_multi-column_layouts)
- [column-rule-color@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/column-rule-color)
- [break-inside@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/break-inside)
- [纯css实现瀑布流（multi-column多列及flex布局）@segmentfault](https://segmentfault.com/a/1190000016255824)
