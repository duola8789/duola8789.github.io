---
title: 网络基础03 Web Storage
top: false
date: 2017-01-17 21:06:30
updated: 2020-03-07 19:06:21
tags:
- localStorage
- sessionStorage
categories: 网络基础
---

Web Storage知识总结。

<!-- more -->

[TOC]

# Web Storage

## 基础概念

WebStorage分为localStorage和sessionStorage。

浏览器支持情况：（基本IE7以上都可以使用）

![image](http://images2015.cnblogs.com/blog/728493/201606/728493-20160626102341735-27421870.jpg)

有如下的方法：

- `localStorage.setItem("a", "b")`：为本地存储键值对，等同于`localStorage['a'] = b`
- `localStorage.getItem("a")`：获取属性名a对应的属性值，等同于`localStorage['a']`
- `localStorage.clear()`：删除所有值
- `localStorage.removeItem("a")`：删除指定的键值对
- `localStorage.key(1)`：获取指定位置的属性名

WebStorage都要受到同源策略的限制。

## 存值类型

目前所有的浏览器中都会把WebStorage的值类型限定为`string`类型:

```JS
window.localStorage.setItem('a', 3);

console.log(window.localStorage.getItem('a')); // string
```

直接向WebStorage中存入对象是不行的：

```JS
let a = { name: 'joe' }
window.sessionStorage.setItem('chow', a)
window.sessionStorage.getItem('chow')
// "[object Object]"
```

存入对象的时候，需要使用`JSON.stringify()`这个方法，将对象序列化，读取时要将字符串反序列化，使用`JSON.parse()`方法

```JS
var data = {
  name: 'xiecanyong',
  sex: 'man',
  hobby: 'program'
};
var d = JSON.stringify(data);
storage.setItem("data", d);

// 将JSON字符串转换成为JSON对象输出
var json = storage.getItem("data");
var jsonObj = JSON.parse(json);
```


## localStorage和sessionStorage的区别

LocalStorage与SessionStorage有两个比较明显的区别点：

（1）localStorage属于永久性存储，而sessionStorage存储的值会在会话结束的时候（当前标签页关闭）被清空

（2）localStorage可以在同源策略的基础上，不同的浏览器标签页之间共享，而sessionStorage的生命周期是于当前Tab相同的，不能在不同的标签页间共享

## 监听WebStorage事件

当localStorage或sessionStorage被修改时，会触发`StorageEvent`事件，我们可以通过监听`storage`来捕获这个事件：

```JS
window.addEventListener('storage', (e) => {
  console.log(e.oldValue); // 正在被更改的键的旧值
  console.log(e.newValue); // 正在被更改的键的新值
  console.log(e.url); // 触发事件的文档的地址
  console.log(e.storageArea); // 受影响的存储对象
})
```

要注意：事件在同一个域下面的不同的页面之间才会被触发，即在A页面注册了`storage`的监听，只有在A同域名下的B页面操作Storage对象，A页面才会被触发storage事件。在A页面本身修改Storage对象是不会触发此事件的。

## Cookie与WebStorage的区别

（1）Cookie参与网络通信，可以随着HTTP请求的头部信息发送到服务器，WebStorage不参与网络通信，只在本地储存信息

（2）Cookie容量较小，一般为4KB左右，WebStorage容量较大，为5M左右

（3）Cookie可以指定生效时间，过期后失效，LocalStorage除非被主动清除，否则永久有效，SessionStorage仅在本次会话有效，在浏览器关闭后失效，如果要为WebStorage设定有效期需要自己进行封装

（4）Cookie跨域传递的方法是在客户端与服务器通信的方案（CORS/方向代理/JSONP等），而WebStorage的跨域传递是多个客户端之间进行（`postMessage`）

## 关于WebStorage的几个问题

（1）`a.baidu.com`和`b.baidu.com`能共享同一个localStorage吗？

> 不能，因为本地存储都要受到同源策略的限制，也就是说只要是符合跨域的情况都不能互访本地存储。主域名、二级域名、端口号、协议不同，都属于跨域。如果希望共享，可以使用`postMessage`方法

（2）浏览器打开的不同Tab之间能否共享localStorage？

> 只要符合同源策略的限制就可以，但是sessionStorage不可以

（3）如果localStorage满了，再继续往里存东西，会发生什么？

存不进去并且报错`QuotaExceededError`


## 参考

- [storage@MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/storage_event)
- [StorageEvent@MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/StorageEvent)
- [localStorage使用总结@博客园](https://www.cnblogs.com/st-leslie/p/5617130.html)
- [browser sessionStorage. share between tabs?@stackoverflow](https://www.cnblogs.com/st-leslie/p/5617130.html)
