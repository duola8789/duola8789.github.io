---
title: Egg04 cookie
top: false
date: 2019-05-30 20:07:38
updated: 2019-05-30 20:07:40
tags:
- Cookie
- encodeURIComponent
- Base64
categories: Egg
---

通过`ctx.cookie`可以很便捷的在Controller中设置、读取Cookie。

开发之前还是应该精读文档啊。

<!-- more -->

## 设置Cookie

```JS
ctx.cookies.set(key, value, options)
```

设置Cookie其实是通过在HTTP响应中设置`set-cookie`头完成的，每个`set-cookie`都会让浏览器在Cookie中存储一个键值对。

![](http://image.oldzhou.cn/FvJ32UF9As8kZChS66GusZCTFbOB)

在设置Cookie时还支持很多参数来配置Cookie的传输、存储和权限，具体[参考文档](https://eggjs.org/zh-cn/core/cookie-and-session.html)。

其中有几个参数需要注意：

- `{ Boolean } overwrite`，设置`key`相同时如何处理，如果设置为`true`，后设置的值会覆盖前面设置的值
- `{ Boolean } singed`，设置是否对Cookie进行签名，如果设置为`true`，则设置键值对是会同时对这个键值对的值进行前面，后面取的时候会进行校验，防止前端对这个值进行篡改。默认为`true`
- `{ Boolean } encrypt`，设置是否对Cookie进行加密，如果设置为`true`，会在发送Cookie前对这个键值对的值进行加密，客户端无法读取Cookie的明文值。默认为`false`

**在默认配置下，Cookie是加签不加密的，浏览器可以看到明文，JS不能访问，不能被客户端手工篡改。**

在设置Cookie时需要考虑清楚这个Cookie的作用，需要保存多久，是否可以被JS获取，是否可以被前端修改。

（1）Cookie可以在前端访问并修改：

```JS
ctx.cookies.set(key, value, {
  httpOnly: false,
  signed: false,
});
```

（2）如果想要Cookie在浏览器端不能被修改，不能看到明文：

```JS
ctx.cookies.set(key, value, {
  httpOnly: true, // 默认就是 true
  enctypt: true,
});
```
由于浏览器对Cookie有长度限制限制，所以尽量不要设置太长的Cookie。一般来说不要超过4093 bytes。当设置的`value`大于这个值时，框架会打印一条警告日志。

## Cookie设置的编码

由于浏览器和其他客户端实现的不确定性，为了保证Cookie可以写入成功，应该将`value`通过Base64或者其他编码方式进行编码后写入。

如果直接写入中文字符会失败：

```JS
ctx.cookies.set('test', '你好')
```

可以使用`encodeURIComponent`转义，它会转义除了字母、数字、`(`、`)`、`.`、`!`、`~`、`*`、`'`、`-`和`_`之外的所有字符。

> `encodeURI`自身无法产生能适用于GET或POST请求的URI，例如对于XMLHTTPRequests, 因`&`, `+`和`=`不会被编码，然而在GET和POST请求中它们是特殊字符。然而`encodeURIComponent`这个方法会对这些字符编码。

```JS
ctx.cookies.set('test', '你好')
```

这样写入的就是通过编码后的信息：

![](http://image.oldzhou.cn/Fk2tw-Lq0iKfPmMmSQ0FJtr5a2TA)

也可以使用Base64进行加密。

## 关于Base64的题外话

之前面试的时候有面试官问我的项目中为什么要用到[base64-js](https://www.npmjs.com/package/base64-js)这个库进行Base64加密，为什么不用原生的。我说的是API更简单，其实并不是的，是因为我当时不知道浏览器已经原生支持Base64的加解密了。

在浏览器环境下的`window`对象有两个方法，`window.atob`用于解密，`window.btoa`进行加密：

```JS
let str = 'hello'

let x = btoa(str)
// "aGVsbG8="

let y = atob(x)
// 'hello'
```

既然有原生了，那么`base64-js`在npm上还有8559097的周下载量呢，原因有二：

（1）兼容性，`window.atob`和`window.btoa`方法不支持IE10以下的浏览器，而`base64-js`就都解决了：

![](http://image.oldzhou.cn/FmTBlKMbGFkc92yz7I4MJLpWTofk)

（2）对中文的支持，原生的方法直接对中文编码是会报错的：

```JS
let str = '你好'
btoa(str)

// Uncaught DOMException: Failed to execute 'btoa' on 'Window': The string to be encoded contains characters outside of the Latin1 range.
```

解决方法就是对中文字符进行安全转码：

```JS
let str = '你好'

let x = btoa(encodeURIComponent(str))
// "JUU0JUJEJUEwJUU1JUE1JUJE"

let y = decodeURIComponent(atob(x))
// '你好'
```

`base64-js`有中文的支持，减少了我们的工作量。

选用一个包，也有这么大的学问，自己还是太不认真。

## Node中的Base64加解密

说了这么多题外话，回到整体，在Egg中也可以使用Base64加密后再Set-cookie。在Node环境中不能使用刚才提到的`atob`和`btoa`方法，我们也不一定需要引用`base64-js`，Node原生也是支持Base64加解密的，要利用到`Buffer`对象的`toString`方法


```JS
> let str = '你好'

> let x = new Buffer(str).toString('base64')
// '5L2g5aW9'

> let y = new Buffer(x, 'base64').toString()
// '你好'
```

也可以不新建`Buffer`实例，而是使用Buffer的静态方法`from`来实现，原理是一样的

```JS
const str = '你好'

ctx.cookies.set('base64', Buffer.from(str).toString('base64'))

const base64 = ctx.cookies.get('base64')
const result = Buffer.from(base64, 'base64').toString()
// '你好'
```

![](http://image.oldzhou.cn/FuEo2Q_lq9CWJLzl2ripT_ziOaEy)

## 读取Cookie

```JS
ctx.cookies.get(key, options)
```
HTTP请求中的Cookie是在Header中穿过来的，而且是字符串中键值对的形式：

```TEXT
"test=%E4%BD%A0%E5%A5%BD; test.sig=bQpP88NGWWHrDUIuTcFH-D8DqTPU4VmalNWGQDIjnwU; base64=5L2g5aW9; base64.sig=Xbh30SgqJt8y9WY7fdicbtq-YE2E_qUUK5CMthAH4UQ"
```
框架提供的`get`方法可以快速的解析Cookie并获取对应的键值对的值。由于在传输时指定了`options.signed`对Cookie进行签名，所以Cookie中会自动带来`base.sig`这个签证，用来对Cookie进行校验。

- 如果设置的时候指定了`singed`，获取时未指定，则不会再获取时对渠道的值做验签，导致Cookie可能经过客户端篡改而无法发现
- 如果设置是指定了`encrypt`，获取时未指定，则无法获取到真实的值，而是加密过后的密文

如果要获取前端或者其他系统设置的cookie，需要指定参数`singed`为`false`，避免对它做验签而导致获取不到cookie值

```JS
ctx.cookies.get('frontend-cookie', {
  signed: false,
});
```

由于没有好好看文档，在使用Postman发送Cookie时，无论如何都无法成功，就是因为没有设置`signed`为`false`，这个时候要想获取成功，只能从浏览器里面将`base.sig`的值复制到Postman中一起发送。

要注意的是，即便设置了`signed`为`false`，在Postman里面直接设置Cookie的键值对的值为中文（这很方便），

![](http://image.oldzhou.cn/FgXij2axXiyWDGPgqKPRH6e0t1tp)

在Egg中也是无法解码的，它不会报错，Postman也给不出原因：

![](http://image.oldzhou.cn/Fpxdy6P3Y2o86b9KS2DzUfyXTYJa)

还是应该在Postman中对中文字符进行安全编码，选中中文部分，点击右键，在菜单中选择`encodeURIComponet`，在接受到之后进行解码就OK了

![](http://image.oldzhou.cn/FsIK1VCv2bWyG7_8hCjZBGEeE0bo)


## Cookie密钥

在Cookie中需要用到加解密和验签，需要配置一个密钥供加密使用，在`config/config.default.js`中

```JS
module.exports = {
  keys: 'key1,key2',
};
```
`kyes`配置成为一个字符串，使用逗号分割多个`key`。Cookie在使用这个配置进行加解密时：

- 加密和加签只会使用第一个密钥。
- 解密和验签时会遍历`keys`进行解密

这样做的好处是，如果我们想要更新Cookie的密钥，但是又不希望之前设置到用户浏览器的Cookie失效，可以将新的密钥配置到`keys`的最前面，过一段事件后再删除不需要的密钥即可。




## 参考
- [Cookie 与 Session@egg](https://eggjs.org/zh-cn/core/cookie-and-session.html)
- [encodeURIComponent()@MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent)
- [encodeURI()@MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/encodeURI)
- [原来浏览器原生支持JS Base64编码解码@鑫空间鑫生活](https://www.zhangxinxu.com/wordpress/2018/08/js-base64-atob-btoa-encode-decode/)
- [How to do Base64 encoding in node.js?@stack overflow](https://stackoverflow.com/questions/6182315/how-to-do-base64-encoding-in-node-js)
- [Node.js Base64 Encoding和Decoding@Jaxu's home](https://www.cnblogs.com/jaxu/p/6087109.html)
