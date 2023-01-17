---
title: HTML+CSS35 可替换元素
top: false
date: 2019-08-14 10:20:24
updated: 2019-08-14 10:20:28
tags:
- 可替换元素
categories: HTML+CSS
---

为何img、input等内联元素可设置宽高？

<!-- more -->

## 一道面试题

在美团面试的时候遇到过这样一个小问题：

> Q：你知道行内元素和块级元素有什么区别吗？
> 
> A：行内元素只会占据内部元素宽度的空间，并且不会以新的一行开始，不能设置宽度；块级元素会占满整个父容器，会新起一行，可以设置宽度；
> 
> Q：`<img>`是什么元素？
> 
> A: 行内元素
> 
> Q：那为什么`<img>`可以设置宽度？
> 
> A：呃.....

答不出来了。其实是一个很基础的问题，但是由于很多时候对基础，尤其是CSS/HTML方面的基础不甚在意，稀里糊涂也能实现想要的效果，不求甚解，真的刨根问题的时候就傻眼了。

先给出答案：**因为`<img>`虽然是行内元素，但是它们也都属于可替换元素。可替换元素一般有内在尺寸和宽高比，所以可以设置宽度和高度**

## 可替换元素

从元素本身的特点来说可以分为可替换元素和不可替换元素

HTML大多数元素都是不可替换元素，内容直接由DOM内容确定，直接展现给用户，比如`<p>123</p>`，这就是一个不可替换元素，展示的内容就是DOM的文本节点`123`

与之不同的就是可替换元素，具体显示的内容是由元素的标签和属性确定的，比如`<img>`显示的内容实际上是由`src`属性的值来读取图片的信息并展示的

典型的可替换元素：

- `<iframe>`
- `<video>`
- `<embed>`
- `<img>`

HTML规范规定了`<input>`元素可替换，因为`<image>`类型的`<input>`元素就像`<img>`一样可替换，但是其他类型的`<input>`被明确的列为非可替换元素（来源：MDN，但是实际上比如`type`为`text`的`<input>`也是可以设置宽度的，我认为可以将`<input>`也理解为条可替换元素，因为其展示形式也是与`type`属性的取值有关系的）

**几乎所有的替换元素都是行内元素**，替换元素有内在尺寸，所以可以设置`width`和`height`。如果不设置其宽度和高度时，就按照其内在尺寸显示。

## 控制可替换元素中的对象尺寸和位置

可以使用某些CSS属性来指定可替换元素中包含的内容对象，在该元素的和区域内的位置的位置和定义方式，比如：

`object-fit`用来指定可替换元素的内容对象在元素盒区域内的填充方式（类似于`background-size`），取值有`contain`、`cover`、`fill`、`none`、`scale-down`

```CSS
img {
  border: 1px solid red;
  width: 200px;
  height: 100px;
  object-fit: fill; /* 默认值 */
}
```

![](http://image.oldzhou.cn/Fvc5Tcwdi48Y_3WZKr1_QzkHuIih)


`object-position`用来指定可替换元素的内容对象在盒元素区域中的位置：

![](http://image.oldzhou.cn/FmvGu_JWWMq7StNujprONOtkv-_4)

## 参考

- [Q:为何img、input等内联元素可设置宽高@掘金](https://juejin.im/post/5a1b7319f265da43333e1d60)
- [可替换元素@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Replaced_element#%E5%8F%AF%E6%9B%BF%E6%8D%A2%E5%85%83%E7%B4%A0)
