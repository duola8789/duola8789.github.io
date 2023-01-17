---
title: Node05 常用模块
top: false
date: 2017-04-19 09:25:17
updated: 2019-04-30 11:02:38
tags:
- HTTP
- URL
- querystring
categories: Node
---

重新学习Node，整理以前的日志。常用模块的介绍（未完成）

<!-- more -->

## 1 http模块

略

## 2 url模块

用来生成和解析URL，使用前需要通过`require`加载

### 2.1 `url.resolve(base, path)`

用来生成URL，第一个参数是基准URL，其余参数是根据基准URL，生成对应的位置

```JS
url.resolve('/one/two/three', 'four')
// '/one/two/four'
```

## 3 querystring模块

用来解析查询字符串，将一个查询字符串解析为JavaScript对象

```JS
var str = 'foo=bar&abc=xyz&abc=123';

querystring.parse(str)
// { foo: 'bar', abc: [ 'xyz', '123' ] }
```
一共接受四个参数

```JS
querystring.parse(str[, sep[, eq[, options]]])
```
- `str`是需要解析的查询字符串
- `seq`是多个键值对之间的分隔符，默认为`&`
- `eq`是键名与键值之间的分隔符，默认为`=`
- `options`是配置对象，有两个属性，`decodeURIComponent`属性是一个函数，用来将编码后面的字符串还原，默认是`querystring.unescape()`，`maxKeys`属性指定最多解析多少个属性，默认是`1000`

完整调用形式如下：

```JS
querystring.parse(
  'w=%D6%D0%CE%C4&foo=bar',
  null,
  null,
  { decodeURIComponent: gbkDecodeURIComponent }
)
```

## 参考

- [url模块@JavaScript标准参考教程](https://javascript.ruanyifeng.com/nodejs/url.html)
- [querystring模块@JavaScript标准参考教程](https://javascript.ruanyifeng.com/nodejs/querystring.html)
