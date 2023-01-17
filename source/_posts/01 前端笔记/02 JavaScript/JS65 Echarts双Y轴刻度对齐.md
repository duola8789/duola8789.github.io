---
title: JS65 ECharts双Y轴刻度对齐
top: false
date: 2019-11-11 10:38:53
updated: 2019-11-11 10:38:56
tags:
- ECharts
categories: JavaScript
---

实现ECharts双Y轴刻度对齐。

<!-- more -->

## 数组中的空位

## 需求

系统中有这样一个图标，X轴为天，双Y轴，左侧为百分比，右侧为当日各种车辆数目的汇总，理想状态如下：

![](http://image.oldzhou.cn/FhlY78XuPHiSszbAhUYxSSoNRm0e)

但是线上发现了问题，由于左侧Y轴是强制按照`20`为间隔进行分割，而右侧没有设置分割，是有Echarts自动计算得来，按照`50`进行分割，Y轴总共分割为5份，当右侧Y轴的`max`为`250`时，恰好左右侧Y轴可以对其，但是当右侧外轴的`max`为`300`时，那么两个Y周的刻度就不会重合，不是很美观：

![](http://image.oldzhou.cn/FhlY78XuPHiSszbAhUYxSSoNRm0e)

需要解决这个问题

## 手动计算最大值

解决这个问题，就需要指定两个Y轴的最大值、最小值和间隔，让两个Y轴刻度强制对齐。以左侧外轴为基准，`min`为`0`，`max`为`100`，`interval`为`20`，右侧Y轴也按照这样的比例手动计算`max`和`interval`

首先计算出`max`，其实Echarts的`max`属性可以接受一个函数作为值，函数的参数`value`包含两个属性`max`和`min`就是各个系列（我的例子中就是按天为单位的数组）的实际最大值。但是为了动态计算间隔，我没有使用这个参数，而是手动计算了最大值：

```JS
function getCeilMaxSumByColumn(arr = []) {
  // 默认值
  const DEFAULT_MAX = 100;
  // 近似的间隔值
  const IDEAL_INTERVAL = 50;
  if (arr.length === 0) {
    return DEFAULT_MAX;
  }
  // 按列求和，取最大值
  const getMaxSumByColumn = arr => arr.reduce((result, current) = > {
    current.forEach((num, day) = > {
      if (!result[day]) {
        result[day] = {
          total: 0
        };
      }
      result[day].total += num;
    });
    return result;
  }, []).sort((a, b) => b.total - a.total)[0].total;
  // 
  const max = getMaxSumByColumn(arr);
  if (max <= 0) {
    return DEFAULT_MAX;
  }
  // 按照间隔
  return Math.round(max);
},
```

然后在第二Y轴的设置中，`max`就取这个值，`interval`取`max * 0.2`，结果如下：

![](http://image.oldzhou.cn/Fv93xXyxdEKuKrgqjk4UsIMVqoj6)

可以看到，两个Y轴对齐了，但是为了对齐，右侧Y轴的刻度会有小数，虽然可以通过`formatter`来消除，但是结果仍然不是整齐的数字，所以对上面的函数进行了优化，按照`50`为间隔来取`max`，比如`5`，结果就是`50`，`51`对应`100`，`105`对应`150`，计算的方法一开始难住了我，后来发现也没那么难，挺简单，对结果除以`50`，取结果的整数部分向上四舍五入，然后再乘以`50`即可：

```JS
/**
 * 获取二维数组按列求和的最大值，并按照 50 为分割，向上近似取值，例如输入
 * [
 *     [1, 2, 3],
 *     [4, 5, 6],
 *     [7, 8, 9]，
 * ]
 * 按列求和，第三列求和值最大为18，位于[0, 50]，输出结果为 50
 * @param arr { array } 输入的二维数组
 * @returns {number} 返回按列求和后的最大近似值
 */
function getCeilMaxSumByColumn(arr = []) {
  // 默认值
  const DEFAULT_MAX = 100;
  // 近似的间隔值
  const IDEAL_INTERVAL = 50;
  if (arr.length === 0) {
    return DEFAULT_MAX;
  }
  // 按列求和，取最大值
  const getMaxSumByColumn = arr => arr.reduce((result, current) = > {
    current.forEach((num, day) => {
      if (!result[day]) {
        result[day] = {
          total: 0
        };
      }
      result[day].total += num;
    });
    return result;
  }, []).sort((a, b) = > b.total - a.total)[0].total;
  // 
  const max = getMaxSumByColumn(arr);
  if (max <= 0) {
    return DEFAULT_MAX;
  }
  // 按照间隔
  return Math.ceil(max / IDEAL_INTERVAL) * IDEAL_INTERVAL;
},
```

结果：

![](http://image.oldzhou.cn/Fu1dcnz05Le40nJSjkXYrMTH9dyj)

Done!

## 参考

- [yAxis.interval@Echarts](https://www.echartsjs.com/zh/option.html#yAxis.interval)
