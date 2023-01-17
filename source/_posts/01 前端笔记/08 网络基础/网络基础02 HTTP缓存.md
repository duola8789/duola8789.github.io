---
title: 网络基础02 HTTP缓存
top: false
date: 2017-10-04 19:01:30
updated: 2019-05-11 11:45:33
tags:
- HTTP
- 缓存
categories: 网络基础
---

HTTP缓存学习笔记。

<!-- more -->

## WEB缓存分类

Web缓存大致可以分为：数据库缓存、服务器端缓存（代理服务器缓存、CDN缓存）、浏览器缓存。

浏览器缓存也包含很多内容：HTTP缓存、indexDB、cookie、localstorage 等等。

## 强缓存和协商缓存

强缓存：可以理解为无须验证的缓存策略。对强缓存来说，响应头中有两个字段`Expires`/`Cache-Control`来表明规则。

协商缓存：缓存的资源到期了，并不意味着资源内容发生了改变，如果和服务器上的资源没有差异，实际上没有必要再次请求。客户端和服务器端通过某种验证机制验证当前请求资源是否可以使用缓存。

浏览器第一次请求数据之后会将数据和响应头部的缓存标识存储起来。再次请求时会带上存储的头部字段，服务器端验证是否可用。如果返回`304 Not Modified`，代表资源没有发生改变可以使用缓存的数据，获取新的过期时间。反之返回`200`就相当于重新请求了一遍资源并替换旧资源。

==强制缓存的优先级高于协商缓存==

## 缓存机制

