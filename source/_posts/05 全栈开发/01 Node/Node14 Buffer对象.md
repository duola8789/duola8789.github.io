---
title: Node14 Buffer对象
top: false
date: 2019-05-05 09:53:00
updated: 2019-05-05 09:53:03
tags:
- Buffer
categories: Node
---

Buffer对象是Node处理二进制数据的一个接口，它是Node原生提供的全局对象，可以直接使用，不需要`require`

<!-- more -->

## 概述

Buffer对象是Node处理二进制数据的一个接口，它是Node原生提供的全局对象，可以直接使用，不需要`require`

Buffer对象是一个构造函数，生成的实例代表了V8引擎分配的一段内存，是一个类数组对象，成员是0到255的数值，即一个8位的字节

```JS
let btyes = new Buffer(2)

// <Buffer 00 00>
```

## 与二进制数组的关系

TypedArray构造函数可以接受Buffer实例作为参数，生成一个二进制数组，例如`new Unit32Array(new Buffer([1, 2, 3, 5]))`会生成一个4个成员的二进制数组

这时二进制数组对应的内存是从Buffer对象拷贝的，而不是共享的，二进制数组的`buffer`属性，保留指向原Buffer对象的指针。

## Buffer构造函数

Buffer作为构造函数，可以使用`new`命令生成一个实例，可以接受多种形式的参数：

```JS
// 参数是整数，指定分配多少个字节内存
var hello = new Buffer(5);

// 参数是数组，数组成员必须是整数值
var hello = new Buffer([0x48, 0x65, 0x6c, 0x6c, 0x6f]);
hello.toString() // 'Hello'

// 参数是字符串（默认为utf8编码）
var hello = new Buffer('Hello');
hello.length // 5
hello.toString() // "Hello"

// 参数是字符串（不省略编码）
var hello = new Buffer('Hello', 'utf8');

// 参数是另一个Buffer实例，等同于拷贝后者
var hello1 = new Buffer('Hello');
var hello2 = new Buffer(hello1);
```

## 类的方法

（1）`Buffer.isEncoding()`，返回布尔值，表示是否为指定编码

```JS
Buffer.isEncoding('utf8')；
// true
```

（2）`Buffer.isBuffer()`，接受一个对象作为参数，返回布尔值，表示该对象是否是Buffer实例：

```JS
Buffer.isBuffer(Date)；
// false
```
（3）`Buffer.byteLength()`，返回字符串实际占据的字节长度，默认编码为`utf-8`

```JS
Buffer.byteLength('Hello', 'utf8') // 5
```

（4）`Buffer.concat()`，用来将一组Buffer对象合并为一个Buffer对象

```JS
var i1 = new Buffer('Hello');
var i2 = new Buffer(' ');
var i3 = new Buffer('World');
Buffer.concat([i1, i2, i3]).toString()
// 'Hello World'
```

可以接受第二个参数，指定合并后的Buffer对象的总长度


## 实例属性/方法

（1）`length`，返回Buffer对象所占据的内存涨肚，与Buffer对象内容无关

```JS
buf = new Buffer(1234);
buf.length // 1234

buf.write("some string", 0, "ascii");
buf.length // 1234
```
上面`length`返回值总是Buffer对象的空间长度，内容的长度通过`Buffer.byteLength`来获取

`length`属性是可写的，但是会导致意外，不建议使用，如果想修改Buffer对象的长度，建议使用`slice`方法返回新的Buffer对象

（2）`write()`，、向指定的Buffer对象写入数据，第一个参数是写入的内容，第二个参数是写入的起始位置（可省略，默认从0开始），第三个参数是编码方式（默认是`utf-8`，可省略）

```JS
let buf = new Buffer(5)
buf.write('Hello')

console.log(buf)
// <Buffer 48 65 6c 6c 6f>

console.log(buf.toString())
// 'Hello'
```

（3）`slice()`，返回一个按照指定位置，所原对象切割的Buffer实例，两个参数是切割的起始位置和终止位置（`[start, end)`

（4）`toString()`，将Buffer实例按照指定编码（默认为`utf-8`）转换为字符串

（5）`toJSON()`，将Buffer实例转换为JSON对象

## 参考
- [Buffer对象@JavaScript 标准参考教程（alpha）](https://javascript.ruanyifeng.com/nodejs/buffer.html)





