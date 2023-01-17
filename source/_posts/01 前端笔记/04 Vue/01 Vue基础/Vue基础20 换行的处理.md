---
title: Vue基础20 换行的处理
top: false
date: 2019-06-12 17:32:55
updated: 2019-06-12 17:32:57
tags:
- 换行
- white-space
- innerHTML
- innerText
categories: Vue
---

Vue中的数据使用双大括号插值插入DOM中，如果数据中含有标签或者换行符`\n`时，是无法正常被解析为HTML标签的，而是解析为纯文字的

要想正常换行有一下几个解决方法：

<!-- more -->

## `v-html`

使用`v-html`会将数据内容作为HTML标签插入到DOM中，这时候就可以用`<br>`来代替`\n`进行换行，这个时候就可以换行成功了

## `white-space`

使用`v-html`要注意防范XSS攻击，我们也可以直接使用CSS属性来实现换行：

![](http://image.oldzhou.cn/FnnJv1CghuIFf3llLUUl4nCx-veg)

将容器的`white-sapce`属性设置为`pre-wrap`就可以了，注意，如果设置为`pre`和使用`<pre>`标签效果是一样的，它虽然会正常换行了，但是它们式中会保留HTML标签。

如果一定需要动态插入HTML标签，那么还是需要使用`v-html`


## `innerHTML`和`innerText`

设置一个节点的`innerHTML`和`innerText`都会保留`\n`，但是区别是`innerHTML`会插入HTM标签，`innerText`则会以字符串的形式显示HTML标签

```JS
document.querySelector('test').innerText = '<em>123\n\r45<br />6</em>'
// <em>123
//
// 45<br />6</em>

document.querySelector('test').innerHtml = '<em>123\n\r45<br />6</em>'
// 123 45
// 6
```

Vue中使用双大括号差值与这两种方式都是不同的，猜测可能是它对`\n`有特殊的处理

## 参考
- [插值@Vue](https://cn.vuejs.org/v2/guide/syntax.html#%E6%8F%92%E5%80%BC)
- [white-space@MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/white-space)