![1.19-2.png](http://image.oldzhou.cn/1.19-2.png)

1. 浏览器判定是否有缓存（`Cache-Control`）
2. 浏览器根据`expires`或者`max-age`（`max-age`会覆盖`expires`）判定缓存是否过期，如果未过期，则使用缓存（即==强缓存==，不需要与服务器交互，返回码`200 OK (from cache)`）
3. 如果已过期，则浏览器向服务器发送请求，如果上次缓存中有`Last-modified`和`Etag`字段，这次请求的请求头中会加入`If-Modified-Since`（对应于`Last-modified`）和`If-None-Match`（对应于`Etag`）。如果服务器确定内容未更改，则使用缓存（即==协商缓存==，返回码`304 Not Modified`），否则重新请求资源
4. 服务器将`Cache-control`、`Expires`、`Last-modified`、`Date`、`Etag`等字段在响应头中返回，便于下次缓存。

## 缓存控制字段

### `Expires`

`Expires`指缓存过期的时间，超过了这个时间点就代表资源过期。

取值是一个日期：

```
Expires: new Date('2018/12/06')
Expires: Wed, 21 Oct 2015 07:28:00 GMT
```
当设置为一个无效的日期，比如`0`或者`-1`，代表者此资源已经过期，相当于==禁止使用缓存==

Expires是HTTP/1.0的标准，如果同时设置了`Cache-Control`中的`max-age`或者`s-max-age`，Expires会被忽略

### `Cache-Control`

Cache-Control主要有以下几个取值：

1. `max-age`: 设置缓存的最大的有效时间，单位为秒，会覆盖掉`expires`
2. `s-maxage`: 只用于共享缓存，比如CDN缓存，在私有缓存中被忽略，会覆盖`max-age`和`expires`
3. `public`：响应可以被任何对象（发送请求的客户端、代理服务器等等）缓存，并且在多用户间共享
4. `private`: 响应只作为私有的缓存，不能在用户间共享
5. `no-cache`: 表明资源不进行缓存。但是设置了`no-cache`之后==并不代表浏览器不缓存==，而是在使用缓存前要向服务器确认资源是否被更改。因此有的时候只设置`no-cache`防止缓存还是不够保险，还可以加上`private`指令，将过期时间设为过去的时间
6. `no-store`: 绝对禁止缓存，每次请求都要向服务器重新获取数据
7. `must-revalidate`: 如果页面过期，则去服务器进行获取。

这里面最后三个属性需要单独分析一下，

1. `no-store`是拒绝禁止缓存，不再查看过期时间，每次都向服务器去请求新资源据
2. `no-cache`是拒接直接使用缓存，无论时间是否过期，都强制向服务器确认缓存的有效性
3. `must-revalidate`是拒绝使用过期缓存，只要时间过期，不再像服务器确认缓存有效性，而是直接请求新资源

### `Last-modified`/`If-Modified-Since`

==响应头部==的`Last-modified`，表示服务器端资源的最后修改时间

第一次请求之后，浏览器记录这个时间，再次请求时，==请求头部==带上 `If-Modified-Since`即为之前记录下的时间。

服务器端收到带`If-Modified-Since`的请求后会去和资源的最后修改时间对比。若修改过就返回最新资源，状态码`200`，若没有修改过则返回`304`。

![](http://image.oldzhou.cn/18-12-5/9402141.jpg)

### `Etag`/`If-None-Match`

`Etag`是由服务端生成的一段hash字符串

![](http://image.oldzhou.cn/18-12-5/41399521.jpg)

第一次请求时服务器在响应头中带上`ETag: abcd`，之后客户端再次发送的请求中会带上`If-None-Match: abcd`，服务器比对`ETag`，返回`304`或`200`。

![](http://image.oldzhou.cn/18-12-5/78422726.jpg)

### `last-modified`和`Etag`区别

某些服务器不能精确得到资源的最后修改时间，这样就无法通过最后修改时间判断资源是否更新。

一些资源的最后修改时间改变了，但是内容没改变，使用`Last-modified`看不出内容没有改变。

`Etag`的精度比`Last-modified`高，属于强验证，要求资源字节级别的一致，==优先级高==。

计算`ETag`也是需要占用资源的，如果修改不是过于频繁，看自己的需求用`Cache-Control`是否可以满足。

### 考虑缓存的内容：

- CSS样式文件
- JS文件
- logo、图标
- HTMLl文件
- 可以下载的内容

可缓存的内容又分为几种不同的情况：

（1）不经常改变的文件

==给`max-age`设置一个较大的值==，例如设置`Cache-Control: max-age=31536000`

> 标准中规定`max-age`的值最大不超过一年，所以设成`31536000`

引入的一些第三方文件、打包出来的带有`hash`后缀的CSS、JS文件。一般来说文件内容改变了，会更新版本号、hash值，相当于请求另一个文件。

（2）可能经常需要变动的文件

一般设置`Cache-Control: no-cache max-age=0`

比如入口`index.html`文件、文件内容改变但名称不变的资源。选择`ETag`或`Last-Modified`来做验证，在使用缓存资源之前一定会去服务器端做验证，命中缓存时会比第一种情况慢一点点，毕竟还要发请求进行通信。

### 不能被缓存的请求

1. HTTP信息头中包含`Cache-Control:no-cache`，`Pragma:no-cache`，或`Cache-Control: max-age=0`等告诉浏览器不用缓存的请求
2. **Post请求无法被缓存**
3. HTTP响应头中不包含缓存控制字段(`Last-Modified/Etag`/`Cache-Control`/`Expires`的请求无法被缓存

==业务敏感的GET请求不应该被缓存==

## 禁止浏览器进行缓存的方法

（1）在响应头中设置：

1. `Cache-control: no-store`
2. `Cache-control: no-cache max-age=0`
3. `Cache-control: no-cache Expires: -1`

> `Pragma: no-cache`（与`Cache-Control: no-cache`效果一致，用来向后兼容只支持 HTTP/1.0 协议的缓存服务器）

（2）针对HTML文件：可以在HTML文件的`<Meta>`标签中进行设置`http-eqiv`：

```HTML
<meta http-equiv="Expires" content="Wed, 20 Jun 2007 22:33:00 GMT"> 

<!--设置每次访问都需要请求最新html代码-->
<meta http-equiv="Expires" content="0">

<meta http-equiv="Cache-Control" content="no-store">

<meta http-equiv="Pragma" content="no-store">
```

`http-eqiv`规定了能改变服务器和用于引擎行为的编译，具体属性值[参考文档](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/meta)。

（3）通过对文件命名增加时间戳或者Hash值，来强制浏览器重新获取新文件。

## 用户行为对缓存的影响

（1）CTRL+F5强制刷新浏览器

浏览器不使用缓存，发送的请求头中带有`Cache-Control: no-cache`，明确告诉Web服务器，客户端不使用缓存。 

（2）F5刷新浏览器；

如果是在地址栏输入网址然后回车，浏览器会查找内存缓存（Memory Cache）中是否有匹配。如有则使用（缓存命中）；如没有则发送网络请求

（3）在地址栏输入网址然后回车

同F5效果相同，区别是F5时在内存缓存（Memory Cache）中查找缓存是否匹配，而这里是在磁盘缓存（Disk Cache）中进行查找。


## 参考

- [浏览器缓存@掘金](https://juejin.im/entry/5a5450dff265da3e5033a066)
- [Expires@MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Expires)
- [Pragma@MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Pragma)
- [HTTP 缓存机制一二三@知乎](https://zhuanlan.zhihu.com/p/29750583?group_id=901822935768125440)
