---
title: JS54 无法使用Map遍历Array创建的数组的原因
top: false
date: 2019-01-18 14:45:29
updated: 2019-01-18 14:45:29
tags:
- Map
- Array
- 翻译
categories: JavaScript
---

介绍了使用Map遍历Array创建的数组失效的原因

<!-- more -->

> 原文：[Here’s Why Mapping a Constructed Array in JavaScript Doesn’t Work](https://itnext.io/heres-why-mapping-a-constructed-array-doesn-t-work-in-javascript-f1195138615a)   
> 作者：shawn.webdev

![](https://cdn-images-1.medium.com/max/917/1*bFIR37BFmQcxyPd7UPs6xg.png)

## 示例

为了便于说明，假设现在需要你生成一个数组，数组由数字 0~99 组成。你会怎么做？下面是一种方案：

```JS
const arr = [];
for (let i = 0; i < 100; i++) {
  arr[i] = i;
}
```
也许你和我一样，不太喜欢在 JavaScript 中使用传统的 `for` 循环。实际上，由于像 [forEach](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)、[map](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/map)、[filter](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)、[reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce) 等高阶函数的出现，我已经很久没有使用传统的 `for` 循环了。声明性函数式编程太棒了！

也许你还没有使用过函数式编程，认为上面的方法已经相当不错了。从技术层面上看没错，但是当你体会到函数式编程的魔力后，你可能就会思考是不是有更好的方法。

我对这个问题的第一反应是，“我可以创建一个长度为 `100` 的空数组，然后使用 `map` 遍历数组每个成员的索引！”在 JavaScript 中，我们可以使用 `Array` 构造函数去创建一个长度为 `n` 的空数组，就像下面这样：

```JS
const arr = Array(100);
```
完美，对吧？我们创建了长度为 `100` 的数组，接下来我只需要 map 遍历每个元素的索引。

```JS
const arr = Array(100).map((_, i) => i);
console.log(arr[0] === undefined);  // true
```
什么情况！数组的第一个元素应该是 `0`，但实际上是 `undefined`。

## 原理

为了解释上面的现象，我必须介绍一个重要的技术特性。在 JavaScript 内部，数组就是用数字作为键名的对象。比如：

```JS
['a', 'b', 'c']
```
本质上它等于下面的对象：

```JS
{
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3
}
```
当访问数组中索引 `0` 的元素时，实际上访问的是对象中键名为 `0` 的属性的键值。这很重要，因为当你把数组作为对象看待，再结合高阶函数的运行原理，上面的问题就很好理解了。

当你使用 `Array` 构造函数创建了一个新的数组时，实际上是创建了一个新的数组对象，它的 `length` 属性等于你传给 `Array` 的参数，除此以外，这个对象是一个空对象。对象中并没有数组对应的索引键（index key）。

```JS
{
  //no index keys!
  length: 100
}
```
当你试图访问索引值为 `0` 的数组成员时，访问结果是 `undefined`，但这不是因为在索引键为 `0` 的位置存储的值是 `undefined`，而是因为 JavaScript 规定，当访问一个对象中并不存在的键名对应的键值时，会返回 `undefined`。

当 `map`、`reduce`、`filter`、`forEach` 等高阶函数沿着 0 到数组长度的索引键遍历数组对象时，就会发生上面的现象，但是只有当对象的键值存在时，回应的回调函数才会执行。所以，当我们使用 `map` 对数组遍历时没有执行回调函数--就是因为索引键并不存在。

## 解决方法

正如你所了解的，我们需要的是这样的数组，它内部对应的对象形式包含着从 0 到数组长度的每一个键值。最好的办法就是将数组展开到另一个空数组中。


```JS
const arr = [...Array(100)].map((_, i) => i);
console.log(arr[0]);
// 0
```

将数组展开到一个空数组后会生成一个新数组，它每个成员都是 `undefined`：

```JS
{
  0: undefined,
  1: undefined,
  2: undefined,
  ...
  99: undefined,
  length: 100
}
```
这是因为，扩展运算符比 `map` 方法更简单。它对数组（或者任何可遍历对象）进行从 0 到数组长度的简单循环，在当前的索引处，根据展开后的数组返回值，生成一个新的索引键。而 JavaScript 对展开数组的每一项都会返回 `undefined` （记住，这一些都是默认行为，因为访问的值对应的索引键并不存在），我们就得到了一个新的数组，数组成员都具备了索引键，因此是可以使用 `map` 进行遍历的了（同样也可以使用 `reduce`、`filter`、`forEach` 进行遍历）

## 结论

我们发现了在 JavaScript 中数组的内部实质上就是对象，学习了创建任意长度、任意填充值的数组的最好的额办法。

同以往一样，在下面留下您的评论、疑问和反馈吧！

编程愉快！
