---
title: HTML+CSS06 Grid布局
top: false
date: 2017-12-11 10:40:58
updated: 2020-07-21 17:09:50
tags:
- Grid
- 多列布局
categories: HTML+CSS
---

Grid布局学习笔记。

<!-- more -->

# 与Flex的比较

Flexbox是一维布局系统，适合做局部布局，比如导航栏组件。Grid是二维布局系统，通常用于整个页面的规划。

二者从应用场景来说并不冲突。虽然Flexbox也可以用于大的页面布局，但是没有Grid强大和灵活。二者结合使用更加轻松。

# 兼容性

兼容性并不乐观，虽然至2017年3月，许多浏览器都提供了对 CSS Grid 的原生支持，但是都体现在比较新版本上：

![](http://image.oldzhou.cn/FuwPGoJYZFJH0g7r5cpcWXxIpbmT)

# 基本概念

Grid容器里面，水平区域成为行，垂直区域称为列，行和列的交叉区域成为单元格（cell）。正常情况下`n`行`m`列会产生`n x m`个单元格。

划分网格的线就成为网格线，`n`行有`n+1`根水平网格线，`m`列有`m+1`根垂直网格线

![](http://image.oldzhou.cn/FnUw1EpSCGXtbPn0TASA2q8uhDou)

一个`4x4`的网格，共有5根水平网格线和5根垂直网格线

# 容器属性

## 改变容器

使用`display: grid`指定容器采用网格布局（或者使用`display: linline-grid`指定容器采用行内网格布局）。

设为网格布局后，容器子元素的`float`、`display: inline-block`、`display: tabel-cell`、`vertical-align`和`column-*`设置都会失效

## 划分网格

容器指定了网格布局后需要划分行和列：

- `grid-template-columns`属性定义每一列的列宽
- `grid-template-rows`属性定义每一行的行高

```CSS
.container {
  display: grid;
  grid-template-columns: 100px 100px 100px;
  grid-template-rows: 100px 100px 100px;
}
```

上面的代码将容器划分为三行散列的网格，列宽和行高都是`100px`

![](http://image.oldzhou.cn/FnvaUaKlNcTXgaHDpfikqqh94a24)

### `fr`关键字

划分网格的宽度单位除了是`px`、百分比等常规单位之外，也可以使用`fr`关键字，如果两列的宽度是`1fr`和`2fr`，那么代码后者的宽度是前者的两倍

如果同时使用`px`和`fr`时，使用`fr`的宽度会将剩余的宽度都充满

### `repeat`函数

可以使用`repeat`函数来书写重复的值，它接受两个参数，第一个是重复的次数，第二个是重复的值

```CSS
.container {
  grid-template-columns: repeat(2, 100px 20px 80px);
  /* 等于 grid-template-columns: 100px 20px 80px 100px 20px 80px */
}
```

### `auto-fill`关键字

在使用`repeat`函数时，可以使用`auto-fill`关键字，表明在容器大小不确定时，希望每一行或者每一列容纳尽可能多的单元格

```CSS
.container {
  grid-template-columns: repeat(auto-fill, 100px);
}
```

上面代码表示每列宽度是`100px`，然后自动填充，直到不能放置更多的列

### 网格线名称

`grid-template-columns`属性和`grid-template-rows`属性里面，可以使用`[]`指定每一根网格线的名字，方便后面引用

```CSS
.container {
  display: grid;
  grid-template-columns: [a] 50% [b] 50% [c];
}
```

上面的代码，将列的网格线分别命名为`a`、`b`、`c`，后面引用的时候可以直接使用名称

## 网格间距

- `grid-row-gap`，设置行间距
- `grid-column-gap`，设置列间距
- `grid-gap`，是上面两者的缩写

`grid-gap`需要两个参数：

```CSS
.container {
  grid-gap: 20px 20px;
}
```

如果省略的第二个值，浏览器认为第二个值等于第一个值

> 最新的标准中，上面三个属性的`grid-`前缀都已经删除，直接使用`row-gap`、`column-gap`、`gap`即可

## 划分区域

使用`grid-template-areas`将一个网格的划分成为各个命名的区域，一个区域由单个或多个单元格组成：

```CSS
.container {
  display: grid;
  grid-template-columns: 100px 100px 100px;
  grid-template-rows: 100px 100px 100px;
  grid-template-areas: 'a b c'
                       'd e f'
                       'g h i';
}
```

要注意的是，`grid-template-areas`的属性包含的区域数量一定要和网格划分后的总数一致。上面的代码会划分出9个单元格，然后将其命名为`a`到`i`的九个区域，分别对应多个单元格

多个单元格合并为一个时使用同一个命名即可（注意，不相邻的单元格不可以使用同一个命名）

```CSS
.container {
  grid-template-areas: 'a a a'
                       'b b b'
                       'c c c';
}
```

如果某些区域用不到，使用`.`表示：

```CSS
.container {
  grid-template-areas: 'a . a'
                       'b b b'
                       'c c .';
}
```

对区域的命名会影响到网格线，每个区域的起始网格线会命名为`区域名-start`，终止网格线会自动命名为`区域名-end`

## 网格填充顺序

划分网格之后，容器的子元素会按照顺序放到每一个网格，默认的放置顺序是`先行后列`，即先填满第一行，再开始放入第二行

![](http://image.oldzhou.cn/FnvaUaKlNcTXgaHDpfikqqh94a24)

`grid-auto-flow`属性决定了这个顺序，默认取值是`row`，即先行后列的顺序进行放置，其他取值有：

- `column`，先列后行
- `row dense`
- `column dense`

后两个取值用于指定如果出现剩余空间后如何摆放，例如`1`和`2`各占两个单元格，在默认情况下第一行会出现空格：

![](http://image.oldzhou.cn/Fue2PE4FOCikX_lnBZB_RLPIXMfQ)

如果修改为`row dense`，表示先行后列，并且尽可能紧密填满，尽量不出现空格，这时候`3`就会部位到第一行

![](http://image.oldzhou.cn/FlS1MvvKYe-wawnI47Yt6izqpbHK)

## 单元格内容对齐

- `justify-items`，设置单元格内容水平位置，取值`start`/`end`/`center`/`center`
- `align-items`，设置单元格内容的垂直位置，取值`start`/`end`/`center`/`center`
- `place-items`，上面两个属性的缩写，如果省略第二个值认为与第一个值相等

`justify-items`取`start`时：

![](http://image.oldzhou.cn/Fkv7TrbYJ-skBy-Uzy-nRFNLj3tt)


## 容器对齐

- `justify-content`，设置整个内容区域在容器里面的水平位置，取值`start`/`end`/`space-around`/`space-between`/`space-evenly`
- `align-content`，设置整个内容区域在容器里面的垂直位置，取值`start`/`end`/`space-around`/`space-between`/`space-evenly`

`justify-content`取`start`时：

![](http://image.oldzhou.cn/Fq9KPSu_1mwieFs2gtJu1nJvq_vN)

`justify-content`取`space-between`时：

![](http://image.oldzhou.cn/FlyHTrES_bCbU5i8N9lCDHrdSIqp)

## 缩写

`grid-template`属性`grid-template-columns`、`grid-template-rows`和`grid-template-areas`这三个属性的合并简写形式。

`grid`属性是`grid-template-rows`、`grid-template-columns`、`grid-template-areas`、 `grid-auto-rows`、`grid-auto-columns`、`grid-auto-flow`这六个属性的合并简写形式。

# 项目属性

## 分配网格

- `grid-column-start`，左边框所在的垂直网格线
- `grid-column-end`，右边框所在的垂直网格线
- `grid-row-start`，上边框所在的水平网格线
- `grid-row-end`，下边框所在的水平网格线

通过这一组属性，指明网格的起始位置，取值可以指定为第几根网格线（从`1`开始），也可以使用之前网格线的命名网格线

属性取值还可以使用`span`关键字，表示跨越，及这个单元格跨越多少网格，如果出现重叠使用`z-index`指定重叠顺序

一般情况下我习惯使用下面的两个缩写来分配项目占据的网格：

- `grid-column`，`grid-column-start`和`grid-column-end`的合并简写形式
- `grid-row`，`grid-row-start`属性和`grid-row-end`的合并简写形式

```CSS
.item-1 {
  grid-column: 1 / 3;
  grid-row: 1 / 2;
}

/* 等同于 */
.item-1 {
  grid-column-start: 1;
  grid-column-end: 3;
  grid-row-start: 1;
  grid-row-end: 2;
}
```

缩写方式中仍然可以使用命名网格线和`span`关键字

如果省略`/`及后面的部分，则默认跨域一个网格

## 分配区域

`grad-ared`属性指定项目放在哪个区域，取值就是之前在`grid-template-areas`中指定的区域名。它还可以作为`grid-row-start`、`grid-column-start`、`grid-row-end`、`grid-column-end`的合并简写：

```CSS
.item-1 {
  grid-area: 1 / 1 / 3 / 3;
}
```

## 指定项目位置

- `justify-self`，设置单元格内容的水平位置，与`justify-items`属性用法一致，但只作用于的单个项目
- `align-self`，设置单元格内容的垂直位置，与`align-items`属性用法一致，但只作用于的单个项目
- `place-self`，上面二者的缩写

# 实例

## 两栏布局

![](http://image.oldzhou.cn/Fh00YMqYp-oh7e99V7rJLNP7UYyy)

```HTML
<!DOCTYPE html>
<html lang="en">
  <head>
    <style>
      * {
        box-sizing: border-box;
        margin: 0;
        padding: 0;
      }

      html,
      body {
        width: 100%;
        height: 100%;
      }

      .container {
        height: 100%;
        display: grid;
        grid-template-columns: 200px 1fr;
      }

      .left {
        background: #0bc1ff;
      }

      .right {
        background: #b8c885;
      }
    </style>
  </head>
  <body>
    <div class="container">
      <div class="left"></div>
      <div class="right"></div>
    </div>
  </body>
</html>
```

## 圣杯布局

![](http://image.oldzhou.cn/Fol4hp_uhmIsuES71iuI7JOQC4wY)

```HTML
<div class="container">
  <div class="head"></div>
  <div class="middle-left"></div>
  <div class="middle-right"></div>
  <div class="bottom"></div>
</div>
```

可以使用网格线来实现：

```CSS
.container {
  height: 100%;
  display: grid;
  grid-template-columns: [a] 50% [b] 50% [c];
  grid-template-rows: 80px 1fr 80px;
}

.head {
  grid-column: a/c;
  grid-row: 1/2;
  background: #0bc1ff;
}

.middle-left {
  grid-column: a/b;
  grid-row: 2/3;
  background: #4b0560;
}

.middle-right {
  grid-column: b/c;
  grid-row: 2/3;
  background: #257c2b;
}

.bottom {
  grid-column: a/c;
  grid-row: 3/4;
  background: #917551;
}
```

也可以使用区域来实现：

```CSS
.container {
  height: 100%;
  display: grid;
  grid-template-columns: 50% 50%;
  grid-template-rows: 80px 1fr 80px;
  grid-template-areas:
    'a a'
    'c a'
    'e e';
}

.head {
  grid-area: a;
  background: #0bc1ff;
}

.middle-left {
  grid-area: c;
  background: #4b0560;
}

.middle-right {
  grid-area: d;
  background: #257c2b;
}

.bottom {
  grid-area: e;
  background: #917551;
}
```

# 参考

- [CSS Grid 网格布局教程@阮一峰的网络日志](https://www.ruanyifeng.com/blog/2019/03/grid-layout-tutorial.html)
- [分钟学会 CSS Grid 布局@HTML中文网](http://www.css88.com/archives/8506)
