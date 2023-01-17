---
title: 零散专题09 Moment.js和date-fns
top: false
date: 2017-04-26 12:05:30
updated: 2019-05-09 18:43:15
tags:
- Moment
- date-fns
categories: 其他
---

介绍JavaScript的两个日期处理工具库Moment.js和date-fns。

<!-- more -->

## Moment.js

Moment.js是一个（轻量级）的Javascript日期处理类库，使用它可以轻松解决前端开发中遇到的种种日期时间问题。

Moment.js不依赖任何第三方库，支持字符串、`Date`、时间戳以及数组等格式，可以格式化日期时间，计算相对时间，获取特定时间后的日期时间等等。

支持中文在内的多种语言。

### 格式化日期

```JS
moment().format('MMMM Do YYYY, h:mm:ss a');     // 四月 26日 2017, 12:12:53 中午
moment().format('dddd');                        // 星期三
moment().format("MMM Do YY");                   // 4月 26日 17
moment().format('YYYY [escaped] YYYY');         // 2017 escaped 2017
```

### 相对时间

```JS
moment("20111031", "YYYYMMDD").fromNow();       // 5 年前
moment("20120620", "YYYYMMDD").fromNow();       // 5 年前
moment().startOf('day').fromNow();              // 12 小时前
moment().endOf('day').fromNow();                // 12 小时内
moment().startOf('hour').fromNow();             // 14 分钟前     
```

### 日历时间

```JS
moment().subtract(10, 'days').calendar();       // 2017年4月16日
moment().subtract(6, 'days').calendar();        // 上周四中午12点14
moment().subtract(3, 'days').calendar();         // 上周日中午12点14
moment().subtract(1, 'days').calendar();        // 昨天中午12点14分
moment().calendar();                            // 今天中午12点14分
moment().add(1, 'days').calendar();             // 明天中午12点14分
moment().add(3, 'days').calendar();             // 本周六中午12点14
moment().add(10, 'days').calendar();            // 2017年5月6日
```
Moment.js提供了丰富的说明文档，使用它还可以创建日历项目等复杂的日期时间应用。日常开发中最常用的是格式化时间，下面是常用的格式：

![clipboard.png](http://image.oldzhou.cn/clipboard.png)

## date-fns

date-fns是另外一个比热门的JavaScript日期处理工具库，越来越多的人用它来替换Moment.js。它一样提供了大量的函数来操作日期。

安装：

```BASH
npm install date-fns --save
```
date-fns是使用纯函数构建的，并且可以再不改变传递日期实例的情况下保持不变。并且由于date-fns里面每一个方法都是一个文件，所以可以非常方便的只引入需要的部分，这相比于Moment.js可以更方便的降低打包体积。

```JS
const moment = require('moment');
const format1 = moment.format;
const format2 = require('date-fns/format');
```

具体使用方法参考[官方文档](https://date-fns.org/docs/Getting-Started)。

## 比较

Moment.js存在一些问题，导致了date-fns越来越流行，Moment.js存在的一些问题时hi：

1. Moment.js是可变的
2. 具有复杂的面向对象的API
3. 复杂的API带来大量性能开销
4. 使用Webpack将其打包在构建结果中，会导致构建结果尺寸增加很多

那么date-fns是如何解决这些问题呢？一个一个来看：

（1）Moment.js是可变的

Moment.js中的日期是可变的，这可能会导致不可预料的行为，什么意思呢：

```JS
const moment = require('moment');

const now = moment(new Date());
console.log('now: ', now.format());
// now:  2019-05-09T18:08:40+08:00

now.add(3, 'days');
console.log('now: ', now.format());
// now:  2019-05-12T18:08:40+08:00

now.add(3, 'days');
console.log('now: ', now.format());
// now:  2019-05-15T18:08:40+08:00
```

Momment.js生成的`moment`对象是可变的，`now`在一开始代表5月9日，加3天后变为了5月12日，再加三天变为了5月15日。

这是因为Moment.js是面向对象的原因，它产生的是一个可变的对象。

而date-fns是纯函数式，它更加纯净简洁，它产生的对象是不可变的，每次调用都会返回新的对象：

```JS
const addDays = require('date-fns/add_days');
const now = new Date();
console.log('now: ', now);
// now:  2019-05-09T10:10:30.079Z

const day1 = addDays(now, 3);
console.log('day1: ', day1);
// day1:  2019-05-12T10:10:30.079Z
console.log('now: ', now);
// now:  2019-05-09T10:10:30.079Z

const day2 = addDays(now, 3);
console.log('day2: ', day2);
// day2:  2019-05-12T10:10:30.079Z
console.log('now: ', now);
// now:  2019-05-09T10:10:30.079Z
```

在加3天后，产生新的日期，但是原来的`now`对象一直都是5月9日，是不变的。

（2）具有复杂的面向对象的API和复杂的API带来大量性能开销

相对来说，date-fns作为纯函数，API可能会稍微更简洁一下，这个还是看官网的文档吧。


（3）使用Webpack将其打包在构建结果中，会导致构建结果尺寸增加很多

Moment加载时默认会将所有方法和所有的语言`locale`文件打包，打包出体积Gzip压缩后可能也要60kb，如果需要按需加载需要Webpack的配合，在Webpack的配置文件中使用自带的`IgnorePlugin`插件来过滤掉所有的locale文件：

```JS
const webpack = require('webpack');
module.exports = {
  //...
  plugins: [
    // Ignore all locale files of moment.js
    new webpack.IgnorePlugin(/^\.\/locale$/, /moment$/),
  ],
};
```

当需要语言包时手动引入：

```JS
const moment = require('moment');
require('moment/locale/ja');
 
moment.locale('ja');
```

但是这样只能排除不必要的`locale`文件，但是没使用的方法还是会全部引入的。

而date-fns每个方法、每个locale都是一个单独的文件：

![WX20190509-181909.png](http://image.oldzhou.cn/WX20190509-181909.png)

在使用的时候完全做到了按需引入：

```JS
const addDays = require('date-fns/add_days');
const format = require('date-fns/format');
const eoLocale = require('date-fns/locale/eo');

const result = format(
  addDays(new Date(2014, 6, 2), 3),
  'Do [de] MMMM YYYY',
  {locale: eoLocale}
);
console.log(result);
// 5-a de julio 2014
```

在Webpack构建的时候，自动就可以做到按需引入，非常方便。

## 参考

- [Moment.js 文档@Moment](http://momentjs.cn/docs/)
- [date-fns文档@date-fns](https://date-fns.org/docs/Getting-Started)
- [date-fns —— 轻量级的 JavaScript 日期库@可译网](http://coyee.com/article/12360-introduction-to-date-fns-a-lightweight-javascript-date-library)
- [在webpack打包时精简moment.js@CSDN](https://blog.csdn.net/qq_31061615/article/details/80745538)
