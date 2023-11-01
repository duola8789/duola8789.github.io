---
title: 算法基础06 KMP算法
top: false
date: 2021-06-29 11:09:49
updated: 2023-11-01 11:10:13
tags:
- KMP
categories: 算法
---

KMP算法

<!-- more -->
[TOC]

# 题目

[LeetCode的第28题，实现`strStr()`](https://leetcode-cn.com/problems/implement-strstr/)，实现字符串的匹配，实际上就是实现一个JS的`indexOf`函数

```JS
const haystack = 'hello', needle = 'll';
console.log(strStr(haystack, needle); // 2

const haystack = 'aaaaa', needle = 'bba';
console.log(strStr(haystack, needle); // -1

const haystack = '', needle = '';
console.log(strStr(haystack, needle); // 0
```

这道题目可以用暴力算法，逐个移动起始位置来进行匹配实现，如下：

```JS
var strStr = function (haystack, needle) {
  if (!needle) {
    return 0;
  }
  const len1 = haystack.length,
    len2 = needle.length,
    len = len1 - len2;
  let found = false;

  for (let i = 0; i <= len; i++) {
    found = true;
    for (let j = 0; j < len2; j++) {
      if (haystack[i + j] !== needle[j]) {
        found = false;
        break;
      }
    }
    if (found) {
      return i;
    }
  }

  return -1;
};
```

暴力算法的时间复杂度是`${O((n - m) * m)}$`，想要降低算法复杂度，就需要使用KMP算法来进行字符串匹配

# KMP算法思路

KMP是字符串匹配的最常用的算法之一，它的作用就是：如何快速在原字符串中找到匹配字符串，KMP的算法复杂度是`${O(m + n)}$`。它之所以能够降低复杂度，是因为能够在『非完全匹配』的过程中提取到有效信息进行复用，减少重复匹配的消耗，提高效率

[阮一峰的博客](http://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html)讲解的很好，但是思路与下面[宫水三叶](https://leetcode-cn.com/problems/implement-strstr/solution/shua-chuan-lc-shuang-bai-po-su-jie-fa-km-tb86/)的思考角度有一点不一样，下面主要记录的是宫水三叶的思考角度


# 匹配过程

假设原字符串是`abeababeabf`，匹配串为`abeabf`：

![image](https://note.youdao.com/yws/res/159339/WEBRESOURCEcb36fc0d72f569a13129ab05a92b1dcf)

（1）先逐个进行比较，找到第一个匹配的位置，继续向下进行，直到出现第一个不匹配的位置（红标）：

![image](https://note.youdao.com/yws/res/6/WEBRESOURCE99beef716f233c10b5dd7ca849a1cd26)


（2）如果是暴力匹配的话，会将原字符串的指针移动到本次『发起点』的下一个位置`b`，匹配串的指针移动到其实位置，继续尝试匹配，如果发现对不上，原串的指针会继续向后移动

![image](https://note.youdao.com/yws/res/f/WEBRESOURCEf0d3cafd9644599b8ec956185b4f6aaf)

这样虽然可行，但是效率很差，因为需要把搜索位置移动到已经比较过的位置，再重新比较一遍。而实际上，当`a`与`f`不匹配时，已经知道前面的刘个字符是`abeab`。

KMP的思想就是设法利用这个已知信息，不要把搜索位置移回已经比较过的位置，而是继续向后移动，这样就提高了效率

如果利用已知信息呢？需要针对搜索出，计算出一张『部分匹配表』（Partial Match Table），我们在代码称作`next`数组，数组中的每个位置的就是匹配串对应下标应该跳转的目标位置

    // next数组
    [0, 0, 0, 1, 2, 0]

这张表的计算后面会再介绍，这里先用起来

（3）利用这张表，发现匹配串的`f`不匹配了，它对应的`next[j - 1]`值是`2`，所以将匹配串的指针移动到`2`，继续进行匹配：

![image](https://note.youdao.com/yws/res/c/WEBRESOURCEacc3a6d4dae64ff95c652c07e9c3e14c)

（4）跳转到下一个匹配位置后，尝试匹配，返现`a`和`e`对不上，继续从`next`数组中寻找跳转位置，`next[j - 1]`值是`0`，这样只能回到字符串的起始位置进行匹配

![image](https://note.youdao.com/yws/res/5/WEBRESOURCE49a189ec4fd41fcccb4baf1cbe2c8ea5)


（5）继续匹配，匹配成功

# KMP的原理

KMP之所以更快，主要是因为原串指针不会进行回溯，这意味着，随着匹配过程的进行，原串指针不断右移，本质上是在不断地否决一些不可能的方案

当我们的原串指针从`i`位置移动到`j`位置，不仅仅代表着原串下标范围为`[i, j)`的字符与匹配串匹配或者不匹配，更是在否决那些以原串下标范围为`[i, j)`的『匹配发起点』的子集

# 构建`next`数组

上面的过程中，忽略了一个步骤，就是`next`数组的构建。这个构建过程的时间复杂度是`${O(m)}$`

（1）假设有匹配串`aaabbab`，定义两个指针，`i`（红色）和`j`（黑色），`j`从`0`位置开始，`i`从`1`位置开始

![image](https://note.youdao.com/yws/res/5/WEBRESOURCE49a189ec4fd41fcccb4baf1cbe2c8ea5)


（2）如果`p[i] === p[j]`，那么`next[i] = j + 1`，`i`和`j`同时向后移动，这个过程一直循环进行

![image](https://note.youdao.com/yws/res/d/WEBRESOURCE0124439db12ea8d0a16c61caac309efd)

（3）移动到此时，`p[i] !== p[j]`，那么将`j`指向前一位置的`next`数组对应的值，即`j = next[j - 1]`，循环这个过程，直到`p[i] === p[j]`或者`j === 0`

（4）如果`j === 0`，`p[i]`和`p[j]`仍不相等，那么`next[i] = 0`，`i`向后移动，`j`不动

![image](https://note.youdao.com/yws/res/4/WEBRESOURCEac74d194c066485dd4e5edb75d5db794)

（5）继续上述过程，此时`j === 0`，`p[i]`和`p[j]`仍不相等，所以`next[i]`一直为`0`，`i`指针后移，直到`b`再次出现

![image](https://note.youdao.com/yws/res/6/WEBRESOURCE3a582b1e996ab62b3b0fb34cfe5000c6)

（6）继续，`p[i]`和`p[j]`相等，那么`next[i] = 1`，`i`和`j`同时前进

![image](https://note.youdao.com/yws/res/9/WEBRESOURCE3b92dd7b496b5a43c1e1d5bfeaddc1e9)

（7）`i`移动到了最后一位，此时`p[i]`和`p[j]`不等，给`j`赋值`next[j - 1]`，即`j`为`0`，时`p[i]`和`p[j]`仍不等，所以`next[i]`为`0`，循环结束

![image](https://note.youdao.com/yws/res/0/WEBRESOURCE6248b1eae7059d7d4f4a262b46d41740)

至此循环完成

这个部分的代码实现：

```JS
function genNext(pattern) {
  // 第一位填充 0
  const next = [0],
    len = pattern.length;

  // 预处理 next 数组, 数组长度为匹配串的长度
  for (let i = 1, j = 0; i < len; i++) {
    // 匹配不成功的话, j 向后退
    while (j > 0 && pattern[j] !== pattern[i]) {
      j = next[j - 1] || 0;
    }
    // 相等了
    if (pattern[j] === pattern[i]) {
      j += 1;
    }

    next[i] = j;
  }

  return next;
}
```

# 代码实现

```JS
function strStr(haystack, needle) {
  if (!needle) {
    return 0;
  }
  const next = [0],
    len1 = haystack.length,
    len2 = needle.length;

  // 预处理 next 数组, 数组长度为匹配串的长度
  for (let j = 0, i = 1; i < len2; i++) {
    // 匹配不成功的话, j 向后退
    while (j > 0 && needle[i] !== needle[j]) {
      j = next[j - 1] || 0;
    }

    if (needle[j] === needle[i]) {
      j += 1;
    }
    next[i] = j;
  }

  // 真正的匹配过程开始
  for (let i = 0, j = 0; i < len1; i++) {
    // 匹配不成功，匹配串移动到 next 数组存储的位置
    while (j > 0 && haystack[i] !== needle[j]) {
      j = next[j - 1];
    }
    // 匹配成功，j 向前走
    if (haystack[i] === needle[j]) {
      j++;
    }
    // 整一段匹配成功，直接返回下标
    if (j === len2) {
      return i - len2 + 1;
    }
  }

  return -1;
}
```

# 总结

个人感觉KMP算法还是挺复杂的，但是它的应用还是比较广泛的，理解之后，记住它，相当于大脑里有了一个时间复杂度为`${O(n)}$`的API可以使用，用来返回匹配串在原串的位置

所以，理解一下，背住

# 练习

[这道题目](https://leetcode.cn/problems/maximum-repeating-substring/solutions/1943410/zui-da-zhong-fu-zi-zi-fu-chuan-by-leetco-r4cp/)也可以用KMP算法解决

# 参考

*   [【宫水三叶】简单题学 KMP 算法@LeetCode](https://leetcode-cn.com/problems/implement-strstr/solution/shua-chuan-lc-shuang-bai-po-su-jie-fa-km-tb86/)
*   [字符串匹配的KMP算法@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html/)
