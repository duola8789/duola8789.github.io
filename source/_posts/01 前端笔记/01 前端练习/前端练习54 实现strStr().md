---
title: 前端练习54 实现strStr()
top: false
date: 2019-02-22 10:23:14
updated: 2019-02-22 10:23:14
tags:
- 练习
- 算法
categories: 前端练习
---

Leetcode初级算法练习。

<!-- more -->

## 题目

> 题目来自[LeetCode](https://leetcode-cn.com/problems/implement-strstr/)

实现`strStr()`函数。

给定一个`haystack`字符串和一个`needle`字符串，在`haystack`字符串中找出`needle`字符串出现的第一个位置(从`0`开始)。如果不存在，则返回`-1`。

例1:

```BASH
输入: haystack = "hello", needle = "ll"
输出: 2
```

示例 2:

```BASH
输入: haystack = "aaaaa", needle = "bba"
输出: -1
```
当`needle`是空字符串时我们应当返回`0`。这与C语言的`strstr()`以及Java的`indexOf()`定义相符。

## 实现

其实就是实现JavaScript中的`indexOf`函数，对于空字符串`indexOf`返回的也是`0`

所以有些答案直接把`indexOf`方法摆在那里，就是在让人无语了，你来LeetCode的目的是什么呢？刷成绩吗，还是真心想要提高你的算法和逻辑思维的能力呢？

我一开始想对`haystack`进行一次遍历，声明一个指针来标识内部的`needle`的遍历的位置，结果发现没有想象的简单

还是应该老老实实用两次遍历来实现，我一开始额外声明了变量来标识循环是否完成：

```JS
/**
 * @param {string} haystack
 * @param {string} needle
 * @return {number}
 */
var strStr = function (haystack, needle) {
  if (needle === '') {
    return 0;
  }
  let end = true;
  for (let i = 0; i < haystack.length; i++) {
    let count = i;
    end  = true;
    for (let j = 0; j < needle.length; j++) {
      if (haystack[count++] !== needle[j]) {
        end = false;
        break;
      }
    }
    if (end) {
      return i;
    }
  }
  return -1;
};
```
执行用时竟然达到了骇人听闻的4912ms。

看了排名在前的答案，进行了优化，优化后执行用时只有100ms：

```JS
var strStr = function (haystack, needle) {
  if (needle === '') {
    return 0;
  }
  for (let i = 0; i <= haystack.length - needle.length; i++) {
    for (let j = 0; j < needle.length; j++) {
      if (haystack[i + j] !== needle[j]) {
        break;
      }
      if (j === needle.length - 1) {
        return i;
      }
    }
  }
  return -1;
};
```
看一下优化的点：

首先就是额外的两个变量根本没有必要，只需要在内层循环判断循环是否是最后一个字符串了即可，没有必要声明`flag`，而`count`的话，直接使用`i+j`代替就行

其次是外层循环没有必要完全循环，如果剩下的长度已经小于`needle`的长度，那么就没有必要再循环了，因为已经不能再产生结果了。

所以，还是智商的差距。

努力吧。
